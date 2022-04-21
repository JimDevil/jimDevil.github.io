### K8s 资源模型

资源模型： 3种QoS

guaranteed(保证)>burstable(可爆发的)>besteffort（尽最大努力）

+ Guaranteed: 每一个container都设置了limit与request且值相同，或只设置了limit

+ Burstable: 至少有一个container设置了request
+ Besteffort:没有设置limit和request



Eviction(驱逐): 分soft（缓冲时间）和hard（立即），依靠Cgroup以及cAdvisor

删除pod时参考QoS



Cpuset: 将pod的QoS设置为guaranteed,cpu资源设置为整数

+ 当node存在压力是会被打上taint,不会被调度到



### K8S 调度器

职责： 为pod选出一个最合适的node

Predicate：检查每个node(多个goroutine,以节点粒度并发执行)

Priority: 给node打分（mapreduce进行计算汇总）

调度过程：
Informer path: informer watch etcd->push queue->update local cache

Schedule path: pop queue->predicates->prioritys-> bind

Cache作用: 集群信息（node、pod、svc等）cache化，提高调度算法效率

bind: 修改pod的nodename,只会更新cache，并不调用apiserver。“乐观”假设的API对象更新方式，称作“assume"(假设)。之后闯将一个goroutine异步的想APIserver更新

Kubelet: admit操作，即通过predicates里的一些基本调度算法（资源、端口等情况）做二次确认



predicates算法调度策略：

1.GeneralPredicates:基础调度策略，检查resource,host,port等

2.volume相关：挂载是否冲突，节点上volume是否超过一定数目

3.node相关过滤：污点和容忍

4.pod相关过滤：affinity/anti-affinity

priority算法：

常用的LeastRequestedPriority：实际上就是选择空闲资源最多的主机



举例：predicates选出10台node都满足request,limit，priority则要根据每台node的实际空闲的资源大小进行打分



#### 优先级和抢占（preemption）

2个队列，

activeQ: 调度器工作的queue

unschedulableQ：调度失败的Pod

流程：schedule failed->push unschedulableQ->Trigger preemption->check filed resaon->copy node info,模拟 preemption

真正抢占：

1.删除牺牲者pod的nominatedNodeName

2.update 抢占者nominatedNodeName，push to activeQ

3.go delete Victim

