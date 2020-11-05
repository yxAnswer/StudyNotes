# Go语言的映射——map

[TOC]

> - 类似其它语言中的哈希表或者字典，以key-value形式存储数据
> - Key必须是支持==或!=比较运算的类型，不可以是函数、map或slice；value 可以是任意类型。
> - Map查找比线性搜索快很多，但比使用索引访问数据的类型慢100倍
> - Map使用make()创 建，支持 := 这种简写方式
> - make(map[keyType]valueType, cap)，cap表示容量，可省略
> - 超出容量时会自动扩容，但尽量提供一个合理的初始值,预先申请内存，有助于提升性能 
> - 使用len()获取元素个数
> - 键值对不存在时自动添加，使用delete()删除某键值对
> - 使用 for range 对map和slice进行迭代操作

# Map介绍

go语言中的map是一种数据结构，用于存储一些列无序的键值对。map是一个无序的集合，因为它底层是一个hash表（散列表）。map的键只要支持==或!=比较运算的类型都可以，但是不可以是函数，map，slice等不能比较的。value可以是任意的类型。

# 1、Map 的创建

1.1 使用make函数

```go
// 创建一个映射，键的类型是 string，值的类型是 int
dict := make(map[string]int)
```

make(map[keyType]valueType, cap),cap表示容量，可以在创建的时候指定一个合理初始的容量大小，这样就会申请一大块内存，避免在后续使用中频繁扩张浪费性能。比如：`m := make(map[string]int, 1000) `

1.2 使用字面值创建

```go
// 创建一个映射，键和值的类型都是 string
// 使用两个键值对初始化映射
dict := map[string]string{"Red": "#da1337", "Orange": "#e95a22"}
```

# 2、使用映射

## 2.1为映射赋值

```go 
// 创建一个空映射，用来存储颜色以及颜色对应的十六进制代码
colors:=map[string]string{}
// 将 Red 的代码加入到映射
colors["Red"]="#da1337"
```

## 2.1 未初始化的map,赋值报错,即 nil map

```go
// 通过声明映射创建一个 nil 映射
var colors map[string]string
// 将 Red 的代码加入到映射
colors["Red"] = "#da1337"
Runtime Error:
panic: runtime error: assignment to entry in nil map
```

## 2.2  从map 中获取值，并判断键是否存在

由于go的多返回值，map获取值的时候，会返回值，和一个boolean的参数，表示成不成功，有没有，对不对。

**在 Go 语言里，通过键来索引映射时，即便这个键不存在也总会返回一个值。在这种情况下，返回的是该值对应的类型的零值。** 

```go
// 获取键 Blue 对应的值
value, exists := colors["Blue"]// 这个键存在吗？
if exists {
	fmt.Println(value)
}
//当然了也可以直接用 类型的零值来判断----是一样的
// 获取键 Blue 对应的值
value := colors["Blue"]
// 这个键存在吗？
if value != "" {
	fmt.Println(value)
}
```

## 2.3 用range迭代map

```go
// 创建一个映射，存储颜色以及颜色对应的十六进制代码
colors := map[string]string{
	"AliceBlue": "#f0f8ff",
	"Coral": "#ff7F50",
	"DarkGray": "#a9a9a9",
	"ForestGreen": "#228b22",
}
// 显示映射里的所有颜色
for key, value := range colors {
	fmt.Printf("Key: %s Value: %s\n", key, value)
}
//
```

range迭代和数组、slice 都一样，只不过这里返回的是map的键值对，而array ,slice返回的是索引和值

## 2.4 map中的delete函数

delete(map,key)  函数可以从map中删除指定key的键值对。

```go
// 删除键为 Coral 的键值对
delete(colors, "Coral")
// 显示映射里的所有颜色
for key, value := range colors {
	fmt.Printf("Key: %s Value: %s\n", key, value)
}	

```

## 2.5 map作为参数传递

和slice一样，都是引用类型，都是指向了底层数据结构。slice指向的是数组，map指向的是hash表。map在函数之间作为参数传递的时候，是进行map指针的拷贝，相对于指针来说是值拷贝，相对于底层来说是引用传递。 其实我觉得go所有的传递都是值传递，只不过有的值是值，有的值是指针。

所以，在函数中传递map，对map进行修改会对底层数据进行修改。

```GO
// removeColor 将指定映射里的键删除
func removeColor(colors map[string]string, key string) {
	delete(colors, key)
}
```

```go
// 创建一个映射，存储颜色以及颜色对应的十六进制代码
colors := map[string]string{
	"AliceBlue": "#f0f8ff",
	"Coral": "#ff7F50",
	"DarkGray": "#a9a9a9",
	"ForestGreen": "#228b22",
}
// 显示映射里的所有颜色
for key, value := range colors {
	fmt.Printf("Key: %s Value: %s\n", key, value)
}
fmt.Println()
// 调用函数来移除指定的键
removeColor(colors, "Coral")
// 显示映射里的所有颜色
for key, value := range colors {
	fmt.Printf("Key: %s Value: %s\n", key, value)
}
//输出结果
Key: AliceBlue Value: #F0F8FF
Key: Coral Value: #FF7F50
Key: DarkGray Value: #A9A9A9
Key: ForestGreen Value: #228B22

Key: AliceBlue Value: #F0F8FF
Key: DarkGray Value: #A9A9A9
Key: ForestGreen Value: #228B22
```

**map的常见操作**

```go
m := map[string]int{
	"a": 1,
}
if v, ok := m["a"]; ok { // 判断 key 是否存在。
	println(v)
}
println(m["c"]) // 对于不存在的 key，直接返回零值，不会出错。

m["b"] = 2 // 新增或修改。

delete(m, "c") // 删除。如果 key 不存在，不会出错。

println(len(m)) // 获取键值对数量。 cap ⽆效。

for k, v := range m { // 迭代，可仅返回 key。随机顺序返回，每次都不相同。
	println(k, v)
}
```

# 3、从map取出的value是一个拷贝，对其成员修改没有意义

```go
type user struct{ name string }
m := map[int]user{	 // 当 map 因扩张⽽重新哈希时，各键值项存储位置都会发⽣改变。 因此， map
	1: {"user1"},  	// 被设计成 not addressable。 类似 m[1].name 这种期望透过原 value
} 					// 指针修改成员的⾏为⾃然会被禁⽌。
m[1].name = "Tom" 	// Error: cannot assign to m[1].name
```

因为取出的是一个user实例的拷贝，不能直接对其成员修改，如果要实现，可以有两种方式

3.1完整的替换这个value

```go
u := m[1]
u.name = "Tom"
m[1] = u // 替换 value。
```

3.2 使用指针

```go
m2 := map[int]*user{
	1: &user{"user1"},
}
m2[1].name = "Jack" // 返回的是指针复制品。透过指针修改原对象是允许的。
```





























