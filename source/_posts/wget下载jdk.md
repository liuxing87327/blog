title: "wget下载jdk"
date: 2015-06-17 00:02:00
category: 系统配置
tags: [linux,wget]

---

*通常需要下载jdk时，直接用wget命令是不行的。那么，如何解决呢？*
*只需要在wget的时候加上一个特殊的cookie就可以搞定*

**JDK 7**

    wget --no-cookies --no-check-certificate --header "Cookie:gpw_e24=http%3a%2f%2fwww.oracle.com%2ftechnetwork%2fjava%2fjavase%2fdownloads%2fjdk7-downloads-1880260.html;oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/7u75-b13/jdk-7u75-linux-x64.tar.gz


**JDK 8**
    
    wget --no-cookies --no-check-certificate --header "Cookie:gpw_e24=http%3a%2f%2fwww.oracle.com%2ftechnetwork%2fjava%2fjavase%2fdownloads%2fjdk8-downloads-2133151.html;oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/8u40-b26/jdk-8u40-linux-x64.tar.gz