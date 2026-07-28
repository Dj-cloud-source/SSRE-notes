



## 镜像从哪来？

镜像最开始从Docker hub 或 Harbor 私有仓库



如
```yml
containers:
- name: nginx
  image: nginx:1.27
```
这里  nginx:1.27  只能说是一个地址简写
完整的路径应该是
`image : docker.io/library/nginx:1.27`  


如果node在工作时发现本地没有镜像，就会启动 `containerd` 从 `docker.io`    pull到本地。然后创建容器。




## 镜像存在哪？

现在k8s 常用 containerd 

存储位置类似 `/var/lib/containerd` 

containerd 里存有
- images
- content
- snapshots



images 里实际上是
1. Linux基础文件
2. nginx程序
3. 配置文件
4. metadata


类似 docker file   ,每一步形成一层
```
Dockerfile:

FROM ubuntu

RUN apt install nginx

COPY nginx.conf
```