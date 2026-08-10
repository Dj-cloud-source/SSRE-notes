

[toc]



# yum 包 下载工具



$yum （Yellowdog$ $Updater$ $Modified) 是一个软件包管理工具,用来解决RPM（Red$ $Hat$ $Package$$Manager)麻烦的依赖关系和手动下载rpm的烦恼能够自动化处理软件包的安装、卸载和更新。$
$后来Fedora开发了更快更稳的DNF，但为了兼容，yum命令链接到dnf，yum命令实际运行的是dnf$







yum命令



## 查找软件包

> 有些命令与软件包名不一样

* `yum` `provides` `命令名字`  通过命令名字查找属于哪个软件包



* `yum` `search` `“` `包名` `”`  模糊搜索相关包名

找不到包？
```bash
yum install -y epel-release 
```

## 安装软件包

* `yum` `install` `包名` `-y`
* `yum` `install` `包名1` `包名2` `包名3` `-y`  安装多个软件包
* `yum` `reinstall` `包名` 重新安装
* `yum` `update` `包名` 更新



## 卸载软件包

* `yum` `remove` `包名` `-y`



## 仓库管理

* `yum` `repolist` 查看现有软件仓库
* `yum` `clean` `all` 清空缓存及其他文件
* `yum` `makecache` 重建缓存
* `yum` `list` `installed` `|` `grep` `包名` 查看已安装的软件包
* `yum` `info` `包名` 查看软件包详细信息




[无网状态安装软件（RHCE）](https://ncloud.eagleslab.com/Linux%E5%9F%BA%E7%A1%80/%E8%BD%AF%E4%BB%B6%E5%8C%85%E7%AE%A1%E7%90%86.html#yum%E8%87%AA%E5%BB%BA%E6%BA%90) 

## ==换源==

```bash
cd /etc/yum.repos.d/
mkdir backup

cp -a /etc/yum.repos.d/*.repo backup/
ls /etc/yum.repos.d/backup/
```

```bash 

sed -e 's|^mirrorlist=|#mirrorlist=|g'     -e 's|^#baseurl=http://dl.rockylinux.org/$contentdir|baseurl=https://mirrors.aliyun.com/rockylinux|g'     -i.bak     /etc/yum.repos.d/rocky*.repo

```

```bash
yum clean all
yum makecache
yum repolist -v
```

[阿里云镜像网站](https://developer.aliyun.com/mirror/   )







[内网安装](https://ncloud.eagleslab.com/Linux%E5%9F%BA%E7%A1%80/%E8%BD%AF%E4%BB%B6%E5%8C%85%E7%AE%A1%E7%90%86.html#yum%E8%87%AA%E5%BB%BA%E6%BA%90 '气笑了，怎么都实验不成功🤣')





