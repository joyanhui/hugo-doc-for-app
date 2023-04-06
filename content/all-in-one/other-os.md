---
title: '常见all in one 系统的一些问题'
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

我选择 arch-aio的理由 
> arch-aio的简介 [https://dev.leiyanhui.com/all-in-one/diy/](https://dev.leiyanhui.com/all-in-one/diy/)

##### pve
- 磁盘分区 需要12-20G左右，备份起来不方便 ，虚拟磁盘名称不方便自定义 
- 多了不太常用的lxc，没有原生docker 需要自己处理
##### hyper-v
- 宿主机占用资源偏多
- 部分硬件的直通存在一些问题
- hyper-v 虽然免费，但win是收费的  破解版不适合商用
##### esxi
- 个人免费，商用收费
- 非原生linux，功能限制太多。
- 硬件要求尤其是网卡要求多
##### unraid
- 占用一个usb口
- 需要一个非常靠谱的u盘
- 墙内使用麻烦，需要梯子
- 会破坏一块硬盘的数据
- 收费，破解版多数有挖矿后台
##### 爱快
- 这个怎么说呢...也就比较适合不折腾的用户吧
- 虽然支持docker，但是没有shell 扩展性太差，简直可以说是垃圾中的战斗机
- 但是他的dhcp 以及流控 都很简单易用
- 有黑历史 慎重
##### 群晖
- 要么硬件买，钱多可以
- 黑群晖 有一些莫名其妙的问题，有时候可能是致命性的，不少人出现一段时间后无法启动需要接上显示器处理。
##### openwrt
- 路由器系统，不太适合宿主机折腾
