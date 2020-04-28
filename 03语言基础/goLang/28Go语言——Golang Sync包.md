

# 1 Golang sync包

sync包有以下几个内容：
- `sync.Pool` 临时对象池
- `sync.Mutex` 互斥锁
- `sync.RWMutex` 读写互斥锁
- `sync.WaitGroup` 组等待
- `sync.Cond` 条件等待
- `sync.Once` 单次执行
- `sync.Map `并发安全字典结构

## 1.1 临时对象池 (sync.Pool)

- Pool是用于存储那些被分配了但是没有被使用，而未来可能会使用的值，以减小垃圾回收的
  压力。
- Pool是协程安全的，应该用于管理协程共享的变量，不推荐用于非协程间的对象管理。
  调动 New 函数，将使用函数创建一个新对象返回。
- 从Pool中取出对象时，如果Pool中没有对象，将执行New()，如果没有对New进行赋值，则
  返回Nil。
- 先进后出存储原则，和栈类似Pool一个比较好的例子是fmt包，fmt包总是需要使用一些[]byte之类的对象，Go建立了一个临时对象池，存放着这些对象，如果需要使用一个[]byte，就去Pool里面拿，如果拿不到就分配一份。这比起不停生成新的 []byte，用完了再等待gc回收来要高效得多  
  示例代码：

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	var pool sync.Pool
	var val interface{}
	pool.Put("1")
	pool.Put(12)
	pool.Put(true)
	for {
		val = pool.Get()
		if val == nil {
			break
		}
		fmt.Println(val)
	}
}

```

利用goroutine和channel进行go的并发模式，实现一个资源池实例，资源池可以存储一定数量的
资源，用户程序从资源池获取资源进行使用，使用完成将资源释放回资源池  

示例代码：

```go
//一个安全的资源池，被管理的资源必须都实现io.Close接口
type Pool struct{
	m sync.Mutex
	res chan io.Closer
	factory func () (io.Closer, error)
	closed bool
}
```

这个结构体Pool有四个字段

- m 是一个互斥锁，这主要是用来保证在多个goroutine访问资源时，池内的值是安全的。
- res 字段是一个有缓冲的通道，用来保存共享的资源，这个通道的大小，在初始化Pool的时
  候就指定的。注意这个通道的类型是io.Closer接口，所以实现了这个io.Closer接口的类型都
  可以作为资源，交给我们的资源池管理。
- factory 这个是一个函数类型，它的作用就是当需要一个新的资源时，可以通过这个函数创
  建，也就是说它是生成新资源的，至于如何生成、生成什么资源，是由使用者决定的，所以
  这也是这个资源池灵活的设计的地方。
- closed字 段表示资源池是否被关闭，如果被关闭的话，再访问是会有错误的。  

pool.go

```go
package main

import (
	"errors"
	"fmt"
	"io"
	"sync"
	"time"
)

type Pool struct {
	m   sync.Mutex
	res chan io.Closer
	//创建资源的方法，由用户程序自己生成传入
	factory func() (io.Closer, error)
	closed  bool
	//资源池获取资源超时时间
	timeout <-chan time.Time
}

//资源池关闭标志
var ErrPoolClosed = errors.New("资源池已经关闭")
//超时标志
var ErrTimeout = errors.New("获取资源超时")
//新建资源池
func New(fn func() (io.Closer, error), size int) (*Pool, error) {
	if size <= 0 {
		return nil, errors.New("新建资源池大小太小")
	}
	//新建资源池
	p := Pool{
		factory: fn,
		res:     make(chan io.Closer, size),
	}
	// 向资源池循环添加资源，直到池满
	for count := 1; count <= cap(p.res); count++ {
		r, err := fn()
		if err != nil {
			fmt.Println("添加资源失败，创建资源方法返回nil")
			break
		}
		fmt.Println("资源加入资源池")
		p.res <- r
	}
	fmt.Println("资源池已满，返回资源池")
	return &p, nil
}

//获取资源
func (p *Pool) Acquire(d time.Duration) (io.Closer, error) {
	//设置d时间后超时
	p.timeout = time.After(d)
	select {
	case r, ok := <-p.res:
		fmt.Println("获取", "共享资源")
		if !ok {
			return nil, ErrPoolClosed
		}
		return r, nil
	case <-p.timeout:
		return nil, ErrTimeout
	}
}

//放回资源池
func (p *Pool) Release(r io.Closer) {
	//上互斥锁，和Close方法对应，不同时操作
	p.m.Lock()
	defer p.m.Unlock()
	if p.closed {
		r.Close()
		return
	}
	//资源放回队列
	select {
	case p.res <- r:
		fmt.Println("资源放回队列")
	default:
		fmt.Println("资源队列已满，释放资源")
		r.Close()
	}
}

//关闭资源池
func (p *Pool) Close() {
	//互斥锁，保证同步，和Release方法相关，用同一把锁main.go
	p.m.Lock()
	defer p.m.Unlock()
	if p.closed {
		return
	}
	p.closed = true
	//清空通道资源之前，将通道关闭，否则引起死锁
	close(p.res)
	for r := range p.res {
		r.Close()
	}
}

```

main.go

```go
package main

import (
	"fmt"
	"io"
	"math/rand"
	"sync"
	"sync/atomic"
	"time"
)

const (
	maxGoroutines   = 25
	pooledResources = 2
)

//实现接口类型 资源类型
type dbConnection struct {
	ID int32
}

//实现接口方法
func (conn *dbConnection) Close() error {
	fmt.Printf("资源关闭,ID:%d\n", conn.ID)
	return nil
}

//给每个连接资源给id
var idCounter int32
//创建新资源
func createConnection() (io.Closer, error) {
	id := atomic.AddInt32(&idCounter, 1)
	fmt.Printf("创建新资源,id:%d\n", id)
	return &dbConnection{ID: id}, nil
}

//测试资源池
func performQueries(query int, p *Pool) {
	conn, err := p.Acquire(10 * time.Second)
	if err != nil {
		fmt.Println("获取资源超时")
		fmt.Println(err)
		return
	}
	//方法结束后将资源放进资源池
	defer p.Release(conn)
	//模拟使用资源
	time.Sleep(time.Duration(rand.Intn(1000)) * time.Millisecond)
	fmt.Printf("查询goroutine id:%d,资源ID：%d\n", query, conn.
	(*dbConnection).ID)
}

func main() {
	var wg sync.WaitGroup
	wg.Add(maxGoroutines)
	p, err := New(createConnection, pooledResources)
	if err != nil {
		fmt.Println(err)
	}
	//每个goroutine一个查询，每个查询从资源池中获取资源
	for query := 0; query < maxGoroutines; query++ {
		go func(q int) {
			performQueries(q, p)
			wg.Done()
		}(query)
	}
	//主线程等待
	wg.Wait()
	fmt.Println("程序结束")
	//释放资源
	p.Close()
}

```



## 1.2 互斥锁 (sync.Mutex)  

- 互斥锁用来保证在任一时刻，只能有一个协程访问某对象。
- Mutex的初始值为解锁状态，Mutex通常作为其它结构体的匿名字段使用，使该结构体具有Lock和Unlock方法。
- Mutex可以安全的再多个协程中并行使用。
- 如果对未加锁的进行解锁，则会引发panic  

加锁，保证数据能够正确：  

```go
package main

import (
	"fmt"
	"sync"
)

var num int
var mu sync.Mutex
var wg sync.WaitGroup

func main() {
	wg.Add(10000)
	for i := 0; i < 10000; i++ {
		go Add()
	}
	wg.Wait()
	fmt.Println(num)
}
func Add() {
	mu.Lock() // 加锁
	defer func() {
		mu.Unlock() // 解锁
		wg.Done()
	}()
	num++
}

```

并发没有加锁时，会发现结果不等于10000，其原因是因为出现这样的情况，就是其中一些协程刚好读取了num的值，此时该协程刚好时间片结束，被挂起，没有完成加1。然后其他协程进行加1，然后当调度回到原来那个协程，num = 原来的那个num(而不是最新的num) + 1，导致num数据出错  。

## 1.3 读写锁(sync.RWMutex)

- RWMutex比Mutex多了一个“写锁定” 和 “读锁定”，可以让多个协程同时读取某对象。
- RWMutex的初始值为解锁状态。RWMutex通常作为其它结构体的匿名字段使用。
- RWMutex可以安全的在多个协程中并行使用。  

```go
// Lock 将 rw 设置为写状态，禁止其他协程读取或写入
func (rw *RWMutex) Lock()
// Unlock 解除 rw 的写锁定状态，如果rw未被锁定，则该操作会引发 panic。
func (rw *RWMutex) Unlock()
// RLock 将 rw 设置为锁定状态，禁止其他协程写入，但可以读取。
func (rw *RWMutex) RLock()
// Runlock 解除 rw 设置为读锁定状态，如果rw未被锁定，则该操作会引发 panic。
func (rw *RWMutex) RUnLock()
// RLocker 返回一个互斥锁，将 rw.RLock 和 rw.RUnlock 封装成一个 Locker 接口。
func (rw *RWMutex) RLocker() Locker

```

## 1.4 等待组(sync.WaitGroup)  

- WaitGroup 用于等待一组协程的结束。
- 主协程创建每个子协程的时候先调用Add增加等待计数，每个子协程在结束时调用Done减少协程计数。
- 主协程通过 Wait 方法开始等待，直到计数器归零才继续执行。  

```go
// 计数器增加 delta，delte可以时负数
func (wg *WaitGroup) Add(delta int)
// 计数器减少1，等价于Add(-1)
func (wg *WaitGroup) Done()
// 等待直到计数器归零。如果计数器小于0，则该操作会引发 panic。
func (wg *WaitGroup) Wait()
```

## 1.5 条件等待(sync.Cond)  

条件等待和互斥锁有不同，互斥锁是不同协程公用一个锁，条件等待是不同协程各用一个锁，但是wait()方法调用会等待（阻塞），直到有信号发过来，不同协程是共用信号。  

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func main() {
	var wg sync.WaitGroup
	cond := sync.NewCond(new(sync.Mutex))
	for i := 0; i < 3; i++ {
		go func(i int) {
			fmt.Println("协程", i, "启动。。。")
			wg.Add(1)
			defer wg.Done()
			cond.L.Lock()
			fmt.Println("协程", i, "加锁。。。")
			cond.Wait()
			fmt.Println("协程", i, "解锁。。。")
			cond.L.Unlock()
		}(i)
	}
	time.Sleep(time.Second)
	fmt.Println("主协程发送信号量。。。")
	cond.Signal()
	time.Sleep(time.Second)
	fmt.Println("主协程发送信号量。。。")
	cond.Signal()
	time.Sleep(time.Second)
	fmt.Println("主协程发送信号量。。。")
	cond.Signal()
	wg.Wait()
}

```

## 1.6 单次执行(sync.Once)

- Once的作用是多次调用但只执行一次，Once只有一个方法，Once.Do()，向Do传入一个函
  数，这个函数在第一次执行Once.Do()的时候会被调用
- 以后再执行Once.Do()将没有任何动作，即使传入了其他的函数，也不会被执行，如果要执
  行其它函数，需要重新创建一个Once对象。
- Once可以安全的再多个协程中并行使用，是协程安全的。  

```go
// 多次调用仅执行一次指定的函数f
func (o *Once) Do(f func())
```

```go
// 示例：Once
package main

import (
	"fmt"
	"sync"
)

func main() {
	var once sync.Once
	var wg sync.WaitGroup
	onceFunc := func() {
		fmt.Println("旭哥爱你们哟~")
	}
	wg.Add(10)
	for i := 0; i < 10; i++ {
		go func() {
			defer wg.Done()
			once.Do(onceFunc) // 多次调用只执行一次
		}()
	}
	wg.Wait()
}

```

## 1.7 并发安全Map(sync.Map)  

Go语言中的 map 在并发情况下，只读是线程安全的，同时读写是线程不安全的。需要并发读写时，一般的做法是加锁，但这样性能并不高，Go语言在 1.9 版本中提供了一种效率较高的并发安全的 sync.Map，sync.Map 和 map 不同，不是以语言原生形态提供，而是在 sync包下的特殊结构。
sync.Map 有以下特性：

- 无须初始化，直接声明即可。
- sync.Map 不能使用 map 的方式进行取值和设置等操作，而是使用 sync.Map 的方法进行调用，Store 表示存储，Load 表示获取，Delete 表示删除。
- 使用 Range 配合一个回调函数进行遍历操作，通过回调函数返回内部遍历出来的值，Range参数中回调函数的返回值在需要继续迭代遍历时，返回 true，终止迭代遍历时，返回false。  

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	var scene sync.Map
	// 将键值对保存到sync.Map
	scene.Store("大哥", 97)
	scene.Store("二哥", 100)
	scene.Store("三哥", 200)
	// 从sync.Map中根据键取值
	fmt.Println(scene.Load("旭哥"))
	// 根据键删除对应的键值对
	scene.Delete("大哥")
	// 遍历所有sync.Map中的键值对
	scene.Range(func(k, v interface{}) bool {
		fmt.Println("iterate:", k, v)
		return true
	})
}

```

