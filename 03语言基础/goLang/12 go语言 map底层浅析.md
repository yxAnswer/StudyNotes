# Go语言map底层浅析

[toc]

# 1、Hash函数

哈希表就不得不说hash函数。hash函数，有加密型和非加密型。加密型的一般用于加密数据、数字摘要等，典型代表就是md5、sha1、sha256、aes256这种；非加密型的一般就是查找。在map的应用场景中，用的是查找。选择hash函数主要考察的是两点：性能、碰撞概率。**golang使用的hash算法根据硬件选择，如果cpu支持aes，那么使用aes hash，否则使用memhash**，memhash是参考xxhash、cityhash实现的，性能非常好。

具体hash函数的性能比较可以看：<http://aras-p.info/blog/2016/08/09/More-Hash-Function-Tests/>

哈希函数会将传入的key值进行哈希运算，得到一个唯一的值。比如一个key经过哈希函数，生成的哈希值为：`10010111 000011110110110010001111001010100010010110010101010 01010  `， 10010111 和01010 分别就是：前半部分就叫做**高位哈希值**，后半部分就叫做**低位哈希值**。

**高位哈希值：是用来确定当前的bucket（桶）有没有所存储的数据的。**

**低位哈希值：是用来确定，当前的数据存在了哪个bucket（桶）**

# 2、map 源码

## 2.1hmap(a header of map)

```go
// Go map的一个header结构
type hmap struct {
	count     int // map的大小.  len()函数就取的这个值
	flags     uint8 //map状态标识
    B         uint8  // 可以最多容纳 6.5 * 2 ^ B 个元素，6.5为装载因子即:map长度=6.5*2^B
    				//B可以理解为buckets已扩容的次数
	noverflow uint16 // 溢出buckets的数量
	hash0     uint32 // hash 种子

	buckets    unsafe.Pointer //指向最大2^B个Buckets数组的指针. count==0时为nil.
	oldbuckets unsafe.Pointer //指向扩容之前的buckets数组，并且容量是现在一半.不增长就为nil
	nevacuate  uintptr  // 搬迁进度，小于nevacuate的已经搬迁
	extra *mapextra // 可选字段，额外信息
}
```

hmap是map的最外层的一个数据结构，包括了map的各种基础信息、如大小、bucket。首先说一下，buckets这个参数，它存储的是指向buckets数组的一个指针，当bucket(桶为0时)为nil。**我们可以理解为，hmap指向了一个空bucket数组**，并且当bucket数组需要扩容时，它会开辟一倍的内存空间，并且会渐进式的把原数组拷贝，即用到旧数组的时候就拷贝到新数组。

## 2.2 bmap(a  bucket of map)

```go
// Go map 的 buckets结构
type bmap struct {
	// 每个元素hash值的高8位，如果tophash[0] < minTopHash，表示这个桶的搬迁状态
	tophash [bucketCnt]uint8
  // 第二个是8个key、8个value，但是我们不能直接看到；为了优化对齐，go采用了key放在一起，value放在一起的存储方式，
   // 第三个是溢出时，下一个溢出桶的地址
}
```

bucket（桶），每一个bucket最多放8个key和value，最后由一个overflow字段指向下一个bmap，注意key、value、overflow字段都不显示定义，而是通过maptype计算偏移获取的。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190304132823432.png)

**bucket这三部分内容决定了它是怎么工作的：**

- 它的tophash 存储的是哈希函数算出的哈希值的高八位。是用来加快索引的。**因为把高八位存储起来，这样不用完整比较key就能过滤掉不符合的key，加快查询速度**当一个哈希值的高8位和存储的高8位相符合，再去比较完整的key值，进而取出value。

- 第二部分，存储的是key 和value，就是我们传入的key和value，注意，它的底层排列方式是，key全部放在一起，value全部放在一起。**当key大于128字节时，bucket的key字段存储的会是指针，指向key的实际内容；value也是一样。**

- 这样排列好处是在key和value的长度不同的时候，可以消除padding带来的空间浪费。并且每个bucket**最多存放8个键值对**。

- 第三部分，存储的是当bucket溢出时，指向的下一个bucket的指针

  ![key和value排列](https://img-blog.csdnimg.cn/20190304134051406.png)

**bucket的设计细节：**

在golang map中出现冲突时，不是每一个key都申请一个结构通过链表串起来，**而是以bmap为最小粒度挂载，一个bmap可以放8个key和value。这样减少对象数量，减轻管理内存的负担，利于gc。**
 如果插入时，bmap中key超过8，那么就会申请一个新的bmap（overflow bucket）挂在这个bmap的后面形成链表，**优先用预分配的overflow bucket，如果预分配的用完了，那么就malloc一个挂上去。注意golang的map不会shrink，内存只会越用越多，overflow bucket中的key全删了也不会释放**

# 3、hmap和bmap结构图

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190304115315717.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

如图所示：

- hmap存储了一个指向底层bucket数组的指针。

- 我们存入的key和value是存储在bucket里面中，如果key和value大于128字节，那么bucket里面存储的是指向我们key和value的指针，如果不是则存储的是值。

- 每个bucket 存储8个key和value，如果超过就重新创建一个bucket挂在在元bucket上，持续挂接形成链表。

- 高位哈希值：是用来确定当前的bucket（桶）有没有所存储的数据的。

- 低位哈希值：是用来确定，当前的数据存在了哪个bucket（桶）

  简单结构为图：

  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190304143410819.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

  **工作流程**：

  查找或者操作map时，首先key经过hash函数生成hash值，通过哈希值的低8位来判断当前数据属于哪个桶(bucket)，找到bucket以后，通过哈希值的高八位与bucket存储的高位哈希值循环比对，如果相同就比较刚才找到的底层数组的key值，如果key相同，取出value。如果高八位hash值在此bucket没有，或者有，但是key不相同，就去链表中下一个溢出bucket中查找，直到查找到链表的末尾。
  
  碰撞冲突：如果不同的key定位到了统一bucket或者生成了同一hash,就产生冲突。 go是通过链表法来解决冲突的。比如一个高八位的hash值和已经存入的hash值相同，并且此bucket存的8个键值对已经满了，或者后面已经挂了好几个bucket了。那么这时候要存这个值就先比对key,key肯定不相同啊，那就从此位置一直沿着链表往后找，找到一个空位置，存入它。所以这种情况，两个相同的hash值高8位是存在不同bucket中的。
  
  查的时候也是比对hash值和key 沿着链表把它查出来。  还有一种情况，就是目前就 1个bucket，并且8个key-value的数组还没有存满，这个时候再比较完key不相同的时候，同样是沿着当前bucket数组中的内存空间往后找，找到第一个空位，插入它。这个就相当于是用寻址法来解决冲突，查找的时候，也是先比较hash值，再比较key,然后沿着当前内存地址往后找。 
  
  go语言的map通过数组+链表的方式实现了hash表，同时分散各个桶，使用链表法+bucket内部的寻址法解决了碰撞冲突，也提高了效率。因为即使链表很长了，go会根据装载因子，去扩容整个bucket数组，所以下面就要看下扩容。

#  4、map的扩容

- 当链表越来越长，bucket的扩容次数达到一定值，其实是bmap扩容的加载因数达到6.5，bmap就会进行扩容，将原来bucket数组数量扩充一倍，产生一个新的bucket数组，也就是bmap的buckets属性指向的数组。这样bmap中的oldbuckets属性指向的就是旧bucket数组。
- 这里的加载因子LoadFactor是一个阈值，计算方式为（map长度/2^B ）如果超过6.5，将会进行扩容，这个是经过测试才得出的合理的一个阈值。因为，加载因子越小，空间利用率就小，加载因子越大，产生冲突的几率就大。所以6.5是一个平衡的值。
- map的扩容不会立马全部复制，而是渐进式扩容，即首先开辟2倍的内存空间，创建一个新的bucket数组。只有当访问原来就的bucket数组时，才会将就得bucket拷贝到新的bucket数组，进行渐进式的扩容。当然旧的数据不会删除，而是去掉引用，等待gc回收。

# 5、结语

这篇文章，只是对map底层的结构进行了说明，具体创建、查找、删除等的流程是差不多，但是具体的细节还是有很多。所以，我就不一一写出了，贴一下其他博主写的文章，很详细。
5.1 剖析golang map的实现
	地址：https://www.jianshu.com/p/092d4a746620
5.2 Golang map 的底层实现
	地址：https://www.jianshu.com/p/aa0d4808cbb8

**重点来了，必须看**：非常详细的map源码说明

[https://github.com/cch123/golang-notes/blob/master/map.md](https://github.com/cch123/golang-notes/blob/master/map.md)




参考：

https://www.jianshu.com/p/092d4a746620

https://www.jianshu.com/p/aa0d4808cbb8

https://blog.csdn.net/i6448038/article/details/82057424