
Redis运维
1.  redis.conf  配置管理
	-  `redis-cli config GET *`
	- 控制监听地址+密码+防火墙+安全组
	- port
	- 设置密码    `requirepass   abc123456` 

3. 内存管理
	1. `maxmemory  4gb`
	2. 淘汰策略 `redis-cli  config get maxmemory-policy`
		1. **删除最少使用（least recentyl used）的key   `maxmemory-policy allkeys-lru` 
	3. Bigkey  
		- `redis-cli  --bigkeys` 
		- `redis-cli memmory usage key` 
		- 解决：拆分，分片存储
	4. 内存碎片
4. 热点问题
	1. hotkey
		 -  key 拆分
	2. 大流量排查
5. 性能分析
	1. slowlog
		- `slowlog get`    `slowlog len` 
	2. monitor
	3. latency
6. 故障排查
	1. redis连接不上
	2. 延迟升高
			1. 看cpu `top` `htop` 
			2. `redis-cli  info`
			3. 查看慢查询 `slowlog get` 
			4. 检查大key
	3. 内存爆满
		1. 查看内存 `redis-cli info memory`
		2. 找大key   `redis-cli  --bigkey`
		3. 设置淘汰策略 `maxmemory-policy allkey-lru` 
	4. 主从异常



