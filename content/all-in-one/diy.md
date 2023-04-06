---
title: '选择arch-aio，彻底抛弃pve esxi unraid'
date: 2022-12-29T17:14:20+08:00
tag:
  - arch
  - aio
  - ventoy
  - pve
  - unraid
  - kvm
categories: 
  - arch
  - aio
  - pve
  - unraid
  - ventoy
  - kvm
---

> 本文停止更新，新版查看 [https://dev.leiyanhui.com/all-in-one/arch-aio/](https://dev.leiyanhui.com/all-in-one/arch-aio-use)

## 前言
### 经历
前前后后用过很多 all in one 系统，hyper-v  pve6-7.3  unraid，esxi 还有 群晖 爱快，感觉到很多不方便的地方。所以自己做了一个基于arch 和 ventoy的all in one 母系统。

缺点：门槛略微高一点点。你需要会用linux的基本命令。包括不限`ln mount `   
优点：灵活，免费,不占用额外分区，强大扩展，较新并非常稳定的内核
### 使用方法
https://www.123pan.com/s/EqorVv-K2nPA 提取码:arch  
下载后解压，然后用ventoy，用支持x11转发的ssh客户端（例如 `MobaXterm`） 登录  用户名`yanhui` 密码 `1`
root 密码也是 `1` 但是root默认禁止登录。

简明使用方法：[https://dev.leiyanhui.com/all-in-one/arch-aio-use/](https://dev.leiyanhui.com/all-in-one/arch-aio-use/)
#### 虚拟机
先挂载分区，并 ` ln -s /var/lib/libvirt/images  /目录` 或者直接把这个目录挂载到分区
MobaXterm 中运行 `virt-manager` 会弹出` virt-manager `界面  
如果你只想用命令行，也可以卸载掉它。 
> 在不支持x11 转发的设备上管理虚拟机，请继续看后面
#### docker
先挂载分区，并ln /var/lib/docker 目录 或者直接把这个目录挂载到分区
如果要图形界面，可以安装portainer
```
docker run -d -p 8000:8000 -p 9443:9443 \
--name portainer --restart=always \
-v /var/run/docker.sock:/var/run/docker.sock \
portainer/portainer-ce:latest
```
然后浏览器访问：https://ip:9443

#### 新建用户
新建用户请 添加到wheel组 和 libvirt  组，例如用户名  xiaolei
```
sudo useradd -m -G wheel -s /bin/bash xiaolei
sudo usermod -a -G libvirt xiaolei
```
然后设置密码
```
sudo passwd xiaolei
```
#### 清理日志和缓存文件
```
sh /root/del-log.sh
```
#### 扩容
如果默认容量不够，需要扩容。请参考
[https://dev.leiyanhui.com/arch/ventoy-vdisk-resize](https://dev.leiyanhui.com/arch/ventoy-vdisk-resize)
#### exfat4 和ntfs支持
```
sudo pacman -S exfat-utils
sudo pacman -S ntfs-3g
```
> exfat 和 ntfs 在linux下性能一般，ntfs稳定性更是一般，不建议高强度使用
#### 关于sudo 和 vi vim
用doas 和nano替代的 软连接对应的sudo 和 vi vim
#### 运行浏览器和其他软件
请尽量挂载到其他分区后用appimages
例如火狐
```
sudo pacman -S fuse #AppImage 依赖
sudo pacman -S dbus-glib # 火狐依赖
cd /挂载目录
wget https://apprepo.de/appimage/download/firefox --output-document=Firefox.AppImage
chmod +x ./*.AppImage
./Firefox.AppImage
```
火狐在x11转发下首次启动很卡，等一会关掉再次打开就流畅很多
中文字体支持
```
sudo pacman -S wqy-zenhei
```
中文输入法：[https://wiki.archlinuxcn.org/wiki/Fcitx5](https://wiki.archlinuxcn.org/wiki/Fcitx5)
#### 开机自动启动脚本
rc-local服务：`/usr/lib/systemd/system/rc-local.service` 和 `/etc/rc.local`

sh脚本放到 /etc/rc.local.d 目录
#### 开机自动挂载
建议放到 /etc/rc.local.d目录。
参考编辑文件`sudo nano /etc/rc.local.d/auto-mount-and-start-aio.sh`  
详细说明参考这里：[https://dev.leiyanhui.com/arch/auto-mount-dock-kvm](https://dev.leiyanhui.com/arch/auto-mount-dock-kvm)
#### 开机自动启动其他分区的vm 或者docker
参考编辑文件`sudo nano /etc/rc.local.d/auto-mount-and-start-aio.sh`  
详细说明参考这里：[https://dev.leiyanhui.com/arch/auto-mount-dock-kvm](https://dev.leiyanhui.com/arch/auto-mount-dock-kvm)
#### 桥接虚拟机网络
[https://dev.leiyanhui.com/all-in-one/arch-aio-use-bridge-network/](https://dev.leiyanhui.com/all-in-one/arch-aio-use-bridge-network/)

#### 关于远程访问虚拟机
如果客户端非ios设备，网络尚可 直接用支持x11转发的ssh客户端管理   
如果是ios设备，因为没有可用的客户端，建议kvm 或者docker中运行一个 有桌面的win 或者linux，再远程过去.再套娃访问宿主机  
> docker 运行linux桌面参考 [https://dev.leiyanhui.com/docker/run-deskto-gpu-usb/](https://dev.leiyanhui.com/docker/run-deskto-gpu-usb/)  
> 上文中的docker 支持网页vnc和xrdp微软远程链接，你甚至可以v挂载虚拟磁盘目录进去，里面有一些基本的软件，包括浏览器完成网盘备份等操作。

当然，你也可以直接在宿主系统上安装一个gnome kde等桌面系统，然后开xrdp或者vnc 或者 向日葵 rustdesk 等等 尽情想想...
#### 关于win/winpe下访问
因为镜像是vhd格式的，在win系统和多数winpe下可以直接挂载。 brtfs分区的访问请安装驱动，[详情](https://dev.leiyanhui.com/win/winbtrfs/)

### 系统内软件包
#### 母盘
archlinux-2022.12.01-x86_64.iso  arch滚动更新，我用的母盘的版本不影响你 你可随意随时升级到最新版   
用archinstall 直接安装的 minimal 模式  
源 改为清华源，未开启archlinuxcn源，如果你需要aur软件包自行开启 [https://mirrors.tuna.tsinghua.edu.cn/help/archlinuxcn/](https://mirrors.tuna.tsinghua.edu.cn/help/archlinuxcn/)     
内核考虑到安卓x86 选择 linux-zen   
#### 额外的软件包和配置
##### 基本
开启了非root用户的x11转发，依赖`xorg-xauth`
基本包： `openssh dhcpcd nano doas`   
ventoy依赖：`which lvm2`
> sudo用doas替换了，但是同时ln到/bin/sudo  
> vi 未安装 如果不喜欢用nano自行处理

##### kvm和docker相关
` exfat-utils rsync  ` `qemu-base samba dnsmasq dmidecode iptables-nft bridge-utils openbsd-netcat libvirt virt-manager docker`
### 开机自定启动的额外服务
`sshd  dhcpcd`

> 注意 docker 和 kvm的libvirt 默认是通过 `/etc/rc.local.d/auto-mount-and-start-aio.sh` 启动的


### 我对all in one的一点要求
自定义、灵活、内核新，不破坏原来硬盘。
### 简单说一下各个系统的缺陷
https://dev.leiyanhui.com/all-in-one/other-os/
### 为什么选择arch
- 只使用qemu 等几个软件的情况下，也不存在滚挂的可能性。除非你真的大半年不更新。
- 相对 debian centos系，内核新，很多新特性都可以支持
- wiki资料丰富，扩展容易
### 为什么选择ventoy引导
ventoy新版已经有限的支持无损安装到硬盘，同时也兼容grub4dos
### 磁盘容量的纠结
控制到3.7G 以内是最好的选择，但是可用空间实在是太少了。安装qemu的常用依赖后，剩余空间只有几百M。无法满足正常使用的需求。
考虑许久后，决定抛弃兼容fat32和4G U盘的不实际的想法，改为7.3G。 即便是要安装一个桌面系统 应该也勉强够用。
另外 home opt等目录都可以挂载出来。
### 磁盘格式的选择
ventoy只支持 固定大小的vhd  vdi  raw/img 三种格式 虚拟磁盘启动linux。kvm只支持其中raw，vbox支持前两者，win可以直接挂载vhd   
linux可以用qemu-img随意转换其他格式，vhd文件也可以转换成raw/img后挂载。
综合考虑，还是选择了vhd格式
### vdisk内efi分区的大小
考虑到换内核的可能性和内核更新的情况，还是用多数情况适用的512M（正常只占用60m左右）
### vdisk内根分区文件系统的选择
我这里选择brtfs，支持压缩和快照。并且几乎所有winpe下都可以 右键加载驱动后访问，虽然不够稳定但是要比其他文件系统简单一些。
### swap的选择
默认不开启，如果需要自行创建swap文件即可


