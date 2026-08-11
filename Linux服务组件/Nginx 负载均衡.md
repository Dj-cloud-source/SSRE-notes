
反向代理是 Nginx对应一个后端，
而负载均衡是 Nginx对应一组后端。
核心字段是 `upstream` 



### `upstream` 定义后端服务区组

比如有三台Web服务器
```
192.168.1.10:8080
192.168.1.11:8080
192.168.1.12:8080
```

配置文件
```nginx

http {
	upstream backend{
		server 192.168.1.10:8080;
		server 192.168.1.10:8080:
		server 192.168.1.10:8080;
	}
	
	server {
		listen 80;
		location / {
			proxy_pass http://backend;
		}
	}

}
```


`upstream backend` ：相当于创建了一个后端服务器组，组名叫 `backend` 
`proxy_pass http://backend;` ：转给 `backend` 这一组机器，至于具体选哪台，让Nginx自己决定。

相当于一个代号






Nginx 默认算法：轮询
就s1s2s3s1s2s3

### weight：权重

权重是用来控制某台后端服务器相对分到多少请求
毕竟**后端性能不同** 


```nginx
upstream backend{
	server 192.168.1.10:8080 weight=4;
	server 192.168.1.11:8080 weight=2;
	server 192.168.1.12:8080 weight=1;
}
```

所以大概每个各会分得  4/7 , 2/7 , 1/7 

而且还可以和轮询结合，变成加权轮询





### `least_conn`：最少连接
一言以蔽之，这个就是将新请求优先交给当前活跃连接数较少的后端服务器

配置示例
```nginx
upstream backend {
	least_conn;
	
	server 192.168.1.10:8080;
	server 192.168.1.11:8080; 
	server 192.168.1.12:8080;
}
```


和轮询有什么区别？

轮询是，即使你在忙，但还是硬塞给你
最少连接是看“谁没有那么忙”。所以比较适合请求时间差异比较大的业务。



而且这个也可以和权重结合在一起。这时Nginx会结合权重考虑连接负载



### `ip_hash`  
根据客户端IP做哈希，让同一个客户端尽量（固定）落到同一台后端服务器

```nginx

upstream backend{
	ip_hash;
	
	server 192.168.1.10:8080;
	server 192.168.1.11:8080;
	server 192.168.1.12:8080;
}
```

所谓固定，就是Session / 回话保持
(Session和Cookie)

不过生产中，这个比较少用。更经常用的是无状态Token哈。

>“Nginx 可以通过 `ip_hash` 实现一定程度的会话保持，但生产系统通常更倾向于应用无状态化，把 Session 放到 Redis 等共享存储中，这样扩缩容和故障切换更加方便。”




### `keepalive` 长连接

首先，Nginx的代理链路是 ：客户端连接 → Nginx → upstream 连接 → 后端服务器 。
这里`keepalive`  长连接管的是 **Nginx  <–>  后端app服务器                

其次是为什么需要长连接？懂得都懂，TCP反复建立连接是有成本的

`keepalive 32;` 

不是 “最多允许32个用户连接” “最多32个并发连接”

而是 **每个 worker 进程为这个 upstream 保留的空闲 keepalive 连接数上限** 


```nginx

upstream backend{
	server 10.0.0.1:8080;
	server 10.0.0.2:8080;
	
	keepalive 32;
}


location / {
	
	proxy_pass http://backend;
	proxy_http_version 1.1;
	
	proxy_set_header Connection "";
}
```
>这样配置，可以复用Nginx与后端的连接，减少频繁建立/关闭连接的开销




### `max_fails` 与 `fail_timeout` 
后端失败处理：后端老是访问失败，Nginx可以暂时认为它不可用，先别往那里发请求


```nginx

upstream backend{
	server 10.0.0.1:8080  max_fails=3  fail_timeout=30s;
	server 10.0.0.2:8080  max_fails=3  fail_timeout=30s;
	
}
```
> 意思是：在30秒内达到3次失败，然后会暂时把它认定为 “不可用” 30秒，这期间，Nginx尽量不向它发请求。过了这30秒后，Nginx会重新尝试