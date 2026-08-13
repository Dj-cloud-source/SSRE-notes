


## 基础命令 
### 查看本地已有镜像

`docker images` 

- `tag` 版本或名字标签



### 下载镜像

`docker  pull  nginx:1.28` 
>在生产环境里务必明确版本




### 删除镜像
```bash
docker ps -a
docker rm -f   id

docker images
docker rmi   images_id
```




### tag打标签
```bash

docker tag   images_id      new_image_name:tag
```

tag 还挺重要，因为以后推送镜像到仓库经常需要打标签
`docker push  ……` 





### 查看 `Dockerfile` 创建过程
`docker history nginx` 




## 镜像仓库管理


```bash
systemctl stop firewalld
setenforce 0

wget https://github.com/goharbor/harbor/releases/download/v2.15.0/harbor-online-installer-v2.15.0.tgz


tar -zxvf harbor-online-installer-v2.15.0.tgz

vim /etc/docker/daemon.json

"data-root": "/data/docker",
"insecure-registries": [ "192.168.60.130" ],

systemctl restart docker
docker info | sed -n '/Insecure Registries/,+5p'


cd harbor

cp harbor.yml.tmpl  harbor.yml

vim harbor.yml

#
hostname: 你的IP

# certificate: /your/certificate/path 
# private_key: /your/private/key/path





./install.sh
docker compose ps

docker login   IP

#用户名：admin 密码：Harbor12345

docker images

docker tag   images_id      IP/projects/images_tag

docker push                 IP/projects/images_tag


#在web界面有个复制按钮，可以复制docker pull命令
```

