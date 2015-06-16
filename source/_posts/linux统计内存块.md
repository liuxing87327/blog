title: "linux统计内存块"
date: 2015-06-17 01:02:00
category: 系统配置
tags: [Java,linux]

---
*纯粹备忘*

##统计内存块
pmap -x $pid | awk '{ if($3 > 64000 && $3 < 65537) count++ } END { print count }’

##导出内存块明细
pmap -x $pid > pmap.log

##导出核心进程内存

sudo gdb -q --pid=4990
 
--pid后面跟着的是jvm的进程id
(gdb) generate-core-file 
 
这里调用命令生成gcore的dump文件
(gdb) detach 
 
detach是用来断开与jvm的连接的
(gdb) quit

指定内存块：
内存地址从pmap结果中查询
 dump memory memory.bin 0x0007f5f38000000 0x0007f5f394af000

导出核心进程内存（正式库数据太大不建议）
gdb --pid $pid
gcore   [文件名]    #   产生core dump文件

http://blog.chinaunix.net/uid-24020646-id-2419921.html

##核心进程内存转换为heap dump
$JAVA_HOME/bin/jmap -dump:format=b,file=heap.hprof $JAVA_HOME/bin/java core.63278 
/usr/java/jdk1.8.0_40/bin/jmap -dump:format=b,file=heap.hprof /usr/java/jdk1.8.0_40/bin/java memory.bin
/usr/local/java/jdk1.8.0_20/bin/jmap -dump:format=b,file=heap.hprof /usr/local/java/jdk1.8.0_20/bin/java memory.bin
http://itindex.net/detail/50907-jmap-gcore-dump

http://www.ibm.com/developerworks/cn/java/j-memoryanalyzer/


##导入本地使用MAT工具分析

##使用libtcmalloc优化linux内存管理
gperftools+libunwind


##查找文件

find / -name "libunwind*” 


jhat -J-Xmx1024M heap.hprof 
