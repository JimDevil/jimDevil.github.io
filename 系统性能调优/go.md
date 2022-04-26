### GO GC

+ Mutator: 赋值器，修改对象之间的引用关系
+ allocator: 分配器，为应用分配内存
+ Collector: 垃圾收集器，回收堆内存，返回给操作系统



> 分配内存

新建一个对象需要申请内存

1. 由mutator通过mmap系统调用向操作系统申请内存
2. 有allocator将申请的内存与用户对象进行映射

go内存分配器基于tcmalloc,按内存大小分级，分为3类

+ Tiny: <16B
+ Small:<16B,<32KB
+ Large:>32KB

