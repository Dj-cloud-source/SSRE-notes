

1. URL 结构
	1. 协议：http、https
	2. 域名：www.baidu.com
	3. 端口：80/443
	4. 路径：/api/users
	5. 参数：?id=1&name=zhangsan


2. http报文
	1. 请求
		1. 请求行
		2. 请求头
			1. Host域名
			2. Content-Type 正文格式
				1. application/json
				2. text/html
				3. multipart/form-data
				4. application/octet-stream
			
			3. Content-Length 正文字节数
			4. Authorization 认证
			5. Cookie
			6. User-Agent
			7. Referer请求来源
			
		3. 空行
		4. 请求体
	
	2. 响应 
		1. 状态体
		2. 响应头
			1. Set-Cookie
			2. Location
		3. 空行
		4. 响应体
	
	
	3. 请求方法
		1. GET 查询
		2. POST 新增
		3. PUT 整体修改
		4. PATCH  修改
		5. DELETE 删除
		6. HEAD 只获取相应头
		7. OPTIONS 查询支持的方法




3. 状态码
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
		2. ==401 没登录==
		3. ==403 无权限==
		4. ==404 资源不存在==
		5. 405 请求方法不允许
		6. 429 请求过多



	4. 5xx 服务器有问题
		1. ==500 后端程序异常==
		2. ==502 代理连接后端失败==
		3. ==503 服务不可用==
		4. ==504 等待后端相应超时== 



4. 连接机制
	1. 短连接：一次请求建立一次tcp连接
	2. 长连接：一个tcp连接发送多个http请求
	3. Keep-Alive
		1. HTTP Keep-Alive
		2. TCP Keepalive
		3. Nginx upstream


5. 缓存机制
	1. 强缓存 Cache-Control
		1. max-age
		2. no-cache
		3. no-store
	
	2. 协商缓存 
		1. ETag / If-None-Match
		2. Last-Modified / If-Modified-Since
		3. 304 Not Modified



6. 代理与Nginx
	1. 正向代理
	2. 反向代理
	3. 负载均衡
	4. 虚拟主机
	5. 请求链路


7. 超时与故障排查
	1. 连接超时 proxy_connect_timeout
	2. 发送超时 proxy_send_timeout
	3. 读取超时 proxy_read_timeout
	
	
	4. 故障判断 
		1. 404 路径或资源问题
		2. 401 认证问题
		3. 403 权限问题
		4. 500 后端权限问题
		5. 502 Nginx 连不上后端
		6. 503 服务不可用
		7. 504 后端响应太慢
		8. 499 客户端主动断开
	
	
	5. 排查方向 
		1. DNS是否正常
		2. TCP端口是否可达
		3. HTTP状态码是什么
		4. Nginx日志是否报错
		5. 后端服务是否存活
		6. 请求在那一层超时




 8. HTTP 版本
	 1. HTTP/1.0 默认短连接
	 2. HTTP/1.1 长连接、Host、分块传输
	 3. ==HTTP/2  二进制分帧、多路复用、头部压缩（常用）==
	 4. HTTP/3 基于QUIC、基于UDP








