
## 云计算的演变
从 IAAS基建即服务，到PAAS 平台即服务，再到SAAS 软件即服务

所以到底什么是云计算？
不是说把服务放到云服务器上就叫云计算
而是 服务自动扩容、弹性伸缩、自动修复
这些才是云计算的特点


首先回到 Docker

Docker 是单机容器管理
会出现以下问题：
1. 容器数量变多了，怎么管理？
2. 有很多物理机，容器应该跑在哪台机器上？
3. Docker 挂了怎么办？
4. Docker 的IP是动态变化的，怎么找到重启后的容器？

因为Docker只管单机容器，不能管理多容器，
所以k8s应运而生
k8s主要解决的问题是，容器数量增多后，部署、调度、服务发现、扩缩容和滚动更新的问题

Kubernetes 的本质就是 **容器编排和调度系统** 


什么叫编排？
==多个独立的部分按照规则自动协同工作== 
Orchestration ，编排二字来源于管弦乐队/交响乐团的指挥。
Docker 容器就像 乐团里的提琴手、鼓手。能熟练得吹奏自己的部分。
如果只有乐手而没有指挥，上百个乐手坐在台上，没人控制节奏提琴手拉得太快，鼓手敲得太慢，最后一团糟。

所以到底还是要一个指挥家滴。

k8s就像那位指挥家
1. 自己不演奏任何乐器
2. 拿着总谱
3. 拿着“指挥棒”告诉乐手，这个声部大声一点。




==回到技术，编排的核心技术如下== 
1. 部署与调度（schedule）：这么多容器应该分配到哪台机器？（青柚服务）
2. 生命周期管理（lifecycle）：容器启动顺序是什么（依赖）？谁先谁后？
3. 服务发现和网络（networking）：pod和pod之间如何通信？
4. 弹性伸缩（scaling）：流量来了，怎么10秒内把2个容器扩成200个？
5. 存储（storage）：实现数据持久化存储




## 一些概念：
pod
	“豆荚”。pod是k8s最小调度单位[^1]。
	一个pod里可以有多个容器


```markdown
Deployment
      │
Replica Set
      │
Pod
      │
Container
```


资源清单yaml
（附图）


==生命周期==Pause
	init容器
	hook启动前钩子、启动后钩子
	probe启动探针、就绪探针、存活探针
	（附图）
	（青柚运维，集群、组件初始化及作用）

Deployment（部署）控制器：
	数量期望
	==滚动升级、灰度发布和应用回滚== （青柚服务代码更新）
	手动扩容与缩容

HPA：自动扩容

ingress：
	原来pod 只跨 node 通信，外面的流量通常不能直接访问pod，
	但可以通过ingress（就像Nginx的反向代理），使得外部流量也能进来。
	说白了这个ingress也是网络分发的抽象设计。
	易错点：Ingress 负责外部 http/https **入口规则** ，Ingress controller负责转发流量

service[^2]：
负责集群内部稳定访问
负载均衡



PV与PVC：这个也是抽象性的设计
	物理存储——> PV。
	PVC在PV的基础上，申请使用存储空间




用户流量的路径是怎么样的？

在浏览器访问https://qq.com

 1. dns——>负载均衡 /  nodeport集群ip
 2. 进入集群入口，也就是 Ingress controller
 3. controller 再根据 Ingress 域名和路径 Rule 决定转发到哪个 service
 4. service 再在后面的一组pod 里选一个
 5. 请求进入某个 pod
 6. pod 里的业务容器处理请求












Yaml 资源清单
![[Pasted image 20260725101554.png]]





pod生命周期

![[Pasted image 20260725101532.png]] 





k8s组件

 ![[c6fabf75b8e264691d6339ab6c324c09_720.png]]




集群初始化

控制面组件：
kube-apiserver ：集群统一入口，所有操作都经过他

etcd ：保存集群状态

kube-scheduler ：给新pod选择节点

kube-controller-manager ：不断把实际状态修正为期望状态


节点node组件
kubelet ：创建本节点上的pod

containerd ：真正创建和运行容器

kube-proxy ：维护 service 的网络转发规则

CNI插件 ：负责pod 网络，例如cilium、calico

CoreDNS ：集群内域名解析
CSI ：存储插件

ingress controller ：http/https 入口
metrics-server ：资源指标






























[^1]: 为什么以pod而不以容器？因为pod间网络通信、共享存储、统一分配资源、统一生死。（或者说谷歌实践下来，觉得这样打包着管理是更优解）（还是回到cpu memory net process？）

[^2]: sevice如何找到pod？1标签选择器，2端点控制器。service通过selector匹配pod的标签，如app=nginx。

[^3]: 
