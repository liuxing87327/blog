title: "GitHub博客搭建"
date: 2015-06-14 02:52
category: 系统配置
tags: [GitHub Pages, Blog, Godaddy, DNSPod, Hexo]

---

*GitHub带你装逼带你飞！你值得拥有！*

---

## 介绍
*{% link GitHub https://github.com/ %}很好的将代码和社区联系在了一起，于是发生了很多有趣的事情，世界也因为他美好了一点点。*
*GitHub作为现在最流行的代码仓库，已经得到很多大公司和项目的青睐，比如{% link jQuery https://github.com/jquery/jquery %}、{% link Twitter https://github.com/twitter/bootstrap %}等。*
*为使项目更方便的被人理解，介绍页面少不了，甚至会需要完整的文档站，GitHub替你想到了这一点，他提供了{% link GitHub Pages http://pages.github.com %}的服务，不仅可以方便的为项目建立介绍站点，也可以用来建立个人博客。*

GitHub Pages有以下几个优点
- *轻量级的博客系统，没有麻烦的配置*
- *使用标记语言，比如 {% link Markdown http://markdown.tw %}*
- *无需自己搭建服务器*
- *根据GitHub的限制，对应的每个站有300MB空间*
- *可以绑定自己的域名*


当然他也有缺点
- *使用{% link Jekyll https://github.com/jekyll/jekyll %}模板系统，相当于静态页发布，适合博客，文档介绍等。*
- *动态程序的部分相当局限，比如没有评论，不过还好我们有解决方案。*
- *基于Git，很多东西需要动手，不像Wordpress有强大的后台。*


大致介绍到此，作为个人博客来说，简洁清爽的表达自己的工作、心得，就已达目标，所以Github Pages是我认为此需求最完美的解决方案了。

## GitHub配置
### 注册账号
传送口：https://github.com/join ，自行搞定，否则放弃吧...

**PS**：*`不要取奇怪的用户名，比如大小写混合，建议小写字母+数字组合，否则pages会碰到问题！`*

### 配置Pages
**新增仓库**：https://github.com/new
- Repository name：github账号.github.io
- Description：随便输入点描述
- public
- Initialize this repository with a README
- .gitignore 选择初始的文件忽略，我选的java
- Licenses：我选的NPL（GNU General Public License v2.0）

**配置**
- 选择右侧操作区的`settings`
- 选择`Launch automatic page generator`
- 输入一些基本说明，非必要
- 选择`Load README.md`
- 继续`Continue to layouts`
- 选择模板（随便选个）
- 发布`Publish page`
- 此时进入`settings`应该会有`Your site is published at http://username.github.io`的条提示，访问一下，神奇吧！
- 如果404，请检查你的仓库名或账号名，删除仓库重来，删除也是在`settings`最底部
      
## 绑定独立域名
### 购买域名

不绑定独立域名则可以直接跳到 **使用hexo**

传送门：https://www.godaddy.com 支持支付宝
域名的购买不用多讲，注册、选域名、支付，有网购经验的都毫无压力。
记得先找优惠券：http://www.dute.me

推荐几个翻译插件
{% link 多词典划译 https://chrome.google.com/webstore/detail/%E5%A4%9A%E8%AF%8D%E5%85%B8%E5%88%92%E8%AF%91/cdonnmffkdaoajfknoeeecmchibpmkmg %}
{% link Google翻译 https://chrome.google.com/webstore/detail/google-translate/aapbdbdomjkkjkaonfhkkikfgjllcleb%}

没有VPN？
注册红杏：http://honx.in/_U9m44oIaA3c2nFTX
公益红杏：http://help.honx.in/posts/view/32854 

### DNS解析
传送门：https://www.dnspod.cn/
- 首先添加域名记录，可参考DNSPod的帮助文档：https://www.dnspod.cn/Support
    添加域名记录后，进入会有个加载配置啥的，不要保存，使用默认的两个解析就行
- 在DNSPod自己的域名下添加一条{%link A记录 http://baike.baidu.com/view/65575.htm %}，地址就是Github Pages的服务IP地址：103.245.222.133（最好自行ping获取最新的ip）
- 在域名注册商处修改DNS服务:去Godaddy修改Nameservers为这两个地址：f1g1ns1.dnspod.net、f1g1ns2.dnspod.net。如果你不明白在哪里修改，可以参考这里：{% link Godaddy注册的域名如何使用DNSPod https://www.dnspod.cn/support/index/fid/119 %}
- 等待域名解析生效

### 绑定
在刚创建的GitHub仓库根目录下添加`CNAME`文件，写入你申请的域名，等待生效。

## 使用hexo
基于github pages的不足，我们使用hexo博客框架

因为hexo的文档写的太好了，就没我啥事了！

传送门：http://hexo.io/zh-cn/




有任何问题，欢迎评论交流！