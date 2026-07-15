

# SELinux



$Security-Enhanced- Linux$



防火墙 + SELinux ，网络 + 进程 双重守护

[极客blog](https://geek-blogs.com/blog/security-enhanced-linux/)

[知乎文章](https://zhuanlan.zhihu.com/p/1910718844818923999)

[课件](https://ncloud.eagleslab.com/Linux%E5%9F%BA%E7%A1%80/%E9%98%B2%E7%81%AB%E5%A2%99%E4%B8%8Eselinux.html#selinux%E5%AE%89%E5%85%A8%E5%AD%90%E7%B3%BB%E7%BB%9F)



使用 `getenforce` 命令查看 SELinux 的运行模式


* getenforce

	* Enforce ，拒绝不符合规则的访问，并记录日志
	* Permissive ，不拒绝，但会记录不符规则的操作（用于**故障排查**）
	* Disable ，完全关闭。**不推荐** 






调整 SELinux 模式

> 考虑到服务器不会重启，用以下方法即可

Permissive [0]

```bash
setenforce 0
```

Enforce [1]

```bash
setenforce 1
```

