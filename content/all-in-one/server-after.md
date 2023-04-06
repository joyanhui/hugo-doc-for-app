---
title: '旧版本 arch-aio all in one系统的，docker和kvm 与挂载磁盘顺序的一个小地方需要自己处理以下'
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

为了灵活管理，arch-aio 单独创建一个了rc-local服务，用来添加开机自动启动的脚本。 同时也是这个原因，建议在rc-local 里面处理自动挂载脚本。

仅限2022-12-29 版本
```
cd /usr/lib/systemd/system/
```
分别编辑 `sudo nano libvirtd.service` 和  `sudo nano docker.service` 在`[Unit]` 段 增加一行
```
After=rc-local.service
```
这行是为了让docker和libvirtd 在rc-local这个服务之后启动，方便我们在 rc-local 内先挂载好磁盘
最后这两个服务设置为自动启动
```
sudo systemctl enable libvirtd
sudo systemctl enable docker
```

关于和自动挂载其他分区配合的方法，可以参考这里：[https://dev.leiyanhui.com/arch/auto--dock-kvm/](https://dev.leiyanhui.com/arch/auto--dock-kvm/)
