---
title: 'win10 搭建remoteapp 服务器，实现app 随处可移动'
date: 2022-11-29T23:14:20+08:00
tag:
  - win
  - rdp
  - remoteapp
categories: 
  - win
  - rdp
  - remoteapp
---

windows server 2012以后的版本都支持，但是考虑到不折腾，win7 win8 win10 都是可以实现的，考虑到兼容性 和免折腾，我选择 win10。  
母盘选择`Win_Pro_10_22H2_64BIT_ChnSimp_Pro_Ent_EDU_N_MLF_X23-20012.ISO`   kms激活到专业工作站版本，专业版也可以。家庭版不确定
`Microsoft Windows [版本 10.0.19045.2006]`
借用第三方开源工具remoteapptool轻松实现

http://www.kimknight.net/remoteapptool  
https://github.com/kimmknight/remoteapptool
> remoteapptool 的权限问题依赖账户系统，也比较简单。如果需要严格详细的限制权限功能，还是用window server自带的remoteapp 来处理。
> 我这里是小团队使用没有那么复杂
## 下载安装RemoteApp.Tool

https://github.com/kimmknight/remoteapptool/releases/download/v6.0.0.0/RemoteApp.Tool.6000.zip

解压就可以了，使用很简单 就是添加可执行文件，

因为是自己用，所以先添加两个
C:\Windows\explorer.exe
C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe

创建rdp文件 然后rdp文件复制到其他win或者macos电脑上。win下直接打开 macos下用微软的 远程桌面客户端 就可以了。

Linux的客户端 还没研究明白

当然需要先自行打开 Windows的远程桌面 并且正常连接后

下一步是需要处理 win10的多用户登录

### win10 多用户登录
这个是非必须的，但不解决只能同时使用一个用户登录，并且不可以
去下载，主页：https://github.com/stascorp/rdpwrap
#### 下载破解程序
https://github.com/stascorp/rdpwrap/releases/download/v1.6.2/RDPWrap-v1.6.2.zip
加载后解压即可，这个程序不少杀毒软件都提示病毒风险的，这个是正常的。也已经很久没更新过了
#### 下载最新的破解配置文件
只要不是太新的win内核版本（最近1-2周的） 一般都支持
https://github.com/sebaxakerhtc/rdpwrap.ini/blob/master/rdpwrap.ini
下载后复制到和RDPWrap 同一个目录下
#### 开始处理
建议全部挪到 C:\Program Files\RDP Wrapper 目录下
先打开RDPConf.exe 查看一下
双击下运行  install.bat 就耐心等待。 一直到提示`You can check RDP functionality with RDPCheck program.` 重新打开 RDPConf.exe 全绿色就好了
#### 常见问题
### 微信黑框的问题
暂时无解
### 聊天软件 和其他软件系统托盘的问题
暂时无解
### 后台程序热键问题
暂时无解
### edge不能启动问题
edge 有后台常驻程序，会导致不能多用户启动。手动运行任务 C:\windows\System32\Taskmgr 结束掉 就可以了。
或者 干脆使用 默认支持多用户的浏览器。火狐 chrome 都可以用户模式安装。
