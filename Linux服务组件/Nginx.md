>好像很多模块需要编译安装才能支持。。
## 下载

### yum安装
```bash
yum install -y yum-utils

mkdir -pv /etc/yum.repos.d
vim /etc/yum.repos.d/nginx.repo

[nginx-stable]
name=nginx stable repo
baseurl=https://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true

[nginx-mainline]
name=nginx mainline repo
baseurl=https://nginx.org/packages/mainline/centos/$releasever/$basearch/
gpgcheck=1
enabled=0
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true



yum install -y nginx
systemctl disable --now firewalld
setenforce 0
systemctl enable --now nginx

```


### 编译安装
```bash

cat /etc/os-release

uname -m

gcc --version

yum install -y gcc make
#如果下载太慢，就换源

gcc --version
make --version
curl -I https://nginx.org/

cd /usr/local/src
curl -O https://nginx.org/download/nginx-1.28.0.tar.gz
ls -lh nginx-1.28.0.tar.gz
yum install -y tar

tar -zxvf nginx-1.28.0.tar.gz

cd nginx-1.28.0

```

```bash
#检查依赖、外部库
./configure


#安装依赖
yum install -y pcre2-devel zlib-devel openssl-devel  

./configure
……

Configuration summary
  + using system PCRE2 library
  + OpenSSL library is not used
  + using system zlib library

  nginx path prefix: "/usr/local/nginx"
  nginx binary file: "/usr/local/nginx/sbin/nginx"
  nginx modules path: "/usr/local/nginx/modules"
  nginx configuration prefix: "/usr/local/nginx/conf"
  nginx configuration file: "/usr/local/nginx/conf/nginx.conf"
  nginx pid file: "/usr/local/nginx/logs/nginx.pid"
  nginx error log file: "/usr/local/nginx/logs/error.log"
  nginx http access log file: "/usr/local/nginx/logs/access.log"
  nginx http client request body temporary files: "client_body_temp"
  nginx http proxy temporary files: "proxy_temp"
  nginx http fastcgi temporary files: "fastcgi_temp"
  nginx http uwsgi temporary files: "uwsgi_temp"
  nginx http scgi temporary files: "scgi_temp"

```
生产中怎么加模块呢[^1]
```bash
#但是呢，openssl  is not used。这不是错误，而是没有告诉Nginx编译SSL模块。
#why？ 因为源码编译时，功能是靠参数决定的，刚才./configure 只是按默认功能编译，如过想加入#HTTPS，就像下面这样。



./configure \
--with-http_ssl_module
……
checking for zlib library ... found
creating objs/Makefile

Configuration summary
  + using system PCRE2 library
  + using system OpenSSL library
  + using system zlib library

  nginx path prefix: "/usr/local/nginx"
  nginx binary file: "/usr/local/nginx/sbin/nginx"
  
  #这里是源码安装后的模块目录
  nginx modules path: "/usr/local/nginx/modules"
  
  nginx configuration prefix: "/usr/local/nginx/conf"
  nginx configuration file: "/usr/local/nginx/conf/nginx.conf"
  nginx pid file: "/usr/local/nginx/logs/nginx.pid"
  nginx error log file: "/usr/local/nginx/logs/error.log"
  nginx http access log file: "/usr/local/nginx/logs/access.log"
  nginx http client request body temporary files: "client_body_temp"
  nginx http proxy temporary files: "proxy_temp"
  nginx http fastcgi temporary files: "fastcgi_temp"
  nginx http uwsgi temporary files: "uwsgi_temp"
  nginx http scgi temporary files: "scgi_temp"



make
ls -lh objs/nginx


make install
#把已经编译好的东西复制到安装目录

ls /usr/local/nginx


/usr/local/
│
├── src/
│   └── nginx-1.28.0       # 源码（以后可以保留）
│
└── nginx/                 # 安装后的程序
    ├── sbin/
    │   └── nginx          # 可执行文件
    │
    ├── conf/
    │   └── nginx.conf     # 配置文件
    │
    ├── html/
    │   └── index.html     # 默认网页
    │
    └── logs/





/usr/local/nginx/sbin/nginx -v


/usr/local/nginx/sbin/nginx -V

nginx version: nginx/1.28.0
built by gcc 11.5.0 20240719 (Red Hat 11.5.0-14) (GCC)
built with OpenSSL 3.5.5 27 Jan 2026
TLS SNI support enabled
configure arguments: --with-http_ssl_module



/usr/local/nginx/sbin/nginx -t

nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful



#启动！
/usr/local/nginx/sbin/nginx

ps aux | grep nginx



#下一步，使用systemctl管理
#如果上面启动了Nginx进程，就关掉它
/usr/local/nginx/sbin/nginx -s quit


vim /usr/lib/systemd/system/nginx.service
```

```

[Unit]
Description=Nginx Web Server
After=network.target

[Service]
Type=forking

ExecStart=/usr/local/nginx/sbin/nginx
ExecReload=/usr/local/nginx/sbin/nginx -s reload
ExecStop=/usr/local/nginx/sbin/nginx -s quit

PrivateTmp=true

[Install]
WantedBy=multi-user.target

```

```bash
systemctl daemon-reload
systemctl start nginx
systemctl status nginx
systemctl enable nginx
```

>ok，你已经会了🥴



这里补充一下yum安装的和源码编译安装，配置文件的区别

yum
- 主配置文件在   `/etc/nginx/nginx.conf`  ，遵循Linux标准文件系统层次，配置全在 `/etc/` 下
 

源码编译安装
- 所有所有的文件（包括二进制、日志、配置）都在 `/usr/local/nginx`  下面
`













## 演示

### 建立网站
```bash

mkdir -pv /data/nginx/site0{1..3}/

mkdir -pv /data/nginx/status


echo "Running 3 websites" > /data/nginx/status/index.html
echo "Hello PC Website!" > /data/nginx/site01/index.html
echo "Hello Mobile Website!" > /data/nginx/site02/index.html
echo "Hello Local Test Website!" > /data/nginx/site03/index.html






vim /etc/nginx/conf.d//vhost.conf

#PC
server{
    listen 8001;
    location / {
        root /data/nginx/site01;

    }
}

#mobile
server{
    listen 80;
    server_name m.test.com;
    location / {
        root  /data/nginx/site02;

    }

}

#Test
server{
    listen 127.0.0.1:8003;
    location / {
        root /data/nginx/site03;

    }
    location /status {
        root /data/nginx;
    }

}





vim /etc/nginx/nginx.conf

http{
	……
   include             /etc/nginx/conf.d//vhost.conf;
   ……
}




nginx -t
nginx -s reload
ss -nlt

curl 0.0.0.0:8001
curl  127.0.0.1:8003
curl  127.0.0.1:80
curl  127.0.0.1:8003/status
curl -I  127.0.0.1:8003/status
curl -L  127.0.0.1:8003/status

```

### 访问控制

```bash

server{
    listen 8001;
    location / {
        root /data/nginx/site01;
        deny 192.168.60.130;

    }
}

nginx -t
#配置文件记得写分号啊
nginx -s reload

# 192.168.60.130
[root@Rocky-test ~] curl -I 192.168.60.134:8001

HTTP/1.1 403 Forbidden
Server: nginx/1.20.1
Date: Thu, 06 Aug 2026 11:19:32 GMT
Content-Type: text/html
Content-Length: 153
Connection: keep-alive


#192.168.60.133
[root@master1 ~]  curl -I 192.168.60.134:8001

HTTP/1.1 200 OK
Server: nginx/1.20.1
Date: Thu, 06 Aug 2026 11:19:44 GMT
Content-Type: text/html
Content-Length: 18
Last-Modified: Thu, 06 Aug 2026 08:15:41 GMT
Connection: keep-alive
ETag: "6a7442ad-12"
Accept-Ranges: bytes







#注意 deny 的位置，在上面和在下面是不一样的
server{
    listen 8001;
    location / {
        root /data/nginx/site01;
        allow 192.168.60.130;
        deny all;

    }
}



[root@Rocky-test ~] curl -I 192.168.60.134:8001
HTTP/1.1 200 OK
Server: nginx/1.20.1
Date: Thu, 06 Aug 2026 12:00:56 GMT
Content-Type: text/html
Content-Length: 18
Last-Modified: Thu, 06 Aug 2026 08:15:41 GMT
Connection: keep-alive
ETag: "6a7442ad-12"
Accept-Ranges: bytes


[root@master1 ~]$ curl -I 192.168.60.134:8001
HTTP/1.1 403 Forbidden
Server: nginx/1.20.1
Date: Thu, 06 Aug 2026 12:02:51 GMT
Content-Type: text/html
Content-Length: 153
Connection: keep-alive







yum install -y httpd
htpasswd -c /etc/nginx/.htpasswd admin #只能有一个管理员
echo "site04" > /data/nginx/site04/admin/index.html
vim /etc/nginx/conf.d/site04.conf


server{
    listen 8004;
    if ($http_user_agent ~* bot ){
        return 403;
    }
    location /admin {
        root /data/nginx/site04;
        auth_basic 'tip:input password ';
        auth_basic_user_file /etc/nginx/.htpasswd;
        allow 127.0.0.1;
        deny all;
    }
}





nginx -t
nginx -s reload
curl -I -u admin:123456 http://127.0.0.1:8004/admin/

[root@node01 ~]$ curl -I -u admin:123456 http://127.0.0.1:8004/admin/
HTTP/1.1 200 OK
Server: nginx/1.20.1
Date: Thu, 06 Aug 2026 12:32:19 GMT
Content-Type: text/html
Content-Length: 7
Last-Modified: Thu, 06 Aug 2026 12:16:33 GMT
Connection: keep-alive
ETag: "6a747b21-7"
Accept-Ranges: bytes





curl -I -u admin:1 http://127.0.0.1:8004/admin/
[root@node01 ~]$ curl -I -u admin:1 http://127.0.0.1:8004/admin/
HTTP/1.1 401 Unauthorized
Server: nginx/1.20.1
Date: Thu, 06 Aug 2026 12:30:39 GMT
Content-Type: text/html
Content-Length: 179
Connection: keep-alive
WWW-Authenticate: Basic realm="tip:input password "


```




### Nginx location语法规则


1. 无修饰符（普通前缀）：最长的匹配被记下，但若后续有正则匹配成功，则覆盖它
2. `/`：匹配所有请求
3. `=` ：精确匹配，完全一致才匹配，匹配后立即停止搜索
4. `^~`：前缀匹配，匹配到指定开头，停止后续正则匹配
5. `~`：区分大小写的正则匹配，`~*` 不区分大小写。按配置文件顺序匹配




### HTTPS与证书

> HTTP跳转HTTPS
```nginx
server{
	listen 80;
	server_name www.qq.com;
	
	return 301 https://$host$request_uri;
}
```

>HTTPS服务
```nginx
server {
	listen 443 ssl;
	server_name www.qq.com;
	
	#证书位置
	ssl_certificate  /etc/nginx/ssl/fullchain.pem;
	
	#私钥（一般放HSM，现阶段不必研究）
	ssl_certificate_key  /etc/nginx/ssl/privkey.pem;
	
	location /{
	……
	}
}
```
```bash
#查看证书、测试tls
openssl x509 -in server.crt  -text  -noout
openssl s_client  -connect example.com:443
nginx -t
```




























### Epoll 、IO多路复用

（8月10日，待写








[^1]: 重新编译！
	不是重装系统，也不是删掉Nginx 。
	而是保留原来的配置和源码  → 重新执行 `./configure` 加参数 →重新生成二进制文件 → 替换Nginx可执行文件
	
	比如半年后老板说加个HTTPS
	```bash
	#进入原来的源码目录
	cd /usr/local/src/nginx-1.28.0
	
	#查看以前编译参数，记到小本本里，很重要！
	nginx -V
	
	#重新执行 ./configure
	
	#原来的参数保留，在最后加上
	
	--with-新模块
	--add-module=/usr/local/src/ngx_xxx别人的模块_module
	--add-dynamic-module=/usr/local/src/xxx
	
	#然后重新编译。这里make会重新编译变化的部分
	make
	
	#备份旧Nginx
	cp /usr/local/nginx/sbin/nginx \
    /usr/local/nginx/sbin/nginx.bak
    
    #替换二进制文件
    cp objs/nginx /usr/local/nginx/sbin/nginx
    #检查
    /usr/local/nginx/sbin/nginx -V
    #上线
    nginx -t
	nginx -s reload
	
	#
	#这就是一次小小的，软件从诞生到上线的过程
	#现在更可能是 Dockerfile → 重新构建镜像 → 测试 →发布新 pod →滚动更新
	






	```

