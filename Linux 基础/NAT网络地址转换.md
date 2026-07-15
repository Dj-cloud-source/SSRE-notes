# NAT 网络地址转换

NAT(Network Address Translation,网络地址转换)



$模拟内网机器不能直接与外部联系，只能用另一台机器当中介，来上网$



* 步骤

1. 恢复快照
2. 克隆出一台虚拟机
3. 编辑虚拟机设置，将克隆的，网络适配器改成仅主机模式。
4. 在原来的电脑添加一网络适配器，仅主机模式
5. 开机
6. 在克隆机上 `nmtui` ,编辑 `gateway` 输入 原本电脑 的ip， `dns`  输入 `114.114.114.114`  
7.  `reboot`  （在虚拟机上重启可以，但在生产环境中不太行）



* 在 原本电脑 上开启数据转发功能

```bash
echo "net.ipv4.ip_forward=1" >> /usr/lib/sysctl.d/50-default.conf
sysctl -w net.ipv4.ip_forward=1
sysctl -p
```



---





## `SNAT`



* 配置 SNAT 让克隆机能上网

```bash
iptables -t nat -A POSTROUTING -s 克隆机ip  -j SNAT --to-source 原本电脑ip
```



然后在克隆机上 `ping www.qq.com`  ，成功 `ping`  通。



---

## `DNAT` 



配置 原本电脑 的  `DNAT` ，使得外部可以放问克隆机上的内部网站

```bash
iptables -t nat -A PREROUTING -p tcp --dport 1234  -j DNAT --to-destination 克隆机ip:80
```



