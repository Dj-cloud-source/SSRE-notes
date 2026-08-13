

```bash

docker stats     docker_name

#内存
docker  run ……   --memory="512m"  ……

#CPU
docker  run ……   --cpus="1"       ……

#指定只能使用的CPU核心
docker  run ……   --cpuset-cpus="0,1" ……

#修改
docker  update  --cpus="2"  --memory="1g"  docker_name 

```