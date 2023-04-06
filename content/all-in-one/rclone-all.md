---
title: 'rclone 常用命令 挂载 ls 临时url等'
date: 2023-04-02T08:14:20+08:00
tag:
  - rclone
  - webdav
  - nas
  - sync
categories: 
  - rclone
  - webdav
  - nas
  - sync
---

rclone  是一个非常强大的 网络文件系统辅助工具！
支持并不限与以下功能
- 挂载 对象储存
- 挂载 多数正常的网盘（非国内阉割盘）
- sync同步
- 文件复制 删除  ls
- 临时获取url link等
- 加密同步


下面整理记录常用命令
### 安装
golang写的 直接解压即可 ，或者用linux自带的包管理器
```
wget https://downloads.rclone.org/rclone-current-linux-amd64.zip
unzip rclone-current-linux-amd64.zip
cp rclone-*-linux-amd64/rclone  /mnt/sda1/rclone &&   rm -rf rclone-*
chown root:root /mnt/sda1/rclone && chmod 755 /mnt/sda1/rclone
```
### 配置文件
#### 向导模式
```
/mnt/sda1/rclone config   # 默认路径在  ~/.config/rclone/rclone.conf  
/mnt/sda1/rclone config   --config /mnt/sda1/rclone.conf  
```
#### 手动编辑
例子
```
cat >/mnt/sda1/rclone.conf   << EOF
[cloudreve]
type = webdav
url = https://cloudreve地址:5213/dav
vendor = nextcloud
user = 密码
pass = 加密后的密码
[oss-sh]
type = s3
provider = Alibaba
access_key_id = XXXXX
secret_access_key = XXXXXX
endpoint = oss-cn-shanghai.aliyuncs.com
acl = private
storage_class = STANDARD
bucket_acl = private
EOF
```
### 备份和恢复
```
rclone copy oss-qd:demo-docker-img-bak/emqx.5.0.17.tar.gz  /demo/docker-img-bak/emqx.5.0.17.tar.gz  --config /demo/etc/rclone.conf   
rclone sync oss-qd:demo-serv-emqx-bak/ /demo/emqx   --config /demo/etc/rclone.conf
rclone sync  /demo/emqx  oss-qd:demo-serv-emqx-bak/  --config /demo/etc/rclone.conf

```
sysnc 和 copy 区别  等于 系统 sysnc 和 cp的区别
### 获取临时链接
仅限部分储存可以用。有一些储存肯定是不可以的用的。
```
/root/opt/rclone link --expire 1m oss-qd:demo-serv-bak/opt_serv1/main.txt
```
### 检查 查询
ls 列出文件
```
rclone ls 列出路径中的对象及其大小和路径。
rclone lsd 列出路径中的所有目录/容器/桶。
rclone lsf 列出 remote:path 格式的目录和对象以供解析。
rclone lsjson 以 JSON 格式列出路径中的目录和对象。
rclone lsl 列出路径中的对象及其修改时间、大小和路径。
```
例如
```
rclone lsd cloudreve:/ --config /demo/etc/rclone.conf
```

### 挂载命令

```
/mnt/sda1/rclone mount cloudreve:/ /mnt/dav   --config /mnt/sda1/rclone.conf
```
如果提示 `fusermount3` 找不到，应该是缺少 fuse3 这个包 `apk add fuse3`