> 底层结构

```go
type hchan struct {
  qcount int
  dataqsiz uint
  buf unsafe.Pointer
  elemsize uint16
  closed uint32
  elemtype *_type
  sendx uint
  recvx uint
  recvq waitq
  sendq waitq
  lock mutex
}
```

> select

+ 底层调用selectgo，每个case是scase
+ 对多路channel,会先按channel顺序加锁，在乱序轮询返回已就绪的case下标

> 非阻塞式

```go
select {
	case ch<-1:
  ...
  default:
  ...
}
```

+ 发生操作对应chansend()，接收操作对应chanrecv()

+ 非阻塞式的send操作对应runtime.selectnbsend()调用
+ 非阻塞式的recv操作对应runtime.selectnbbrecv()调用