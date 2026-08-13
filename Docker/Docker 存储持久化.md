

1. Bind Mount 自己指定目录挂载
	- 宿主机绝对路径 → 容器目录
	-  常用命令  `docker run ……   -v   宿主机路径:容器路径   `
	- `-v  /data/nginx/html:/usr/share/nginx/html`
	- `docker run  --mount   ……` 

	-  常用于配置文件、Nginx网页、日志


2. Volume 数据卷
	1.  和 Bind Mount 的区别。交给Docker管理。
		删除容器，但 volume 还在


	2. 创建 `docker volume create   volume_name  ` 
	3. 查看 `docker volume ls       docker volume inspect  volume_name `
	4. 使用 `docker run ……  -v  卷名:容器路径 `
	5. 删除 `docker volume  rm  volume_name` 

	- 常用于 MySQL 、Redis 等持久化数据
















