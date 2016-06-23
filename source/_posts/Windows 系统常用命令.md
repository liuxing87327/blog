title: "Windows 系统常用命令"
date: 2014-03-20 10:54
category: 系统配置
tags: windows

---

来自小辉哥

## 必备命令
```bash
msinfo32,查看系统各种信息（环境变量、驱动等等）
gpedit.msc 组策略编辑
gpupdate /force 组策略更新 
mstsc 远程桌面 
mmc控制台 
services.msc 服务配置 
taskmgr 任务管理器
regedit：注册表编辑器
compmgmt.msc 计算机管理
devmgmt.msc 设备管理
ServerManager.msc 服务器管理
gpmc.msc组策略管理
ipconfig 查看ip地址信息
tasklist /M 
    显示进程引用的所有dll

tasklist /SVC 
    显示每个进程里的服务。配合find 命令，可以筛选指定服务的进程信息 
    例如： tasklist /SVC | find "AjaxCenter"

fsutil fsinfo ntfsinfo c:
    查看文件系统信息，可知道IO block size,对程序优化也有借鉴作用。
```

## 其他常用命令
```bash
sysdm.cpl：查看计算机属性
dxdiag ： 查看DirectX相关信息,也可以查看内存
devmgmt.msc： 设备管理
wmic diskdrive  可以看出来磁盘和大小.
Wmic logicaldisk 可以看出来磁盘和大小
wmic volume .可以看到有几个盘，每一个盘的文件系统和剩余空间
fsutil volume diskfree c: 每个盘的剩余空间量，其实上一个命令也可以查看的
wmic cpu 上面显示的有位宽，最大始终频率， 生产厂商，二级缓存等信息
wmic memorychip 查看Windows7和2003的内存属性  
WMIC MEMLOGICAL 查看xp内存命令
wmic bios  BIOS信息
systeminfo 详细系统信息，内存，显卡，处理器
wmic  process get Caption,name,commandline | find "red5",可以获取进程调用命令的详细情况
ver 查看操作系统版本。
```