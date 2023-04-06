---
title: 'win10 win11的磁盘压缩'
date: 2022-11-15
tag:
  - windows
  - compact
---

如果是小磁盘安装，默认是关闭预留空间，并且compcat的

## 释放预留空间
    dism /Online /Set-ReservedStorageState /State:Disabled
## NTFS的compact命令
    compact /compactOS:always
    
### 第三方工具

dism++  10.x  开专家模式 ，可以compact lzx
