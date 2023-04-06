---
title: 'windows 使用btrfs分区 '
date: 2022-12-23T01:14:20+08:00
tag:
  - win
  - btrfs
  - linux
  - choco
categories: 
  - win
  - btrfs
  - linux
  - choco
---

btrfs 大有替代 ext4的趋势，支持压缩 支持快照 简直爽的不要不要的

win10下启用btrfs 经过长时间使用发现winbtrfs这个项目算是相对不错 且稳定的 
https://github.com/maharmstone/btrfs

安装方法

下载，https://github.com/maharmstone/btrfs/releases/download/v1.8.1/btrfs-1.8.1.zip 解压，注意 是 `btrfs-XX.zip` 不是debug和pdb那两个

右键  btrfs.inf  ,安装，不用重启即可使用

测试 大文件可以全速读写  4k 未测试。

## 后记
### winpe下 
测试winpe 为 firpe  v1.8.1  ，也支持。  winpe下不需要重启即可使用
### 不建议主力
win下使用linux原生分区格式，建议还是应急为主。   
如果你的系统以win为主，建议还是exfat 兼容性处理起来更简单 也更稳。   
如果是linux为主的话，那么 上btrfs 吧！不会后悔   
