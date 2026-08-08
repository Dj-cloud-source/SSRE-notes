

MySQL下载


先换源
```bash

cd /etc/yum.repos.d/
mkdir backup

cp -a /etc/yum.repos.d/*.repo backup/
ls /etc/yum.repos.d/backup/


sed -e 's|^mirrorlist=|#mirrorlist=|g'     -e 's|^#baseurl=http://dl.rockylinux.org/$contentdir|baseurl=https://mirrors.aliyun.com/rockylinux|g'     -i.bak     /etc/yum.repos.d/rocky*.repo


yum clean all
yum makecache
yum repolist -v

```


下载
```bash
wget https://dev.mysql.com/get/Downloads/MySQL-9.0/mysql-9.0.1-1.el9.x86_64.rpm-bundle.tar

tar -xvf mysql-9.0.1-1.el9.x86_64.rpm-bundle.tar

yum localinstall *.rpm -y

systemctl start mysqld
systemctl enable mysqld
```


```bash
grep 'temporary password' /var/log/mysqld.log 
#复制临时密码

mysql -uroot -p临时密码

ALTER USER 'root'@'localhost' IDENTIFIED BY '临时密码';
FLUSH PRIVILEGES;

SET GLOBAL validate_password.policy = LOW;
SET GLOBAL validate_password.length = 6;
ALTER USER 'root'@'localhost' IDENTIFIED BY '123456';
```


---



字符集 Charset
字符集是一个系统支持的所有抽象字符的集合

![[Pasted image 20260716105807.png]]

MySQL中常见的字符集：
- UTF8
- GBK
```sql
#查看
show charset
#建表
mysql>create table `test`(
`id` int(4) not null auto_increment,
`name` char(20) not null,
primary key (`id`)
)engine = innodb  charset = utf8;

mysql> alter table t1 charset set utf8 ;

```

MySQL数据类型

主要数据类型
- 数值
- 字符
- 二进制
- 时间

Type
- `tinyint` （0-255）的整数
- `int` 整数
- `char` 字符串
- `verchar` 可变长度字符串
- `enum` 枚举
