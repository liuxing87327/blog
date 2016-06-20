title: "使用SDKMAN管理开发环境"
date: 2016-06-18 04:03:00
category: 系统配置
tags: [sdkman,gvm,开发环境]

---

{% blockquote github https://github.com/sdkman/sdkman-cli sdkman/sdkman-cli %}
SDKMAN 是用来在类Unix 系统中管理多个版本的开发环境的工具。提供命令行接口来安装、切换、删除、列出候选版本。
{% endblockquote %}

## 前戏
###缘由
写这篇博文的原因是因为最近在使用GVM(SDKMAN的前身)管理版本的时候，一直提示我一个异常“gvm offline disable”，
在网上找了一通都没解决，后来才发现是GVM已经弃用了，连原先的服务域名`http://api.gvmtool.net`都无法访问了。
所以导致一直无法下载新的东西，只能使用已经安装好的SDK，无奈我本地的groovy版本太老了...

本文不介绍GVM和SDKMAN是啥玩意，请自行谷歌！！！

### 卸载GVM
需要编辑者三个文件
- .bashrc
- .bash_profile
- .profile

然后删除下面类似的代码
```bash
#THIS MUST BE AT THE END OF THE FILE FOR GVM TO WORK!!!
[[ -s "/Users/liuxing/.gvm/bin/gvm-init.sh" ]] && source "/Users/liuxing/.gvm/bin/gvm-init.sh"
```

最后，删除[~/.gvm]或[~/.sdkman]文件夹

## 安装SDKMAN

打开终端运行
> curl -s "https://get.sdkman.io" | bash

然而，你也有可能会出现，一直卡着不动了...

有两种方案
1.把https改为http，然后重试
2.从浏览器访问[https://get.sdkman.io]，然后把打开的内容另存为脚本，然后执行它

