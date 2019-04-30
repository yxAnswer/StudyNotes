# Go语言并发1——Goroutine

[TOC]

# 1、并发和并行

## 1.1 进程和线程

进程：程序启动时（比如qq），操作系统位程序开启一个进程。可以把它看做是操作系统进行资源分配和调度的一个容器，里面包含了该应用程序用到的所有资源。

线程：是一个独立的执行空间，用来被系统调度来运行程序代码。比如我下载文件，操作系统调度会安排到合适的cpu上进行执行，并且不定是该程序进程所在的cpu。这个调度我们不用关心。

也就是一个进程可以有好多个线程，线程用来执行具体的任务。每个进程的初始线程叫做主线程，所以进程至少有一个线程。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190328101903497.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

上图，系统的调度器来调度线程在合适的cpu上运行。

## 1.2 并发和并行的概念

并发：多线程或多协程在一核cpu上运行就是所谓的并发，都是cpu通过切换时间片给人一种并发的感觉。

并行：是真正意义上的并发，就是多核cpu同时去处理多个线程，互不干扰，并行处理。

并发不是并行：并行是让不同的代码片段同时在不同的cpu上执行，利用多核的优势。并发是通过非常快切换时间片来实现“同时”运行。**并行的关键是同时做很多的事情，而并发是指同时管理很多事情，这些事情可能只做了一半就暂停做别的任务。**

总结：并发优于并行，可以有效利用资源。go语言可以利用多核，通过goroutine 高效并发。

## 1.3 go语言逻辑处理器和调度器了解

go语言通过一个叫做goroutine的东西进行并发执行，每一个goroutine就是一个独立的工作单元，可以执行我们的程序代码。goroutine是类似协程（coroutine）的东西，可以理解为一个更轻量的线程。goroutine的具体使用后面再讲，目前我们来看一下go语言是如何通过goroutine实现并发的。

目前可以理解为：goroutine是个执行代码的独立工作单元，需要将它放到合适的线程和cpu上进行执行。这就需要go语言的逻辑处理器和调度器。

go语言中支撑整个scheduler实现的主要有4个重要结构，分别是M、G、P、Sched。

- Sched结构就是调度器，它维护有存储M和G的队列以及调度器的一些状态信息等。
- M结构是Machine，系统线程，它由操作系统管理的，goroutine就是跑在M之上的；M是一个很大的结构，里面维护小对象内存cache（mcache）、当前执行的goroutine、随机数发生器等等非常多的信息。
- P结构是Processor，逻辑处理器，它的主要用途就是用来执行goroutine的，它维护了一个goroutine队列，即runqueue。Processor是让我们从N:1调度到M:N调度的重要部分。
- G是goroutine实现的核心结构，它包含了栈，指令指针，以及其他对调度goroutine很重要的信息，例如其阻塞的channel。

这里借用一张图来看下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190328105654453.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)



注意：go1.5之后为每个cpu创建一个逻辑处理器。

我们从一个goroutine被创建到goroutine被执行的过程来看一下go是如何实现调度的。

（1）创建一个**goroutine**，它会放在**全局运行队列**中，等待调度器调度

（2）**调度器**将这个goroutine  分配给一个**逻辑处理器A**，将它放到了这个逻辑处理器的**本地队列**中，这个goroutine就会**等待**逻辑处理器A**执行**它

（3）每个逻辑处理器默认**绑定了一个线程**，它是在线程中去**执行自己本地队列**中的goroutine。

- 如果逻辑处理器目前运行的goroutine是阻塞的，比如打开文件操作。

（4）逻辑处理器和原来的**线程分离**，调度器重新**创建一个线程**和这个逻辑处理器**绑定**。这时候逻辑处理器在新的线程上继续执行本地运行队列的其他goroutine。 同时，阻塞的goroutine随着线程分离，从本地队列移除。

（5）那个阻塞的goroutine和分离的线程会继续阻塞，等待系统调用的返回。一旦执行完成并返回，这个goroutine就会**重新放回**到原来逻辑处理器的本地队列。

（6）之前的线程目前没有goroutine了，但是它会被保存，以备之后使用。

调度器对可以创建的逻辑处理器的数量没有限制，但语言运行时默认限制每个程序最多创建 10 000 个线程。这个
限制值可以通过调用 runtime/debug 包的 SetMaxThreads 方法来更改。 

小结：

| 概念         | 说明                                                  |
| ------------ | ----------------------------------------------------- |
| 进程         | 一个程序对应的资源容器                                |
| 线程         | 一个独立的执行空间，一个进程可以有多个线程            |
| goroutine    | 同样是独立的执行空间，但是一个线程可以有多个goroutine |
| 逻辑处理器   | 绑定一个线程，运行goroutine                           |
| 调度器       | 将goroutine分配到合适的逻辑处理器                     |
| 全局运行队列 | 所有刚创建的goroutine都在这                           |
| 本地运行队列 | 逻辑处理器的goroutine队列                             |

所以，我们可以利用多核cpu，调度器创建多个逻辑处理器，然后每个逻辑处理器可以绑定一个线程去运行多个goroutine。这样我们就充分利用了多核资源实现并发处理，比单纯的多线程更加优秀，高效，省资源。

注意:上面第（4）（5）步，如果goroutine 是在执行网络io的操作，这个goroutine就不一定就回到这个逻辑处理器了。它实际上会先从逻辑处理器分离，移到集成了网络轮询器的运行时 ，一旦该轮询器指示某个网络读或者写操作已经就绪，对应的 goroutine 就会**重新分配**到逻辑处理器上来完成操作 。

**看到这我们重温下并发和并行：**

go语言实现并发，创建多个goroutine，调度器会将goroutine分配到逻辑处理器的本地运行队列，逻辑处理器去运行goroutine。如果只有一个逻辑处理器，只会实现并发，不会实现并行。 

要实现并行，就需要多个逻辑处理器，在不同的cpu上，然后调度器会平等的将goroutine分配到每个逻辑处理器，这样多个线程多个goroutine就实现了并行和并发。 至于这些算法怎么调度，我们根本不需要关心，我们只要记住goroutine是我们进行并发编程的一个独立单元就可以了。

# 2、goroutine使用

goroutine其实是官方实现的超级“轻量线程池”。每个实例4~5kb的占内存占用，更加轻量。只需在函数调⽤语句前添加 go 关键字，就可创建并发执⾏单元。开发⼈员⽆需了解任何执⾏细节，调度器会⾃动将其安排到合适的系统线程上执⾏。

## 2.1 go 关键字创建goroutine

```go
package main
import (
    "fmt"
    "time"
)
func main() {
    //通过go 关键字 +匿名函数就可以开启一个goroutine
    go func() { 
        fmt.Println("Hello, World!")
    }()
    //由于main函数也是一个goroutine，如果不让线程等待，那么main方法执行完，就退出了，还来不及打印helloworld
    time.Sleep(1 * time.Second)
}
```

## 2.2 简单使用waitgroup同步

WaitGroup能够一直等到所有的goroutine执行完成，并且阻塞主线程的执行，直到所有的goroutine执行完成。

WaitGroup总共有三个方法：Add(delta int)，Done()，Wait()。简单的说一下这三个方法的作用。

| 方法名 | 说明                                           |
| ------ | ---------------------------------------------- |
| Add    | 添加或者减少等待goroutine的数量；              |
| Done   | 相当于Add(-1)，减少一个需要等待的goroutine数量 |
| Wait   | 进行等待，需等待的goroutine数量为0             |

WaitGroup用于线程同步，WaitGroup等待一组线程集合完成，才会继续向下执行。 主线程(goroutine)调用Add来设置等待的线程(goroutine)数量。 然后每个线程(goroutine)运行，并在完成后调用Done。 同时，Wait用来阻塞，直到所有线程(goroutine)完成才会向下执行。

```go
package main
import (
	"fmt"
	"runtime"
	"sync"
)
func main() {
	runtime.GOMAXPROCS(1)//只使用1个物理处理器
	var wg sync.WaitGroup
	wg.Add(2) //添加需要等待goroutine数量
	fmt.Println("开启两个goroutine")
	go func() {
		defer  wg.Done()//函数结束时通知main函数执行完毕
		for i:=0;i<1000;i++{
			fmt.Println("A:",i)
		}
	}()
	go func() {
		defer wg.Done()//函数结束时通知main函数执行完毕
		for i := 0; i < 1000; i++ {
			fmt.Println("B:",i)
		}
	}()
	fmt.Println("等待goroutine运行")
	wg.Wait()
	fmt.Println("程序结束")
}
```

从上面的代码我们可以看到，我们用sync包下的waitgroup 来进行线程等待，避免main函数执行完来不及执行goroutine就退出的情况。waitgroup详情下面会讲。我们用go关键字+匿名函数 开启了两个goroutine，来并发的去打印，但是因为这个1000个打印速度太快了，还没来得及切换goroutine就第一个就已经打印完了，所以你可能会输出，A打印完了才打印B这种顺序输出。我们可以增加一下打印时间，就能看到他们是并发打印的了。比如加个time.Sleep(time.Millisecond) 。 这段代码中有一个runtime.GOMAXPROCS(1),这是go语言可以指定程序运行使用的cpu核数，我们可以设置多个，来实现并行。

一般来说，通过`runtime.GOMAXPROCS(runtime.NumCPU())`可以设置本机逻辑CPU的数量，不是物理CPU，比如一个双核CPU，带有超线程技术，则会被认为是4个逻辑CPU。 **runtime.Gosched ()** 可以让出底层线程，让其他goroutine 使用，**runtime.Goexit** 将立即终止当前goroutine 执行

**runtime 小结：**

```go
runtime.GOMAXPROCS()  //设置使用的逻辑处理器数量
runtime.NumCPU()   //本地逻辑cpu的数量
runtime.Gosched()  // 将当前goroutine的线程让给别的goroutine，自己进入运行队列等待
runtime.Goexit()   //立即终止当前goroutine 运行
runtime.GOROOT()	//获取go的根目录
runtime.GOOS		// 获取操作系统信息
```

## 2.3 WaitGroup 传值问题

sync.WaitGroup  类型的变量是一个值类型，如果在函数间进行传递，是值传递，这样执行Done()和 wait()方法就不是同一个 WaitGroup了，就会出现死锁，所以**传递时必须传递指针**。 代码如下：

```go
package main
import (
    "fmt"
    "sync"
)
func main() {
    wg := new(sync.WaitGroup)
    for i := 0; i < 5; i++ {
        wg.Add(1)
        go func(wg *sync.WaitGroup, i int) {
            fmt.Printf("i=>%d\n", i)
            wg.Done()
        }(&wg, i) //这里要传指针就对了
    }
    wg.Wait()
}
```



# 3、资源竞争

如果两个或者多个 goroutine 在没有互相同步的情况下，访问某个共享的资源，并试图同时读和写这个资源，就处于相互竞争的状态，这种情况被称作竞争状态（race candition）。 我们要做的是：同一时刻只能有一个 goroutine 对共享资源进行读和写操作 。

```go
package main
import (
	"fmt"
	"runtime"
	"sync"
)
var (
	count int
	wg sync.WaitGroup
)
func main(){
	wg.Add(2)
    //开启两个goroutine
	go incCount(1)
	go incCount(2)
	wg.Wait()
	fmt.Println("最终结果：",count)
}
//执行两次  count++
func incCount(id int) {
	defer  wg.Done()
	for i:=0;i<2;i++{
		value:=count
		runtime.Gosched()
		value++
		count=value
	}
}
//输出
最终结果： 2
```

从上面可以看出，我们开启两个goroutine，每个goroutine，都执行了两个value++并赋值给count，也就是说最终的结果应该是4，但是现在确是2。 毫无疑问，在对count 进行读写的时候，两个goroutine进行了资源竞争，并且没有同步。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190329152711330.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

程序运行就像图中所示，两个goroutine在进行切换的时候，并没有同步count的数量，并且他们相互覆盖了对方，导致各自有一般的工作白做了。

## 3.1 实用go 自带的竞争监测命令-race

```go
go run -race goDemo.go//  -race go自带的竞争监测命令，可以查看哪一行哪些方法有资源竞争。
```

```go
==================
WARNING: DATA RACE
Read at 0x0000005fa2d0 by goroutine 7:
  main.incCount()
      D:/gopath/src/awesomeProject/goroutine/godemo.go:23 +0x76

Previous write at 0x0000005fa2d0 by goroutine 6:
  main.incCount()
      D:/gopath/src/awesomeProject/goroutine/godemo.go:26 +0x97

Goroutine 7 (running) created at:
  main.main()
      D:/gopath/src/awesomeProject/goroutine/godemo.go:16 +0x90

Goroutine 6 (finished) created at:
  main.main()
      D:/gopath/src/awesomeProject/goroutine/godemo.go:15 +0x6f
==================
最终结果： 4
Found 1 data race(s)
exit status 66

```



**如何解决资源竞争和线程同步，这就有两类，一类是传统的方式——加锁，另一类是go语言有的通过chanel,采用csp模型，即通过通信去共享内存，而不是通过共享内存而通信。**

# 4、资源同步传统方式——加锁

我们要实现**同一时间只能有一个goroutine对共享资源进行读写操作**，go语言提供了传统的解决方案，atomic和sync 包。 另一种方式是使用channel，下一篇单独讲。

## 4.1原子函数atomic

atomic 包提供了一些函数来保证对资源的读写安全。比如LoadInt32  和 StoreInt32两个函数，一个读取int32类型的值，一个写入int32类型的值。如下：

```go
package main
import (
	"fmt"
	"runtime"
	"sync"
	"sync/atomic"
)
var (
	count int32
	wg sync.WaitGroup
)
func main(){
	wg.Add(2)
	go incCount(1)
	go incCount(2)
	wg.Wait()
	fmt.Println("最终结果：",count)
}
func incCount(id int) {
	defer  wg.Done()
	for i:=0;i<2;i++{
		value:=atomic.LoadInt32(&count)
		runtime.Gosched()
		value++
		atomic.StoreInt32(&count,value)
	}
}
```

这时候执行结果是4，读写安全了。atomic虽然可以解决资源竞争问题，但是比较都是比较简单的，支持的数据类型也有限。所以，sync 提供了互斥锁来解决。

## 4.2 互斥锁 mutex

sync包里提供了一种互斥型的锁，可以让我们自己灵活的控制哪些代码，同时只能有一个goroutine访问，被sync互斥锁控制的这段代码范围，被称之为**临界区**，临界区的代码，同一时间，只能有一个goroutine访问。代码如下：

```go
package main
import (
	"fmt"
	"runtime"
	"sync"
)
var (
	count int32
	wg    sync.WaitGroup  
	mutex sync.Mutex  //声明 mutex  互斥锁变量
)

func main() {
	wg.Add(2)  //2个等待的goroutine
	go incCount()
	go incCount()
	wg.Wait()
	fmt.Println(count)
}

func incCount() {
	defer wg.Done() 
	for i := 0; i < 2; i++ {
		mutex.Lock()  //临界区开始位置
		value := count
		runtime.Gosched()
		value++
		count = value
		mutex.Unlock()//临界区结束位置
	}
}
```

我们还是使用 sync.WaitGroup  来进行等待两个goroutine都执行完再推出main函数。 重点看我们还声明了一个

`mutext sync.Mutex` 这个互斥锁，通过`mutex.Lock()`加锁， `mutex.Unlock()`解锁。它将中间的代码块形成一个临界区，所以，这段代码块同时只能有一个goroutine进行操作，所以，goroutine1 将count赋值给value，让出线程，此时goroutine2也无法进入临界区的代码，等待goroutine1 执行完**临界区**的代码，goroutine2再进行执行。这样就保证了资源的读写安全。



当然goroutine 同步还有更好，更简单的方式，使用channel。即所谓的：**通过通信来共享内存，而不是通过共享内存来通信。**


























