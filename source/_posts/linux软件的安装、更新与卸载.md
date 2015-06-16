title: "linux软件的安装、更新与卸载"
date: 2014-04-25 01:52
category: 系统配置
tags: linux

---

##介绍
Linux常见的安装为tar，zip，gz，rpm，deb，bin等。可以简单的分为三类.

> - 打包或压缩文件tar，zip，gz等，一般解压后即可，或者解压后运行sh文件；
> - 对应的有管理工具的deb，rpm等，通常的这类安装文件可以通过第三方的命令行或UI来简单的安装。
  例如Ubuntu中的apt来安装；deb，Redhat中的yum来安装rpm；
> - 像.bin类，其实就是把sh和zip打包为bin，或把sh和rpm打包为bin等，当在命令行运行bin安装文件时，其实就是bin里面的sh来解压bin中的zip或安装rpm的过程

<br/><br/>

##安装
###rpm安装，更新与卸载

*RPM包，这种软件包就像windows的EXE安装文件一样，各种文件已经编译好，并打了包，哪个文件该放到哪个文件夹，都指定好了，安装非常方便，在图形界面里你只需要双击就能自动安装。但是有一点不好，就是包的依赖关系，很恶心。*

```bash
A. rpm安装

  1) 找到相应的软件包，比如jmagick-6.4.0-3.src.rpm，下载到本机某个目录；
  2) 打开一个终端，su 成root用户；
  3) cd jmagick-6.4.0-3.src.rpm所在的目录；
  4) 输入rpm -ivh jmagick-6.4.0-3.src.rpm

B.rpm更新
   #rpm -Uvh jmagick-6.4.0-3.src.rpm

C.rpm卸载

   1) 查找欲卸载的软件包 rpm -qa | grep ×XXX×
   2) 例如找到软件mysql-4.1.22-2.el4_8.4 ，执行rpm -e mysql-4.1.22-2.el4_8.4

 注意：查询软件的安装目录，用命令 rpm -ql mysql-4.1.22-2.el4_8.4
```
 
###以.bin结尾的安装包

*bin类似rpm包安装，也比较简单*

```bash
bin安装
    1) 打开一个SHELL，即终端
    2) 用CD 命令进入源代码压缩包所在的目录
    3) 给文件加上可执行属性：chmod +x ******.bin(中间是字母x，小写)
    4) 执行命令：./******.bin 或者 直接执行 sh ******.bin

   bin卸载
     把安装时中选择的安装目录删除就OK
```

###tar.gz(bz或bz2等)结尾的源代码包

*这种软件包里面都是源程序，没有编译过，需要编译后才能安装*

```bash
源代码安装
    1) 打开一个SHELL，即终端
    2) 用CD 命令进入源代码压缩包所在的目录
    3) 根据压缩包类型解压缩文件(*代表压缩包名称)
　　   tar -zxvf ****.tar.gz
　　   tar -jxvf ****.tar.bz(或bz2)
    4) 用CD命令进入解压缩后的目录
    5) 输入编译文件命令：./configure(有的压缩包已经编译过，这一步可以省去)
    6) 然后是命令：make
    7) 再是安装文件命令：make install
```

###yum安装

*yum是rpm的管理工具，管理一个软件库，可以很好的解决依赖关系*

```bash
1) yum安装
    yum install -y 软件名

2) yum更新
    yum update -y  软件名

3) yum卸载
    yum remove -y 软件名
    或
    yum erase -y 软件名
```

###apt-get安装

```bash
apt-get 是deb的管理工具，类似yum
apt-get install package 安装包
apt-get reinstall package  重新安装包
apt-get upgrade 更新已安装的包
apt-cache rdepends package 是查看该包被哪些包依赖
apt-cache depends package 了解使用依赖
apt-get clean &&  apt-get autoclean 清理无用的包
apt-cache show package 获取包的相关信息，如说明、大小、版本等
apt-get remove package 删除包
apt-get purge package  删除包，包括删除配置文件等
```