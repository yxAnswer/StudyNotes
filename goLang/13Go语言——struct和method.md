# Go语言——Struct 和Method

[TOC]

struct  特点
> - 使用 type <Name> struct{} 定义结构，名称遵循可见性规则
> - struct是值类型
> - 可以使用字面值对结构进行初始化
> - 支持匿名结构，和匿名字段
> - 允许直接通过指针来读写结构成员
> - 相同类型的成员可进行直接拷贝赋值，支持 == 与 !=比较运算符，但不支持 > 或 <
> - 嵌入结构作为匿名字段看起来像继承，但不是继承
> - Go中的struct没有构造函数，一般可以使用工厂模式来解决这个问题

# 一、结构体struct

## 1、struct介绍

struct 首先是一种类型，**值类型**。**它是由一系列具有相同类型或不同类型的数据构成的数据集合**。和c语言的struct很像，其实就相当于java的class，但是go是没有class 概念的。定义类型的目的其实就是告诉编译器要开辟多少内存，内存中放什么类型，而结构体组合多个类型。

## 2、struct 定义和初始化

```go
type person struct{//定义一个person struct，
	Name string
	Age int
}
func main(){
	a:=person{}//初始化这个struct，这个时候会用类型零值进行初始化
	fmt.Println(a)
	a.Name="tom" //对结构体的属性进行赋值
	a.Age=19
	fmt.Println(a)
}
//打印结果
{ 0}
{tom 19}
```

从上面的例子看到，我们可以使用type <Name> struct{}  来定义一个结构体，里面可以声明结构体拥有的属性。（**结构体的名字遵循大小写访问权限**）。然后，使用a:=person{} 初始化了一个空的结构体，这是时候打印出的是{空字符串 0}，（**初始化结构体时默认使用零值**）。接下来可以通过. 来操作初始化了的结构体，访问它的属性进行赋值操作。

接下来列出struct初始化的方式

2.1 使用var 关键字声明，并初始化为其零值 

```go
var Tom person   //关键字 var 创建了类型为 person 且名为 Tom 的变量
//注意，这里会使用person各个属性的零值进行初始化
```

2.2 使用字面量来声明创建结构体变量，

第一种形式：

```go
tom:=person{  //使用:= 创建了person结构体类型的变量tom,初始值是 tom,和18
    Name:"Tom",
    Age:18,
}
//注意：每一行必须以逗号结尾
```

第二种形式：

```go
tom:=person{"Tom",20} //必须要和结构声明中字段的顺序一致
//当然也可以这样---都是一样的，注意顺序就可以
tom:=person{
    "Tom",
    20,
}
```

2.3 嵌套结构体

```go
type student struct{
    per person		//person结构体作为属性
    grade string  	//student的属性，
}
```

其实使用这种组合的方式，可以将不同的结构体组成一个集合，其实也是实现了面向对象的继承。

小结，struct是值类型，赋值和传参会复制全部内容。可⽤ "_" 定义补位字段，⽀持指向⾃⾝类型的指针

## 3、结构体比较 

struct ⽀持 "=="、"!=" 相等操作符，可⽤作 map 键类型。 相等比较运算符==将比较两个结构体的每个成员 

```go
type Point struct{ X, Y int }
p := Point{1, 2}
q := Point{2, 1}
fmt.Println(p.X == q.X && p.Y == q.Y) // "false"
fmt.Println(p == q) // "false"
```

既然可以比较，那么就可以作为map键类型

```go
type User struct {
	id int
	name string
}
m := map[User]int{
	User{1, "Tom"}: 100,
}
```

## 4、匿名字段

Go语言有一个特性让我们只声明一个成员对应的数据类型而不指名成员的名字；这类成员就叫匿名成员。匿名成员的数据类型必须是命名的类型或指向一个命名的类型的指针。 

```go
type person struct{ //匿名字段
	 string
	 int
}
func main(){
	tom:=person{"name",19} //顺序必须和声明一致
	fmt.Println(tom)
}
```

## 5、结构体嵌入

```go
type human struct {  //定义 human的结构体
	Sex int
 }
 type teacher struct { //定义teacher结构体
	human          //组合的形式，不存在继承直接放入human
	Name  string
	Age  int
 
 }
 
 type  student struct {
	human        //自动以human命名匿名结构
	Name string
	Age int
 }
 func main() {
     // 所以需要用human这个名称进行操作这个匿名结构
	a := teacher{Name:"joe",Age:18,human:human{Sex:0}} 
	b:= student{Name:"joe",Age:18,human:human{Sex:0}}
	a.Name="aaaa"
	a.Age=11
	a.Sex=100     //可以直接修改
	fmt.Println(a,b)
 }

```

从上面的例子可以看出，teacher 和student 都是人，如果把human 结构体作为他们的内部属性，那就相当于teacher和student拥有了 human结构体的所有属性和方法，变相实现了继承。

# 二、Method

> -  只能为当前包内命名类型定义方法。
> -  参数receiver可任意命名。如方法中未曾使用，可省略参数名。
> - 参数 receiver 类型可以是 T 或 *T。基类型 T 不能是接口或指针。
> -  不支持方法重载， receiver 只是参数签名的组成部分。
> -  可用实例 value 或 pointer 调用全部⽅法，编译器自动转换。 
>
> - 类型别名不会拥有底层类型所附带的方法

## 1、方法声名

```go
type A struct{   //定义结构体 A 
	Name string
}
type B struct{	//定义结构体B
	Name string
}

func (a A) Print(){  //结构体A的方法
	fmt.Println("A的方法")
}
func (b B) Print(){ //结构体B的方法
	fmt.Println("B的方法")
}
func main(){
	a:=A{}
	a.Print()
	b:=B{}
	b.Print()
}
```

从上面的代码看出，方法实际上也是函数，只是在声明时，在关键字func 和方法名之间增加了一个参数， 

```go
func (接收者变量名  接收者) 方法名(){

} 
```

receiver 声名的是谁，这个就是哪个结构体的方法。注意：只能为同一个包中的类型定义方法

**没有构造和析构方法，通常用简单工厂模式返回对象实例。** 

```go
type Person struct {
	Name string
}
func SetName() *Person { // 创建对象实例。
    return &Person{}
}
func main(){
	a:=SetName()
	a.Name="zhangsan"
	fmt.Println(a.Name)
}
```

## 2、值接收者和指针接收者

```go
type Data struct {
	x int
}

func (self Data) ValueTest() { // 值接受者方法
	fmt.Printf("值接收者: %p\n", &self)
}
func (self *Data) PointerTest() { // 指针接受者方法
	fmt.Printf("指针接受者: %p\n", self)
}
func main() {
	d := Data{}
	p := &d
	fmt.Printf("原始地址: %p\n", p)
	d.ValueTest()   // ValueTest(d)
	d.PointerTest() // PointerTest(&d)
	p.ValueTest()   // ValueTest(*p)
	p.PointerTest() // PointerTest(p)
}
//输出结果
原始地址: 0xc00005a058
值接收者: 0xc00005a090
指针接受者: 0xc00005a058
值接收者: 0xc00005a098
指针接受者: 0xc00005a058
```

这个实例可以看到，定义结构体的**方法可以是值接收者，也可以是指针接收者**。区别在于方法名前面的receiver一个是指针，一个是实际的值。

并且调用方法的时候，struct 变量的**值可以调用指针接收者的方法，指针也可以调用值接受者的方法**。分开来说这几种情况：

### 2.1 使用值调用值接收者方法

```go
func (self Data) ValueTest() { // 值接受者方法
	fmt.Printf("值接收者: %p\n", &self)
}
```

使用值来调用此方法，receiver是 Data类型。当发起调用的时候，此方法操作的是值调用者的一个值的拷贝。如最上面的代码中，连续值调用最终打印出的地址是不同的，每次都不相同，因为每一次的调用，都是使用的值的拷贝，并且对这个值做任何操作，都不会影响初始值。

###  2.2 使用指针调用指针接收者方法

```go
func (self *Data) PointerTest() { // 指针接受者方法
	fmt.Printf("指针接受者: %p\n", self)
}
```

使用指针来调用此方法，receiver是一个指向Data类型的指针。当调用使用指针接收者声明的方法时，这个方法会共享调用方法时接收者所指向的值。其实也就是，此方法直接可以操作接收者指针指向的底层数据的值。这样方法对值进行任何改变都会对原始值产生影响。 类似引用传递。（指针的拷贝）

### 2.3 使用指针调用值接收者方法

使用指针来调用值接收者方法就有意思了。值接收者前面说了是对值的拷贝进行操作，那么指针怎么样？其实这是go语言编译器做了处理，比如`p.ValueTest()` 这段代码在编译器会修改为`(*p).ValueTest() `,也就是编译器会自动先取出指针指向的值，然后调用值接收者方法。注意：这里的调用也是一个值的拷贝，只不过拷贝的是指针指向的值。再次声明：此方法操作的是指针指向的值的副本。 也就是，此方法怎么操作也不会影响原始值。

### 2.4 使用值调用指针接收者方法

使用值来调用指针接收者方法。这个一看很矛盾，指针怎么接受值。于是乎go的编译器再一次做了调整，如`d.PointerTest()`,经过编译器处理变为`(&d).PointerTest() ` 。这就很明显了，其实就是语法糖，为了方便。编译器会先取出值的地址，然后使用此地址指向的值进行处理。操作的也是原始值。

**总结下：值接收者使用值的副本来调用方法，而指针接受者使用实际值来调用方法**。 

### 2.5、废除指针的多级调用

从1.4开始不再支持指针的多级调用方法（了解）

```go
type X struct{}
func (*X) test() {
	println("X.test")
}
func main() {
	p := &X{}
	p.test()
	// Error: calling method with receiver &p (type **X) requires explicit dereference
	// (&p).test()
}
```

不能将一个指针的指针去调用指针接受者的方法，只能用这个接受者的指针去调用。

## 3、匿名字段

可以像字段成员那样访问匿名字段⽅法，编译器负责查找 

```go
type User struct { //User 结构体
	id   int
	name string
}
type Manager struct {
	User		//匿名字段
}

func (self *User) ToString() string { //User的方法
	return fmt.Sprintf("User: %p, %v", self, self)
}
func main() {
	m := Manager{User{1, "张大仙"}} 
	fmt.Printf("ManagerL: %p\n", &m)
	fmt.Println(m.ToString())
}
//输出·
ManagerL: 0xc000054400
User: 0xc000054400, &{1 张大仙}
```

从上面代码看到，Manager 内部声明了User类型的匿名字段，那么Manager就具备了User的所有方法。

如果匿名字段的方法和外部结构的方法重名怎么办？ 其实还是就近原则，如果外部结构和嵌入结构存在同名方法，则优先调用外部结构的方法

由此可见，我们可以理解为：使用匿名字段，就相当于扩展了此struct的功能，也就是变相实现了继承，就可以直接调用匿名字段的方法。如果我们要实现方法重写，那么就可以在外层定义一个重名方法，修改方法内容，那不就是实现了方法重写吗！！！

```go
type User struct { //User 结构体
	id   int
	name string
}
type Manager struct {
	User  			//匿名字段
	title string	//Manager特有的字段
}

func (self *User) ToString() string { //user的方法
	return fmt.Sprintf("User: %p, %v", self, self)
}
func (self *Manager) ToString() string {//manager 的方法
	return fmt.Sprintf("Manager: %p, %v", self, self)
}
func main() {
	m := Manager{User{1, "张大仙"}, "标题"}
	fmt.Println(m.ToString()) 
	fmt.Println(m.User.ToString())
}
//输出结果
Manager: 0xc00006e240, &{{1 张大仙} 标题}
User: 0xc00006e240, &{1 张大仙}
```



## 4、Method Value和Method Expression

方法值和方法表达式，从某种意义上来说，方法是函数的语法糖，因为receiver其实就是方法所接收的第1个参数（Method Value vs. Method Expression）

根据调⽤者不同，⽅法分为两种表现形式： 

```go
instance.method(args...) ---> <type>.func(instance, args...) 
```

两者都可像普通函数那样赋值和传参，区别在于 method value 绑定实例，⽽ method expression 则须显式传参。 

```go
type TZ int   //自定义一个int类型的类型TZ
func (a *TZ)Print(){
	fmt.Println("TZ")
}
func main(){
	var a TZ  //声明变量a  TZ 类型
	a.Print()	//Method Value 方式
	(*TZ).Print(&a)// Method Expression 方式
}
```

这里借用下go学习笔记的总结，慢慢体会

## 5、方法集：

go语言，每种类型都有与之关联的方法集，方法集定义了一组关联到给定类型的值或者指针的方法，定义方法时使用的接受者的类型决定了这个方法是关联到值，还是关联到指针，还是两个都关联。这对于接口的实现规则非常重要。对于方法集规则，在讲接口的时候会详细写明，此处了解即可。这里只需记住：**直接用实例或者实例的指针调用方法，不受方法集约束，接口的实现规则才会受方法集约束。**

```go
/*
每个类型都有与之关联的方法集，这会影响到接口实现规则
- 类型T方法集包含全部receiver T方法
- 类型*T 方法集包含全部receiver T+ *T方法
- 如类型S包含匿名字段T，则S方法集包含T方法
- 如类型S包含匿名字段*T，则S方法集包含T+*T方法
- 不管嵌入T或者* T，*S方法集总是包含T+*T方法
*/
```

# 三、面向对象

go 没有class关键字，struct替代了class的作用，go对于面向对象的支持是不一样的。拿struct来说，struct没有继承的概念，go语言是通过 组合的概念来进行面向对象编程。面向对象的目的其实就是代码复用，go通过组合不同的结构体，使这个struct具有更多的功能，复用其他结构体的属性和方法，这个就是struct的继承。

所以，对go语言来说，**封装采用首字母大小写的可见性支持，继承采用不同struct的组合来实现**。多态会在讲接口的时候说。

## 1、 封装

go是直接支持strut的封装的，go语言的可见性是根据首字母大小写来实现的。 首字母大写表示对外部可见，等同Java中的public，首字母小写，对外部不可见，等同于Java中的private。

```go
type Student struct{ //声名一个对包外部可见的结构体，
    name string
	age int
}
func (s *Student)setName(name string){
    s.name=name
}
func (s *Student)setAge(age int){
    s.age=age
}
func (s *Student)getName() string{
    return s.name
}
func (s *Student)getAge()int{
    return s.age
}
func main(){
	s:=Student{}
	fmt.Println(s)
	s.setName("张三")
	s.setAge(18)
	fmt.Println(s)
}
//输出
{ 0}
{张三 18}
```

这里的可见不可见是相对于包外部说得，对于同一个包内都是可见的。

## 2、继承

使用匿名字段或者叫结构体嵌入，可以变向的实现继承，并且可以实现多继承

go语言的struct使用组合的方式实现继承，比传统的面向对象更加灵活，可以有效的避免类似Java的超多层级的继承。而go另辟蹊径，根据需要自己组合，也就是组合大于继承。代码见上面方法的匿名字段。

























