

完成[#新机初始化](SSRE/Linux 基础/新机初始化)后


```bash
#master
vim /etc/hosts


192.168.60.133  master1
192.168.60.134  node01
192.168.60.135  node02

```

```bash
scp /etc/hosts  root@node01:/etc/hosts
scp /etc/hosts  root@node02:/etc/hosts
```

多执行模式
### 所有机器一起
```bash

sed -e 's|^mirrorlist=|#mirrorlist=|g' \
    -e 's|^#baseurl=http://dl.rockylinux.org/$contentdir|baseurl=https://mirrors.aliyun.com/rockylinux|g' \
    -i.bak \
    /etc/yum.repos.d/[Rr]ocky*.repo
    


  
dnf makecache 

systemctl stop firewalld
systemctl disable firewalld

yum -y install iptables-services

systemctl start iptables
iptables -F
systemctl enable iptables
service iptables save

setenforce 0

sed -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config
grubby --update-kernel ALL --args selinux=0

timedatectl set-timezone Asia/Shanghai

swapoff -a
sed -i 's:/dev/mapper/rl-swap:#/dev/mapper/rl-swap:g' /etc/fstab

yum install -y ipvsadm

echo 'net.ipv4.ip_forward=1' >> /etc/sysctl.conf
sysctl -p

yum install -y epel-release
yum install -y bridge-utils


modprobe br_netfilter
echo 'br_netfilter' >> /etc/modules-load.d/bridge.conf
echo 'net.bridge.bridge-nf-call-iptables=1' >> /etc/sysctl.conf
echo 'net.bridge.bridge-nf-call-ip6tables=1' >> /etc/sysctl.conf
sysctl -p


sudo dnf config-manager --add-repo https://mirrors.ustc.edu.cn/docker-ce/linux/centos/docker-ce.repo
cd /etc/yum.repos.d

sed -i 's#download.docker.com#mirrors.ustc.edu.cn/docker-ce#g' docker-ce.repo

yum -y install docker-ce

```


```bash

cat > /etc/docker/daemon.json <<EOF
{
  "data-root": "/data/docker",
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m",
    "max-file": "100"
  },
  "registry-mirrors": [
    "https://docker.1ms.run",
    "https://docker.1panel.live",
    "https://docker.xuanyuan.me",
    "https://dockerproxy.net",
    "https://docker.fast360.cn",
    "https://cloudlayer.icu",
    "https://docker-registry.nmqu.com",
    "https://hub.amingg.com",
    "https://docker.amingg.com",
    "https://docker.hlmirror.com"
  ]
}
EOF




mkdir -p /etc/systemd/system/docker.service.d

systemctl daemon-reload && systemctl restart docker && systemctl enable docker


wget http://file.eagleslab.com:8889/pkg/cri-dockerd-0.3.9.amd64.tgz  

tar -zxvf cri-dockerd-0.3.9.amd64.tgz

cp cri-dockerd/cri-dockerd /usr/bin/

chmod +x /usr/bin/cri-dockerd




cat <<"EOF" > /usr/lib/systemd/system/cri-docker.service
[Unit]
Description=CRI Interface for Docker Application Container Engine
Documentation=https://docs.mirantis.com
After=network-online.target firewalld.service docker.service
Wants=network-online.target
Requires=cri-docker.socket
[Service]
Type=notify
ExecStart=/usr/bin/cri-dockerd --network-plugin=cni --pod-infra-container-image=registry.aliyuncs.com/google_containers/pause:3.8
ExecReload=/bin/kill -s HUP $MAINPID
TimeoutSec=0
RestartSec=2
Restart=always
StartLimitBurst=3
StartLimitInterval=60s
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
TasksMax=infinity
Delegate=yes
KillMode=process
[Install]
WantedBy=multi-user.target
EOF


cat <<"EOF" > /usr/lib/systemd/system/cri-docker.socket
[Unit]
Description=CRI Docker Socket for the API
PartOf=cri-docker.service
[Socket]
ListenStream=%t/cri-dockerd.sock
SocketMode=0660
SocketUser=root
SocketGroup=docker
[Install]
WantedBy=sockets.target
EOF





systemctl daemon-reload
systemctl enable cri-docker
systemctl start cri-docker
systemctl is-active cri-docker





wget http://file.eagleslab.com:8889/pkg/kubernetes-1.29.2-150500.1.1.tar.gz

tar -zxvf kubernetes-1.29.2-150500.1.1.tar.gz

cd kubernetes-1.29.2-150500.1.1

 yum install -y * 



systemctl enable kubelet.service

```




### 接下来只有主节点


注意ip地址！！

```bash

kubeadm init --apiserver-advertise-address=192.168.60.133 --image-repository registry.aliyuncs.com/google_containers --kubernetes-version 1.29.2 --service-cidr=10.10.0.0/12 --pod-network-cidr=10.244.0.0/16 --ignore-preflight-errors=all --cri-socket unix:///var/run/cri-dockerd.sock

```

一定保存这一串！！ 

```
kubeadm join 192.168.60.133:6443 --token wojwpk.9jepxnk5vxdleyc3 --discovery-token-ca-cert-hash sha256:f0c637a443beda7191fe32c8e9cbdc624cdf498c773bbdc4655568da2b046517
```


执行以下命令！！
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

```



### 然后是node加入集群


将上面的join复制下来，然后后面加上
```
 --cri-socket unix:///var/run/cri-dockerd.sock
```

```bash

kubeadm join 192.168.60.133:6443 --token wojwpk.9jepxnk5vxdleyc3 --discovery-token-ca-cert-hash sha256:f0c637a443beda7191fe32c8e9cbdc624cdf498c773bbdc4655568da2b046517  --cri-socket unix:///var/run/cri-dockerd.sock


```



### 三个一起执行

```bash
cd 

wget http://file.eagleslab.com:8889/pkg/calico-images.tar.gz 

tar -zxvf calico-images.tar.gz 

cd calico-images/


docker load -i calico-cni-v3.26.3.tar
docker load -i calico-kube-controllers-v3.26.3.tar
docker load -i calico-node-v3.26.3.tar
docker load -i calico-typha-v3.26.3.tar

```


### 换成只有master

```bash

wget http://file.eagleslab.com:8889/pkg/calico-typha.yaml

kubectl apply -f calico-typha.yaml

kubectl get pod -A

kubectl get node

```


如果ready了，就赶紧关机，拍摄快照