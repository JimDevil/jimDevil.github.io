overlay网络：在原有的物理网络上，通过软件构建出一个覆盖在已有网络之上的可以把容器连通在一起的网络。

underlay:在overlay下面的物理网络



### fannel

3种实现方式：

1. udp:最早支持的方式，性能最差，原因：基于隧道（tun）用户态和内核态切换太多,etcd保存每个节点上的子网信息

   container1访问container2网络数据流向：

   container1->docker0->flannel0(tun，内核态)->fanneld(用户态)->node1(eth0)->

   node2(eth0)->flanneld->funnel0->docker0->container2

2.VXLAN: Virtual Extensible LAN,内核态实现封装解封，在三层网络之上，覆盖一层虚拟的。内核VXLAN负责的二层网络

​	通过VTEP（VXLAN Tunnel EndPoint）设备,有IP,MAC

​	container1访问container2网络数据流向：

​	container1->cni0->flannel.1(vtep)->node1(eth0)->

​	node2(eth0)->flannel.1(vtep)->cni0->container2

​	flannel.1 只知道另一端的mac地址,却不知道对应的宿主机地址

| 目的宿主机MAC | 目的宿主机IP                         | Udp header | Vxlan header                | Inner Ethernet header(目的vtep Mac) | Inner IP header(目的IP) | Inner IP body |
| ------------- | ------------------------------------ | ---------- | --------------------------- | ----------------------------------- | ----------------------- | ------------- |
| ARP           | fanneld维护FDB拿到目的vtep的宿主机IP |            | vni 1标志由哪个vtep设备处理 | flanneld维护arp记录                 |                         |               |

3.host-gw:将每个flannel子网的”下一跳“，设置成了对应宿主机ip地址，宿主机充当网关。通过路由表里的”下一跳“设置MAC，经过二层网络到达目的宿主机，要求宿主机之间二层连通。flannled watch etcd 维护路由信息。

### calico

与flannel host-gw方案类似，使用BGP（border gateway protocol）维护路由信息。大规模网络中实现节点路由信息共享的一种协议。

无网桥，每个节点（BGP Peer）有BGP client将路由表信息传输给其它网关，其它client接收到后会分析，将自己需要的信息添加到自己的路由表里。

可靠性与扩展性远非flannel可比

组成部分：CNI插件，Felix(插入路由规则)，BIRD（BGP client 分发路由信息）

缺点：节点增多，BGP client间通信呈指数级增长,建议小于100个节点使用。

解决方案：route reflector节点（专门负责收集路由信息），其它节点只跟这几个节点交互

不同子网会打开IPIP模式，通过TUNL0(IP隧道)发送



### CNI插件

配置容器网络栈