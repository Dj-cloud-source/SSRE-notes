

# firewall 防火墙



## 初体验

```bash
yum install -y httpd 
systemctl start httpd 
```

在浏览器中输入虚拟机 ip 地址，无法访问。因为 firewall 防火墙处于开启状态，把外部流量拒绝掉了





1. 修改 firewalld 策略。通过 *服务名* 开放（推荐），使得外部可以访问网站

```bash
firewall-cmd  --cmd=public  --add-service=http  
……
firewall-cmd  --cmd=public  --add-service=http  --permanent 
firewall-cmd  --reload 
```

或者按服务的端口号

```bash
firewall-cmd  --zone=public  --add-port=80/tcp  --permanent
firewall-cmd --reload
```



2. 删除规则,取消放行

```bash
firewall-cmd  --zone=public  --remove-service=http  --permanent 
firewall-cmd --reload
```



---



## `zone` ?

查看当前 zone 的规则

```bash
firewall-cmd --zone=public  --list-all
```





firewalld 将网络接口划分到不同的 zone ，**不同的 zone 有不同的限制程度**

* zone
	* public（不可信网络）
	* dmz（缓冲区）
	* home



端口转发

```bash
firewall-cmd --permanent --add-forward-port=port=6666:proto=tcp:toport=22
firewall-cmd --reload
```

之后可以在  cmd  上远程连接虚拟机







紧急模式。在紧急模式下，连远程连接和 ping 信号都用不了。

```bash
firewall-cmd --panic-on
……
firewall-cmd --panic-off
```



[富规则](https://ncloud.eagleslab.com/Linux%E5%9F%BA%E7%A1%80/%E9%98%B2%E7%81%AB%E5%A2%99%E4%B8%8Eselinux.html#firewalld%E5%AF%8C%E8%A7%84%E5%88%99)





