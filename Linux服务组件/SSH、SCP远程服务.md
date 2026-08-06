
## SSH

 命令 ：
 `ssh  username@ip`

指定端口
`ssh username@ip   -p 22` 

仅执行某个命令
`ssh username@ip    "cat  /etc/hosts" ` 



SCP远程文件传输

路径：`用户名@ip:路径` 

`scp  []   源路径   目标路径` 

本地🔜服务器
`scp    本机file        username@ip:/root/ ` 

`scp -r Project/       usename@ip:/root/` 



本地 🔙服务器
`scp  username@ip:/root/file         .`
`scp  -r  username@ip:/root/project      .`


指定SSH端口
`scp -P 2222  file      username@ip:/root` 


--- 



## 一台新机器到了，我们该做什么？

```
装系统
    ↓
安装 openssh-server
    ↓
启动 sshd
    ↓
配置公钥
    ↓
关闭密码登录
    ↓
Ansible 接管
```


1. 以root登陆控制台


2.配置主机名

`hostnamectl  set-hostname    name` 

检查  `hostname` 


3. 配置IP
```bash
#查看设备
nmcli device
nmcli  connection  show

#设置ip
#主要是ipv4address

nmcli con mod ens160 \
ipv4.addresses 192.168.60.131/24 \
ipv4.gateway 192.168.60.1 \
ipv4.dns 223.5.5.5 \
ipv4.method manual



nmcli con down  ens160
nmcli con up    ens160

ip a

#测试连通性
ping www.baidu.com

```



4. 安装SSH
```bash

yum install -y  openssh-server  vim


systemctl enable  --now sshd
systemctl status  sshd

```

5. 放行防火墙
```bash

firewall-cmd  --permanent  --add-service=ssh
firewall-cmd  --reload

firewall-cmd  --list-services

```


6. 本地生成密钥
```bash
#本机有了就不用了
ssh-keygen  -t ed25519  -C "your_email@ ? com"
```


7. 本地上传公钥至服务器
```bash
#本地电脑
ssh-copy-id    root@ip
```

```bash
#服务器

cat  .ssh/authorized_keys


# 编辑 /etc/ssh/sshd_config 文件，更改的配置项如下


vim /etc/ssh/sshd_config


PasswordAuthentication no
PubkeyAuthentication yes
PermitRootLogin prohibit-password





systemctl restart sshd

```

8. 测试ssh
```bash

ssh  root@ip

exit
```

9. 配置hosts
```bash
vim /etc/hosts

ip1   master
ip2   node2
……

```

10. 同步时间
```bash
timedatectl set-timezone Asia/Shanghai

timedatectl set-ntp true

timedatectl
```

11. 更新系统（新机器可以，但生产服务器不行）
```bash

yum update -y


yum install -y  wget curl git tar zip unzip  net-tools

```

12. 重启后验证
```bash

reboot

ssh ……
```





rsync
文件同步工具


```bash

yum install -y rsync

#本地人同步到服务器
rsync -av  ./     root@server:/opt/project/

#删除服务器多余文件（生产慎用）
rsync -avz --delete   ./dist/   root@server:/usr/share/nginx/html/


#先走一遍，查看将同步什么。确认没问题再真正同步
rsync -av --dry-run ./ root@server:/opt/project/



rsync -av -e "ssh -p 2222" ./ root@127.0.0.1:/root/
```

