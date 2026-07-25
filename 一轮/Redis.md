
## 理解

之前讲过Mysql，说过Mysql是关系型数据库，但没有深入，这次稍微进阶一点


关系型数据库结构极其严谨，数据必须按预先定义好的 table 存放，有严格的 row 和 
column。

为什么叫关系二字？
就是因为它支持 table 和table 之间建立复杂关系（比如一对多、多对多、外键约束）

然后就是数据安全性
Mysql 存在磁盘上，读写速度较慢；它的核心任务是保障业务数据安全，重视数据一致性和可靠性。比如用户账号、订单、支付记录（和💰相关😄）



而Redis跑在内存中，速度贼快，但它的职责是缓冲流量，提速访问
常用于：热门商品缓存、验证码、登陆 Session、计数器、排行榜、分布式锁、消息队列（靠一些数据结构实现）（我觉得可以用kaflka和redis给南邮高并发的应用）


Redis原理
为什么快？
1. 不用磁盘而是内存（避免磁盘寻址、机械等待、IO等待）
2. 单线程模型。减少线程切换、锁竞争
3. IO多路复用


但 它 是非关系型数据库，常用键值对来存数据

但为什么要强调键值对呢？？
首先是快。
你查key，用hash函数查找的话，时间复杂度是O（1），直接拿到。
为什么说说哈希函数定位快？
Redis 内部用一个全局字典保存 key-value 的映射，这个字典底层类似 hash table。
key --> 通过hash 函数算出位置，在Redis字典里找到对应的value，直接去对应位置拿value  ！

就像手机上已经显示一区38号柜
你不需要查目录，也不需要逐层定位
直接走到38号柜拿外卖

所以用键值对的方式来查找，平均时间复杂度为 O（1）


key更像是这条数据本身的唯一地址。




---

虽然 Redis 主要把数据放在内存中，但它也提供持久化机制，避免进程重启后数据全部丢失

key永远是字符串，但values可以使用不同数据结构
不同数据结构决定不同用途
数据结构：
- string:SDS
- hash:listpack哈希表
- list:quicklist
- set:hash table,intset
- Zset → {Tom: 90, Jack: 80} 有分数的排序集合[^2]


数据持久化
	1. RDB （Redis Database）：定期生成快照，按配置 周期性生成数据快照，保存某一刻的全量数据集
	2. AOF（Append Only File）：增量日志，记录每一次 **写** 操作  ，数据安全




主从复制
	一主多从。主节点负责写，slave 复制 master 数据，以备不时之需（读扩展、数据备份、故障切换）
	1. 全量复制：slave第一次同步时，复制Master 的完整数据
	2. 增量复制：只补缺失的部分

 哨兵架构 sentinel
	 1.哨兵是“独立的”，独立运行的进程
	 2.一旦Master出问题，哨兵会立刻发现。然后多个 Sentinel [^3]之间通信、投票，“提拔”新的主节点
	 （和 k8s 的探针很像，但Sentinel能自动故障转移）


Sentinel已经初步解决高容灾的问题了，但是还有高并发啊！？

Redis Cluster集群架构就是用来解决高并发的
1. 哈希槽。key 通过算法计算哈希，再取模，得到槽位编号，再由”槽“决定数据落入那个节点[^1]
2. 重定向
3. Gossip（流言蜚语、闲聊）。多个节点相互确认集群状态（有点像村口大妈闲聊）


缓存设计

1. 缓存穿透（查询不存在的数据）
	- 布隆过滤器
	- 缓存空值

 2. 缓存击穿 （热点key、热门商品突然过期，大量请求打到MySQL）
	- 互斥锁
	- 永不过期

3. 缓存雪崩（大量key同时过期）
	- 过期时间随机化
	- 多级缓存



Redis运维



## 基本命令



命名规范
1. 使用冒号分隔层级
2. 使用有意义的前缀（功能、业务）
3. 保持统一（小写、日期）


先启动redis客户端
```bash
redis-cli info
redis-cli
```


>实际开发中有个习惯，命令大写、key小写
```redis
#判断key是否存在
EXISTS key

#获取key的类型
TYPE key

RENAME key newkey

#保存数据
SET key value

#获取数据
GET key

#删除
DEL key

```


```redis
#批量设置
MSET name tom age 20 city Nanjing

#批量获取
MGET name age city
```

数量操作
```redis
SET count 10

INCR count
#count=11


INCR article:1000:view
#阅读量+1




INCR count 10
#count=21

#DECR 同理
DECR count


```

设置过期时间
```redis
#300秒过期
SET code 9527  EX 300

#查看还剩多久
TTL code 

#NX 不存在才创建
SET sms:limit:1001 1 EX 60 NX


```


```redis

SELECT 0
```




[^1]: Cluster 有16384 个槽。key 先经过 CRC16 算法计算哈希值，然后再对 16384 取模，得到槽位编号，

[^2]: 可用于排行榜、热搜。为什么ZSet用跳表？答：因为排行榜需要按分数排序、范围查询、插入删除。O(logN)

[^3]: 防止单个哨兵误判，比如说网络抖动
