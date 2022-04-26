### Mutex实现原理

#### 底层结构

```go
type Mutex struct {
  state int32
  sema uint32
}
```

![image-20220425113001156](/Users/chengjin/my-github/blog/jimDevil.github.io/images/image-20220425113001156.png)

#### 加锁过程

Fastpath:

1.  通过atomic cas操作获取锁	

slowPath:

1. 非饥饿状态，先通过自旋尝试获取，超过一定次数获取不到加入等待队列
2. 饥饿模式下，为了锁的公平性，优先让队列头的goroutine获取锁
3. 当waiter已经是最后一个或等待时间小于1ms时，会转换为正常模式



### RWMutex实现原理

#### 底层实现

```go
type RWMutex struct {
  w Mutex
  writerSem uint32
  readerSem uint32
  readerCount int32
  readerWait int32
}
const rwmutexMaxReaders=1<<30
```



#### 加锁过程

> RLock/RUnlock

RLock: 通过 atomic addint32方法给readerCount+1，并判断是否大于0（小于0则代表有写锁在等待），

RUnlock: atomic readerCount-1,并判断是否大于0，小于0（有写锁竞争）还要检查是否所有的读锁都已经释放锁，都释放了再去唤醒写锁

Lock: 首先调用mutex lock(),避免其它写锁竞争，反转readerCount(减去rwmutexMaxReaders),如果有reader持有锁，等待readerwaiter都释放后被唤醒

Unlock: 还原readerCount,唤醒阻塞的reader,mutex unLock()

