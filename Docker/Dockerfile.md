
- `FROM` 基础镜像
- `RUN` 制作镜像的时候执行的命令。在 build 时就会执行，然后才是镜像制作完成

- `COPY` 构建镜像时，把宿主机文件复制进镜像
- `WORKDIR`  设置镜像之上的工作目录（ WORKDIR ≈ cd）
- `CMD` 容器启动后执行
- `ENV` 设置环境变量
- `EXPOSE`  预计监听什么端口[^1]









Dockerfile 实战

```dockerfile

FROM rockylinux:9
LABEL  author="666eagleslab"
RUN cd /etc/yum.repos.d/ \
&& mkdir backup \
&& cp -a /etc/yum.repos.d/*.repo backup/  \
&& ls /etc/yum.repos.d/backup/ \
&& sed -e 's|^mirrorlist=|#mirrorlist=|g'     -e 's|^#baseurl=http://dl.rockylinux.org/$contentdir|baseurl=https://mirrors.aliyun.com/rockylinux|g'     -i.bak     /etc/yum.repos.d/rocky*.repo \
&& yum clean all \
&&yum makecache \
&& yum install -y gcc make vim tar git wget  net-tools pcre2-devel zlib-devel openssl-devel 

# 从这里开始，工作目录切到 /usr/local/src
WORKDIR /usr/local/src

RUN curl -O https://nginx.org/download/nginx-1.28.0.tar.gz \
&& ls -lh nginx-1.28.0.tar.gz \
&& tar -zxvf nginx-1.28.0.tar.gz 

WORKDIR /usr/local/src/nginx-1.28.0 

RUN   ./configure \
   --with-http_ssl_module \
&& make \
&& ls -lh objs/nginx \
&& make install \
&& /usr/local/nginx/sbin/nginx -v \
&& /usr/local/nginx/sbin/nginx -t 


EXPOSE 80

#启动容器时，运行下面命令
CMD ["/usr/local/nginx/sbin/nginx","-g","daemon off;"]
```


```bash
mkdir -p ./test/dockerfile
cd ./test/dockerfile
vim Dockerfile
#复制进去

docker build -t  nginx-test:v1  .



docker tag nginx-test:v1      IP/library/nginx-test:v1
docker images | grep nginx-test

docker push    IP/library/nginx-test:v1





```







[^1]: 但这里有面试问的坑。
	EXPOSE 80 不等于 docker run -p 8080:80 
	
	EXPOSE 主要是声明、描述容器里的服务使用了 80 端口
	
	真正映射端口还是 docker run -p 8080:80 
	
