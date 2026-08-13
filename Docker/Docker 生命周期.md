


## 创建并启动容器
`docker run  -option  image` 


```bash
docker run -d   --name  web01  -p 8080:80  nginx
```





## 查看所有容器
`docker ps -a` 



## 停止、启动容器
`docker stop  web01` 
`docker start web01` 
`docker restart web01` 




## 删除容器
`docker rm -f  web01` 



## 进入运行中的容器
`docker exec  -it  web01  bash` 



## 看日志
`docker logs web` 
`dokcer logs  -f --tail  100 web01` 

看详细信息
`docker inspect   web01`

看资源占用
`docker stats`