
当用户提交pod需求时，k8s需要考虑这个pod放哪个node节点
所以需要kube-scheduler 


k8s提供了四大类调度方式：
1. 自动调度
2. 定向调度 NodeName、NodeSelector
3. 亲和性调度 NodeAffinity、PodAffinity、PodAntiAffinity
4. 污点、容忍调度 Taints、Toleration




## 定向调度
将 pod **强制** 调度到期望 node 上。即使node不存在，也会调度，只是pod运行失败而已


### NodeName
```yml

#pod-node.yaml

apiVersion: v1
kind: Pod
metadata:
  name: pod-nodename
spec:
  containers:
  - name: nginx
    image: aaronxudocker/myapp:v1.0
  nodeName: node01 # 指定调度到nodem01节点上

```
```bash
kubectl apply -f pod-node.yaml

kubectl get pods pod-nodename  -o wide 

#删除pod，并换成node3

kubectl delete -f pod-node.yaml
#修改成node3

kubectl get pods pod-nodename  -o wide 
#由于不存在node3，所以pods无法正常运行


```



### NodeSelector
```bash
#打标签
kubectl label nodes node01  nodeenv=pro   
```
```yaml

apiVersion: v1
kind: Pod
metadata:
  name: pod-nodeselector
spec:
  containers:
  - name: nginx
    image: aaronxudocker/myapp:v1.0
  nodeSelector: 
    nodeenv: pro # 指定调度到具有nodeenv=pro标签的节点上

```
```bash
kubectl apply -f pod-node.yaml

kubectl get pod pod-nodeselector -o wide
#和上面同理，删除yaml，修改标签值，……，再次查看，发现pod无法正常运行
```



## 亲和性调度

上面的定向调度很死板，而k8s在NodeSelect基础上进行拓展，通过配置的形式，实现优先选择满足条件的Node进行调度

Affinity 亲和性
- 亲和性：相互靠近
- 反亲和性：打散分布


- node亲和性 ：以node为出发点。比如优先运行在有 GPU 的node上
- pod亲和性   ：以pod为出发点。比如web服务和缓存服务靠近，以减少网络延迟
- pod反亲和性：不要全挤在同一个node。希望分散以提供高可用





yaml 模版
```yml

pod.spec.affinity.nodeAffinity
  requiredDuringSchedulingIgnoredDuringExecution  Node节点必须满足指定的所有规则才可以，相当于硬限制
    nodeSelectorTerms  节点选择列表
      matchFields   按节点字段列出的节点选择器要求列表
      matchExpressions   按节点标签列出的节点选择器要求列表(推荐)
        key    键
        values 值
        operator 关系符 支持Exists, DoesNotExist, In, NotIn, Gt, Lt
  preferredDuringSchedulingIgnoredDuringExecution 优先调度到满足指定的规则的Node，相当于软限制 (倾向)
    preference   一个节点选择器项，与相应的权重相关联
      matchFields   按节点字段列出的节点选择器要求列表
      matchExpressions   按节点标签列出的节点选择器要求列表(推荐)
        key    键
        values 值
        operator 关系符 支持In, NotIn, Exists, DoesNotExist, Gt, Lt
    weight 倾向权重，在范围1-100。




- matchExpressions:
  - key: nodeenv              # 匹配存在标签的key为nodeenv的节点
    operator: Exists
  - key: nodeenv              # 匹配标签的key为nodeenv,且value是"xxx"或"yyy"的节点
    operator: In
    values: ["xxx","yyy"]
  - key: nodeenv              # 匹配标签的key为nodeenv,且value大于"xxx"的节点
    operator: Gt
    values: "xxx"


```

### node亲和性

演示一下硬限制
```yaml

apiVersion: v1
kind: Pod
metadata:
  name: pod-nodeaffinity-required
spec:
  containers:
  - name: nginx
    image: aaronxudocker/myapp:v1.0
  affinity:  #亲和性设置
    nodeAffinity: #设置node亲和性
      requiredDuringSchedulingIgnoredDuringExecution: # 硬限制
        nodeSelectorTerms:
        - matchExpressions: # 匹配nodeenv的值在["xxx","yyy"]中的标签
          - key: nodeenv
            operator: In
            values: ["xxx","yyy"]

```


```bash
kubectl apply -f pod-nodeaffinity-required.yaml
kubectl get pod -o wide

kubectl describe pod pod-nodeaffinity-required


# 修改文件，将values: ["xxx","yyy"]------> ["pro","yyy"]
kubectl get pod -o wide
```

软限制
`preferredDuringSchedulingIgnoredDuringExecution` （调度时优先考虑，运行时忽略）

```yml

apiVersion: v1
kind: Pod
metadata:
  name: pod-nodeaffinity-preferred
spec:
  containers:
  - name: nginx
    image: aaronxudocker/myapp:v1.0
  affinity:  #亲和性设置
    nodeAffinity: #设置node亲和性
      preferredDuringSchedulingIgnoredDuringExecution: # 软限制
      - weight: 1
        preference:
          matchExpressions: # 匹配env的值在["xxx","yyy"]中的标签(当前环境没有)
          - key: nodeenv
            operator: In
            values: ["xxx","yyy"]

```
```bash
kubectl apply -f  pod-nodeaffinity-preferred.yaml

kubectl get pod -o wide 
```


### pod亲和性
yaml 模版
```yml

pod.spec.affinity.podAffinity
  requiredDuringSchedulingIgnoredDuringExecution  硬限制
    namespaces       指定参照pod的namespace
    topologyKey      指定调度作用域
    labelSelector    标签选择器
      matchExpressions  按节点标签列出的节点选择器要求列表(推荐)
        key    键
        values 值
        operator 关系符 支持In, NotIn, Exists, DoesNotExist.
      matchLabels    指多个matchExpressions映射的内容
  preferredDuringSchedulingIgnoredDuringExecution 软限制
    podAffinityTerm  选项
      namespaces      
      topologyKey
      labelSelector
        matchExpressions  
          key    键
          values 值
          operator
        matchLabels 
    weight 倾向权重，在范围1-100



topologyKey用于指定调度时作用域,例如:
    如果指定为kubernetes.io/hostname，那就是以Node节点为区分范围
    如果指定为beta.kubernetes.io/os,则以Node节点的操作系统类型来区分


```


创建参照pod


```yml
#pod-podaffinity-target.yaml


apiVersion: v1
kind: Pod
metadata:
  name: pod-podaffinity-target
  labels:
    podenv: pro #设置标签
spec:
  containers:
  - name: nginx
    image: aaronxudocker/myapp:v1.0
  nodeName: node01 # 将目标pod名确指定到node01上
```
```bash
kubectl apply -f pod-poddffinity-target.yaml
kubectl get pod -o wide --show-labels
```

```yml
#pod-podaffinity-required.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-podaffinity-required
spec:
  containers:
  - name: nginx
    image: aaronxudocker/myapp:v1.0
  affinity:  #亲和性设置
    podAffinity: #设置pod亲和性
      requiredDuringSchedulingIgnoredDuringExecution: # 硬限制
      - labelSelector:
          matchExpressions: # 匹配env的值在["xxx","yyy"]中的标签
          - key: podenv
            operator: In
            values: ["xxx","yyy"]
        topologyKey: kubernetes.io/hostname

#意思是：新Pod必须要与拥有标签nodeenv=xxx或者nodeenv=yyy的pod在同一Node上，显然现在没有这样pod，接下来，运行测试一下
```
```bash
kubectl apply -f pod-podaffinity-required.yaml
kubectl get pod -o wide

kubectl describe pod pod-podaffinity-required.yaml
```

```yml
#接下来修改  values: ["xxx","yyy"]----->values:["pro","yyy"]
#意思是：新Pod必须要与拥有标签nodeenv=xxx或者nodeenv=yyy的pod在同一Node上
#重新创建pod
```
```bash
 kubectl get pod -o wide
```

>`PodAffinity`的 `preferredDuringSchedulingIgnoredDuringExecution` 同理



### pod反亲和性
以运行的pod为参照，让新创建的pod和参照pod不在一个区域中
> 继续使用上面的目标pod

```bash

kubectl get pod --show-labels

```
```yml
#pod-podantiaffinity-required.yaml

apiVersion: v1
kind: Pod
metadata:
  name: pod-podantiaffinity-required
spec:
  containers:
  - name: nginx
    image: aaronxudocker/myapp:v1.0
  affinity:  #亲和性设置
    podAntiAffinity: #设置pod亲和性
      requiredDuringSchedulingIgnoredDuringExecution: # 硬限制
      - labelSelector:
          matchExpressions: # 匹配podenv的值在["pro"]中的标签
          - key: podenv
            operator: In
            values: ["pro"]
        topologyKey: kubernetes.io/hostname

#意思是：新Pod必须要与拥有标签nodeenv=pro的pod不在同一Node上


```
```bash

kubectl get pod -o wide

```
## 污点与容忍
Taint 和 Toleration


污点是给Node打标签，不允许普通pod进来


容忍呢？如果就是想将一个pod调度一个有污点的node上去，这就需要用到容忍

>污点就是拒绝，容忍就是忽略，Node通过污点拒绝pod调度上去，Pod通过容忍忽略拒绝



### 污点

污点格式
`key=value:effect` 

effect
- preferNoSchedule 尽量别来，除非没办法
- NoSchedule 新的别来，旧的先待着
- Noexecute 全都驱赶走


设置污点与去除污点
```bash
#设置污点
kubectl taint nodes key=value:effect

#去除污点
kubectl taint nodes node1 key:effect-

#去除所有污点
kubectl taint nodes node1 key-

```

```bash
#示例
kubectl taint node node01  tag=eagle:PrefreNoDchedule

kubectl create deployment myapp                                --image=aaronxudocker/myapp:v1.0 --replicas=100


kubectl get pod -o wide 



kubectl taint nodes node01 tag:PreferNoSchedule-
kubectl taint nodes node01 tag=eagle:NoSchedule

kubectl create deployment myapp --image=aaronxudocker/myapp:v1.0 --replicas=100

kubectl get pod -o wide

#关闭node02虚拟机后，再次创建，无法被调度
kubectl get -o wide 


#设置NoExecute 
kubectl taint nodes node01 tag:NoSchedule-

kubectl create deployment myapp --image=aaronxudocker/myapp:v1.0 --replicas=10
deployment.apps/myapp created

kubectl get pod -o wide 

kubectl taint nodes node01  tag=eagle:NoExecute
kubectl get pod -o wide 

```
>之前使用kubeadm搭建的集群，默认就会给master节点添加一个污点标记,所以pod就不会调度到master节点上.



### 容忍

```yml

```



上面的节点已经打上了 NoExecute，现在可以通过添加容忍，然后调度上去

```yml
#pod-toleration.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: myapp
  name: myapp
spec:
  replicas: 10
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - image: aaronxudocker/myapp:v1.0
        name: myapp
      tolerations:      # 添加容忍
      - key: "tag"        # 要容忍的污点的key
        operator: "Equal" # 操作符
        value: "eagle"    # 容忍的污点的value
        effect: "NoExecute"   # 添加容忍的规则，这里必须和标记的污点规则相同

```

```bash
kubectl apply -f pod-toleration.yaml

kubectl get pod -o wide

```