title: "Tomcat的APR优化"
date: 2015-06-17 00:53:00
category: 系统配置
tags: [tomcat,APR]

---
*什么是APR？*

{% blockquote Apache Portable Runtime http://apr.apache.org/ Welcome! - The Apache Portable Runtime Project %}
Tomcat可以使用APR来提供超强的可伸缩性和性能，更好地集成本地服务器技术。
 
APR(Apache Portable Runtime)是一个高可移植库，它是Apache HTTP Server 2.x的核心。
 
APR有很多用途，包括访问高级IO功能(例如sendfile,epoll和OpenSSL)，OS级别功能(随机数生成，系统状态等等)，本地进程管理(共享内存，NT管道和UNIX sockets)。这些功能可以使Tomcat作为一个通常的前台WEB服务器，能更好地和其它本地web技术集成，总体上让Java更有效率作为一个高性能web服务器平台而不是简单作为后台容器。
 
在产品环境中，特别是直接使用Tomcat做WEB服务器的时候，应该使用Tomcat Native来提高其性能。
{% endblockquote %}

##安装apr
yum install -y apr-devel openssl-devel gcc

查看安装目录
rpm -ql apr-devel
rpm -ql openssl-devel

apr目录：/usr/bin/apr-1-config


##安装native

拷贝：tomcat/bin目录下的tomcat-native.tar.gz到某个位置
解压：tar zxvf tomcat-native.tar.gz

    cd  tomcat-native-1.1.32-src/jni/native/
    ./configure --with-apr=/usr/bin/apr-1-config --with-java-home=$JAVA_HOME 
    ./configure --with-apr=/usr/local/apache2/ --with-java-home=/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home --with-ssl=yes


可选：
    
    --with-ssl=yes
    make & make install

##配置tomcat

    vim catalina.sh
    export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/apr/lib

将tomcat/config/service.xml 的protocol 改为
    
    org.apache.coyote.http11.Http11AprProtocol

##重启查看日志


##参考

http://www.cnblogs.com/kgdxpr/archive/2013/08/07/3243657.html

http://blog.csdn.net/qingchn/article/details/7895851

http://tomcat.apache.org/native-doc/

http://neptune.iteye.com/blog/125101

http://www.cnblogs.com/chuncn/archive/2010/10/17/1853915.html

