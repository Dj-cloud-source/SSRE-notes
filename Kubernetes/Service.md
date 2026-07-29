
节点Node就是一台真实的服务器物理机或虚拟机

集群Cluster 是 是一组服务器节点的集合


Service本质是什么？
Service是 一个稳定的虚拟IP + 一套转发规则
```
              Service

            10.96.0.10

                 |

       ---------------------

       |          |         |

     Pod-A      Pod-B     Pod-C

```


为什么需要Service？
- 解决pod创销时 IP 不稳点的问题，提供一个稳定的入口

怎么找到pod？
- Label、Selector
- pod 里 labels:  app : nginx
- Service 里 selector:  app : nginx
请求到了Service，怎么转发？
- kube-porxy
- IPVS

Service类型（面试高频）
- ClusterIP 集群内部访问
- NodePort 高位端口转发
- LoadBalancer     用户→ http80 Loadbalancer  → node ip:30123  →ipvs → pod:80
- ExternaName





## 不多废话，直接看命令和yaml


```yml
#nginx-deploy.yaml
#labels:
#  app: nginx

apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx-deploy

spec:
  replicas: 3

  selector:
    matchLabels:
      app: nginx

  template:
    metadata:
      labels:
        app: nginx

    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```


```yml
#nginx-service.yaml
#selector:
#  app: nginx

apiVersion: v1
kind: Service

metadata:
  name: nginx-service

spec:

  selector:
    app: nginx

  ports:
  - port: 80
    targetPort: 80

  type: ClusterIP
```

```bash
kubectl apply -f nginx-deploy.yaml
kubectl apply -f nginx-service.yaml

```

```bash
#查看pod
kubectl get pod -o wide


#查看service
kubectl  get svc

#查看详细
kubectl describe svc nginx-service

```



修改配置文件再试验
```yaml

type: NodePort

```
然后再走一遍流程



常用命令和排障顺序
```bash


#查看Service
kubectl get svc


kubectl describe svc nginx-service

#查看endpoint
kubectl get endpoints
kubectl get pod  --show-labels


kubectl delete svc  nginx-service
kubectl edit svc  nginx-service

#查看yaml
kubectl get svc nginx-service  -o  yaml
```


## 请求到了Service，怎么转发？

```
Service IP
    ↓
kube-proxy
    ↓
IPVS规则
    ↓
Pod IP

```

为什么选ipvs
	1. 算法丰富     rr轮询，wrr，lc，sh
	2. 底层哈希表，复杂度O(1) ，大规模集群性能极其稳定
	3. 增量更新，网络影响小

### IPVS 试验

1. 创建Service
2. 修改kube-proxy
```bash
#每台node安装、执行（生产中用初始化脚本或Ansible 批量执行）
yum install -y ipset ipvsadm

modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack

cat >/etc/modules-load.d/ipvs.conf <<'EOF'
ip_vs
ip_vs_rr
ip_vs_wrr
ip_vs_sh
nf_conntrack
EOF

systemctl restart systemd-modules-load





#控制端
kubectl edit configmap  kube-proxy   -n  kube-system
```
```yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: kube-proxy
  namespace: kube-system
data:
  config.conf: |    # 注意！必须改这个里面的内容
    apiVersion: kubeproxy.config.k8s.io/v1alpha1
    kind: KubeProxyConfiguration
    mode: "ipvs"    # <--- 在这里改，把 "iptables" 改成 "ipvs"
    # ... 其他配置保持不变
```



重新加载
```bash

kubectl rollout restart daemonset kube-proxy -n kube-system
```

4. 看ipvs表
```bash
ipvsadm  -Ln
```

