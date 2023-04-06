---
title: 'arch-aio 免费高扩展性灵活且开放的all in one 系统的基本使用'
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

### 下载
1、 arch-aio：https://dev.leiyanhui.com/all-in-one/diy/  下载后解压到硬盘或U盘，要求分区格式是（exFAT/NTFS/UDF/XFS/Ext2(3)(4) ）   
2、 www.ventoy.net 下载 ，并安装到U盘 或者硬盘（新版支持无损安装了）   
3、 配置ventoy的自动启动以及VTOY_LINUX_REMOUNT ，可以使用arch-aio 下载链接里面的ventoy文件的json文件，具体参考 ventoy官网说明   
4、 配置 lnk.vtoy 等等，具体参考 ventoy官网说明，也可以在ventoy引导界面按下F2 浏览找到对应的 vhd.vtoy 文件   
### 启动
使用ventoy启动即可，具体参考 ventoy官网说明
### 登录arch-aio
使用默认用户 yanhui 或 root 登录。   
win客户端登录建议使用[MobaXterm免费版](https://mobaxterm.mobatek.net/download-home-edition.html)
### 修改密码
root和yanhui的密码 都要修改,yanhui 
```
passwd yanhui
passwd root
```
### 可选shell和neofetch
```
sudo pacman -S fish neofetch
#编辑
nano .bashrc
```
添加两行
```
neofetch
fish
```
neofetch 是登录后自动打印系统信息，fish是一个还不错的shell终端，命令补充和提示好用一些，另外一个不错的shell 叫做`nushell` 安装后启动命令是`nu`   
`.bashrc` 文件是你命令行登录后自动执行的一个文件。可以让它自动启动一些脚本方便使用。
### 检查和自动配置挂载
#### 检查
使用 `fdisk -l `  和 `lsblk` 查看分区情况，使用 `df -h` 查看挂载情况。 值得注意的是，/dev/mapper/ventoy 的两个分区 是arch-aio 虚拟磁盘镜像的两个分区分别对应efi和`/`
#### 配置自动挂载
建议使用`rc-local` 这个系统服务来处理，不建议使用fstab   
以下实例直接从yanhui这个用户下处理
```
cd ~
mkdir -p  mnt/nvme   mnt/efi  mnt/exfat
# 测试
sudo mount /dev/nvme0n1p6 /home/yanhui/mnt/exfat
sudo mount /dev/nvme0n1p5 /home/yanhui/mnt/nvme
sudo mount /dev/nvme0n1p1 /home/yanhui/mnt/efi
```
没问题后修改 `sudo nano /etc/rc.local.d/auto-mount-and-start-aio.sh`   内容
```
mount /dev/nvme0n1p6 /home/yanhui/mnt/exfat
mount /dev/nvme0n1p5 /home/yanhui/mnt/nvme
mount /dev/nvme0n1p1 /home/yanhui/mnt/efi
```
重启测试
### 迁移和配置docker和kvm
#### docker
```
sudo systemctl stop docker
sudo rsync -avzP /var/lib/docker  /home/yanhui/mnt/nvme
sudo mv /var/lib/docker  /var/lib/docker.bak && sudo ln -s /home/yanhui/mnt/nvme/docker /var/lib/
# 完成后 rm -rf /var/lib/docker.bak
```
#### kvm/libvirtd
此步骤 可以不处理
```
systemctl stop libvirtd
rsync -avzP /var/lib/libvirt  /home/yanhui/mnt/nvme
mv /var/lib/libvirt  /var/lib/libvirt.bak && ln -s /home/yanhui/mnt/nvme/libvirt /var/lib/
# 完成后 rm -rf /var/lib/libvirt.bak
```
#### 配置docker和libvirtd开机自动启动
先禁止他们的开机自动启动，改为从rc.local这个服务里面启动，已方便在挂载完成后再启动对应的服务    
```
systemctl disable docker
systemctl disable libvirtd
```
修改   `sudo nano /etc/rc.local.d/auto-mount-and-start-aio.sh`   内容 添加两行
```
systemctl start docker
systemctl start libvirtd
```
> 2022-12-29版本这个默认文件的libvirtd少了一个字母d，注意修改以下
重启

### 安装一个系统（win10为例）
#### 安装镜像
先去弄一个win安装镜像,我这里有了 
```
mnt/exfat/iso/SW_DVD9_Win_Pro_10_22H2_64BIT_ChnSimp_Pro_Ent_EDU_N_MLF_X23-20012.ISO
```
对应的目录是 `/var/lib/libvirt/images `
#### windows virio驱动镜像
```
sudo pacman -S wget
cd mnt/exfat/iso/
wget -c https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/archive-virtio/virtio-win-0.1.225-2/virtio-win-0.1.225.iso
```
这个网站访问速度很慢，最好挂个代理，或者下载后用MobaXterm上传到对应目录
#### 准备安装
MobaXterm或者其他支持x11的ssh客户端里面 用yanhui登录，执行
```
systemctl start libvirtd
virt-manager
```
会弹出virt-manager 图形界面窗口，先connect上默认的`QEMU/KVM`   
点 file -> New virual Machine  选默认的  local install，点下一步 Forward   
点后面的浏览`Browse` 弹出来一个 locate Iso 管理窗口，点左下角的 ` + ` 号  Name 输入 ISO ，`type` 保持默认 `dir:` ,目标路径 Target 输入ISO储存路径`/home/yanhui/mnt/exfat/iso`  然后点`Finish` 结束

然后选中新建的iso里面的win10安装镜像， 返回虚拟主机的配置界面 去掉`automatically ...` 这个自动识别系统版本的勾号，在上面输入win10 搜索 选中对应的系统。然后 点下一步 Forward   

可能弹出一个权限的提醒，下一步忽略就好，分配cpu和内存。下一步 Forward   创建磁盘。参考前面iso的配置同样的方法创建一个专门放置vdisk的目录`/home/yanhui/mnt/nvme/vdisk`，并创建一个磁盘 win10-base.qcow2 在这里个目录，大小要14G以上哦，不然不够win10安装用的，下一步 Forward ，勾选上 `Customize configuration before install`



继续配置，添加一个ide磁盘cdrom 到 驱动virtio-win-0.1.225.iso。

默认磁盘改为 virtIO,缓存 unfase，

如果需要和宿主机文件的交互，添加 File System，选择 virtio-9p

网络接口等等 其他简单也配置以下，点`begin installxxx` 

开始安装 就好了。
找不到磁盘 记得加载驱动，完成后 记得装完整驱动。别的都简单 不用多说。 

网络如果不会配置桥接的话，暂时先用NAT

> win11 的话记得添加tpm 
> 桥接网络的配置 [点这里查看](https://dev.leiyanhui.com/all-in-one/arch-aio-use-bridge-network/)
