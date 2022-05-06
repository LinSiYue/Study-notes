# Linux云服务搭建项目环境

## 1、搭建环境

### 1.1 安装Docker

1）下载docker

 https://docs.docker.com/engine/install/centos/ 

根据文档步骤：

* 卸载之前下载的docker

```shell
sudo yum remove docker \
                docker-client \
                docker-client-latest \
                docker-common \
                docker-latest \
                docker-latest-logrotate \
                docker-logrotate \
                docker-engine
```

* 下载所需工具

```shell
sudo yum install -y yum-utils

sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

* 安装docker

```shell
sudo yum install docker-ce docker-ce-cli containerd.io
```

* 启动docker，查看镜像。

```shell
sudo systemctl start docker

sudo docker images
```

* 设置开机自启

```shell
sudo systemctl enable docker
```

* 配置阿里云加速镜像，方便docker下载所需镜像。

```shell
sudo mkdir -p /etc/docker

sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://44eevmt9.mirror.aliyuncs.com"]
}
EOF

sudo systemctl daemon-reload

sudo systemctl restart docker
```

2）下载所需镜像 https://hub.docker.com/ 

### 1.2 安装mysql

#### 1.2.1 docker环境

* 安装mysql

>  > ```shell
>  > sudo docker pull mysql:5.7
>  > ```
>  >
>  > 可切换到root用户，就不需要sudo了，密码为vagrant。
>  >
>  > ```shell
>  > su root
>  > ```
>
>  ​	创建实例并启动
>
>  > ```shell
>  > docker run -p 3306:3306 --name mysql \
>  > -v /mydata/mysql/log:/var/log/mysql \
>  > -v /mydata/mysql/data:/var/lib/mysql \
>  > -v /mydata/mysql/conf:/etc/mysql \
>  > -e MYSQL_ROOT_PASSWORD=root \
>  > -d mysql:5.7
>  > ```
>  >
>  > * 参数说明
>  >
>  > -p 3306:3306：将容器3306端口映射到主机的3306端口
>  >
>  > -v /mydata/mysql/log:/var/log/mysql：将配置文件夹挂载到主机
>  >
>  > -v /mydata/mysql/data:/var/lib/mysql：将日志文件夹挂载到主机
>  > -v /mydata/mysql/conf:/etc/mysql：将配置文件夹挂载到主机
>  > -e MYSQL_ROOT_PASSWORD=root：初始化root用户的密码
>  >
>  > 用mysql客户端工具尝试连接192.168.56.10数据库连接成功，用户名和密码均为root。
>  >
>  > 查看容器
>  >
>  > ```shell
>  > docker ps
>  > ```
>  >
>  > MySQL配置
>  >
>  > ```shell
>  > vi /mydata/mysql/conf/my.cnf
>  > ```
>  >
>  > 按i插入模式，编写以下内容，mysql5.5.3之后推荐使用utf8mb4编码，esc+:wq保存退出，
>  >
>  > ```text
>  > [client]
>  > default-character-set=utf8mb4
>  >  
>  > [mysql]
>  > default-character-set=utf8mb4
>  >  
>  > [mysqld]
>  > init_connect='SET collation_connection = utf8mb4_unicode_ci'
>  > init_connect='SET NAMES utf8mb4'
>  > character-set-server=utf8mb4
>  > collation-server=utf8mb4_unicode_ci
>  > ```
>  >
>  > 重启docker容器
>  >
>  > ```shell
>  > docker restart mysql
>  > ```
>  >
>  > 交互模式进入mysql的/bin/bash
>  >
>  > ```shell
>  > docker exec -it mysql /bin/bash
>  > ```
>  >
>  > 进入/etc/mysql里面，查看文件my.cnf，此时文件内容与刚刚主机配置的文件一致。
>  >
>  > ```shell
>  > cd /etc/mysql
>  > cat my.cnf
>  > ```

* 安装完成，默认用户密码都为root。

* 重置密码

  ```shell
  vi /mydata/mysql/conf/my.cnf
  
  #在[mysqld]最后新增
  skip-grant-tables
  
  docker exec -it mysql /bin/bash
  
  # 进入mysql
  mysql
  use mysql;
  
  # 查看用户和密码，以及访问地址
  select user,authentication_string,host from user;
  
  # 设置密码，password()设置密文格式
  update user set authentication_string=password('密码') where user='root';
  
  # 设置远程访问连接数据库，将host设置为%，默认localhost，只允许本地访问
  update user set host='%' where user='root';
  ```

#### 1.2.2 非docker环境

（1）、检查是否已经安装过mysql，执行命令

```shell
rpm -qa | grep mysql
```

如果安装过mysql，想删除则执行

```shell
rpm -e --nodeps mysql-libs-5.1.73-5.el6_6.x86_64
```

（2）、下载mysql压缩包

```shell
wget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.24-linux-glibc2.12-x86_64.tar.gz
```

也可以到官网下载，https://downloads.mysql.com/archives/community/，之后将安装包拷到Linux服务器上。

（3）解压压缩包

```shell
tar xzvf mysql-5.7.24-linux-glibc2.12-x86_64.tar.gz
```

解压文件夹最好放到/usr/local文件目录下，因为可以全局使用命令。

（4）、启动mysql

```shell
systemctl start mysqld
```

### 1.3 安装redis

#### 1.3.1 docker环境

* 安装redis

> > ```shell
> > docker pull redis
> > ```
> >
> > 创建并启动实例，因为还没有redis.conf文件，需要先创建
> >
> > ```
> > mkdir -p /mydata/redis/conf
> > touch /mydata/redis/conf/redis.conf
> > 
> > docker run -p 6379:6379 --name redis \
> > -v /mydata/redis/data:/data \
> > -v /mydata/redis/conf/redis.conf:/etc/redis/redis.conf \
> > -d redis redis-server /etc/redis/redis.conf
> > ```
> >
> > 此时redis并不是持久化，重启redis之后，保存的数据丢失，所以设置持久化aof方式，在redis.conf文件中加入，
> >
> > ```text
> > appendonly yes
> > ```
> >
> > 交互模式进入redis测试
> >
> > ```shell
> > docker exec -it redis redis-cli
> > ```

* 安装完成，默认密码为空。

### 1.4 安装tomcat

```shell
docker pull tomcat
```

启动

```shell
docker run -d -p 8088:8080 --name tomcat1 --restart=always tomcat
# 或者
docker run -d -p 8088:8080 --name tomcat1 -v /usr/local/dev/docker-tomcat:/usr/local/tomcat/webapps --restart=always tomcat

# 之后
docker exec -it tomcat /bin/bash

# 因为webapps里面文件都没有了，他默认webapps里面的每个文件夹都是一个jsp站点，需要有完整的结构，
# 即WEB-INF文件夹、web.xml文件，可以将webapps.dist文件夹里面的内容拷贝到webapps里面
cp webapps.dist/* webapps/

# 就可以访问tomcat了，如果想自己测试index.html文件，则将index.html放到ROOT文件夹下即可

```


### 1.5 安装nginx（非docker）

1. 下载http://nginx.org/en/download.html
2. 安装环境

```shell
yum install gcc-c++
yum install -y zlib zlib-devel
yum install -y openssl openssl-devel
tar -zxvf nginx-1.8.0.tar.gz
cd nginx-1.8.0
./configure --prefix=/usr/local/nginx
make
make install
```

* nginx.conf

```conf
server {
	listen       80;
	server_name  localhost;

	location / {
        root   html;
        index  index.html index.htm;
        try_files $uri $uri/ /index.html;
    }

    location /api {
        proxy_set_header Host $http_host;
        proxy_pass http://localhost:8080/freesay;
    }
}
```

启动：

```
./nginx
```

停止：

```shell
ps -ef |grep nginx
netstat -lnp|grep 80
kill -9 <PID>
```

### 1.6 nginx
1. docker安装
```shell
docker pull nginx:latest
```

2. 运行
```shell
docker run -itd -p 80:80 -v /freesay/nginx/conf/nginx.conf:/etc/nginx/nginx.conf -v /freesay/nginx/html:/usr/share/nginx/html --name nginx nginx
```

3. 配置
```conf
server {
    listen       80;
    server_name  localhost;

    location / {
        root   /usr/share/nginx/html/dist;
        index  index.html index.htm;
        try_files $uri $uri/ /index.html;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   html;
    }
}
```