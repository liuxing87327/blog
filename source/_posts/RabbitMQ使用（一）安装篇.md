title: "RabbitMQ使用（一）安装篇"
date: 2016-01-31 13:22:00
category: MQ
tags: [RabbitMQ,MQ,消息队列]

---

## 安装erlang语言环境

安装依赖文件

> yum install ncurses-devel
  yum -y install openssl*
  yum -y install ssl*
  yum -y install xmlto
  yum -y install python-simplejson
  yum -y install python

进入 http://www.erlang.org/download.html 选择源文件下载

> wget http://www.erlang.org/download/otp_src_17.5.tar.gz
tar zxvf otp_src_17.5.tar.gz
cd otp_src_17.5
    
阅读HOTO/INSTALL.md文件 

> ./configure
  make && make install 
  
安装完成以后，执行erl看是否能打开eshell，用’halt().’退出，注意后面的点号，那是erlang的结束符。

> [root@iZ113aowxo2Z ~]# erl
  Erlang/OTP 17 [erts-6.4] [source] [64-bit] [async-threads:10]      [hipe] [kernel-poll:false]
  Eshell V6.4  (abort with ^G)
  1> 9+1.
  10
  2> halt().
  
## 安装RabbitMQ 
安装依赖
> yum install xmlto

创建主文件夹
> mkdir rabbitmq
  cd rabbitmq

直接使用RPM
> wget http://www.rabbitmq.com/releases/rabbitmq-server/v3.6.0/rabbitmq-server-3.6.0-1.noarch.rpm
  rpm -ivh rabbitmq-server-3.6.0-1.noarch.rpm
  
编译安装包
进入http://www.rabbitmq.com/download.html选择最新的源码包
>  wget http://www.rabbitmq.com/releases/rabbitmq-server/v3.6.0/rabbitmq-server-3.6.0.tar.xz
  xz -d rabbitmq-server-3.6.0.tar.xz
  tar xvf rabbitmq-server-3.6.0.tar
  cd rabbitmq-server-3.6.0
  make
  make install TARGET_DIR=/opt/rabbitmq SBIN_DIR=/opt/rabbitmq/sbin MAN_DIR=/opt/rabbitmq/man DOC_INSTALL_DIR=/opt/rabbitmq/doc
  
## 使用
启动MQ
> rabbitmq-server -detached

查看状态
> rabbitmqctl status

启用管理插件
> mkdir /etc/rabbitmq/
  rabbitmq-plugins enable rabbitmq_management 

停止服务
> rabbitmqctl stop

添加账号
PS：默认账号guest只能在localhost访问
> rabbitmqctl add_user admin admin

设置管理员
> rabbitmqctl set_user_tags admin administrator

设置读写权限
命令使用户admin具有/vhost1这个virtual host中所有资源的配置、写、读权限以便管理其中的资源
> rabbitmqctl set_permissions -p /vhost1 admin '.*' '.*' '.*'


查看账号
> rabbitmqctl list_users

加入账号到配置
> vi /etc/rabbitmq/rabbitmq.config
> [
    {rabbit, [{loopback_users, ["admin"]}]}
  ].

重启后 http://ip:15672 登录管理界面了