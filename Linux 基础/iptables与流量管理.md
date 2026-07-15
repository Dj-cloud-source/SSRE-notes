[toc]



# 流量管理







$防火墙有 firewalld 和 iptables 。 所谓防火墙，也不过是我们定义内核中 netfilter 的规则，以此来过滤、修改、转发数据$ 









先[换源](  'yum安装命令与内网安装.md')

初体验

```bash
yum install -y epel-release
```

> **Extra Packages for Enterprise Linux** 这个是提供  拓展软件包资源

```bash
yum install -y nginx
systemctl start nginx
```

> nginx 是一个web服务器，一个**“高性能 HTTP 文件分发器”** 



* 然后在浏览器输入虚拟机的IP地址

![image-20260212150711524](image-20260212150711524.png)

* 可以看见访问测试通过



使用 iptables 进行访问控制

```bash
iptables -A INPUT -p tcp --dport 80 -j DROP
```

再刷新发现已经无法打开



![image-20260212150926796](image-20260212150926796.png)





查看刚才添加过的过滤规则

```bash
iptables -vnL INPUT --line-numbers
```



![image-20260212151235105](image-20260212151235105.png)



---



<!-- 五链四表 -->

## iptables 的组成



iptables 的设置有五小段

![高级用法](高级用法-17724502779932.png)



*  **`-t` ** [`表`]( https://ncloud.eagleslab.com/Linux%E5%9F%BA%E7%A1%80/%E9%98%B2%E7%81%AB%E5%A2%99%E4%B8%8Eselinux.html#%E4%BA%94%E9%93%BE) 
	
	* `filter`  ( 过滤。默认是 filter ，增删改查都可以加 `-t`  `表`来指定) 
	* `nat`  (修改数据包的来源、目的地和端口 )
	* `raw`  (确定是否对数据包进行状态追踪)
	* `mangle` 
	
	


* **`-A`** `链`  / **`-D`** `链` 
	* `	PERROUTING`  (进入路由之前)
	* `INPUT`  (入站的数据包)
	* `OUTPUT`  (出站)
	* `FORWARD`  (转发)
	* `POSTROUTING`  



* **`-s`** `ip来源` 
* **`-J`** `处理动作` 
	* `accept`  接受
	* `drop`  丢弃
	* `reject`  拒绝

 

* **`-p`** `数据包协议` 

	* tcp (Transmission Control Protocol)

		* --dport：指定目的端口

		* --sport：指定源端口

	* udp (User Datagram Protocol)
	
	* icmp 
	
		



## 1. 增加规则



* 屏蔽 `ping` 包

(ping包是icmp协议的)

```bash
iptables -A INPUT  -s 192.168.xx.xx  -p icmp  -j DROP  
```



![image-20260302200410943](image-20260302200410943.png)

> `DROP` 为丢弃数据包，不给予对方回应，对方会一直等待



```bash
iptables -A INPUT  -s 192.168.xx.xx  -p icmp  -j REJECT 
```

> `REJECT` 是拒绝对方，会将拒绝的数据包发给对方，对方就会知道我们拒绝的访问


![image-20260302201417167](image-20260302201417167.png)



---



* 屏蔽 sshd 服务

(sshd 是 tcp 协议的，端口号为22 )

```bash
iptables -A INPUT  -p tcp --dport 22  -j REJECT 
```



然后远程连接就中断了。重新恢复

```bash
iptables -D INPUT -p tcp --dport 22 -j REJECT
```





## 2. 查看规则

```bash
iptables -nL --line-numbers
```





## 3. 删除规则

先查看规则

- 通过 Chain 序号删除

```bash
iptables -D INPUT 1
```

* 像上面 sshd 一样，通过完整的规则删除
* 清空所有规则

```bash
iptables -F 
```





## 4. 修改规则

1. 通过 `iptables -D` 删除原有的规则后添加新的规则
2. 通过`iptables -R`可以对具体某一个num的规则进行修改

```bash
iptables -R INPUT 1 -p tcp --dport 8080 -j ACCEPT
```







## 5. 保存规则，持久化

```bash
yum install -y iptables-services 

service iptables save
```





---



## 自定义链

当我们在 `INPUT` 链中放了许多规则，httpd、sshd、nginx ，太多东西杂糅在一起，难以维护 。

可以自定义链，来给不同规则分类，方便管理

比如：我们自定义一个WEB_CHAIN链，专门管理跟web网站相关的规则策略



### 创建

```bash
iptables -t filter -N chain_name
```

### 改名

```bash
iptables -t filter -E  old_name  new_name  
```



### 引用自定义链

必须引用自定义链才能生效

```bash
iptables -t filter  -A INPUT  -p tcp   --dport 80    -j  chain_name
```

> 在 `INPUT` 链中添加了一条规则，访问本机80端口的tcp报文将会被这条规则跳转到chain_name链



### 在自定义链中添加规则

```bash
iptables -t filter  -A  chain_name  -p tcp  -j ACCEPT 
```



### 删除自定义链

```bash
iptables -t filter -F  chain_name
iptables -t filter -X  chain_name 
```







## 其他用法

筛选字符

```bash
yum -y install httpd
systemctl start httpd
systemctl enable httpd

echo "<h1>hello world</h1>" > /var/www/html/index.html
```



![image-20260303190420146](image-20260303190420146.png)

---



```bash
iptables -A OUTPUT -p tcp --sport 80 -m string --algo bm --string "world" -j REJECT
```

刷新浏览器发现打不开

```bash
echo "<h1>hello linux</h1>" > /var/www/html/index.html
```

刷新浏览器，可以打开



