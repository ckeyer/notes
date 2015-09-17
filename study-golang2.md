<!--
{
"author":"ckeyer",
"name": "20150704",
"head":"http://blog.ckeyer.com/blog/img/logo_l.jpg",
"date":"2015-07-04",
"title":"Golang中的map",
"tags":["编程语言", "Golang", "数据结构"],
"category":["学习笔记", "Golang"],
"status":"publish",
"summary":"学习golang中的map"
}
-->


### 数据结构
go语言中map在底层是通过哈希表实现的，以下为hmap中其中几个关键属性（源码在$GOROOT/src/pkg/runtime/hashmap.go）:
```
type hmap struct {
	count int // # live cells == size of map.  Must be first (used by len() builtin)
	flags uint8
	B     uint8  // log_2 of # of buckets (可以容纳2^B个项)

	buckets    unsafe.Pointer // array of 2^B Buckets. may be nil if count==0.
	oldbuckets unsafe.Pointer // 前一个buckets，只有当正在扩容时才不为空
	nevacuate  uintptr        // progress counter for evacuation (buckets less than this have been evacuated)

	overflow *[2]*[]*bmap
}
```

这个hash结构使用的是一个可扩展哈希的算法，由hash值mod当前hash表大小决定某一个值属于哪个桶，而hash表大小是2的指数，即上面结构体中的2^B。每次扩容，会增大到上次大小的两倍。结构体中有一个buckets和一个oldbuckets是用来实现增量扩容的。正常情况下直接使用buckets，而oldbuckets为空。如果当前哈希表正在扩容中，则oldbuckets不为空，并且buckets大小是oldbuckets大小的两倍。

#### 增量扩容

大家都知道哈希表表就是以空间换时间，访问速度是直接跟填充因子相关的，所以当哈希表太满之后就需要进行扩容。

如果扩容前的哈希表大小为2^B，扩容之后的大小为2^(B+1)，每次扩容都变为原来大小的两倍，哈希表大小始终为2的指数倍，则有(hash mod 2^B)等价于(hash & (2^B-1))。这样可以简化运算，避免了取余操作。

假设扩容之前容量为X，扩容之后容量为Y，对于某个哈希值hash，一般情况下(hash mod X)不等于(hash mod Y)，所以扩容之后要重新计算每一项在哈希表中的新位置。当hash表扩容之后，需要将那些旧的pair重新哈希到新的table上(源代码中称之为evacuate)， 这个工作并没有在扩容之后一次性完成，而是逐步的完成（在insert和remove时每次搬移1-2个pair），Go语言使用的是增量扩容。

为什么会增量扩容呢？主要是缩短map容器的响应时间。假如我们直接将map用作某个响应实时性要求非常高的web应用存储，如果不采用增量扩容，当map里面存储的元素很多之后，扩容时系统就会卡往，导致较长一段时间内无法响应请求。不过增量扩容本质上还是将总的扩容时间分摊到了每一次哈希操作上面。

扩容会建立一个大小是原来2倍的新的表，将旧的bucket搬到新的表中之后，并不会将旧的bucket从oldbucket中删除，而是加上一个已删除的标记。

正是由于这个工作是逐渐完成的，这样就会导致一部分数据在old table中，一部分在new table中， 所以对于hash table的insert, remove, lookup操作的处理逻辑产生影响。只有当所有的bucket都从旧表移到新表之后，才会将oldbucket释放掉。

扩容的填充因子是多少呢？如果grow的太频繁，会造成空间的利用率很低， 如果很久才grow，会形成很多的overflow buckets，查找的效率也会下降。 这个平衡点如何选取呢(在go中，这个平衡点是有一个常量来确定的( const loadFactor = 6.5 ), 它的意思是这样的，如果table中元素的个数大于table中能容纳的元素的个数， 那么就触发一次grow动作。

#### 查找过程
1. 根据key计算出hash值。
2. 如果存在old table, 首先在old table中查找，如果找到的bucket已经evacuated，转到步骤3。 反之，返回其对应的value。
3. 在new table中查找对应的value。

这里一个细节需要注意一下。不认真看可能会以为低位用于定位bucket在数组的index，那么高位就是用于key/valule在bucket内部的offset。事实上高8位不是用作offset的，而是用于加快key的比较的。

#### 插入过程分析
1. 根据key算出hash值，进而得出对应的bucket。
2. 如果bucket在old table中，将其重新散列到new table中。
3. 在bucket中，查找空闲的位置，如果已经存在需要插入的key，更新其对应的value。
4. 根据table中元素的个数，判断是否grow table。
5. 如果对应的bucket已经full，重新申请新的bucket作为overbucket。
6. 将key/value pair插入到bucket中。


