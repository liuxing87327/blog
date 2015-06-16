title: "linux命令备忘"
date: 2015-06-16 23:58:00
category: 系统配置
tags: [linux]

---

*记录一下自己常用的linux命令*

独立用户需要配置path,切换到根目录查看path
    
    vi .bash_profile

重新给文件夹赋权限
    
    chown -R yishou apache-tomcat-7.0.47

修改密码
    
    passwd yishou

新增用户，会自动创建同名文件夹

    useradd loupan

删除用户
    
    userdel keybox

创建文件夹
    
    mkdir yishou

防止环境配置修改之后不立即生效，退出重新登录也可以
    
    source .bash_profile

根据名字查找进程

    ps -aux | grep estat

关闭防火墙
    /etc/init.d/iptables stop

关闭开机启动
    
    chkconfig --level 2345 iptables off

赋权限
    
    chmod 777 origimagesdisk

MAC修改hosts

    sudo vi /etc/hosts

查看文件夹使用情况
    
    du --max-depth=1 -h

linux新建tomcat无法启动
`Cannot find bin/catalina.sh `
*The file is absent or does not have execute permission*
*This file is needed to run this program*

原因： 没有权限
解决 ： chmod 777 *.sh 

添加开机启动
    
    vi /etc/rc.local


jvisualvm

修改mac最大连接数，默认128
    
    sudo sysctl -w kern.ipc.somaxconn=

linux ssh互信
    
    ssh-keygen -t rsa
    cd ~/.ssh 
    scp -r id_rsa.pub keyuan@192.168.3.51:/home/keyuan/.ssh/authorized_keys
    
配置java环境变量

    export JAVA_HOME=/usr/local/java/jdk1.8.0_40
    export JRE_HOME=$JAVA_HOME
    export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
    export PATH=$JAVA_HOME/bin:$PATH