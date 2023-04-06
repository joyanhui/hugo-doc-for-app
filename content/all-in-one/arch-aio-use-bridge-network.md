---
title: 'arch-aio 的 virt-manager上配置桥接网络'
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

本文基于[ arch-aio](https://dev.leiyanhui.com/all-in-one/diy/)，如果你是其他系统，请先处理好相关依赖。
> 未完成

### 查看

```
cat /etc/qemu/bridge.conf
--------------
allow virbr0
```
virbr0  是自动创建的，并不能完成虚拟机 映射到外网   
继续查看其他情况
```
ip a
sudo brctl show 
```
判断出来物理口是 `enp1s0`
### 安装 networkmanager
我这里还是选用传统的networkmanager来处理，感觉更简单无脑一些。
```
pacman -S networkmanager
systemctl enable NetworkManager
systemctl start NetworkManager
```
### 配置桥接br0
根据你的实际情况修改ip网关dns等
```
nmcli connection add type bridge ifname br0 con-name br0 ipv4.addresses 10.1.1.1/24 ipv4.gateway 10.1.1.254 ipv4.dns 114.114.114.114 ipv4.method manual
```
我这里还是打算用dhcp所以执行下面的
```
nmcli connection add type bridge ifname br0 con-name br0
```
### 桥接网卡子网卡 关联物理卡
```
nmcli connection add type bridge-slave con-name br0-slave1 ifname enp1s0 master br0 
```
### 开启
```
nmcli connection up br0-slave1 
nmcli connection up br0
nmcli connection show 
# systemctl restart NetworkManager
```
此时网络会断开，原来ip无法连接，可以使用br0上的ip 重新连接。 如果是dhcp 可以从路由器看到多分配到br0上的ip。  
重启 NetworkManager 后，或者重启物理机器后，两个ip都可以连接。
### 后续可选处理
关闭br0的stp   
禁止物理卡开机启动 可以避免要占用两个ip的问题

### 开启ip4转发
```
sysctl net.ipv4.ip_forward=1
nano /etc/sysctl.d/99-sysctl.conf
--------
net.ipv4.ip_forward = 1
```
### 加载tun
```
modprobe tun
nano /etc/modules-load.d/tun.conf
-------------
tun
```

### 恢复之前的操作(未经测试)
```
sudo ip link set br0 down
sudo brctl delif br0 enp1s0
sudo ip link set enp1s0 down
sudo ip link set up enp1s0 #重启物理网卡
sudo ip link set dev br0 down #关闭 删除br0
sudo brctl delbr br0
```

参考文章：

https://wiki.archlinux.org/title/Network_bridge

https://blog.csdn.net/qq_42197548/article/details/120655449

