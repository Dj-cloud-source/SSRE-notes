
### `access.log` 

```bash

cd  /var/log/nginx
tail -n 10 access.log

192.168.60.133 - - [11/Aug/2026:10:16:22 +0800] "HEAD / HTTP/1.1" 200 0 "-" "curl/7.76.1" "-"

```

#### 日志字段解析：

`192.168.60.133`   客户端IP
`- - `  远程登录于HTTP认证用户名占位符（少用）
`[11/Aug/2026:10:16:22 +0800]`  时间与时区


`HEAD` 请求方法
`/`       URI
`HTTP/1.1` HTTP版本

`200` HTTP状态码

`0` 返回给客户端的数据大小（这里0字节，意思是没有返回网页正文，状态是正常的

`-`  来源页面。如果是从某个网页点进来的，就会显示

`curl/7.76.1`  客户端工具（可以是浏览器

`-`   X-Forwarded-For 代理链（这里没经过代理，直接访问，所以为空





#### 状态码

1. 2xx 成功
		1. 200 成功
		2. 201 创建成功
		3. 204 成功但无响应
	
	
2. 3xx 重定向 
		1. 301 永久重定向
		2. 302 临时重定向
		3. 304 缓存有效
	
	
3. 4xx 客户端有问题
		1. 400 请求格式错误
		2. ==401 没登录、需要身份认证==
		3. ==403 无权限==
		4. ==404 资源不存在==
		5. 405 请求方法不允许
		6. 429 请求过多



4. 5xx 服务器有问题
		1. ==500 服务器懵了== 
		2. ==502 Nginx连接后端失败==
		3. ==503 服务不可用==
		4. ==504 等待后端响应超时== 





### `error.log` 


1. `connect() failed  Connection  refused`  后端连接失败，因为对方端口拒绝连接
2. `upstream timed out`  后端响应超时
3. `Permission denied`  权限问题
4. `No such file or directory` 文件不存在
5. `host not found in upstream` 域名解析失败





#### 502 Bad Gateway 排障

502 →Nginx 连接后端失败


```bash
tail -n 50 /var/log/nginx/error.log



nginx -T | grep -n "proxy_pass"
# 出现 proxy_pass http://ip111:8080;


curl -v http://ip111:8080/
```
> Connection refused →端口没监听、服务没启动、IP端口写错
> Connection timed out →网络、防火墙、安全策略、路由


```bash

ss -nltp | grep   :端口
```
>若什么都没有 →后端服务没起来
```bash
systemctl status  <后端服务>
ps  -ef |grep <进程名>
```


```bash
firewall-cmd  --list-port
getenforce
```








#### 504 Gateway Timeout
503 等待后端响应超时

Nginx把请求送到后端了，但后端在规定时间内没给出响应

```bash
tail -n 50 /var/log/nginx/error.log
```

`upstream timed out`                     后端超时
`while reading response header`  Nginx当时正在等待后端响应头
`upstream`                                      出问题的是代理后端的这一段


绕过Nginx，直接请求后端
```bash
#假设 后端是192.168.60.20:8080

curl -v  http://192.168.60.20:8080/api/user

curl -o /dev/null -s -w 'time_total: %{time_total}\n' http://192.168.60.20:8080/api/user
```


查后端为什么慢
```bash
top
free -h
ss -s
journalctl -u   <后端服务>
```

最后查配置文件、业务需求







#### 403 Forbidden

看
文件权限、SELinux
allow / deny


#### 404 Not Found

URI 写错
location 匹配到哪里
文件是否存在
`proxy_pass`  后端返回的404





### 慢请求与性能定位
>这个一般写在配置文件里



”网站为什么这么慢？“


1. `$request_time`  
2. `$upstream_response_time` 



`$request_time` ：从收到请求到返回给客户响应，整体花了多久

`$upstream_response_time` ：后端处理并返回花了多久


request很高，response也很高  →可以判断出是后端慢

比如：
- 数据库慢查询
- 接口处理慢
- 第三方接口慢
- 后端CPU高

所以可以继续往后端查

不过一般都是写在配置文件
```nginx
log_format main '$remote_addr - $remote_user [$time_local] "$request" '
				'$status $body_bytes_sent "$http_referer" '
				'"$http_user_agent" "$http_x_forwarded_for"'
				'request_time=$request_time '
				'upstream_time=$upstream_response_time';
```
