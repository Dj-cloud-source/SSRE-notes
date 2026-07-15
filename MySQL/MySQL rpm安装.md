

MySQL下载


先换源
```bash

mkdir /repobackup

cp /etc/yum.repos.d/* /repobackup/

sed -e 's|^mirrorlist=|#mirrorlist=|g' \
    -e 's|^#baseurl=http://dl.rockylinux.org/$contentdir|baseurl=https://mirrors.ustc.edu.cn/rocky|g' \
    -i.bak \
    /etc/yum.repos.d/rocky-extras.repo \
    /etc/yum.repos.d/rocky.repo
    
    
yum makecache

yum install -y wget tar
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


