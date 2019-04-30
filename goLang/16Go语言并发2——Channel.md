# Go语言并发2——Channel

[TOC]

## 1、什么是channel

channel 是一种架设在goroutine之间进行 通信的管道，类似队列。channel是引用类型，类型为chan，可以通过make关键字进行创建指定类型的channel。channel存在的意义是让goroutine通过通信来共享内存，一个往通道发送数据，一个从通道获取数据，来实现数据同步。

## 2、channel的创建和传递

声明通道时，需要指定将要被共享的数据的类型。可以通过通道共享内置类型、命名类型、结构类型和引用类型的值或者指针。 

### 2.1 make关键字创建

```go
ch:=make(chan int) //创建一个int类型的channel,所以这个channel只能发送接收int型的数据
ch1:=make(chan int 10)//这个是有缓冲的buffer，这个下面会解释
```

### 2.2 <-  运算符 读和取

```go
ch <- 2 //发送数值2给这个通道
x:=<-ch //从通道里读取值，并把读取的值赋值给x变量
<-ch //从通道里读取值，然后忽略
```

### 2.3 close 函数关闭channel

```go
close(ch)   //可以使用内置函数close关闭通道
```

如果一个通道被关闭了，我们就不能往这个通道里发送数据了，如果发送的话，会引起`painc`异常。但是，我们还可以接收通道里的数据，如果通道里没有数据的话，接收的数据是`nil`。



```go
package  main
import "fmt"
func main() {
	ch:=make(chan int)//创建int 型的无缓冲的channel
	go func() { 
		sum:=0
		for i:=0;i<100;i++{
			sum+=i
		}
		ch<-sum  //将goroutine 算出的数放进通道
	}()
	fmt.Println(<-ch) //从通道获取数据，退出main函数，如果channel还没有存入数据，就会阻塞等待
}
```

上面这个简单的例子， 执行顺序是先创建一个channel，开启一个goroutine进行计算，然后打印从channel取出的数。会先执行`fmt.Println(<-ch)`，这时候goroutine还没有往里面写数据，此时main函数进入等待。一直到goroutine运算完将sum发送给channel，这时候main函数马上收到数据，打印完退出。

## 3、无缓冲的channel

**无缓冲的通道指的是通道的大小为0，也就是说，这种类型的通道在接收前没有能力保存任何值，它要求发送goroutine和接收goroutine同时准备好，才可以完成发送和接收操作。**

接下来看一个小案例，channel 好比是乒乓球的球桌，球好比数据，数据在channel通信就好比乒乓球在球桌上来回弹，而两个goroutine就是两位选手，这个就是无缓冲的channel的例子

```go
package main

import (
	"fmt"
	"math/rand"
	"sync"
	"time"
)
var wg sync.WaitGroup
func main() {
	ch := make(chan int) //乒乓球台看做通道
	wg.Add(2)            //2个goroutine
	//相当于两个选手对打
	go player("张继科", ch)
	go player("马龙", ch)
	ch <- 1   //发球
	wg.Wait() //等待比赛结束

}
func player(name string, ch chan int) {
	defer  wg.Done()//一方输了就告诉main函数，裁判不要等了
	for {
		ball, ok := <-ch
		if !ok { //如果通道关闭
			fmt.Printf("%s赢了！！\n", name)
			return
		}
		n := rand.Intn(100)
		if n%13 == 0 { //随机数来决定自己是否失误
			fmt.Printf("%s输了\n", name)
			close(ch) //输了就得关闭通道
			return
		}
		fmt.Printf("%s击球第%d次\n", name, ball)
		ball++
		ch <- ball

	}
}

```

从上面的例子看出，通道是球桌，球在球桌上来回传递，统计次数。两个选手用for循环持续的在接收和发送数据，也就是要么接球要么发球。之所以他们在相互等待对方是因为这个channel是一个无缓冲的channel，也就是球不能放在球桌上，球桌只管传递，不能存储。再看一个例子：

```go
package main
import (
	"fmt"
	"sync"
	"time"
)
var wg sync.WaitGroup
func main() {//4x100米接力比赛
	ch:=make(chan int) //接力棒
	wg.Add(1) //需要等待的是最后一棒
	go runing(ch)  
	ch<-1
	fmt.Println("比赛开始")
	wg.Wait()
}
func runing(ch chan int){
	var newRunner int
	runner:=<-ch   //接棒
	fmt.Printf("第%d棒正在跑\n",runner)

	time.Sleep(time.Second)//跑步中
	if runner==4 {//第四棒
		fmt.Printf("跑完了\n")
		wg.Done()
		return
	}else{//没跑完，创建下一棒
		newRunner=runner+1
		fmt.Printf("第%d棒准备就绪\n",newRunner)
		go runing(ch)//等待接棒
		fmt.Printf("接力棒传递给第%d棒\n",newRunner)
		ch<-newRunner //接力
	}
}
```

上面的例子很有趣，模拟4x100米接力，我们创建一个无缓冲的channel，比作接力棒，只有双方都准备好接收和发送，接力才会发生，不然一方就会处于等待期。 传递给channel 一个数字1，表示比赛开始，第一棒取出runner，只要不是第4棒就需要往下一棒传递，所以，就创建了第二棒，让他准备继续接力。知道runner==4，比赛结束。

总结下：无缓冲channel，可存储的大小为0，它保证进行发送和接收的 goroutine 会在同一时间进行数据交换 。如果发送方没有准备好发送，接收方会进入阻塞，等待发送。

## 4、有缓冲的channel

有缓冲通道，其实是一个队列，这个队列的最大容量就是我们使用`make`函数创建通道时，通过第二个参数指定的。

```go
ch := make(chan int, 5)
```

这里创建容量为5的，有缓冲的通道。对于有缓冲的通道，向其发送操作就是向队列的尾部插入元素，接收操作则是从队列的头部删除元素，并返回这个刚刚删除的元素。

**当队列满的时候，发送操作会阻塞；当队列空的时候，接收操作会阻塞。有缓冲的通道，不要求发送和接收操作时同步的，相反可以解耦发送和接收操作。** 

```go
// cap  和 len 函数同样对于有缓冲的channel可用，
cap(ch)    	//channel 容量
len(ch)		//当前channel内的数量
```

看代码：

```go
func mirroredQuery() string {
	responses := make(chan string, 3)
	go func() { responses <- request("asia.gopl.io") }()
	go func() { responses <- request("europe.gopl.io") }()
	go func() { responses <- request("americas.gopl.io") }()
	return <-responses // return the quickest response
} 
func request(hostname string) (response string) { /* ... */ }
```

我们定义了一个容量为3的通道`responses`，然后同时发起3个并发goroutine向这三个镜像获取数据，获取到的数据发送到通道`responses`中，最后我们使用`return <-responses`返回获取到的第一个数据，也就是最快返回的那个镜像的数据。

## 5、单项通道

为了避免channel混乱使用，还可以在定义的时候定义这个channel是单项的，即只能发送数据，或者只能接受数据。比如：

```go
var send chan<- int //只能发送——只能往channel里发数据
var receive <-chan int //只能接收——只能从channel中取

//我们定义函数的时候，可以明确声明接受的参数
func  test(ch chan<- int){
    //接受的是一个只能发送数据的channel，
}
```

区别主要在于 `<-`符号的位置，在后面，往里发，，在前面，从里面取。好像一列车穿过隧道。

**注意：不能把单项channel 转换为普通channel**

```go 
d := (chan int)(send) // Error: cannot convert type chan<- int to type chan int
d := (chan int)(receive) // Error: cannot convert type <-chan int to type chan int
```



## 6、forange 迭代

```go
package main

import (
	"fmt"
)

func main() {
	data := make(chan int) // 数据交换队列
	exit := make(chan bool) // 退出通知
	go func() {
		for d := range data { // 从队列迭代接收数据，直到 close 。
			fmt.Println(d)
		}
		fmt.Println("recv over.")
		exit <- true // 发出退出通知。
	}()
	data <- 1 // 发送数据。
	data <- 2
	data <- 3
	close(data) // 关闭队列。
	fmt.Println("send over.")
	<-exit // 等待退出通知。
}
```

**forange 用于channel 有一个特点，就是一直进行迭代，不管channel有没有数据，直到channel （close）关闭**。这样既安全又便利，当channel关闭时，for循环会自动退出，无需主动监测channel是否关闭，可以防止读取已经关闭的channel，造成读到数据为通道所存储的数据类型的零值。

## 7、select 关键字

之前的例子都是使用1个channel进行通信，当我们使用多个channel进行通信时，就需要用到 select关键字来进行管理。

- 可处理一个或多个channel的发送和接收
- 同时又多个可用的channel的按随机顺序处理
- 可用空的select来阻塞main函数
- 可设置超时
- **当通道为nil时，对应的case永远为阻塞，无论读写。特殊关注：而普通情况下，对nil的通道写操作是要panic的**。

###  7.1 处理多个channel发送和接收

```go
package main

import (
	"fmt"
	"os"
)
func main() {
	a, b := make(chan int, 3), make(chan int)
	go func() {
		v, ok, s := 0, false, ""
		for {
			select { // 随机选择可用 channel，接收数据。
			case v, ok = <-a:
				s = "a"
			case v, ok = <-b:
				s = "b"
			}
			if ok {
				fmt.Println(s, v)
			} else {
				os.Exit(0)
			}
		}
	}()
	for i := 0; i < 5; i++ {
		select { // 随机选择可用 channel，发送数据。
		case a <- i:
		case b <- i:
		}
	}
	close(a)
	select {} // 没有可用 channel，阻塞 main goroutine。
}

```

我们的例子中，使用select有点像switch， 它可以管理多个channel，随机的发送也可以随机的获取数据。最后我们用**select{}** 很巧妙的阻塞了main goroutine ,因为没有可用的channel，它进入阻塞直到channel 关闭，执行了os.Exit(0) main函数才推出。

### 7.2 设置超时

```go
package main

import (
    "fmt"
    "time"
)
func main() {
    exit := make(chan bool)
    c1 := make(chan int, 2)
    c2 := make(chan string, 2)

    go func() {
        select {
        case vi := <-c1:
            fmt.Println(vi)
        case vs := <-c2:
            fmt.Println(vs)
        case <-time.After(time.Second * 3):
            fmt.Println("timeout.")
        }

        exit <- true
    }()
	//我们先把发送数据代码注释掉。
    //这里我们并没有向c1 和c2 发送任何数据，select 超时后就会打印 timeout
    //c1<-10
    //c2<-"加油"
    <-exit
}
```



**当然select 还有default ，但是在循环中使用default一定要小心，小心，小心。**

## 8、channel 总结

channel存在3种状态

- nil，未初始化的状态，只进行了声明，或者手动赋值为`nil`
- active，正常的channel，可读或者可写
- closed,已关闭的channel。 关闭的channel存储的是类型零值。

| 操作  | nil通道 | closed 关闭的通道 | active正常通道 |
| ----- | ------- | ----------------- | -------------- |
| close | panic   | panic             | 成功           |
| ch<-  | 死锁    | panic             | 阻塞或成功     |
| <-ch  | 死锁    | 零值              | 阻塞或成功     |

对于nil通道的情况，也并非完全遵循上表，**有1个特殊场景**：**当`nil`的通道在`select`的某个`case`中时，这个case会阻塞，但不会造成死锁。**

- channel分为有缓冲和无缓冲
- channel 有单项channel
- 可以使用forange迭代，直到channel 关闭
- 可以用select 管理多个channel，随机处理读和写
- 可以用select{}阻塞main函数
- select 可以设置超时



## 9、channel 应用场景小结

### 9.1 forange 迭代，无需关注channel 是否关闭

场景：在需要不断从channel 取数据时，而不用关心channel是否关闭

```go
for x := range ch{
    fmt.Println(x)
}
//会一直迭代，直到channel 关闭
```

### 9.2 使用`_,ok`判断channel是否关闭

场景：在不确定channel是否关闭时，使用

```go
if v, ok := <- ch; ok {
    fmt.Println(v)
}
```

ok的含义：

- `true`：读到数据，并且通道没有关闭。
- `false`：通道关闭，无数据读到。

### 9.3使用select处理多个channel

场景：需要对多个通道进行同时处理，但只处理最先发生的channel时，见上面的例子

注意：**当通道为nil时，对应的case永远为阻塞，无论读写。特殊关注：普通情况下，对nil的通道写操作是要panic的**。

### 9.4 使用channel 传递结构体时，用指针

场景：channel 传递的数据是结构体时，最好用指针。

channel本质上传递的是数据的拷贝，拷贝的数据越小传输效率越高，传递结构体指针，比传递结构体更高效



### 9.5 简单⼯⼚模式打包并发任务和 channel

```go
package main
import (
	"math/rand"
	"time"
)
func NewTest() chan int { //简单工厂方法返回一个channel
	c := make(chan int)
	rand.Seed(time.Now().UnixNano())
	go func() {
		time.Sleep(time.Second)
		c <- rand.Int()  
	}()
	//并且返回的channel 是已经准备好发送的，阻塞中，只要接收方一准备好，立马数据就传递出去了
	return c
}
func main() {
	t := NewTest()
	println(<-t) // 等待 goroutine 结束返回。
}
```

### 9.6 ⽤channel 实现信号量 (semaphore) 

简单解释下信号量，也叫信号灯，是可以用来保证两个或多个关键代码段不被并发调用。信号量有四种操作，1、初始化，2、等信号3、发信号4、清理

```go
package main
import (
	"fmt"
	"sync"
)
func main() {
	wg := sync.WaitGroup{}
	wg.Add(3)
	sem := make(chan int,1)
	for i := 0; i < 3; i++ {
		go func(id int) {
			defer wg.Done()
			sem <- 1 // 向 sem 发送数据，阻塞或者成功。
			fmt.Printf("第%d个\n",id)
			for x := 0; x < 3; x++ {
				fmt.Println(id, x)
			}
			<-sem // 接收数据，使得其他阻塞 goroutine 可以发送数据。
		}(i)
	}
	wg.Wait()
}
//输出
第2个
2 0
2 1
2 2
第0个
0 0
0 1
0 2
第1个
1 0
1 1
1 2

```

这里的channel 是一个容量为1的有缓冲的通道。也就是说，它只能存一个信号，比如开了3个goroutine，只有一个能发送进去，其他的都会阻塞，等到这个goroutine处理完自己的事情，将数据取出<-，那么第二个goroutine就会发送，执行，然后取出。接着是第三个goroutine。  这样就实现了信号量，保证goroutine一个个的执行。

### 9.7 利用从closed channel取值，发出退出的通知

**应用场景：关闭所有下游的goroutine**

nil 的channel在select 中是永久阻塞的，case是不会走的，但是关闭了的channel，就会走。

从关闭了的channel中取值  <-   是不会引发panic，会取出零值。

实现思路就是： 在channel取值，是阻塞的，只要一关闭channel，取值就是零值，然后执行退出就可以了。

通过将nil  channel关闭，使select的 阻塞channel 变为取出零值， case退出代码执行，所有读取这个channel的goroutine就会执行关闭代码。

```go
package main
import (
	"sync"
	"time"
)
func main() {
	var wg sync.WaitGroup
	quit := make(chan bool)
	for i := 0; i < 2; i++ {
		wg.Add(1)
		go func(id int) {
			defer wg.Done()
			for {
				select {
				case <-quit: // closed channel 不会阻塞，会取出零值，因此可用作退出通知。
					return
				default: // 执行正常任务。
					func() {
						println(id, time.Now().Nanosecond())
						time.Sleep(time.Second)
					}()
				}
			}
		}(i)
	}
	time.Sleep(time.Second * 5) // 让测试 goroutine 运⾏⼀会。
	close(quit)                 // 发出退出通知。
	wg.Wait()
}

```

### 9.8 channel作为结构成员 

channel 可以做未结构体的成员，可以封装的更好。

```go
package main
import (
	"fmt"
)

type Request struct {
	data []int
	ret  chan int
}
func NewRequest(data ...int) *Request {
	return &Request{data, make(chan int, 1)}
}
//使用结构体指针，效率更高。
func Process(req *Request) {
	x := 0
	for _, i := range req.data {
		x += i
	}
	req.ret <- x
}
func main() {
	req := NewRequest(10, 20, 30)
	Process(req)
	fmt.Println(<-req.ret)
}

```



