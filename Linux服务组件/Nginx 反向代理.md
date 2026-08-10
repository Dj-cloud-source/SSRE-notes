
普通访问，是用户直接访问服务器，

而反向代理，
```
	用户
	 ↓ 访问 qq.com
	 ↓
	 Nginx
	 ↓
	 ↓ 请求转发
	 ↓
	后端服务
	Java:8080
	Python:5000
	Node:3000 
```

为什么需要反向代理？
1. 暴露端口不美观
2. 后端不应该暴露（？
3. 统一入口




## 配置文件字段解析


### `proxy_pass` 把当前请求转发给目标服务器

```nginx

server {

	listen 80;
	server_name   qq.com;
	
	location /api/ {
		proxy_pass http://127.0.0.1:8080;
		
	}
}
```

> 用户访问 http://www.qq.com/api/user ，Nginx收到 /api/user ，然后转发到127.0.0.1:8080 




###  `proxy_set_header`  

用户请求 
```bash
GET /api/user
Host:  qq.com
```

但是到了Nginx，会默认转发
```bash
GET /api/user
Host: 127.0.0.1
```
导致后端不知道请求原来的域名和用户ip
所以有了  `proxy_set_header` 

```nginx
……

location /api/ {
	proxy_set http://127.0.0.1:8080;
	
	proxy_set_header  Host  $host;
	proxy_set_header  X-Real_IP  $remote_addr;
	proxy_set_header  X-Forwarded-For  $proxy_add_x_forwarded_for;
	
}
```

`proxy_set_header  Host  $host` ：告诉后端，用户访问的原始域名。
`proxy_set_header X-Real-IP $remote_addr`  ：告诉后端，用户真实IP是多少。
`proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for` ：记录代理链，方便日志分析（比如用户IP → CDN加速 →Nginx → 后端）。




### Timeout 超时

Nginx不能无限等待后端，所以需要超时时间
常见三个字段
`proxy_connect_timeout` 
`proxy_send_timeout` 
`proxy_read_timeout`



1. `proxy_connect_timeout` 
	后端网络异常，无法建立连接，超过时间就不等了


2. `proxy_read_timeout` 
	Nginx → Java → 执行数据库查询 →很慢 
	连接成功，但是后端在规定时间内一直没有响应数据，Nginx会认为后端超时
```
实际排障比较常见。比如遇到
504 Gateway Timeout

看到504，就要想到，Nginx还活着，是不是后端太慢
```


3. `proxy_send_timeout` 
建立连接后，向后端发送**完整数据**（包括请求头和请求体）的最大时间






### Buffer 代理缓冲

当后端返回数据，Nginx是否收到一点点就马上发给用户？

假设返回了一个10 MB 的大 JSON，没有缓冲，后端一点一点的传，可能会导致
- 连接被占用很久
- 慢客户拖累后端
- 网络波动影响后端


所以我们可能需要一个缓冲
后端快速返回给 Nginx Buffer ，Nginx再慢慢发送给用户。

所以 Buffer 的作用是暂存后端响应数据，然后再发送给客户端




1. `proxy_buffering  on; `  开关
	常用于
		- WebSocket
		- 流式输出
		- Server-Sent Events


2. `proxy_buffers 8  16k; `
	缓冲区数量和大小


3. `proxy_buffer_size  16k` 
	第一个响应头缓冲。主要存 HTTP响应头 和 部分响应内容
	


运维排障
```
#响应头太大
upstream sent too big header

#可能需要
proxy_buffer_size 32k;
proxy_buffer  8  32k;
  
```




### WebSocket 代理

引入WebSocket前，先看普通 HTTP 请求。

普通 HTTP ，都是 客户端服务端 一次请求，一次响应。

但很多实时场景
- 游戏
- 在线客服
- 实时行情
- AI 对话流式输出

不希望一应一答，所以有了WebSocket

WebSocket 的特点是，建立连接以后，可以长期保持，并且双方都可以双向主动发送

```nginx
location /WS/ {
	proxy_pass  http://backend;
	
	proxy_http_version 1.1;
	
	proxy_set_header Upgrade $http_upgrade;
	
	proxy_set_header Connection  "upgrade";

}
```

