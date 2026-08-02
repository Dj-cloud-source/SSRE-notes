## Service
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



## Ingress


前面的 Nodeport和LB 都有一定的缺点
Nodeport 会占用集群机器的端口，当集群变多就会很麻烦。暴露端口，很丑
LB 缺点是每个Service 需要一个LB，浪费，且需要k8s之外的设备支持




![[Pasted image 20260801093318.png]]  
**Ingress**  将来自集群外部的http和https路由暴露给集群内的服务。流量路由由配置文件的 **规则** 控制

Ingress Controller，真正转发之人😝，一般是nginx （不过现在k8s-nginx不维护了。更多的是用Gateway API）


如图，Ingress 只需要一个Nodeport或者一个LB就可以满足暴露多个Service的需求；
如图，Service 解决集群内部服务发现，Ingress 解决外部http请求如何进入集群





### 看yaml配置规则示例

```yaml

#假设已经有 user-service  order-service


apiVersion: networking.k8s.io/v1
kind: Ingress

metadata:
  name: shop-ingress

spec:

  rules:
 
  - host: shop.com              #当用户访问 shop 时，规则才生效

    http:

      paths:

      - path: /user              #URL 以/user开头的
        pathType: Prefix         #只要是/user开头就算命中规则

        backend:
          service:
            name: user-service   #命中后，把流量转给集群内名叫 user-service 
            port:
              number: 80         #service的端口


      - path: /order
        pathType: Prefix        #逻辑同上，只要是/order开头的路径，转给 order-service的80端口

        backend:
          service:
            name: order-service
            port:
              number: 80
```



### Controller

```bash
#安装 Ingress Controller

kubectl apply \
-f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml


kubectl get pods  -n  ingress-nginx

```

```bash
#安装helm包管理器，helm在k8s中相当于yum。将yaml文件打包成 chart 的安装包，一键安装升级卸载

curl -fsSL  -o get_helm.sh  
https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3

chmod 700 get_helm.sh
./get_helm.sh



helm repo add bitnami  https://charts.bitnami.com/bitnami
helm repo list 
helm repo update

helm search repo nginx
helm search hub  nginx
helm show chart  bitnami/nginx


#安装部署（示例）
helm install   nginx   bitnami/nginx  
helm uninstall nginx-185942
```
```bash
#真安装，不过现在k8s-nginx不维护了。更多的是用Gateway API
helm  pull ingress-nginx/ingress-nginx
tar -zxvf  ingress-nginx.tar.gz


#修改 value.yaml

hostNetwork: true
dnsPolicy: ClusterFirstWithHostNet
kind: DaemonSet
ingressClassResource.default: true





#如果是本地安装，使用命名空间
kubectl create ns  ingress 
helm install ingress-nginx  -n  ingress  . -f  values.yaml
kubectl get pod  -n ingress

```

