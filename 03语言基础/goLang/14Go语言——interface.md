# Go语言——接口interface详解

[TOC]

# 1、Duck Typing 概念

go语言中的duck  typing并不是真正的duck typing,但是他是类似的概念，go语言接口的实现就可以看做为duck typing。举例：什么是鸭子？ 

其他面向对象的语言可能认为，活生生的鸭子才是鸭子，要定义它的属性和方法。但是go语言中，鸭子的定义并不是真实的对象，它是由使用者来决定到底什么是鸭子。比如，真的鸭子和玩具的鸭子。像鸭子走路，像鸭子叫（长得像鸭子）,那么就是鸭子。所以，go语言中的接口就是这样，它不必显示的去声明它，它只关注是否实现了相应的方法。**它描述的事物的外部行为，而非内部结构**

面向对象的继承、抽象接口等等目的都是代码的复用。既然是复用，那就要从使用者的角度去想，我认为是什么样子它就是什么样子。我只关心这段代码结构能做哪些事情，我复用它，我才不管它符不符合常识。

go语言就是一个结构化类型系统，类似 duck typing。只要实现了接口的所有方法，就表示该类型实现了该接口。



# 2、GO 语言interface特点

- 接口是一个或多个方法签名的集合


- 只要某个类型拥有该接口的所有方法签名，即算实现该接口，无需显示声明实现了哪个接口，这称为 Structural Typing

- 接口只有方法声明，没有实现，没有数据字段
- 接口可以匿名嵌入其它接口，或嵌入到结构中
- 将对象赋值给接口时，会发生拷贝，而接口内部存储的是指向这个复制品的指针，既无法修改复制品的状态，也无法获取指针

- 只有当接口存储的类型和对象都为nil时，接口才等于nil
- 接口调用不会做receiver的自动转换
- 接口同样支持匿名字段方法
- 接口也可实现类似OOP中的多态
- 空接口可以作为任何类型数据的容器

# 3、接口定义

## 3.1 接口类型

go语言中的接口也是一种类型，他具体描述了一系列方法的集合，一个实现了这些方法的具体类型是这个接口类型的实例。 定义接口很简单，使用type关键字，其实就是定义一个结构体，但是内部只有方法的声明，没有实现。

```go
type Stringer interface {//接口的定义就是如此的简单。
	String() string
}
```

## 3.2 接口的实现方式

go语言接口的独特之处就是，不需要显示的去实现接口。**一个类型如果拥有一个接口需要的所有方法，那么这个类型就实现了这个接口。** 这种隐式实现接口的方式，同时也提高了灵活性。

```go
type Stringer interface {
	String() string
}
type Printer interface {
	Stringer // 接口嵌⼊。
	Print()
}
type User struct {
	id int
	name string
}
func (self *User) String() string {
	return fmt.Sprintf("user %d, %s", self.id, self.name)
}
func (self *User) Print() {
	fmt.Println(self.String())
}
func main() {
	var t Printer = &User{1, "Tom"} // *User ⽅法集包含 String、 Print。
	t.Print()
}
```

上面的代码可以看到，一个类型只要实现了接口定义的所有方法（是指有相同名称、参数列表 (不包括参数名) 以及返回值 ），那么这个类型就实现了这个接口，可以说这个类型现在是这个接口类型，可以直接进行赋值（其实也是隐式转换），比如`var t Printer = &User{1, "Tom"}`。 那么既然如此，一个类型就可以实现多个接口，只要它拥有了这些接口类型的所有方法，那么这个类型就是实现了多个接口。同时这个类型也就是多种形式的存在，反过来说一个接口可以被不同类型实现，这就是go语言中的多态了。

## 3.3 interface{}空接口的实现

空接⼝ interface{} 没有任何⽅法签名，也就意味着任何类型都实现了空接⼝。其作⽤类似⾯向对象语⾔中的根对象 object。 

## 3.4 类型断言

一个类型断言检查接口类型实例是否为某一类型 。语法为x.(T) ,x为类型实例，T为目标接口的类型。比如

`value, ok := x.(T)`  x ：代表要判断的变量,T ：代表被判断的类型,value：代表返回的值,ok：代表是否为该类型。即：ok partern方式。**注意：x 必须为inteface类型，不然会报错。**

不过我们一般用switch进行判断，叫做 type switch。注意：**不支持fallthrough**.

```go
func main() {
	var o interface{} = &User{1, "Tom"}
	switch v := o.(type) {
		case nil: 			// o == nil
			fmt.Println("nil")
		case fmt.Stringer: 	// interface
			fmt.Println(v)
		case func() string: // func
			fmt.Println(v())
		case *User: 		// *struct
			fmt.Printf("%d, %s\n", v.id, v.name)
		default:
			fmt.Println("unknown")
	}
}
```

## 3.5 接口转换

**可以将拥有超集的接口转换为子集的接口，反之出错**。

```go
type User struct {
	id int
	name string
}
func (self *User) String() string {
	return fmt.Sprintf("%d, %s", self.id, self.name)
}
func main() {
	var o interface{} = &User{1, "Tom"}
	if i, ok := o.(fmt.Stringer); ok { // ok-idiom
		fmt.Println(i)
	}
	u := o.(*User)
	// u := o.(User) // panic: interface is *main.User, not main.User
	fmt.Println(u)
}
```

通过类型判断，如果不同类型转换会发生panic.

## 3.6 匿名接口

匿名接口可用作变量类型，或者是结构成员。

```go
type Tester struct {
	s interface {
		String() string
	}
}
type User struct {
	id int
	name string
}
func (self *User) String() string {
	return fmt.Sprintf("user %d, %s", self.id, self.name)
}
func main() {
	t := Tester{&User{1, "Tom"}}
	fmt.Println(t.s.String())
}
//输出：
user 1, Tom
```

# 4、接口的内部实现

## 4.1 接口值

接口值可以使用 == 和 !＝来进行比较。两个接口值相等仅当它们都是nil值或者它们的动态类型相同，并且动态值也根据这个动态类型的==操作相等。因为接口值是可比较的，所以它们可以用在map的键或者作为switch语句的操作数。
然而，如果两个接口值的动态类型相同，但是这个动态类型是不可比较的（比如切片） ，将它们进行比较就会失败并且panic。

那么接口值内部到底是什么结构呢？

## 4.2 接口内部结构

```go
// 没有方法的interface
type eface struct {
    _type *_type   //类型信息
    data  unsafe.Pointer  //数据指针
}

// 记录着Go语言中某个数据类型的基本特征，_type是go所有类型的公共描述
//可以简单的认为，接口可以通过一个  _type *_type 直接或间接表述go所有的类型就可以了
type _type struct {
    size       uintptr	//类型的大小
    ptrdata    uintptr	//存储所有指针的内存前缀的大小
    hash       uint32	//类型的hash
    tflag      tflag	//类型的tags
    align      uint8	//结构体内对齐
    fieldalign uint8	//结构体作为field时的对齐
    kind       uint8	//类型编号 定义于 runtime/typekind.go
    alg        *typeAlg	// 类型元方法 存储hash 和equal两个操作。
    gcdata    *byte		//GC 相关信息
    str       nameOff	//类型名字的偏移
    ptrToThis typeOff
}

// 有方法的interface
type iface struct {
    tab  *itab
    data unsafe.Pointer
}

type itab struct {
    inter  *interfacetype	//接口定义的类型信息
    _type  *_type			//接口实际指向值的类型信息
    link   *itab
    hash   uint32
    bad    bool
    inhash bool
    unused [2]byte
    fun    [1]uintptr		//接口方法实现列表，即函数地址列表，按字典序排序
}

// interface数据类型对应的type
type interfacetype struct {
    typ     _type
    pkgpath name
    mhdr    []imethod
}
```

存在两种interface，一种是带有方法的interface，一种是不带方法的interface。

对于不带方法的接口类型，Go语言中的所有变量都可以赋值给interface{}变量，interface可以表述go所有的类型，_type存储类型信息，data存储类型的值的指针，指向实际值或者实际值的拷贝。

对于带方法的接口类型，`tab  *itab`  存储指向了iTable的指针，ITable存储了类型相关的信息以及相关方法集，而data 同样存储了实例值的指针，指向实际值或者是实际值的一个拷贝。

 实现了interface中定义方法的变量可以赋值给带方法的interface变量，并且可以通过interface直接调用对应的方法，实现了其它面向对象语言的多态的概念。

go语言interface的源码表示，接口其实是一个两个字段长度的数据结构。所以任何一个interface变量都是占用16个byte的内存空间。从大的方面来说，如图：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190310221415491.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)



`var n notifier     n=user("Bill")` 将一个实现了notifier接口实例user赋给变量n。那我们先来看有方法的接口的内部是怎么样的。接口n 内部两个字段    tab  *itab 和 data unsafe.Pointer， 第一个字段存储的是指向ITable(接口表)的指针，这个内部表包括已经存储值的类型和与这个值相关联的一组方法。第二个字段存储的是，指向所存储值的指针。**注意：这里是将一个值赋值给接口，并非指针，那么就会先将值拷贝一份，开辟内存空间存储，然后将此内存地址赋给接口的data字段。也就是说，值传递时，接口存储的值的指针其实是指向一个副本。**

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019031022411252.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

如果是将指针赋值给接口类型，那么第二个字段data存储的就是指针的拷贝，指向的是原来的内存。

再进一步了解，内部是如何存储的。

没有方法的interface 内部变量第一个字段为`*_type` 类型，这个`_type`记录这某种数据类型的基本信息，比如占用内存大小（size）,数据类型名称等等。然后，第二个字段存储的还是指向值的指针。

每种数据类型都存在一个与之对应的_type结构体（Go语言原生的各种数据类型，用户自定义的结构体，用户自定义的interface等等）。

在这里引用的https://www.jianshu.com/p/70003e0f49d1一张图说明两种方式：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190310225703502.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

小结：总的来说接口是一个类型，它是一个struct，是一个或多个方法的集合。任何类型都可以实现接口，并且是隐式实现，可以同时实现多个接口。接口内部只有方法声明没有实现。接口内部存储的其实就是接口值的类型和值，一部分存储类型等各种信息，另一部分存储指向值的指针。如果是将值传给接口，那么这里第二个字段存储的就是原值的副本的指针。接口可以调用实现了接口的方法。

# 5、方法集

## 5.1 方法集定义

方法集：方法集定义了一组关联到给定类型的值或者指针的方法。定义方法时使用的接受者的类型决定了这个方法是关联到值，还是关联到指针，还是两个都关联。

```go
// 这个示例程序展示 Go 语言里如何使用接口
 package main
 import (
 	"fmt"
 )

 // notifier 是一个定义了
 // 通知类行为的接口
 type notifier interface {
	 notify()
 }

 // user 在程序里定义一个用户类型
 type user struct {
 	name string
 	email string
 }

 // notify 是使用指针接收者实现的方法
 func (u *user) notify() {
	 fmt.Printf("Sending user email to %s<%s>\n",
	 u.name,
	 u.email)
 }

 // main 是应用程序的入口
 func main() {
 // 创建一个 user 类型的值，并发送通知30 
    u := user{"Bill", "bill@email.com"}
 	sendNotification(u)
 // ./listing36.go:32: 不能将 u（类型是 user）作为
 // sendNotification 的参数类型 notifier：
 // user 类型并没有实现 notifier
 // （notify 方法使用指针接收者声明）
 }

 // sendNotification 接受一个实现了 notifier 接口的值
 // 并发送通知
 func sendNotification(n notifier) {
	 n.notify()
 }
```

如上面代码，当为struct实现接口的方法notify()方法时，定义的接受者receiver是一个指针类型，所以，它要遵循方法集的规则，**如果方法集的receiver 是\*T  即指针类型，那么属于接口的值必须同样是\*T  指针类型。**

user 实现了notify 方法，也就是它实现了notifier 接口，当时如果将user 实例传给notifier实例，必须是一个指针类型，因为它实现的方法的receiver是一个指针类型。所以方法集的作用也就是规范接口的实现。

## 5.2 方法集规则

```go
Values					Methods Receivers
-----------------------------------------------
T 						(t T)
*T 						(t T) and (t *T)



Methods Receivers  		Values
-----------------------------------------------
(t T)				 	T and *T
(t *T)				 	*T
```

如果方法的接受者是 指针类型 ，那么用指针接受者方式实现这个接口，只有指向那个类型的指针才能够算实现对应的接口，所以接口值接收的只能也是一个指针类型。

如果方法的接受者是 值类型，那么用值接收者实现接口，那个类型的值和指针都能够实现对应的接口。

简单讲就是，**接受者是（t T）,那么T 和 \*T 都可以实现接口，如果接受者是（t \*T）那么只有 \*T才算实现接口。**

反过来看稍微复杂点，判断这个类型变量是否实现了接口，看一下他是值类型还是指针类型，如果是T 值类型，那就看它实现接口方法的receiver是什么类型，如果也是值类型，那么它就实现了接口，如果不是，就没有实现，就不能进行传递。如果他是指针类型，那么不管它的receiver是值还是指针都实现了接口。所以记住上面的图就好。

**原因：编译器并不是总能自动获得一个值的地址 。**

# 6、嵌入类型时接口实现

重温一下什么是嵌入类型，go语言为了实现类似继承的代码复用，通过组合的方式来提高面向对象的能力。通过嵌入类型来实现代码复用和扩展类型字段和方法。

**嵌入类型**：是将已有的类型直接声明在新的结构类型里。被嵌入的类型被称为新的外部类型的**内部类型**。

**实现方法重写**：外部类型也可以通过声明与内部类型标识符同名的标识符来覆盖内部标识符的字段或者方法。

- 注意声明字段和嵌入类型在语法上的不同 ，嵌入类型直接是写个类型名就行
- 内部类型的标识符提升到了外部类型，可以直接通过外部类型的值来访问内部类型的标识符。 也可以通过内部类型的名间接访问内部类型方法和标识符。
- 内部类型实现接口外部类型默认也实现了该接口。注意方法集的规则。
- 如果内部类型和外部类型同时实现一个接口，就近原则，外部类型不会直接调用内部类型实现的同名方法，而是自己的。当然可以通过内部类型间接显示的去调用内部类型的方法。

## 6.1嵌入类型的实现

```go
// 这个示例程序展示如何将一个类型嵌入另一个类型，以及
// 内部类型和外部类型之间的关系
package main

import (
	"fmt"
)

// user 在程序里定义一个用户类型
type user struct {
	name  string
	email string
}

// notify 实现了一个可以通过 user 类型值的指针
// 调用的方法
func (u *user) notify() {
	fmt.Printf("Sending user email to %s<%s>\n",
		u.name,
		u.email)
}

// admin 代表一个拥有权限的管理员用户
type admin struct {
	user  // 嵌入类型
	level string
}

// main 是应用程序的入口
func main() {
	// 创建一个 admin 用户
	ad := admin{
		user: user{
			name:  "john smith",
			email: "john@yahoo.com",
		},
		level: "super",
	}
	// 我们可以直接访问内部类型的方法
	ad.user.notify()
	// 内部类型的方法也被提升到外部类型
	ad.notify()
}

```



**总之：嵌入类型，就是外部类型拥有内部类型所有的字段和方法，就好比直接定义在外部类型一样。就像继承。**

在来看下，内部类型实现接口是什么情况？

## 6.2 嵌入类型实现接口，同样应用到外部类型

```go
// 这个示例程序展示如何将一个类型嵌入另一个类型，以及
// 内部类型和外部类型之间的关系
package main
import (
	"fmt"
)
// notifier 是一个定义了
// 通知类行为的接口
type notifier interface {
	notify()
}

// user 在程序里定义一个用户类型
type user struct {
	name  string
	email string
}
// 通过 user 类型值的指针
// 调用的方法
func (u *user) notify() {
	fmt.Printf("Sending user email to %s<%s>\n",
		u.name,
		u.email)
}

// admin 代表一个拥有权限的管理员用户
type admin struct {
	user  // 嵌入类型
	level string
}
// main 是应用程序的入口
func main() {
	// 创建一个 admin 用户
	ad := admin{
		user: user{
			name:  "john smith",
			email: "john@yahoo.com",
		},
		level: "super",
	}
	// 给 admin 用户发送一个通知
	// 用于实现接口的内部类型的方法，被提升到
	// 外部类型
	sendNotification(&ad)
}
// sendNotification 接受一个实现了 notifier 接口的值
// 并发送通知
func sendNotification(n notifier) {
	n.notify()
}

```

**由于内部类型的提升，内部类型实现的接口会自动提升到外部类型，即外部类型同样也实现了该接口。不过要注意的是方法集的规则。总结：内部类型实现接口外部类型默认也实现了该接口。**

## 6.3 内部类型和外部类型同时实现接口

```go
// 这个示例程序展示如何将一个类型嵌入另一个类型，以及
// 内部类型和外部类型之间的关系
package main

import (
	"fmt"
)

// notifier 是一个定义了
// 通知类行为的接口
type notifier interface {
	notify()
}

// user 在程序里定义一个用户类型
type user struct {
	name  string
	email string
}

// 通过 user 类型值的指针
// 调用的方法
func (u *user) notify() {
	fmt.Printf("Sending user email to %s<%s>\n",
		u.name,
		u.email)
}

// admin 代表一个拥有权限的管理员用户
type admin struct {
	user  // 嵌入类型
	level string
}

// 通过 admin 类型值的指针
// 调用的方法
func (a *admin) notify() {
	fmt.Printf("Sending admin email to %s<%s>\n",
		a.name,
		a.email)
}

// main 是应用程序的入口
func main() {
	// 创建一个 admin 用户
	ad := admin{
		user: user{
			name:  "john smith",
			email: "john@yahoo.com",
		},
		level: "super",
	}

	// 给 admin 用户发送一个通知，就近原则
	sendNotification(&ad)
	// 我们可以直接访问内部类型的方法
	ad.user.notify()

	// 内部类型的方法没有被提升
	ad.notify()
}

// sendNotification 接受一个实现了 notifier 接口的值
// 并发送通知
func sendNotification(n notifier) {
	n.notify()
}

```

输出

```go
Sending admin email to john smith<john@yahoo.com>
Sending user email to john smith<john@yahoo.com>
Sending admin email to john smith<john@yahoo.com>
```

外部类型和内部类型同时实现接口，就近原则，外部类型优先调用自己实现的方法。如果要调用内部类型的方法，需要用内部类型字段间接调用。类似于方法重写的方式，实现和内部类型同名的方法，也是就近原则。

