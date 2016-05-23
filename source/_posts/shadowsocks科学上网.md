title: "shadowsocks科学上网"
date: 2015-09-09 10:09:00
category: 系统配置
tags: [linux,ss,shadowsocks]

---

由于一些大家都懂的原因，访问GitHub或者Google需要进行特殊的”配置”。
之前一直用红杏的方式，后来红杏不行了，最近发现了ShadowSocks，折腾一下发现稳定性、速度都比红杏要好很多，推荐给大家使用。

---
`有内网代理的可以直接跳到`[配置浏览器插件](#配置浏览器插件)
`有内网代理的可以直接跳到`[配置浏览器插件](#配置浏览器插件)
`有内网代理的可以直接跳到`[配置浏览器插件](#配置浏览器插件)
重要的事情说三遍!

---

先购买ss套餐，推荐下面两个

只是看看网页,推荐**按需订购套餐**,支持 电脑/手机/平板 等多客户端同时在线
http://ss.fanque.wang/aff.php?aff=102

这个是按年付费,只能单平台同时在线,限制比较多,但是不限流量
https://portal.shadowsocks.com/aff.php?aff=2773


##安装shadowsocks

- 下载各平台的软件
    Windows 链接: http://pan.baidu.com/s/1mggYFNy 密码: w9qw
    MAC 链接: http://pan.baidu.com/s/1o6EK2Oq 密码: 4erq
    Android 链接: http://pan.baidu.com/s/1dDflgBZ 密码: 75wt
- 运行软件,设置账号信息(以`MAC`平台为例)
    ![打开设置](/images/ss/01.png)
    ![设置服务器信息](/images/ss/03.png)
    ![使用自动代理模式](/images/ss/02.png)

##配置浏览器插件

`以下教程的使用场景是chrome`

**离线插件地址**
链接: http://pan.baidu.com/s/1o607yjg 密码: wxrt

**安装到chrome**

![chrome安装插件](/images/ss/04.png)

**配置插件**

1.使用自己购买的服务,启动shadowsocks后,本机启动一个socks5的代理
代理协议选择`socks5`,地址是`127.0.0.1`,端口是`1080`
配置完成后,点应用选项保存

2.使用内网代理地址的设置对应的 协议 IP 端口
socks5 192.168.3.51 1080
http 192.168.3.51 8123
配置完成后,点应用选项保存

![配置代理信息](/images/ss/05.png)

3.设置自动切换
![添加规则列表](/images/ss/06.png)


规则列表地址:`http://autoproxy-gfwlist.googlecode.com/svn/trunk/gfwlist.txt`

注意：上面的规则地址已过期

*[new]*规则列表地址:`https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt`

![添加规则列表](/images/ss/07.png)


![改为自动切换](/images/ss/08.png)


##视频教程

<object width='541' height='450'><param name='allowFullScreen' value='true'><param name='movie' value='http://i7.imgs.letv.com/player/swfPlayer.swf?autoPlay=0&id=23744815'/><embed src='http://i7.imgs.letv.com/player/swfPlayer.swf?autoPlay=0&id=23744815' width='541' height='450' allowFullScreen='true' type='application/x-shockwave-flash'/></object>

有问题欢迎留言交流!!!