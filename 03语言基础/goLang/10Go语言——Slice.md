# Go语言——slice

[TOC]

# 一、切片介绍

切片是一种数据结构，这种数据结构便于使用和管理数据集合。切片是围绕动态数组的概念构建的，可以按需自动增长和缩小。切片的动态增长是通过内置函数 append 来实现的。这个函数可以快速且高效地增长切片。还可以通过对切片再次切片来缩小一个切片的大小。因为切片的底层内存也是在连续块中分配的，所以切片还能获得索引、迭代以及为垃圾回收优化的好处。 

切片是引用类型，指向底层数组，切片语法和数组很像，只是没有长度而已。slice底层是连续内存，动态增加长度其实如果超过底层数组容量，也会重新分配内存。

> - 指向底层的数组，作为变长数组的替代方案，可以关联底层数组的局部或全部
> - slice 为引用类型,但自身是结构体，值拷贝传递。 
> - 如果多个slice指向相同底层数组，其中一个的值改变会影响全部
> - 可以直接创建或从底层数组获取生成，一般使用make()创建
> - make([]T, len, cap)  ,其中cap可以省略，则和len的值相同。
> - 属性 len 表示可用元素数量，读写操作不能超过该限制。
> - 属性 cap 表示最⼤扩张容量，不能超出数组限制。
> - 如果 slice == nil，那么 len、 cap 结果都等于 0。

# 二、内部实现和原理

```go
type slice struct {
	array unsafe.Pointer
	len   int
	cap   int
}
```

切片是有 3 个字段的数据结构 ，这 3 个字段分别是指向底层数组的指针、切片访问的元素的个数（即长度）和切片允许增长到的元素个数（即容量）。 

![切片示例](https://img-blog.csdnimg.cn/20190226112317444.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

可以看出，slice是引用类型。时刻注意一点，slice传递的时候也是值拷贝，把slice的指针，长度，容量拷贝过去，那么同样可以去操作底层数组

# 三、创建和初始化

## 1、普通初始化（var或者：=  ,字面量）

```go
var slice []int //只声明一个slice,len  和 cap 都是0
var slice0 []int=[]int{1,2,3}//完整声明，字面量初始化
var slcie1=[]int{1,2,3}  //直接var 字面量初始化
slice2:=[]int{1,2,3}//函数内部可以用:=替代var 初始化
```

## 2、使用make() 函数进行初始化

一般使用make()进行创建 make([]T, len, cap)  指定类型，长度，容量，make()会在按照容量分配内存空间，也就是分配的数组

```go
slice:=make([]string,5)//使用长度声明一个字符串切片，长度、容量都是5
slice0:=make([]int,3,5)//使用长度和容量声明int切片，长度为3，容量为5
```

**容量小于长度的切片会在编译时报错** 

```go
slice := make([]int, 5, 3)//注意，不允许,
```

注意：使用make([]int,3,5)创建slice,指定的容量cap 其实就是初始化的底层数组的长度。但是此slice 长度为3，只能操作底层数组的前3个，后两个是不能操作的，但是可以通过append函数添加到此slice中。如果基于这个切片创建新的切片，新切片会和原有切片共享底层数组，也能通过后期操作(如append)来访问多余容量的元素。

## 3、使用索引声明切片 

```go
// 创建字符串切片
// 使用空字符串初始化第 100 个元素
slice := []string{99: ""}
```

记住，如果在[]运算符里指定了一个值，那么创建的就是数组而不是切片。只有不指定值的时候，才会创建切片 。这里字面量声明索引为99的位置为空字符串，所以，长度和容量都是 100 个元素，最少开辟了100个内存空间。

## 4、从数组创建slice

（1）通过两个冒号创建切片，`slice[x:y:z]`切片实体`[x:y]`切片长度`len = y-x`，切片容量`cap = z-x`

```go
data := [...]int{0, 1, 2, 3, 4, 5, 6} //初始化一个数组
slice := data[1:4:5] // [low : high : max]  通过两个冒号创建切片
```

使用两个冒号[1:4:5] 从数组中创建切片，长度为4-1=3，也就是索引从1到3 的数据（1,2,3），然后，后面是最大是5，即容量是5-1=4，即，创建的切片是长度为从索引为 1、2、3 的切片，底层数组为[ 1,2,3,4] 

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190226115425496.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

（2）通过单个冒号，索引创建slice

```go
data := [...]int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}

expression slice len cap comment
------------+----------------------+------+-------+---------------------
data[:6:8] [0 1 2 3 4 5] 		//6 8 省略 low.
data[5:] [5 6 7 8 9] 			//5 5 省略 high、 max。
data[:3] [0 1 2] 				//3 10 省略 low、 max。
data[:] [0 1 2 3 4 5 6 7 8 9]	//10 10 全部省略。
```

## 5、创建空的slice

首先说一下，slice和array的区别，其实就是[]中有没有值，有值就是数组，无值就是slice

```go
// 使用 make 创建空的整型切片
slice := make([]int, 0)
// 使用切片字面量创建空的整型切片
slice := []int{}
```

空切片在底层数组包含 0 个元素，也没有分配任何存储空间。想表示空集合时空切片很有用.
例如，数据库查询返回 0 个查询结果时 

# 四、切片的使用

## 1、直接用索引赋值

```go
// 创建一个整型切片
// 其容量和长度都是 5 个元素
slice := []int{10, 20, 30, 40, 50}
// 改变索引为 1 的元素的值
slice[1] = 25
```

## 2、reslice 也就是通过slice创建 slice

```go
s := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
s1 := s[2:5] // [2 3 4]
s2 := s1[2:6:7] // [4 5 6 7]
s3 := s2[3:6] // Error。 不能超过父slice的容量。
//索引s[2:5]  表示从索引2开始，到索引4结束，包括2但是不包括5 
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190226145305668.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70) 

- s1:=s[2:5] 得到的是len为  5-2=3， cap为10-2=8。 这个公式 可以计算。 所以，s1指向底层数组的从第二个索引开始，三个长度的内存，2,3,4.  但是他的容量cap为8，也就是，s1指向内存起始位置是原数组的2，一直到9，都是一块连续的内存。如果要增加s1的长度，不需要开辟新的内存空间，只需要往s1里面添加就行，他继续指向这块内存地址，直到他的长度超过容量，就会重新开辟内存空间。

- s2:=s1[2:6:7]  ,s2是通过s1创建的新的slice，它同样指向了和s1，s1一样的底层数组，只不过是起始的索引位置不同。s2表示从s1索引为2开始到索引为5的长度为4个的slice，并且还指定了他的容量为7，也就是它不使用它起始索引到底层数组末尾的长度作为容量，而是使用自己指定的容量7. 如图，s1长度为3，而s2的长度为4，并且是从s1索引为2开始的，但是却没有报错。 是因为，reslice,是根据 子slice相对于父slice的容量创建的，只要子slice的长度没有超过父slice的容量，那么就是允许的，因为他们指向的是同一个底层数组。
- 所以，s3:=s2[3:6]  他报错了。因为s3是创建 从s2作为为3，长度为3的slice，但是s2的长度为4，容量为5，从索引3开始算，s2只能最大创建出长度为2的 子slice，不能创建出长度为3的slice
- 总结下，就是，父slice 和子slice 都是指向了同一块连续内存，底层是个数组。 只是根据长度和容量的不同，创建的slice 允许操作的内存块是不同的。如果子slice创建时不指定自己的容量cap,那么它的容量默认为，从他指向的那个底层数组的索引开始，一直到这个数组的最末端。这块是它的容量，也就是它创建以后，可以继续扩展而不改变内存地址。注意，它可以操作的连续内存是由它的长度决定的。比如，长度为3，容量为5,的slice，它只能操作从它指向底层数组的那个首索引开始，往后三个数，另外两个是不允许操作的。

由于reslice 的slice指向了同一块连续内存空间，所以操作是会相互影响的

```go
// 创建一个整型切片
// 其长度和容量都是 5 个元素
slice := []int{10, 20, 30, 40, 50}
// 创建一个新切片
// 其长度是 2 个元素，容量是 4 个元素 {20,30}
newSlice := slice[1:3]    
// 修改 newSlice 索引为 1 的元素
// 同时也修改了原来的 slice 的索引为 2 的元素
newSlice[1] = 35      
//最终底层为   {10,20,35,40,50}
```

## 3、append  增长slice

相对于数组而言，使用切片的一个好处是，可以按需增加切片的容量。 函数 append 总是会增加新切片的长度，而容量有可能会改变，也可能不会改变，这取决于被操作的切片的可用容量。 (也就是，如果容量够直接加，不够重新分配内存，拷贝原数组加)

### （1）向 slice 尾部添加数据，返回新的 slice 对象。

```go
s := make([]int, 0, 5)
fmt.Printf("%p\n", &s)
s2 := append(s, 1)
fmt.Printf("%p\n", &s2)
fmt.Println(s, s2)
//输出结果
0xc000004440
0xc000004480
[] [1]        
```

**append函数做的事就是更改slice指向底层数组的值**

```go
data := [...]int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
s := data[:3]
s2 := append(s, 100, 200) // 添加多个值。  
fmt.Println(data)
fmt.Println(s)
fmt.Println(s2)
//输出----------append函数更改了slice执行底层数组data，增加了s的长度，并将相应位置的值改变
[0 1 2 100 200 5 6 7 8 9]
[0 1 2]
[0 1 2 100 200]
```
### （2） 未超过原slice容量，底层数组不会重新分配
```go
package main

import "fmt"

func main(){
	data := [...]int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
	s := data[:2:3] //长度为2，容量为3
	fmt.Printf("len: %d,cap: %d\n",len(s),cap(s))
	fmt.Printf("原地址对比\n%p\n%p\n",&data[0],&s[0])
	s2 := append(s, 100) // 添加1一个值，未超过容量
	fmt.Printf("len: %d,cap: %d\n",len(s2),cap(s2))
	fmt.Printf("未超过容量时地址对比\n%p\n%p\n",&data[0],&s2[0])
	s3 := append(s2, 200) // 再次添加一个值，超过容量3
	fmt.Printf("len: %d,cap: %d\n",len(s3),cap(s3))
	fmt.Printf("超过容量时地址对比\n%p\n%p\n",&data[0],&s3[0])

}
//输出
len: 2,cap: 3
原地址对比
0xc00008e000
0xc00008e000
len: 3,cap: 3
未超过容量时地址对比
0xc00008e000
0xc00008e000
len: 4,cap: 6
超过容量时地址对比
0xc00008e000
0xc00007e030
```
### （3）但是，一旦超出原 slice.cap 限制，就会重新分配底层数组，即便原数组并未填满。 

```go
data := [...]int{0, 1, 2, 3, 4, 10: 0}
s := data[:2:3]
s = append(s, 100, 200) // ⼀次 append 两个值，超出 s.cap 限制。
fmt.Println(s, data) // 重新分配底层数组，与原数组无关。
fmt.Println(&s[0], &data[0]) // 比对底层数组起始指针。
//输出
[0 1 100 200] [0 1 2 3 4 0 0 0 0 0 0]
0xc00007e030 0xc00004a060
```

上面代码表示，切片s 的长度为2，容量为3，而s通过append一次性添加两个值，也就是想把s变为长度为4的切片，这超过了s的原始容量，所以，这样append会重新开辟一段内存地址，长度为4，容量为4。可以看大他们的内存地址是不一样的。

### （4）append超过容量时，由增长因子决定开辟多大新内存

```go
s := make([]int, 0, 1)
c := cap(s)
for i := 0; i < 1000000; i++ {
	s = append(s, i)
	if n := cap(s); n > c {
		fmt.Printf("cap: %d -> %d  %.2f\n", c, n,float32(n)/float32(c))
		c = n
	}
}
//输出
cap: 1 -> 2  2.00
cap: 2 -> 4  2.00
cap: 4 -> 8  2.00
cap: 8 -> 16  2.00
cap: 16 -> 32  2.00
cap: 32 -> 64  2.00
cap: 64 -> 128  2.00
cap: 128 -> 256  2.00
cap: 256 -> 512  2.00
cap: 512 -> 1024  2.00
cap: 1024 -> 1280  1.25
cap: 1280 -> 1696  1.33
cap: 1696 -> 2304  1.36
cap: 2304 -> 3072  1.33
cap: 3072 -> 4096  1.33
cap: 4096 -> 5120  1.25
cap: 5120 -> 7168  1.40
cap: 7168 -> 9216  1.29
cap: 9216 -> 12288  1.33
cap: 12288 -> 15360  1.25
cap: 15360 -> 19456  1.27
cap: 19456 -> 24576  1.26
cap: 24576 -> 30720  1.25
cap: 30720 -> 38912  1.27
cap: 38912 -> 49152  1.26
cap: 49152 -> 61440  1.25
cap: 61440 -> 76800  1.25
cap: 76800 -> 96256  1.25
cap: 96256 -> 120832  1.26
cap: 120832 -> 151552  1.25
cap: 151552 -> 189440  1.25
cap: 189440 -> 237568  1.25
cap: 237568 -> 296960  1.25
cap: 296960 -> 371712  1.25
cap: 371712 -> 464896  1.25
cap: 464896 -> 581632  1.25
cap: 581632 -> 727040  1.25
cap: 727040 -> 909312  1.25
cap: 909312 -> 1136640  1.25
```

从输出看，容量最开始以2倍的方式进行开辟，数据量越大，增长因子会稳定在1.25左右。下面看一下源码

```go
if cap > doublecap {
		newcap = cap
	} else {
		if old.len < 1024 {
			newcap = doublecap
		} else {
			// Check 0 < newcap to detect overflow
			// and prevent an infinite loop.
			for 0 < newcap && newcap < cap {
				newcap += newcap / 4
			}
			// Set newcap to the requested cap when
			// the newcap calculation overflowed.
			if newcap <= 0 {
				newcap = cap
			}
		}
	}
```



**长度小于1024，以2倍的方式增长。超过1024每次增长上次的1/4。也就是会每次增加 25%的容量。随着语言的演化，这种增长算法可能会有所改变。**

在大批量添加数据时，建议一次性分配足够大的空间，以减少内存分配和数据复制开销。或初始化足够长的 len 属性，改用索引号进行操作。及时释放不再使用的 slice 对象，避免持有过期数组，造成 GC 无法回收。 

但是如果是要避免多个slice同时操作同一内存出现错误时，就需要将将slice创建为长度=容量，避免append新slice指向同一内存操作数据，具体情况根据实际灵活选择。

### （5）将一个切片追加到另一个切片 ,使用...运算符 

```go
// 创建两个切片，并分别用两个整数进行初始化
s1 := []int{1, 2}
s2 := []int{3, 4}
// 将两个切片追加在一起，并显示结果
fmt.Printf("%v\n", append(s1, s2...))   //使用... 就可以
//输出:
[1 2 3 4]
//就像通过输出看到的那样，切片 s2 里的所有值都追加到了切片 s1 的后面。使用 Printf时用来显示 append 函数返回的新切片的值
```

## 4、使用for  range  迭代slice

```go
	// 创建一个整型切片
	// 其长度和容量都是 4 个元素
	slice := []int{10, 20, 30, 40} // 迭代每一个元素，并显示其值
	for i, v := range slice {
		fmt.Printf("Index: %d Value: %d\n", i, v)
	}
	//输出
	Index: 0 Value: 10
	Index: 1 Value: 20
	Index: 2 Value: 30
	Index: 3 Value: 40
```

**关键字 range 会返回两个值。第一个值是当前迭代到的索引位置，第二个值是该位置对应元素值的一份拷贝** 

```go
// 创建一个整型切片
// 其长度和容量都是 4 个元素
slice := []int{10, 20, 30, 40}
// 迭代每个元素，并显示值和地址
for index, value := range slice {
	fmt.Printf("Value: %d Value-Addr: %X ElemAddr: %X\n",
	value, &value, &slice[index])
}
//输出:-----明显看出内存地址不同，所以这里的v，是拷贝，其实只是一个内存地址不断更新存不同的拷贝
Value: 10 Value-Addr: C000058058 ElemAddr: C0000560C0
Value: 20 Value-Addr: C000058058 ElemAddr: C0000560C8
Value: 30 Value-Addr: C000058058 ElemAddr: C0000560D0
Value: 40 Value-Addr: C000058058 ElemAddr: C0000560D8
```

因为迭代返回的变量是一个迭代过程中根据切片依次赋值的新变量，所以 value 的地址总是相同的。要想获取每个元素的地址，可以使用切片变量和索引值 

**迭代返回索引和位置非常有用，如果我们只想用其中一个，可以通过占位符 “_” 操作**

```go
slice := []int{10, 20, 30, 40}
for _, value := range slice {// 迭代每个元素，并显示其值，通过占位符，不需要得到索引
	fmt.Printf("Value: %d\n", value)
}
//输出:
Value: 10
Value: 20
Value: 30
Value: 40
```

## 5、copy 复制slice

函数 copy 在两个 slice 间复制数据，复制长以 len 小的为准。两个 slice 可指向同一底层数组，允许元素区间重叠。 

### （1）不指定位置

```go
s1 := []int{1, 2, 3, 4, 5, 6}
s2 := []int{7, 8, 9}
fmt.Println(s2)
copy(s2, s1)        //将s1  copy到 s2, 谁的长度小听谁的，此处只能拷贝前三个
fmt.Println(s2) 
//输出------长度大的往长度小的复制最大只能复制以小长度为准的位数
[7 8 9]
[1 2 3]

```

### （2）指定位置

还可以指定复制slice 和被复制slice 的位置进行copy，不指定默认从前往后，如上面代码

```go
s1 := []int{0,1, 2, 3, 4, 5, 6}
s2 := []int{7, 8, 9}
fmt.Println("s1=",s1)
fmt.Println("s2=",s2)
copy(s1[4:6], s2[1:3]) //将s2指定位置复制到s1指定的位置
fmt.Println("s1=",s1)
//输出
s1= [0 1 2 3 4 5 6]
s2= [7 8 9]
s1= [0 1 2 3 8 9 6]
```

### （3）注意，copy以后的数组，指向的底层数组也会随之改变

```go
data := [...]int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
s := data[8:]
s2 := data[:5]
copy(s2, s) // dst:s2, src:s
fmt.Println(s2)
fmt.Println(data)
//输出
[8 9 2 3 4]
[8 9 2 3 4 5 6 7 8 9]
```
##  6、切片的传递
### 切片的传递，都是值传递，是切片的拷贝

在 64 位架构的机器上，一个切片需要 24 字节的内存：指针字段需要 8 字节，长度字段需要8字节，容量字段需要 8 字节。由于与切片关联的数据包含在底层数组里，不属于切片本身，所以将切片复制到任意函数的时候，对底层数组大小都不会有影响。复制时只会复制切片本身，不会涉及底层数组 。所以不管多大的slice，值传递都不会影响性能。

# 五、多维切片

多维切片和多维数组差不多，只不过[]没有值

```go
// 创建一个整型切片的切片
slice := [][]int{{10}, {100, 200}}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190226164332322.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)



使用append 给slice[0]进行增长，查看内存变化

```go
// 创建一个整型切片的切片
slice := [][]int{{10}, {100, 200}}
// 为第一个切片追加值为 20 的元素
slice[0] = append(slice[0], 20)
```
以上代码会先增长切片，会为新的整型切片分配新的底层数组，然后将切片复制到外层切片的索引为 0 的元素 
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190226164544625.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)