---
title: 'arch-aio 常见问题'
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

## libvirtd/virt-manager 找不到磁盘
libvirtd 应该在rc.local 之后启动

## virt-manager 弹不出界面
1、 使用支持x11转发的客户端，win下使用 MobaXterm Macos和linux andriod下也又对应的客户端。如果是ios设备也有弯道解决方案
2、后台有virt-manager进程，那么先关掉 `killall virt-manager`

## virt-manager 无法连接
```
Unable to connect to libvirt qemu:///system.
```
### 连不到virtqemud-sock
因为 libvirtd 没启动，可以用下面两种方式启动
```
systemctl enable libvirtd
systemctl start libvirtd
```
### 修改docker/kvm目录后，无法自动启动
 自动挂载建议放到rc-local 服务用，docker和libvirtd 在rc-local之后启动 [具体方法](https://dev.leiyanhui.com/arch/auto-mount-dock-kvm/)

