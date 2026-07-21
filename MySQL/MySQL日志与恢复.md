
## 日志类型

运行状态类日志：排查MySQL 报错
1. `error_log` 错误日志
	- `tail -n 100 /var/log/mysqld.log` 
2. `slow_query_log` 慢查询日志
3. `general_log` 通用查询日志（不建议使用）


事务恢复类日志
1. `redo_log` 
2. `undo_log` 


数据同步类日志：保证主从复制、数据恢复
1. ==`binlog` 二进制日志==
2. `relay_log` 中继日志





## `binlog` 二进制日志

二进制日志会记录对数据库发生修改的操作
`binlog` 的作用：数据的备份恢复；主从复制

二进制日志模式：row 模式，记录每一行数据的具体变化


开启 `binlog` 
```bash
vim /etc/my.cnf

#server-id=1 表示这是主库。每台MySQL的ID必须唯一

[mysqld]
server-id=1 
log-bin=mysql-bin
binlog_format=ROW
```

然后重启 MySQL 
```bash
systemctl restart mysqld

systemctl status mysqld
```

验证是否开启

```sql
#进入MySQL
show variables like 'log_bin_basename';
show variables like 'log_bin';
show variables like 'binlog_format';
show master status;
show binary logs;
```



## 备份恢复

1. 逻辑备份
	- `mysqldump` 将数据导出成SQL
	- `source` 导入数据


2. 物理备份 `XtraBackup` 
	- 直接备份数据库文件
	- 数据文件
	- `redo log`
	- `undo log` 


备份工具的使用
- `mysqldump`
	- `-u -p -h -P  -S ` 连接服务端参数
	- `-A` 全库备份


命令
```bash
mysqldump  -uroot -p123456  -A  >  /back/full.sql
mysqldump  -uroot -p123456  db1  > /back/db1.sql

#只备份表
mysqldump  -uroot -p123456  school student > /backup/student.sql
```

恢复
```sql
mysql>set sql_log_bin = 0 ;
mysql>source  /backup/student.sql
mysql>set sql_log_bin = 1;
```
```bash
mysql -uroot -p123456  <  /backup/student.sql
```





`-R` ：备份存储过程和函数数据
`--triggers` ：备份触发器数据
`--single-transaction`  ：快照备份
`--master-data=2` ：记录binlog的位置，利于后续增量备份
`|gzip` ：压缩备份




所以，常用的全量热备命令
```bash

mysqldump -uroot -p123456 -A -R --triggers --master-data=2 --single-transaction |gzip > /backup/full_$(date +%F).sql.gz
```




全备后，发送到新的 MySQL 上
```bash
scp  /backup/full_$(date +%F).sql.gz  root@10.1.1.1:/tmp

#进入新机器
cd /tmp
gzip -d full_$(date +%F).sql.gz  
```

```sql
#在新库中导入数据
mysql>create database  backup;


mysql>set sql_log_bin = 0 ;
mysql>source  /tmp/full_$(date +%F).sql

```



增量备份
补上凌晨0点到误删前的数据
这就是binlog 的作用

PITR(Point In Time Recovery) 
全量备份+binlog 恢复到指定时间点


看备份文件
```bash
head -50 full_$(date +%F).sql |grep -i 'change master to'
```
可能会看见
```bash
CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000017',
MASTER_LOG_POS=268002;

#binlog 文件：
#mysql-bin.000017

#位置：
#268002
#也就是全量备份结束的位置
```

回到生产库
```sql
show binlog events in 'mysql-bin.000017'\G



Position: 268002
...
Position: 671148
DROP TABLE backup.new

#起点：
#268002

#终点：
#671148
```




截取这段binlog，再用 binlog 重放 0 点到误删之前的操作

```bash
mysqlbinlog \
--start-position=268002 \
--stop-position=671147 \
mysql-bin.000017 \
> /tmp/inc.sql
    
#stop-datetime 要取在误操作之前，不能取误删的位置，否则误删 SQL 也会被重新执行。    

```

```sql
mysql>use backup;
mysql>source /tmp/inc.sql
```

然后新表就修好了，照常发送回去就行

[企业灾备恢复实战演示](https://ncloud.eagleslab.com/%E6%95%B0%E6%8D%AE%E5%BA%93/MySQL.html#%E3%80%90%E5%AE%9E%E6%88%98%E3%80%91%E4%BC%81%E4%B8%9A%E6%95%85%E9%9A%9C%E6%81%A2%E5%A4%8D) 



