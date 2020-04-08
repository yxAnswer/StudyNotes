[TOC]

# log日志（转）

本文转载自：[https://www.flysnow.org/2017/05/06/go-in-action-go-log.html](https://www.flysnow.org/2017/05/06/go-in-action-go-log.html)

可以关注原作者博客网站<http://www.flysnow.org/

## 1、日志使用

日志分析，就是根据输出的日志信息，分析挖掘可能的问题，我们使用`fmt.Println`系列函数也可以达到目的，因为它们也可以把我们需要的信息输出到终端或者其他文件中。不过`fmt.Println`系列函数输出的系统比较简单，比如没有时间，也没有源代码的行数等，对于我们排查问题，缺少了很多信息。

对此，Go语言为我们提供了标准的`log`包，来跟踪日志的记录。下面我们看看日志包`log`的使用。

```go
func main() {
	log.Println("飞雪无情的博客:","http://www.flysnow.org")
	log.Printf("飞雪无情的微信公众号：%s\n","flysnow_org")
}
```

使用非常简单，函数名字和用法也和`fmt`包很相似，但是它的输出默认带了时间戳。

```bash
2017/04/29 13:18:44 飞雪无情的博客: http://www.flysnow.org
2017/04/29 13:18:44 飞雪无情的微信公众号：flysnow_org
```

这样我们很清晰的就知道了，记录这些日志的时间，这对我们排查问题，非常有用。

有了时间了，我们还想要更多的信息，必然发生的源代码行号等，对此日志包`log` 为我们提供了可定制化的配制，让我们可以自己定制日志的抬头信息。

```go
func init(){
	log.SetFlags(log.Ldate|log.Lshortfile)
}
```

我们使用`init`函数，这个函数在`main`函数执行之前就可以初始化，可以帮我们做一些配置，这里我们自定义日志的抬头信息为时间+文件名+源代码所在行号。也就是`log.Ldate|log.Lshortfile`,中间是一个位运算符`|`，然后通过函数`log.SetFlags`进行设置。现在我们再运行下看看输出的日志。

```go
2017/04/29 main.go:10: 飞雪无情的博客: http://www.flysnow.org
2017/04/29 main.go:11: 飞雪无情的微信公众号：flysnow_org
```

比着上一个例子，多了源文件以及行号，但是少了时间，这就是我们自定义出来的结果。现在我们看看`log`包为我们提供了那些可以定义的选项常量。

```go
const (
	Ldate         = 1 << iota     //日期示例： 2009/01/23
	Ltime                         //时间示例: 01:23:23
	Lmicroseconds                 //毫秒示例: 01:23:23.123123.
	Llongfile                     //绝对路径和行号: /a/b/c/d.go:23
	Lshortfile                    //文件和行号: d.go:23.
	LUTC                          //日期时间转为0时区的
	LstdFlags     = Ldate | Ltime //Go提供的标准抬头信息
)
```

这是log包定义的一些抬头信息，有日期、时间、毫秒时间、绝对路径和行号、文件名和行号等，在上面都有注释说明，这里需要注意的是：如果设置了`Lmicroseconds`，那么`Ltime`就不生效了；设置了`Lshortfile`， `Llongfile`也不会生效，大家自己可以测试一下。

`LUTC`比较特殊，如果我们配置了时间标签，那么如果设置了`LUTC`的话，就会把输出的日期时间转为0时区的日期时间显示。

```go
log.SetFlags(log.Ldate|log.Ltime |log.LUTC)
```

那么对我们东八区的时间来说，就会减去8个小时，我们看输出：

```bash
2017/04/29 05:46:29 飞雪无情的博客: http://www.flysnow.org
2017/04/29 05:46:29 飞雪无情的微信公众号：flysnow_org
```

最后一个`LstdFlags`表示标准的日志抬头信息，也就是默认的，包含日期和具体时间。

我们大部分情况下，都有很多业务，每个业务都需要记录日志，那么有没有办法，能区分这些业务呢？这样我们在查找日志的时候，就方便多了。

对于这种情况，Go语言也帮我们考虑到了，这就是设置日志的前缀，比如一个用户中心系统的日志，我们可以这么设置。

```go
func init(){
	log.SetPrefix("【UserCenter】")
	log.SetFlags(log.LstdFlags | log.Lshortfile |log.LUTC)
}
```

通过`log.SetPrefix`可以指定输出日志的前缀，这里我们指定为`【UserCenter】`，然后就可以看到日志的打印输出已经清晰的标记出我们的这些日志是属于哪些业务的啦。

```bash
【UserCenter】2017/04/29 05:53:26 main.go:11: 飞雪无情的博客: http://www.flysnow.org
【UserCenter】2017/04/29 05:53:26 main.go:12: 飞雪无情的微信公众号：flysnow_org
```

`log`包除了有`Print`系列的函数，还有`Fatal`以及`Panic`系列的函数，其中`Fatal`表示程序遇到了致命的错误，需要退出，这时候使用`Fatal`记录日志后，然后程序退出，也就是说`Fatal`相当于先调用`Print`打印日志，然后再调用`os.Exit(1)`退出程序。

同理`Panic`系列的函数也一样，表示先使用`Print`记录日志，然后调用`panic()`函数抛出一个恐慌，这时候除非使用`recover()`函数，否则程序就会打印错误堆栈信息，然后程序终止。

这里贴下这几个系列函数的源代码，更好理解。

```go
func Println(v ...interface{}) {
	std.Output(2, fmt.Sprintln(v...))
}

func Fatalln(v ...interface{}) {
	std.Output(2, fmt.Sprintln(v...))
	os.Exit(1)
}

func Panicln(v ...interface{}) {
	s := fmt.Sprintln(v...)
	std.Output(2, s)
	panic(s)
}
```

## 2、实现原理

通过上面的源代码，我们发现，日志包`log`的这些函数都是类似的，关键的输出日志就在于`std.Output`方法。

```go
func New(out io.Writer, prefix string, flag int) *Logger {
	return &Logger{out: out, prefix: prefix, flag: flag}
}

var std = New(os.Stderr, "", LstdFlags)
```

从以上源代码可以看出，变量`std`其实是一个`*Logger`，通过`log.New`函数创建，默认输出到`os.Stderr`设备，前缀为空，日志抬头信息为标准抬头`LstdFlags`。

`os.Stderr`对应的是UNIX里的标准错误警告信息的输出设备，同时被作为默认的日志输出目的地。除此之外，还有标准输出设备`os.Stdout`以及标准输入设备`os.Stdin`。

```go
var (
	Stdin  = NewFile(uintptr(syscall.Stdin), "/dev/stdin")
	Stdout = NewFile(uintptr(syscall.Stdout), "/dev/stdout")
	Stderr = NewFile(uintptr(syscall.Stderr), "/dev/stderr")
)
```

以上就是定义的UNIX的标准的三种设备，分别用于输入、输出和警告错误信息。理解了`os.Stderr`，现在我们看下`Logger`这个结构体，日志的信息和操作，都是通过这个`Logger`操作的。

```go
type Logger struct {
	mu     sync.Mutex // ensures atomic writes; protects the following fields
	prefix string     // prefix to write at beginning of each line
	flag   int        // properties
	out    io.Writer  // destination for output
	buf    []byte     // for accumulating text to write
}
```

1. 字段`mu`是一个互斥锁，主要是是保证这个日志记录器`Logger`在多goroutine下也是安全的。
2. 字段`prefix`是每一行日志的前缀
3. 字段`flag`是日志抬头信息
4. 字段`out`是日志输出的目的地，默认情况下是`os.Stderr`。
5. 字段`buf`是一次日志输出文本缓冲，最终会被写到`out`里。

了解了结构体`Logger`的字段，现在就可以看下它最重要的方法`Output`了，这个方法会输出格式化好的日志信息。

```go
func (l *Logger) Output(calldepth int, s string) error {
	now := time.Now() // get this early.
	var file string
	var line int
	//加锁，保证多goroutine下的安全
	l.mu.Lock()
	defer l.mu.Unlock()
	//如果配置了获取文件和行号的话
	if l.flag&(Lshortfile|Llongfile) != 0 {
		//因为runtime.Caller代价比较大，先不加锁
		l.mu.Unlock()
		var ok bool
		_, file, line, ok = runtime.Caller(calldepth)
		if !ok {
			file = "???"
			line = 0
		}
		//获取到行号等信息后，再加锁，保证安全
		l.mu.Lock()
	}
	//把我们的日志信息和设置的日志抬头进行拼接
	l.buf = l.buf[:0]
	l.formatHeader(&l.buf, now, file, line)
	l.buf = append(l.buf, s...)
	if len(s) == 0 || s[len(s)-1] != '\n' {
		l.buf = append(l.buf, '\n')
	}
	//输出拼接好的缓冲buf里的日志信息到目的地
	_, err := l.out.Write(l.buf)
	return err
}
```

整个代码比较简洁，为了多goroutine安全互斥锁也用上了，但是在获取调用堆栈信息的时候，又要先解锁，因为这个过程比较重。获取到文件、行号等信息后，继续加互斥锁保证安全。

后面的就比较简单了，`formatHeader`方法主要是格式化日志抬头信息，然后存储在`buf`这个缓冲中，最后再把我们自己的日志信息拼接到缓冲`buf`的后面，然后为一次log日志输出追加一个换行符，这样每次日志输出都是一行一行的。

有了最终的日志信息`buf`，然后把它写到输出的目的地`out`里就可以了，这是一个实现了`io.Writer`接口的类型，只要实现了这个接口，都可以当作输出目的地。

```go
func (l *Logger) SetOutput(w io.Writer) {
	l.mu.Lock()
	defer l.mu.Unlock()
	l.out = w
}
```

`log`包的`SetOutput`函数，可以设置输出目的地。这里稍微简单介绍下`runtime.Caller`，它可以获取运行时方法的调用信息。

```go
func Caller(skip int) (pc uintptr, file string, line int, ok bool)
```

参数`skip`表示跳过栈帧数，`0`表示不跳过，也就是`runtime.Caller`的调用者。`1`的话就是再向上一层，表示调用者的调用者。

log日志包里使用的是`2`，也就是表示我们在源代码中调用`log.Print`、`log.Fatal`和`log.Panic`这些函数的调用者。

以`main`函数调用`log.Println`为例，是`main->log.Println->*Logger.Output->runtime.Caller`这么一个方法调用栈，所以这时候，skip的值分别代表：

1. `0`表示`*Logger.Output`中调用`runtime.Caller`的源代码文件和行号
2. `1`表示`log.Println`中调用`*Logger.Output`的源代码文件和行号
3. `2`表示`main`中调用`log.Println`的源代码文件和行号

所以这也是`log`包里的这个`skip`的值为什么一直是`2`的原因。

## 3、定制自己的日志

通过上面的源码分析，我们知道日志记录的根本就在于一个日志记录器`Logger`，所以我们定制自己的日志，其实就是创建不同的`Logger`。

```go
var (
	Info *log.Logger
	Warning *log.Logger
	Error * log.Logger
)

func init(){
	errFile,err:=os.OpenFile("errors.log",os.O_CREATE|os.O_WRONLY|os.O_APPEND,0666)
	if err!=nil{
		log.Fatalln("打开日志文件失败：",err)
	}

	Info = log.New(os.Stdout,"Info:",log.Ldate | log.Ltime | log.Lshortfile)
	Warning = log.New(os.Stdout,"Warning:",log.Ldate | log.Ltime | log.Lshortfile)
	Error = log.New(io.MultiWriter(os.Stderr,errFile),"Error:",log.Ldate | log.Ltime | log.Lshortfile)

}

func main() {
	Info.Println("飞雪无情的博客:","http://www.flysnow.org")
	Warning.Printf("飞雪无情的微信公众号：%s\n","flysnow_org")
	Error.Println("欢迎关注留言")
}
```

我们根据日志级别定义了三种不同的Logger，分别为`Info`,`Warning`,`Error`，用于不同级别日志的输出。这三种日志记录器都是使用`log.New`函数进行创建。

这里创建Logger的时候，`Info`和`Warning`都比较正常，`Error`这里采用了多个目的地输出，这里可以同时把错误日志输出到`os.Stderr`以及我们创建的`errors.log`文件中。

`io.MultiWriter`函数可以包装多个`io.Writer`为一个`io.Writer`，这样我们就可以达到同时对多个`io.Writer`输出日志的目的。

`io.MultiWriter`的实现也很简单，定义一个类型实现`io.Writer`，然后在实现的`Write`方法里循环调用要包装的多个`Writer`接口的`Write`方法即可。

```go
func (t *multiWriter) Write(p []byte) (n int, err error) {
	for _, w := range t.writers {
		n, err = w.Write(p)
		if err != nil {
			return
		}
		if n != len(p) {
			err = ErrShortWrite
			return
		}
	}
	return len(p), nil
}
```

这里我们通过定义了多个Logger来区分不同的日志级别，使用比较麻烦，针对这种情况，可以使用第三方的log框架，也可以自定包装定义，直接通过不同级别的方法来记录不同级别的日志，还可以设置记录日志的级别等。

