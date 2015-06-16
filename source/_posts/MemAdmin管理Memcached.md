title: "MemAdmin管理Memcached"
date: 2015-06-17 00:24:00
category: 系统配置
tags: [Memcached,MemAdmin]

---

*MemAdmin是一款可视化的Memcached管理与监控工具，基于 PHP5 & JQuery 开发，体积小，操作简单。*

{% blockquote 开源中国社区 http://www.oschina.net/p/memadmin MemAdmin首页、文档和下载 %}

主要功能：

- 服务器参数监控：STATS、SETTINGS、ITEMS、SLABS、SIZES实时刷新
- 服务器性能监控：GET、DELETE、INCR、DECR、CAS等常用操作命中率实时监控
- 支持数据遍历，方便对存储内容进行监视
- 支持条件查询，筛选出满足条件的KEY或VALUE
- 数组、JSON等序列化字符反序列显示
- 兼容memcache协议的其他服务，如Tokyo Tyrant (遍历功能除外)
- 支持服务器连接池，多服务器管理切换方便简洁
{% endblockquote %}


##查看依赖

###查看Apache版本
    apachectl -v

###查看PHP环境
    php -v

http://nan1hao.blog.51cto.com/753570/602610/

##安装Apache和PHP（如果没有的话）
http://blog.csdn.net/czp11210/article/details/8750506

##安装PHP的memcached扩展
    RPM –ivh php-pear-1.9.4-4.el6.noarch.rpm 

    RPM –ivh php-pecl-memcache-3.0.5-4.el6.x86_64.rpm

或 `yum install` 安装



##修改配置

追加内容

    echo "abcd" >> a.txt #命令示例

重启Apache 

    service httpd restart

找不到服务？http://blog.csdn.net/zwfcan/article/details/8231864

搜索Apache目录
    
    find / -name httpd.conf

##安装memadmin
    tar –zxvf memadmin-1.0.12.tar.gz
    mv memadmin  /var/www/html
    
    vi /etc/httpd/conf/httpd.conf
1.DocumentRoot "/var/www/html”
2.DirectoryIndex index.html index.html.var index.php
3.<Directory "/var/www/html”> ...


*很久没用了，还有些使用截图需要补上，待续...*
