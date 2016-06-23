title: "使用google-perftools优化tomcat"
date: 2015-06-17 00:41:00
category: 系统配置
tags: [linux,perftools,tomcat]

---

{% blockquote 开源中国社区 http://www.oschina.net/p/perftools Google PerfTools首页、文档和下载 %}
这个工具可让开发创建更强大的应用程序，特别是那些用C++模版开发的多线程应用程序，包括{% link TCMalloc http://www.oschina.net/p/tcmalloc %}, heap-checker, heap-profiler 和cpu-profiler。
{% endblockquote %}

## 前置依赖
避免后续安装错误

    yum install -y gcc*
    yum install zlib* openssl* -y   

## 安装
切换到工作目录
    
    cd /usr/local/src 或 ~/src

下载：
    
    wget http://download.savannah.gnu.org/releases/libunwind/libunwind-0.99-alpha.tar.gz
    wget http://googledrive.com/host/0B6NtGsLhIcf7MWxMMF9JdTN3UVk/gperftools-2.4.tar.gz

1.针对 64 位操作系统必须安装 libunwind 库
    
    tar zxvf libunwind-1.1.tar.gz
    cd libunwind-1.1/
    CFLAGS=-fPIC ./configure --enable-shared --enable-frame-pointers
    make CFLAGS=-fPIC
    make CFLAGS=-fPIC install
    cd ../

查找：
    
    find /usr/ -name “libunwind*”

卸载：
    
    make CFLAGS=-fPIC uninstall

2.安装 google-perftools 优化

    tar zxvf gperftools-2.0.tar.gz
    cd gperftools-2.0/
    ./configure --enable-shared --enable-frame-pointers
    make && make install
    echo "/usr/local/lib" > /etc/ld.so.conf.d/usr_local_lib.conf
    /sbin/ldconfig

`有依赖没有安装?`

    ./configure --enable-shared --enable-frame-pointers  

`make check 依然报错？`

http://xkorey.iteye.com/blog/1648567

`./libtool: line 1125: g++: command not found`

    yum install -y gcc* 


tomcat启动程序配置

    export LD_PRELOAD=/usr/local/lib/libtcmalloc.so

查看是否生效
    
    /usr/sbin/lsof -n | grep tcmalloc

## 参考
### 示例
https://www.centos.bz/2012/01/google-perftools-speed-up-mysql-tcmalloc/
http://blog.sina.com.cn/s/blog_8d05143b01012b87.html
http://xkorey.iteye.com/blog/1648567
http://shopwwi.com/thread-673-1-1.html
http://blog.csdn.net/wind19/article/details/10381291

http://blog.chinaunix.net/uid-20687780-id-3029851.html
http://blog.hackroad.com/operations-engineer/linux_server/1285.html

### linux安装软件
http://www.cnblogs.com/chuncn/archive/2010/10/17/1853915.html

http://www.educity.cn/wenda/353955.html
    
