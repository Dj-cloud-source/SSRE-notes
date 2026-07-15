[toc]



#      Linux命令行





安装好虚拟机后

```bash
echo "PermitRootLogin yes" >> /etc/ssh/sshd_config
systemctl restart sshd
ip a |grep 192
```

修改名字


```bash
yum install -y vim
vim /etc/hostname
```

输入新名字



修改颜色

```bash
export PS1="\[\e[37;40m\][\[\e[37;41m\]\u\[\e[37;41m\]@\h\[\e[37;40m\] \W\[\e[0m\]]\\$ "
```









#### 1. 显示文件

```bash
ls
ls / 
ls -a
ls -l
ls -la

```

> `ls  文件名`  （list files 查看目录）
>
> `-a` 是显示所有文件
>
> `-l`  是显示详细信息
>
> `-t`  是按时间排列

[课件详细 ](https://ncloud.eagleslab.com/Linux%E5%9F%BA%E7%A1%80/Linux%E5%91%BD%E4%BB%A4%E8%A1%8C.html#ls  "跳转课件") 



---



#### 2. 跳转命令

```bash
cd 
cd ..
```

> `cd .. `   跳转到上一层（change directory）
>
> `cd 文件名  ` （用绝对路径跳转,也可以用tab补全）

#### 3. 显示当前路径

```bash
pwd
```

> （print work directory）

#### 4. 输出字符串

```bash
echo "hello"
echo -e "he\nllo"
```

**想要 \n 这种 *加反斜线字符* 生效，就要加 -e**

> `\a` 让电脑响一下
>
> `\f `跳到下一行同位置
>
> `\r` 回车符
>
> `\t` 水平制表



#### 5. 关机、重启

```bash
poweroff
reboot
```



---





#### 6. 快捷键

| `^C` | 终止前台运行的程序       |
| ---- | ------------------------ |
| `^D` | 退出 等价exit            |
| `^L` | 清屏                     |
| `^A` | 光标移动到命令行的最前端 |
| `^E` | 光标移动到命令行的后端   |
| `^U` | 删除光标前所有字符       |
| `^K` | 删除光标后所有字符       |
| `^R` | 搜索历史命令，利用关键词 |





---

#### 7. 帮助命令

```bash
history
```

> `history -c`   假装清除历史记录，上下键就找不到以前的命令了
>
> `!6`   (英文感叹号)执行第六条命令
>
> `!@@@`   执行最近的 @@@ 命令

