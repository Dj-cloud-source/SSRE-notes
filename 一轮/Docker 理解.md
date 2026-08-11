## Docker 



先说虚拟机、KVM

最下层，叫“宿主机”，是真正的物理机和Linux系统

其次是虚拟层

然后才有 Guest OS，虚拟机里必须安装一个完整的操作系统，

最后才是 业务代码



它每台虚拟机都需要一个完整的OS，就会导致很笨 **重**

* 系统占用资源占用大

* 启动慢



因为 KVM虚拟机太重太慢，所以，容器技术应运而生

Docker



Docker 并不虚拟化任何硬件、网络，也不装操作系统，

Docker 快的原因，是因为容器共享底层宿主机的[内核]( ' 所谓内核，不过是那些：进程（PID）、内存（memory）、网络、磁盘、文件系统、系统调用（syscall）') ， “宿主机已经有内核了，我直接用它”

{

当然，Linux 分为两层：

* Linux
	* kernel
	* 用户空间（bash、MySQL）

}



 **共享内核**，更像是 **一个被隔离的进程空间** 

也就是说，在容器内部，容器以为自己拥有一整个世界。

但出到宿主机上，会发现，其实容器里运行的，本质上也是宿主机的进程。



Docker 使用两个技术

* Namespace
* Cgroup





`Namespace` 

进到容器内部  `ps aux` ，就会发现，容器有自己的 PID，所以才说“在容器里看，它以为自己拥有整个世界”
1. PID
2. NET 网络
3. MNT 挂载
4. IPC 进程通信隔离
5. UTC hostname隔离
6. USER 用户权限




`Cgroup` 

这个是资源限制的，规定这个容器只能用 这么多CPU和内存



AI经典比喻：

KVM 虚拟机是盖公寓楼，每套房都有承重墙、地基等，虽然安全隔离性好，但盖起来很慢、占地方大



Docker 容器则是”租房子“，共享地基、电梯，互不干扰，快速入住。



---



镜像是模版，容器才是跑起来的。

就像类与对象









1. Docker 基本命令，管理容器，exec，run



2. 管理镜像

3. 制作镜像：Dockerfile （就像资源清单：拉取镜像版本，写配置文件，容器内运行命令

4. 公有仓库Docker hub，公司私有仓库Registry

5. 数据存储（数据卷 volumes ，但只是单纯关联了 dir？）
6. 网络管理[^1]（没学会，需要一点计网知识）



7. 单机编排 compose （写yaml配置文件，服务组件，镜像版本，映射端口，服务启动顺序……）

8. 存储（最底下的只读层，上面的镜像层，各自的可写层。联合文件系统将多个 dir 联合挂载到同一dir，统一文件系统视图）




## 命令



```bash
docker info
```


```bash
docker search nginx
#实际工作中，通常去hub看官方镜像说明


docker pull nginx:latest
docker pull nginx:1.27


```


```bash
docker image ls


docker stats  
```



```bash
docker rmi nginx

docker rmi abc123456

```

```bash
docker run  -d nginx

docker run -d --name web01  nginx
#以后可以用 容器名 web01 操作


docker run -d  \
--name web01 \
-p  8080:80  \
nginx
```


```bash
#排障常用
docker ps -a

docker inspect web01
```



```bash

docker exec -it web01  bash

docker exec -it web01  sh

docker exec web01  ps  aux

exit
```
`-i` ：保持标准输入
`-t` ：分配一个终端
`-it` ：像登录Linux一样操作容器



```bash
docker stop web01
docker start web01
docker restart  web01
```



```bash

docker logs  web01
docker logs  --tail 20 web01
docker logs  --since 10m web01

#实时排障常用
docker logs -f --tail  100  web01

```










[^1]: 下面
```bash

docker network ls
```

重点学 bridge

bridge，docker0 相当于一个网络中转站、虚拟交换机


容器IP：172.17.0.2
宿主机IP：192.168.1.10

容器访问网络
```markdown
172.17.0.2
    │
    ▼
docker0
    │
    ▼
宿主机
    │
    ▼
NAT 地址转换
    │
    ▼
192.168.1.10
    │
    ▼
互联网
```

外部网站看到的来源通常不是容器的 `172.17.0.2` ，而是宿主机对外的IP
`172.17.0.2` 是私有地址，不能直接在互联网中路由。

就像
手机：192.168.1.3
        ↓ NAT
路由器公网IP
        ↓
互联网


外部访问容器
```markdown
客户端访问 宿主机IP:8080
              │
              ▼
       Docker 端口转发
              │
              ▼
        容器IP:80
```


容器间通信
两个容器通过`docker0` 通信
```markdown
Container A
      │
      ▼
docker0
      │
      ▼
Container B
```



总结
bridge模式（最常用）

有独立容器 IP
通过 NAT 上网
通过 -p 对外暴露服务


