


Docker Compose 是单机容器“启动管理工具” 
简化容器的创建、启动和停止操作



1. `compose.yaml` 结构
	1. `services:`
		1. `image`
		2. `container_name`
		3. `ports`
		4. `volumes`
		5. `environment`
		6. `networks`
		7. `depends_on` 可以控制服务的启动依赖顺序
		8. `restart` 


2. 镜像来源
	- image  
	- 根据 `Dockerfile` 构建 `build .` 


3. 多容器通信
	- Compose 默认会创建网络
	- 服务之间通过 service 名通信

4. Volume / Network
	1. volumes:
	2. networks:


5. 常用命令
	1. `docker compose up -d`
	2. `docker compose down` 
	3. `docker compose ps` 
	4. `docker compose logs` 
	5. `docker compose restart` 
	6. `docker compose stop/start` 
	7. `docker compose exec` 
	8. `docker compose pull` 










#### 实战
①
```bash
mkdir -p lnmp
cd lnmp
vim compose.yaml
```







>compose文件统一用空格，不能用制表符（改一下tab）

```yaml
#compose.yaml




networks:
    my_network:
        driver: bridge
        
        
        
services:
    
    #服务名（在容器内部网络，可作为主机名访问）
    mysql:
        image: mysql:8.0
        container_name: mysql_container
        environment:
            MYSQL_ROOT_PASSWORD: "1"
            MYSQL_DATABASE: mydb	
        volumes:
            #命名卷。由 Docker 管理。
            - mysql_data:/var/lib/mysql	  
        ports:
            - "3306:3306" 
        restart: always
        networks:
            - my_network
              
              
    php:
        build: ./php
        container_name: php_container
        volumes:
            #宿主机同级别下的 www
            - ./www:/var/www/html
              
        networks:
            - my_network
        
        restart: always
          
        depends_on:
            - mysql
              
    nginx:
        image: nginx:latest
        container_name: nginx_container
        ports:
            - "80:80"
              
        networks:
            - my_network    
        restart: always
        
        depends_on:
            - php
              
        volumes:
            - ./www:/var/www/html
            - ./nginx/default.conf:/etc/nginx/conf.d/default.conf:ro
              
              

   
              
    


#声明命名卷（位于文件最底部，与 services 平级）
volumes:
    #如果没有额外配置，Docker 将使用默认的 Local 驱动
    mysql_data:  

```


②
>给Nginx关联到php。然后挂载一下
>`fastcgi_pass`
```bash

mkdir -p  nginx
vim     nginx/default.conf

```
```nginx

server {
    listen 80;
    server_name localhost;
    
    root /var/www/html;
    index index.php  index.html;
    
    location / {
        try_files $uri $uri/ =404;
    }
    
    location ~ \.php$ {
        fastcgi_pass  php:9000;  #把 PHP 请求交给 PHP-FPM
        fastcgi_index index.php; #默认 PHP 首页
        include fastcgi_params;  #带上标准 FastCGI 参数
        
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                      # 告诉 PHP 要执行那个文件
    }
}
```



④
创建 PHP 页面
```bash
mkdir  -p www
vim  www/index.php
```
```php
<?php

$host = "mysql";
$user = "root";
$password = "1";
$database = "mydb";

$conn = new mysqli($host,$user,$password,$database);

if ($conn->connect_error){
    die("MySQL connection failed: "  .$conn->connect_error);
}

echo "PHP -> MySQL connection successful !";




phpinfo();
?>
```


③
PHP 访问MySQL
```bash
mkdir -p php
vim php/Dockerfile
```
```dockerfile
FROM php:8.2-fpm

RUN docker-php-ext-install mysqli  pdo_mysql
```
>然后前面那里得改成build



⑤ 检查创建了什么
```bash
find . -type f
```



⑥ `docker compose config` 检查配置文件 

⑦ `docker compose up -d --build` 

`docker compose ps`
`docker compose logs` 



OK啊也是跑通了

#### 收尾 

```bash

docker compose config

docker compose  up -d 
docker compose  up -d --build

docker compose  ps

docker compose logs
docker compose logs -f


docker compose logs nginx
docker compose restart nginx



docker compose stop
docker compose down
docker compose start
docker compose pull
```