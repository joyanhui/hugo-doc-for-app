---
title: '一个可以不动硬盘分区，直接从一个文件启动的proxmox（pve） 打造all in one '
date: 2022-12-22T03:14:20+08:00
tag:
  - aio
  - pve
  - unraid
  - kvm
categories: 
  - aio
  - pve
  - unraid
  - kvm
---
## 有什么用
- 可以在不修改你磁盘分区的情况下，只占用5G的硬盘空间(新版占用12G，剩余7G+)，使用vhd虚拟磁盘的方式 启动一个性能和功能完全不受影响在物理机运行的pve系统。
- 可以和其他系统共存并不需要额外的分区
- 可以同时存在多个pve系统 自由切换，例如一个开启gvt-g的一个开启核显直通的
- 可以很方便的整个系统备份和恢复
## 开始使用
1、准备一个空白U盘，或者硬盘安装好 ventoy,下载 解压 pve.vhd 文件到 任意一个分区（exFAT/NTFS/UDF/XFS/Ext2(3)(4)格式的分区），重命名为  pve.vhd.vtoy       
2、开机用ventoy磁盘引导，按下F2 浏览，找到 pve.vhd.vtoy，正常启动  用户名root 密码root    
3、配置网卡和ip等 重启，[详细说明看这里](https://dev.leiyanhui.com/pve/changip-and-netcard)      
4、挂载其他磁盘修改虚拟机和ct容器目录  详细说明看后文 
> 只是修改了pve内部的引导挂载路径,使用方法 和正常安装的pve一样 ，不影响直通和其他任何功能。更不影响任何性能。
## 进阶
1、扩容方法（以后补 剩余7G其实足够了... 先用工具扩容vhd，然后在从pve内部处理lvm即可）    
2、配置开机启动启动pve [查看这里](https://dev.leiyanhui.com/os/ventoy-auto-boot)    
3、不使用ventoy的启动（整理中..） 
## pve通用进阶教程
- [开启pci直通简明教程](https://dev.leiyanhui.com/pve/igpu-pass/) 
- [开启虚拟化嵌套](https://dev.leiyanhui.com/pve/pve-vm-esxi)
- [安装macos](https://dev.leiyanhui.com/pve/install-macos-ventura)
## 更新说明
2022-12-23 
- 更新为12G 容量，剩余空间7G左右，放弃5G镜像，之所以做一个容量大一些的，是发现1.3G的剩余空间在安装几个常用软件后经常就占满了，再次去扩容过于折腾。而7G容量足够安装常用软件，甚至可以安装上一个桌面系统。
- ~~安装firefox 和 hrub~~
- 为了减少体积，网盘包改为 7zip 格式 linux解压命令，debian/ubuntu:`apt-get install p7zip && 7z x pve20221223.vhd.vtoy.7z -r -o /目标路径 `
## 概述
- 基于 Virtual Environment 7.3-3 内核最新版（目前是 5.15.74-1）
- 基于vtoyboot-1.0.25（ventoy） 支持硬盘/U盘安装的 vtoyboot，也支持grub4dos插件启动，推荐用vtoyboot
- 硬盘占用空间5G  剩余空间1.1G 左右（可扩容）
- 默认ip：5G版：10.0.0.8  12G初始版：10.0.0.120
- 默认用户名root  密码 root
- Local-lvm 默认没有启用(低容量会默认禁用)Local开启全功能
- 开启x11 转发，方便在上面运行图形化软件，支持浏览器 文件管理等所有linux图形化软件 
- 开启了sftp 可以用alist rclone之类的软件挂载 [详情](https://dev.leiyanhui.com/pve/open-sftp)
- 修改为国内源
- ~~显示cpu主频 cpu温度 nvme硬盘温度 sata硬盘温度~~
- ~~去掉订阅弹窗~~
- 支持升级
- 不影响pci直通等操作
> 绝对没有后门，所有修改过的文件，都备份了一份在 /root/bak-sources, 所有的历史命令和操作记录 都保留，[你可以自行删除](https://dev.leiyanhui.com/pve/del-log)
## 计划中的功能
精简到4G 以内 方便fat32 分区使用
## 已经处理的步骤和常见问题
### 挂载其他分区
查看所有磁盘和分区
```
fdisk -l 
```
挂载一个分区到/mnt/nvme1
```
mkdir /mnt/nvme1
mount /dev/nvmen0p2 /mnt/nvme1
```
开机自动挂载，修改fstab 或者 添加开机自动执行脚本。
查看本文 [https://dev.leiyanhui.com/pve/auto-mount-exfat](https://dev.leiyanhui.com/pve/auto-mount-exfat)
更多内容请自行google "linux 挂载分区"关键词
> pve 可以挂载ntfs分区，但是不建议长期使用，性能和稳定性都有问题。
### 修改ct和虚拟机主机的路径
[https://dev.leiyanhui.com/pve/pve-mv-dir](https://dev.leiyanhui.com/pve/pve-mv-dir)

### 国内源
已经更换到清华源 [https://dev.leiyanhui.com/pve/guonei](https://dev.leiyanhui.com/pve/guonei)

### ~~显示温度~~
~~已经处理过，不需要你再处理~~
```
cp /usr/share/perl5/PVE/API2/Nodes.pm /root/bak-sources
cp /usr/share/pve-manager/js/pvemanagerlib.js  /root/bak-sources
#覆盖文件后 执行下面命令

apt-get install lm-sensors && apt-get install nvme-cli && apt-get install hddtemp && chmod +s /usr/sbin/nvme && chmod +s /usr/sbin/hddtemp && chmod +s /usr/sbin/smartctl && systemctl restart pveproxy
```

### ~~订阅弹出窗口~~
~~已经处理过，不需要你再处理~~
```
cp /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js  /root/bak-sources
sed -Ezi.bak "s/(Ext.Msg.show\(\{\s+title: gettext\('No valid sub)/void\(\{ \/\/\1/g" /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js && systemctl restart pveproxy.service
```
可能需要清理缓存
### X11 转发
已经处理过，不需要你再处理：[https://dev.leiyanhui.com/pve/x11](https://dev.leiyanhui.com/pve/x11)
用 Xming 或者MobaXterm 连接上，安装一个图形界面软件 测试，比如 `apt install firefox-esr thunar` 然后执行firefox 即可打开火狐，thunar 也是一个简单的文件管理工具。
> 注意：x11 转发 只适合简单的场景使用。如果你要流畅，还是考虑 扩容后安装一个完整的桌面系统kde gnome等，然后xrdp  vnc或者todesk之类的连接上使用。

### 清理日志~~和历史命令~~
[https://dev.leiyanhui.com/pve/del-log/](https://dev.leiyanhui.com/pve/del-log/)
### 系统升级到最新版
```
apt update && apt full-upgrade
cd /root/vtoyboot-*/ && sh vtoyboot.sh
```
### 更新vtoyboot
```
cd /root/vtoyboot-*/ && sh vtoyboot.sh
```
> 应该在系统更新和修改grub后，重启之前执行一次，详情参考  https://www.ventoy.net/cn/plugin_vtoyboot.html

### 怎么访问 vhd所在的分区
参考 [ventoy faq手册](https://www.ventoy.net/cn/faq.html) 启动类问题的说明第四条 ` 默认情况下，Linux系统启动之后，存放ISO文件的分区无法挂载（会提示 Busy）。如果确实需要挂载，请使用 全局控制插件 中的 VTOY_LINUX_REMOUNT 选项。 `

[也可以查看本文 关于自动启动的说明](https://dev.leiyanhui.com/os/ventoy-auto-boot)

### winpe/win下如何备份和配置pve
#### 备份
直接复制压缩 XXX.vtoy  文件
#### 配置ventoy
建议用完整win，或者firpe，然后 用它自带的工具 VentoyPlugson.exe ，默认地址是 http://127.0.0.1:24681
#### 修改pve内部文件
重命名为 .vhd，然后挂载到Windows，而后用支持ext4分区的软件来操作 推荐 ExtFS 最稳定，免费10天，也有XXX的版本  
[https://china.paragon-software.com/home-windows/extfs-for-windows/download.html](https://china.paragon-software.com/home-windows/extfs-for-windows/download.html)


## ventoy和pve的使用需要注意
- ventoy 不支持差分vhd磁盘，不要尝试改为vhd文件后再次差分，但是可以自己转换为vdi或者img格式
- 修改grub后或者内核升级后记得运行一次 vtoyboot.sh 再重启。平时不需要
- vhd.vtoy文件可以放在ntfs分区，不影响使用。但不建议pve内的虚拟机放在ntfs分区，所有的非win系统，对ntfs的支持都不够稳定。
- 记得修改密码！！！！！
## 下载地址
如果失效，请联系 joyanhui@qq.com

https://www.123pan.com/s/EqorVv-fPnPA 提取码:0pve

其他未尽事宜 请参考ventoy手册：[https://www.ventoy.net/cn/doc_news.html](https://www.ventoy.net/cn/doc_news.html)
