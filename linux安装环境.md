# Linux 安装环境

## 一、安装redis
### 1. 下载redis，官网 <a>https://redis.io/download/</a>
``` shell
# 切换到/usr/local目录
cd /usr/local
# 下载
wget https://download.redis.io/releases/redis-6.2.6.tar.gz
```

### 2. 解压
``` shell
tar -xzvf redis-6.2.6.tar.gz
```

### 3. 编译、安装
``` shell
# 进入redis目录
cd /usr/local/redis-6.2.6
# 编译，报错则往下看解决方案
make
# 安装，会将redis安装到目标路径
make PREFIX=/usr/local/redis-6.2.6 install
```
* 解决make报错
``` shell
# 如果make报错，则要安装gcc
yum -y install gcc automake autoconf libtool make
# 报错致命错误则需要
make MALLOC=libc
```

### 4. 修改配置
``` shell
cd /usr/local/redis-6.2.6

vi redis.conf
# 修改daemonize为yes，开启守护进程，修改为后台运行
# 注释掉bind 127.0.0.1 -::1，不然默认只有 127.0.0.1 ip才能访问
# 修改protected-mode为no，才可以远程访问
# 修改密码，打开requirepass foobared注释，将foobared改为自己的密码
# 修改端口，port 6379
```

### 5. 运行
``` shell
cd /usr/local/redis-6.2.6
# 后台运行
./bin/redis-server redis.conf
```

### 6. 停止
``` shell
cd /usr/local/redis-6.2.6/bin
./redis-cli
# 在redis客户端输入
shutdown
```

### 7. 查询进程
``` shell
ps -ef |grep redis
```

## 二、安装rabbitmq，不指定目录
### 1. 下载rabbitmq和erlang，官网 <a>https://www.rabbitmq.com/changelog.html</a>

1.1 点击Release notes，下载rabbitmq-server-3.9.15-1.el7.noarch.rpm

1.2 查看rabbitmq对应erlang版本 <a>https://www.rabbitmq.com/which-erlang.html</a>， 下载erlang， <a>https://github.com/rabbitmq/erlang-rpm/releases/</a><br>
本次下载erlang-23.3.4.11-1.el7.x86_64.rpm
### 2. 上传、解压、安装
``` shell
cd /usr/local
mkdir rabbitmq
cd rabbitmq
# 将压缩包上传到服务器
# 安装erlang
rpm -ivh erlang-23.3.4.11-1.el7.x86_64.rpm
# 查看erlang是否安装成功
erl -v
# 安装socat插件
yum install -y socat
# 安装rabbitmq
rpm -ivh rabbitmq-server-3.9.15-1.el7.noarch.rpm
```
### 3. 修改配置，解决只能localhost访问
``` shell
# 进入【/etc/rabbitmq】文件夹下
cd /etc/rabbitmq

# 编辑【rabbitmq.config】文件
vim rabbitmq.config

# 加入以下语句
[{rabbit,[{loopback_users,[]}]}].
```
### 4. 开启管理界面
``` shell
# 开启管理界面命令
rabbitmq-plugins enable rabbitmq_management
```

### 5. 启动、停止、重启
``` shell
# 启动rabbitmq命令：
systemctl start rabbitmq-server
 
# 查看启动状态命令：
systemctl status rabbitmq-server

# 停止rabbitmq命令：
systemctl stop rabbitmq-server
 
# 重启命令：
systemctl restart rabbitmq-server
```
### 6. 登录管理界面
URL地址：http://[ip]:15672/<br>
默认账号：guest<br>
默认密码：guest

### 7. 卸载
``` shell
# 先停止服务
systemctl stop rabbitmq-server
# 查看rabbitmq安装的相关列表
yum list | grep rabbitmq
# 卸载rabbitmq-server.noarch
yum -y remove rabbitmq-server.noarch
# 查看erlang安装的相关列表
yum list | grep erlang
# 卸载erlang已安装的相关内容
yum -y remove erlang-*
# 删除有关的所有文件
rm -rf /usr/lib64/erlang 
rm -rf /var/lib/rabbitmq
rm -rf /usr/local/erlang
rm -rf /usr/local/rabbitmq
```

## 三、安装rabbitmq，指定目录
### 1. 下载rabbitmq和erlang，官网 <a>https://www.rabbitmq.com/install-generic-unix.html</a><br>
1.1 点击rabbitmq-server-generic-unix-3.9.15.tar.xz，下载rabbitmq-server-generic-unix-3.9.15.tar.xz<br>
1.2 查看rabbitmq对应erlang版本 <a>http://erlang.org/download/</a>， 下载erlang， <br>
本次下载otp_src_24.2.tar.gz
### 2. 上传、解压、安装erlang
``` shell
cd /usr/local
mkdir rabbitmq
cd rabbitmq
# 将压缩包上传到服务器
# 安装erlang
yum -y install ncurses-devel
# yum -y install openssl-devel 
# yum -y install gcc-c++
# yum -y install unixODBC-devel
yum -y install make gcc gcc-c++ kernel-devel m4 ncurses-devel openssl-devel
tar -xvf otp_src_24.2.tar.gz
cd otp_src_24.2
./configure --prefix=/usr/local/rabbitmq/erlang --without-javac
make
make install
# 查看erlang是否安装成功
cd /usr/local/rabbitmq/erlang/
./bin/erl
```
### 3. 上传、解压、安装rabbitmq
``` shell
cd /usr/local/rabbitmq
yum install python -y
yum install xmlto -y
xz -d rabbitmq-server-generic-unix-3.9.15.tar.xz
tar -xvf rabbitmq-server-generic-unix-3.9.15.tar
mv rabbitmq_server-3.9.15/ rabbitmq
```
### 4. 修改配置
``` shell
vi /etc/profile
# 在 profile 文件末尾加上下面这句话（同时包含了erlang的环境变量配置）
export PATH=$PATH:/usr/local/rabbitmq/erlang/bin:/usr/local/rabbitmq/rabbitmq/sbin
source /etc/profile
```
### 5. 启动、关闭
``` shell
# 加了环境变量可以这样启动，没加环境需要到sbin目录操作
# 非后台启动，但是会打日志
rabbitmq-server

# 后台启动
./rabbitmq-server -detached

cd /usr/local/rabbitmq/rabbitmq/sbin
./rabbitmqctl stop

# 查看运行状态
rabbitmqctl status
```
### 6. 开启管理界面
``` shell
# 开启管理界面命令
cd /usr/local/rabbitmq/rabbitmq/sbin
./rabbitmq-plugins enable rabbitmq_management
```
### 7. 登录管理界面
URL地址：http://[ip]:15672/<br>
默认账号：guest<br>
默认密码：guest
* 一开始会没权限进入，显示只有本地才可以访问
  ``` shell
  # 创建之前，可以先查看一下当前有哪些用户和权限
  ./rabbitmqctl list_users
  ./rabbitmqctl list_user_permissions guest
  # 添加用户
  ./rabbitmqctl add_user admin admin
  # 设置超级管理员
  ./rabbitmqctl set_user_tags admin administrator
  ./rabbitmqctl set_permissions -p "/" admin ".*" ".*" ".*"
  ```