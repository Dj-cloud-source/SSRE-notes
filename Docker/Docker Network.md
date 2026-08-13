
1. 容器网络概述
2. bridge 模式
3. 端口映射
4. 容器之间通信
5. 自定义网络
6. 常用命令





Docker 网络解决了什么问题？

- 容器怎么访问互联网？
- 宿主机怎么访问容器？
- 容器A 怎么访问 容器B ？
- 为什么 docker run 经常写 -p 8080:80 ？




bridge 模式
bridge 模式是Docker 基本的网络模式
```bash
docker network ls
```



容器通过一个虚拟网络（docker0）连接到宿主机
```
                 Rocky 宿主机
              192.168.60.130
                     │
                  docker0
                 172.17.0.1
                     │
          ┌──────────┴──────────┐
          │                     │
      容器 nginx             容器 app
      172.17.0.2             172.17.0.3
```




端口映射

```

-p          8080              :    80
	
	rocky宿主机的8080端口           Nginx容器里的80端口

```

```
#大概链路：

浏览器
   ↓
192.168.60.130:8080
   ↓
Rocky 宿主机 8080
   ↓
Docker 端口映射
   ↓
nginx 容器 :80

```

>记得Dockerfile里的EXPOSE和这里不是同一回事






容器之间的通信

两个容器在同一个 Docker 网络里，就可以互相通信
**实际使用时优先通过“容器名 --name ”找对方** 



常用命令
```bash
docker run -d --name nginx1    --network net_name nginx

docker network ls

docker network create   net_name

docker network inspect  net_name

docker network  rm      net_name

docker network  connect net_name  nginx1
docker network  disconnect  net_name nginx1
```