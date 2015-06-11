title: "在windows和linux上安装ImageMagick与Jmagick"
date: 2013-12-20 00:29
category: [Java]
tags: [jmagick,linux,windows]
---

##Linux安装Jmagick
###下载JMagick和ImageMagick
http://downloads.jmagick.org/6.4.0/jmagick-6.4.0-src.tar.gz  
http://downloads.jmagick.org/6.4.0/ImageMagick-6.4.0-0.tar.gz  

文件存到一个指定目录，如/usr/local/ImageMagick，该目录就是后续的安装目录

###安装依赖包
yum install libpng 
yum install libpng-devel 
yum install libjpeg 
yum install libjpeg-devel 
yum install gd 
yum install gd-devel 
yum install libtiff 
yum install libtiff-devel 
yum install gcc （很重要）
yum install zlib(可选)
是zlib通用压缩库，图形格式png使用zlib中的deflate压缩算法


###安装ImageMagick

`cd /usr/local/ImageMagick`
 
1.解压

`tar zxf ImageMagick-6.4.0-0.tar.gz `

2.切换到解压目录

`cd ImageMagick-6.4.0`

3.编译源文件

`./configure --prefix=/usr/local/ImageMagick --enable-shared --without-perl --with-quantum-depth=8`

configure参数说明：
--enable-shared 编译成共享库(建议)
--disable-static 不编译成静态库
--with-quantum-depth=8 使用8位色深。我的1200万像素数码相机，照出的图片就是8位色深。(建议)
--with-windows-font-dir=目录 ，指明字体文件的目录（后面将人工复制中文字体文件到这个目录）(可选)
--disable-openmp 禁用多线程，使用多线程性能并没有提高，但CPU占用达到了100%，所以禁用了。(可选)


###安装（需要几分钟时间）
`make && make install`

 
由于ImageMagic被安装在我们自行指定的/usr/local/ImageMagick，后面安装JMagic会找不到需要用到的ImageMagic的命令和库，因此需要配置一下操作系统： 


1.编辑/etc/profile里面的PATH环境变量：
`vi /etc/profile`

结尾加入：
`export PATH=/usr/local/ImageMagick/bin:$PATH `

2.编辑/etc/ld.so.conf
`vi /etc/ld.so.conf`

加入：
`/usr/local/ImageMagick/lib `

3.执行命令，将ImageMagick的库加入系统联接库：
`ldconfig `

4.重新登录
5.查看版本：
`convert --version`

 
例如：

`convert —version`
`Version: ImageMagick 6.4.0 12/27/11 Q16 http://www.imagemagick.org`
`Copyright: Copyright (C) 1999-2008 ImageMagick Studio LLC`


###安装并配置JMagick


1.解压
`tar xzvf jmagick-6.4.0-src.tar.gz `

2.解压源码移入/usr/local/jmagick
`mv 6.4.0/ /usr/local/jmagick`

3.切换到目录
`cd /usr/local/jmagick/`

4.查看所有环境变量
`env`

5.编译源文件
`./configure --with-java-home=/usr/java/jdk1.7.0/ --with-magick-home=/usr/local/ImageMagick`

6.安装（需要几分钟）
`make && make install`

7.保证已经配置好环境变量
`$JAVA_HOME 和  $JRE_HOME`
切换到用户根目录查看
`vi /.bash_profile`
`vi /home/loupan/.bash_profile`
...

如：
`export JAVA_HOME=/usr/local/java/jdk1.7.0_45`
`export JRE_HOME=/usr/local/java/jdk1.7.0_45/jre`
`export PATH=$JAVA_HOME/bin:$PATH`
`export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar `


8.拷贝依赖文件到jdk
`cp /usr/local/jmagick/lib/libJMagick-6.4.0.so $JAVA_HOME/jre/lib/amd64/libJMagick.so`
`cp /usr/local/jmagick/lib/jmagick-6.4.0.jar $JRE_HOME/lib/jmagick.jar`


备注：

`$JAVA_HOME=/usr/java/jdk1.7.0/`
`$JRE_HOME=$JAVA_HOME/jre`

##windows安装Jmagick 

以ImageMagick-6.3.9-0-Q16-windows-dll.exe安装到windows为例，

下载ImageMagick-6.3.9-0-Q16-windows-dll.exe和jmagick-win-6.3.9-Q16.zip

1.下载ImageMagick-6.3.9-0-Q16-windows-dll.exe安装，在那个多选界面请选择所有内容(是一些依赖的库，避免出现不可预知的错误)。
2.检查看系统环境变量PATH里面是否添加了环境变量，如果没有需要添加安装路径。
3.将jmagick.dll拷贝到C:/windows/system32目录下。
4.将jmagick.dll拷贝到%TOMECAT%/bin目录下。
5.将jmagick.dll拷贝到%JAVA_HOME%/bin目录下。
6.还要把jmagick.jar复制到%JAVA_HOME%/jre/lib/ext下
7.还要把jmagick.jar复制到%TOMCAT%/lib下
8.使用时将jmagick.jar拷贝到项目，在类里加上静态块，用系统的类加载器指定，否则会出现类无法加载。

```java
static {
     // 如果部署到WEB应用，就要加下面这句。不然会报“UnsatisfiedLinkError: no JMagick in
    // java.library.path”。
    System. setProperty("jmagick.systemclassloader", "no");
}
```
 
##参考
http://xlogin.blog.51cto.com/3473583/717321
http://elf8848.iteye.com/blog/455675