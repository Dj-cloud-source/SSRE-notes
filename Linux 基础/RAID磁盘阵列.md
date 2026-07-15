[toc]



# 磁盘阵列





## 什么是磁盘阵列？

RIAD (Redundant Array of Independent Disks,独立磁盘冗余阵列)

$之前讲到磁盘分区，把磁盘“分开”管理，现在这里，是把硬盘“合起来”使用$

---> 将多块独立的磁盘，结合起来，以提高读写性能、数据安全或增加磁盘空间



## 所谓的磁盘阵列有哪些

* RAID 0

* RAID 1

* RAID 10

* RAID 5

* RAID 6

	

### RAID 0

* 将一份数据打散存储
	* 优点：很快
	* 缺点：不安全，盘坏了，数据直接没了
	* 适用于临时存储、高性能I/O场景



### RAID1

* 一个数据存两份

	* 优点：数据安全，坏东西不影响
	* 缺点：占用磁盘容量，只能用一半的磁盘容量
	* 适用于数据库，操作系统

	







### RAID10

* 结合RAID 0和RAID 1，先镜像，再打散存储

	* 优点：稳+快
	* 缺点：贵
	* 适用于数据库MySQL、高负载业务系统

	

	 



### RAID 5

* 有个奇偶校验，数据和校验分散在所有盘
	* 优点：坏了能重新算回来
	* 缺点：大盘重建时间长；只能坏一个盘，第二块盘坏直接爆炸
	* 适用预算有限的场景，但已经几乎不用了



### RAID 6

* RAID 5的升级版，允许坏两块



---



# `mdadm` 工具

(multiple device admin)

`mdadm`   命令

> 首先安装这个工具 `yum install mdadm` 



| 选项                | 作用         |
| ------------------- | ------------ |
| `-C`                | 创建         |
| `-a`                | 检测设备名称 |
| `-n` + `设备数量`   | 指定设备数量 |
| `-l` + ` 0/1/5/10 ` | 指定RAID级别 |
| `-v`                | 显示过程     |
| `-D`                | 查看详细信息 |
| `-S`                | 停止RAID阵列 |







## 部署磁盘阵列



1. 首先关闭虚拟机，再添加新磁盘，然后拍快照，以此模拟添加物理硬盘。



```bash
mdadm -Cv /dev/md123 -a yes-l 0 -n 2 /dev/nvme0n2  /dev/nvme0n3
……
lsblk
```

2. 格式化文件系统

```bash
mkfs.ext4   /dev/新名字
```

3. 创建挂载点，对设备进行[挂载]( ' 可以把挂载信息写入/etc/ fstab 中 ，永久挂载')

```bash
mkdir  /mnt/raid0
……
mount /dev/新名字   /mnt/raid0
```

4. 重新加载，并查看磁盘阵列的详细信息

```bash
systemctl daemon-reload
```

```bash
mount -a
……
df -hT
……
mdadm -D  /dev/md0
```



5. 挂载好之后，测试一下读写速度

> 通过 fio磁盘测试工具来测试读写速度 ，   `yum install -y fio` 

```bash
fio --name=write_test --filename=/mnt/RAID0/testfile --size=1G --bs=1M --rw=write --direct=1 --numjobs=1
```



6. 对照实验

```bash

mkdir /mnt/test
 mkfs.ext4 /dev/nvme0n4
 mount /dev/nvme0n4 /mnt/test
 df -h
 ……
 fio --name=write_test --filename=/mnt/test/testfile --size=1G --bs=1M --rw=write --direct=1 --numjobs=1
```

![image-20260204162757085](image-20260204162757085.png)

 *数据符合结论*







## 删除磁盘阵列

1. 先取消[挂载]( ' 如果写入到 fstab 中，要记得删掉 ')

```bash
umount /dev/新名字
```

2. 停止磁盘阵列

```bash
mdadm -S  /dev/新名字
```

3. 删除阵列中的数据块信息

```bash
mdadm --zero-superblock /dev/nvme0n2 /dev/nvme0n3
……
lsblk
```



---

