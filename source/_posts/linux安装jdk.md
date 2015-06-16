title: "linux安装jdk"
date: 2015-06-17 00:17:00
category: 系统配置
tags: [linux,jdk]

---

*记录一下*

##安装
    
    cd /usr/local/java/ 

或

    cd /usr/java/

    tar -zxvf 文件名.tar.gz

可以修改文件夹读写、所有者、所属组
http://www.tuicool.com/articles/b6bimiz

##环境变量配置

全局方式：

```bash
vi /etc/profile

export JAVA_HOME=/usr/java/jdk1.8.0_40
export JRE_HOME=$JAVA_HOME
export CLASSPATH=.:$JRE_HOME/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$JAVA_HOME/bin:$PATH

source /etc/profile #使更改的配置立即生效
```

用户环境变量

```bash
vi ~/.bash_profile

export JAVA_HOME=/usr/local/java/jdk1.8.0_40
export JRE_HOME=$JAVA_HOME
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$JAVA_HOME/bin:$PATH

source ~/.bash_profile #使更改的配置生效

java -version #查看版本

```