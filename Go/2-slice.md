> 底层结构

```go
type slice struct {
	array unsafe.Pointer
	len   int
	cap   int
}
```

+ 通过make函数初始化
+ array是指向底层数组的指针



> 扩容

先预估容量

+ oldCap*2  < newCap,直接扩到newCap
+ oldCap>1024, newCap = 1.25*oldCap，
+ oldCap<1024, newCap = 2*oldCap

根据预估容量*元素大小，寻址最匹配的内存规格，再得出实际扩容大小，所以实际扩容后的容量与上面公司可能存在一定出入

