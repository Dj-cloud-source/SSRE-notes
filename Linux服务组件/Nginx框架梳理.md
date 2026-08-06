
## 框架梳理

1. 基础运维
	1. 安装/卸载
	2. `nginx -t`                  检查配置文件语法有无错误
	3. `nginx -s  reload`  重载配置
	4. `systemctl  start  nginx`
	5. `systemctl  restart  nginx` 
	6. `systemctl  status  nginx`
	7. `journalctl -u nginx`

2. 配置文件
	1. /etc/nginx/nginx.conf
	2. http {}
	3. server {}
	4. location {}
	5. include 


3. server 块
	1. listen
	2. server_name
	3. root
	4. index / try_files
	5. location 匹配规则
	6. 静态资源
	7. 虚拟主机


4. ==反向代理== 
	1. proxy_pass
	2. Header 请求头转发
	3. Host 主机头
	4. proxy_set_header
	5. X-Real-IP 客户端真实 IP
	6. X-Forwarded-For 代理链 IP
	7. Buffer 缓冲
	8. Timeout 超时
	9. WebSocket 代理



5. ==负载均衡== upstream
	1. round-robin  轮询
	2. weight
	3. least_conn 
	4. ip_hash
	5. keepalive 长连接
	6. max_fails
	7. fail_timeout


6. HTTPS
	1. TLS / SSL
	2. SSL证书、私钥
	3. listen 443 ssl
	4. HTTP > HTTPS 跳转
	5. Cipher 加密套件



7. 流量控制（访问控制）
	1. limit_req
	2. limit_conn
	3. burst
	4. allow / deny 




8. 缓存
	1. proxy_cache 代理缓存
	2. Cache Key 缓存键
	3. Cache Zone 缓存区
	4. TTL 缓存有效期
	5. 缓存绕过/清理



9. ==日志与排障== 
	1. access.log
	2. error.log
	3. request_time 请求耗时
	4. 403/404/502/504
	5. curl
	6. ss
	7. tail  -f  | grep 

10. 安全
	1. 隐藏Nginx版本
	2. 请求大小限制
	3. IP黑白名单
	4. HTTPS / TLS
	5. 基础访问控制





## Nginx 标准配置模版解析

```nginx
#/etc/nginx/nginx.conf

#最外层是Main（全局配置），控制Nginx启动后的进程行为和运行身份


user nginx;              
worker_processes   auto;  #进程数
pid    /run/nginx.pid;    #主进程PID存放路径
worker_rlimit_nofile  65535;



#events层决定Nginx如何处理网络连接
events{
	worker_connections  10240;  #单个进程允许的最大并发连接数
	use epoll;
	multi_accept  on;  #开启可提高高吞吐

}


#HTTP层，定义HTTP服务通用的基础配置
http{
	#include引入其他配置文件,把其他配置文件加载进来
	include     /etc/nginx/conf.d/星.conf;
	include     /etc/nginx/default.d/星.conf;
	include     /etc/nginx/mime.types;
	
	default_type  application/octet-stream;
	
	#日志格式
	log_format main '$remote_addr - $remote_user [$time_local] "$request" ';
	
	#日志访问路径
	access_log  /var/log/nginx/access.log;
	
	sendfile    on;
	tcp_nopush on;
	
	#长连接超时时间
	keepalive_timeout  65;
	
	#是否开始实时压缩传输
	gzip  on;
	
	
	
	#负载均衡。定义一组后端服务器，供 proxy_pass 调用
	upstream  backend_servers{
	#负载均衡算法
	least_conn;
	#ip_hash;
	
	
	#定义后端IP和端口
	server 127.0.0.1:8080  weight=5;
	server 127.0.0.1:8081  weight=3;
	}
	
	
	#server层（虚拟主机），定义一个独立的网站或服务，主要基于 IP、端口和域名 区分。
	server{
		
		#监听端口
		listen  443 ssl;
		
		#匹配的域名
		server_name  www.qq.com;
		
		#站点的根目录路径。本地硬盘文件
		#如果下面的local又写一遍，会把这里的覆盖了
		root /var/www/html;
		
		#默认首页文件
		index index.html;
		
		#证书和私钥路径
		ssl_certificate  cert.pem;
		
		
		
		
		
		#location层（URL路径），是最关键的字段。
		#可以有多个location，Nginx会根据匹配规则选择最合适的一条
		#根据请求URL匹配规则，执行静态文件返回或反向代理
		
		#匹配URL路径
		location = / {
			root   /usr/share/nginx/html;
			index  index.html;
		}
		
		#
		location ~* \.(gif|jpg|png)${
			root /var/www/images;
			
			#浏览器缓存30天
			expires  30d;   
		}
		
		#API 转发
		location  /api/ {
			#proxy_pass 是反向代理的核心。这个字段将请求转发给后端服务器
			proxy_pass http://backend_servers; 
			
			proxy_set_header  Host $host;
			proxy_set_header X-Real-IP  $remote_addr;
			proxy_http_version  2.0;
			
		}
		#
		location  / {
			root  /var/www/spa;
			try_files $uri  $uri/  /index.html; #
		}
		
		
	}
	
	
}

```



```nginx

server {
	#监听端口。监听哪个端口取决于是否是开HTTPS。80是HTTP，443是HTTPS
	#listen 443;
	listen 80;
	
	
	
	#绑定哪个网址
	server_name www.qq.com;
	
	
	
	#网站文件放哪个文件夹（根目录）
	root  /var/www/html;
	
	#默认首页
	index index.html  index.htm;
	
	
	#URL路由
	#两种常见场景
	#1想跑静态网页
	location / {
		try_files $uri  $uri/   /index.html;
	}
	
	#2想把请求发给后端的Java/Python程序
	#所有以 /api/ 开头的请求，都丢给某个ip的8080端口程序去处理
	location /api/ {
		proxy_pass http:// ip :8080;
	}
}

```
