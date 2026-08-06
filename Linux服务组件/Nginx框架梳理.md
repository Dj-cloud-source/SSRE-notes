


1. 基础运维
	1. 安装/卸载
	2. nginx -t               检查配置
	3. nginx -s  reload  重载配置
	4. systemctl  start  nginx
	5. systemctl  restart  nginx 
	6. systemctl  status  nginx
	7. journalctl -u nginx

2. 配置文件
	1. /etc/nginx/nginx.conf
	2. http {}
	3. server {}
	4. location {}
	5. include 


3. 静态web服务
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



5. ==负载均衡==
	1. upstream 后端服务器组
	2. round-robin  轮询
	3. weight
	4. least_conn 
	5. ip_hash
	6. keepalive 长连接
	7. max_fails
	8. fail_timeout


6. HTTPS
	1. TLS / SSL
	2. SSL证书、私钥
	3. listen 443 ssl
	4. HTTP > HTTPS 跳转
	5. Cipher 加密套件



7. 流量控制
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



