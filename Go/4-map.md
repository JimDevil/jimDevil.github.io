>  桶分配算法

+ 取模法：hashcode%m
+ 与运算 ：hashcode&(m-1),m必须是2的次方



> 解决hash冲突

1. 开放地址法
2. 拉链法



> Go map 底层结构

```go 
type hmap struct {
	// Note: the format of the hmap is also encoded in cmd/compile/internal/reflectdata/reflect.go.
	// Make sure this stays in sync with the compiler's definition.
	count     int // 键值对数量
	flags     uint8  //标记包含write
	B         uint8  // 2^B为bucket的数量，hashcode的低B位参与确定bucket的位置
	noverflow uint16 // 记录溢出桶的数量
	hash0     uint32 // hash seed

	buckets    unsafe.Pointer // 桶
	oldbuckets unsafe.Pointer // 扩容时的旧桶
	nevacuate  uintptr        // 扩容阶段下个要迁移的旧桶编号

	extra *mapextra // 溢出桶相关信息
}
```

+ 当B大于4时，会预分配2^(B-4)个溢出桶备用，与常规桶在内存上是连续的

![image-20220428182857462](/Users/chengjin/Desktop/image-20220428182857462.png)

> 扩容-渐进式扩容

+ 翻倍扩容： count/2^B > 6.5

+ 等量扩容：B<=15，noverflow>=2^B;B>15, noverflow>2^15

为了减少性能抖动，采用渐进式扩容，迁移bucket



> 如何判断bucket是否迁移完成

```go
// 空的 cell，也是初始时 bucket 的状态
empty          = 0
// 空的 cell，表示 cell 已经被迁移到新的 bucket
evacuatedEmpty = 1
// key,value 已经搬迁完毕，但是 key 都在新 bucket 前半部分，
// 后面扩容部分会再讲到。
evacuatedX     = 2
// 同上，key 在后半部分
evacuatedY     = 3
// tophash 的最小正常值
minTopHash     = 4

func evacuated(b *bmap) bool {
	h := b.tophash[0]
	return h > empty && h < minTopHash
}
```

+ cell中每个key的存储的时候原hash会加上minTopHash,搬迁后的值会设置成上面几种可以判断搬迁后的去向；

+ evacuate函数进行搬迁，每次最多搬迁2个bucket,
+ 2 层循环，外层遍历 bucket 和 overflow bucket，内层遍历 bucket 的所有 cell；

> 遍历过程

+ 随机选一个bucket,再随机选一个cell开始遍历
+ 如果扩容的过程中遍历，一个bucket中的数据可能分裂到新的2个bucket中，不变的称为X part,变的称为Y part
+ 遍历过程中先判断老的bucket是否迁移完成，如果完成直接遍历当前bucket即可，如果没有迁移完成，先确定当前新bucket是X还是Y，再到老的bucket选择高1位是0（X part）,1(Y part)进行遍历
+ math.NaN()，每次结果都不一样，所以搬迁时只取最低位

> 赋值过程

+ mapassing()函数
+ 判断flag的写标志位是否被置1了，（map不允许并发读写,会panic）
+ 如果扩容过程中进行赋值要等老bucket搬迁完成后才能进行更新或赋值

> 删除过程

+ 同样检查flag的写标志位
+ 如果map正在扩容，则触发一次搬迁操作
+ Count-1,tophash置为empty