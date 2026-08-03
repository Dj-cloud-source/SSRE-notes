
```

真实物理存储
   ↓
PV：集群级存储
   ↓
PVC：应用申请存储
   ↓
Pod：使用 PVC
   ↓
Container：挂载到目录

```


PV与PVC匹配参考指标
```
容量
访问模式
StorageClass
VolumeMode
选择器

```




### PV资源清单

```yml

spec:
  nfs: # 存储类型，与底层真正存储对应
  capacity:  # 存储能力，目前只支持存储空间的设置
    storage: 2Gi
  accessModes:  # 访问模式
  storageClassName: # 存储类别
  persistentVolumeReclaimPolicy: # 回收策略
```


1. 存储类型
2. 存储能力
3. 访问模式
	1. ReadWriteMany  多节点挂载
	2. ReadWriteOnce  单节点挂载
	3. ReadOnlyMany   多节点只读
4. 回收策略
	1. Retain   保留
	2. Delete  删除


### PVC资源配置清单



```yml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc
spec:
  accessModes: # 访问模式
  selector: # 采用标签对PV选择
  storageClassName: # 存储类别
  resources: # 请求空间
    requests:
      storage: 5Gi
```







### 实操

>改成真实ip
```bash
#每台服务器。
#kubectl debug node/node02  -it --image=rockylinux 可以用这个命令实验上去
yum install -y  nfs-utils

mkdir -pv /data/nfs/{pv1,pv2,pv3}  

cat >/etc/exports <<'EOF' 
/data/nfs/pv1 192.168.60.0/24(rw,sync,no_root_squash) 
/data/nfs/pv2 192.168.60.0/24(rw,sync,no_root_squash) 
/data/nfs/pv3 192.168.60.0/24(rw,sync,no_root_squash) 
EOF


systemctl  enable  --now nfs-server
exportfs -rav
exportfs -v
firewall-cmd --permanent --add-service=nfs
firewall-cmd --reload

```

创建PV和PVC
> ！如果同一个yaml文件里有多个资源，每个文件之间必须加  ---  ！
>     vim :set list
```yml
#pv.yaml

apiVersion: v1
kind: PersistentVolume
metadata:
  name:  pv1
spec:
  capacity: 
    storage: 1Gi
  accessModes:
  - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  nfs:
    path: /data/nfs/pv1
    server: 192.168.173.100

```
```yml
apiVersion: v1
kind: PersistentVolume
metadata:
  name:  pv2
spec:
  capacity: 
    storage: 2Gi
  accessModes:
  - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  nfs:
    path: /data/nfs/pv2
    server: 192.168.173.100
```
```yml
apiVersion: v1
kind: PersistentVolume
metadata:
  name:  pv3
spec:
  capacity: 
    storage: 3Gi
  accessModes:
  - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  nfs:
    path: /data/nfs/pv3
    server: 192.168.173.100
```

```bash
kubectl  apply -f pv.yaml

kubectl get   pv 

```


```yml
#pvc.yaml

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc1
spec:
  accessModes: 
  - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
```
```yml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc2
spec:
  accessModes: 
  - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
```
```yml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc3
spec:
  accessModes: 
  - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
```

```bash

kubectl apply  -f  pvc.yaml

kubectl get pvc  -o wide
kubectl get pv   -o wide
```

创建pod
```yml
#pod1.yaml

apiVersion: v1
kind: Pod
metadata:
  name: pod1
spec:
  containers:
  - name: busybox
    image: busybox:1.38
    command: ["/bin/sh","-c","while true;do echo pod1 >> /data/out.txt sleep 10; done;"]
    volumeMounts:
    - name: volume
      mountPath: /data
  volumes:
    - name: volume
      persistentVolumeClaim:
        claimName: pvc1
        readOnly: false
---
apiVersion: v1
kind: Pod
metadata:
  name: pod2
spec:
  containers:
  - name: busybox
    image: busybox:1.38
    command: ["/bin/sh","-c","while true;do echo pod2 >> /data/out.txt; sleep 10; done;"]
    volumeMounts:
    - name: volume
      mountPath: /data
  volumes:
    - name: volume
      persistentVolumeClaim:
        claimName: pvc2
        readOnly: false
```

```bash
kubectl apply  -f  pod1.yaml

more  /data/nfs/pv1/out.txt

more  /data/nfs/pv2/out.txt

```



### 附言


🫪上面的配置老是要手动改，麻烦死了

？helm   、 GitOps yaml  、ConfigMap