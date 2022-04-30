### 内存数据分配

![image-20220427000534803](/Users/chengjin/my-github/blog/jimDevil.github.io/images/image-20220427000534803.png)

+ 程序要执行 的指令在代码段， 全局变量量静态数据在数据段，函数相关变量、参数、返回值在栈帧中
+  如果在编译阶段不能确定数据的大小或者对象生命周期超出当前函数，应该分配在堆上   



#### 三色标记

强三色不变：插入写屏障

弱三色不变：删除写屏障

混合写屏障



#### GC过程

 ![image-20220427003538888](../images/image-20220427003538888.png)

1. GC准备阶段(Mark Setup): 为每个P创建一个mark worker 协程 。把对应的g指针存储到P中 

2. 第一次STW: GC进入_GCMark阶段，gcphase=__GCMark,writeBarrier.enabled = true（是否开启写屏障）,gcblackenabled=1(标记是否允许GC标记工作)

3. start the world,mark work开始标记工作
4.  当没有标记任务时，开始第二次STW，GC 进入GC MarkTermination阶段，确认标记工作完成gcblackenabled=0
5. 进入_GCOff阶段，关闭写屏障 writeBarrier.enabled = false
6. Start the world,进入清扫阶段

![image-20220427003930409](/Users/chengjin/my-github/blog/jimDevil.github.io/images/image-20220427003930409.png)