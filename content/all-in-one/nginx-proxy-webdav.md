---
title: 'nginx 反向代理webdav的问题  无法创建续传 重命名 移动 等问题'
date: 2023-02-12T08:14:20+08:00
tag:
  - alpine
  - nginx
  - webdav
categories: 
  - alpine
  - nginx
  - webdav
---

简单说，是需要 more_set_headers 这个模块，一般情况下nginx没有自带   
https://github.com/openresty/headers-more-nginx-module

以alpine下载的nginx为例
```
apk add nginx-mod-http-headers-more
```
```
nginx -V
```
发现是动态加载的
```
find / -name "*more*"
```
找到了文件 `/usr/lib/nginx/modules/ngx_http_headers_more_filter_module.so`   
修改配置文件 `nginx.conf`添加一行  
```
load_module /usr/lib/nginx/modules/ngx_http_headers_more_filter_module.so;
```
nginx版本必须大于 1.9.11，否则需要自己编译了不能使用动态加载了   
> 新版alpine的nginx默认配置文件会自动加载 `/etc/nginx/modules/` 这里的配置文件，不需要自己 load_module   
## 解决中文文件夹重命名问题
将请求头Destination中的https替换为http，并解决webdav重命名中文文件夹的乱码问题：
修改配置文件 http部分
```
 map $http_destination $webdav_dest {
    ~*https://(.+) http://$1;
    default $http_destination;
  }
```
## 上传文件大小限制问题
这部分内容放到server，或者http部分
```
# 修改包大小上限避免上传错误
client_max_body_size 10G;
```
## 其他的问题
这部分内容放到 location 
```
# 解决webdav不能创建文件夹问题
if ($request_method = MKCOL) { rewrite ^(.*[^/])$ $1/ break; }

# 解决webdav不能重命名问题
if (-d $request_filename) {
    rewrite ^(.*[^/])$ $1/;
    set $webdav_dest $webdav_dest/;
    more_set_input_headers 'Destination: $webdav_dest';
}

# 解决webdav不能复制、移动文件问题
if ($request_method ~ (MOVE|COPY))
{  
    more_set_input_headers 'Destination: $webdav_dest';
}
```

## 最终配置文件
```
load_module /usr/lib/nginx/modules/ngx_stream_module.so;
load_module /usr/lib/nginx/modules/ngx_http_headers_more_filter_module.so;
worker_processes auto;
events {
    worker_connections  1024;
    accept_mutex on;
  }
http {
  include mime.types;
  default_type application/octet-stream;
  keepalive_timeout 75s;
  gzip on;
  gzip_min_length 4k;
  gzip_comp_level 4;
  client_max_body_size 1024m;
  client_header_buffer_size 32k;
  client_body_buffer_size 8m;
  server_names_hash_bucket_size 512;
  proxy_headers_hash_max_size 51200;
  proxy_headers_hash_bucket_size 6400;
  gzip_types application/javascript application/x-javascript text/javascript text/css application/json application/xml;
  map $http_upgrade $connection_upgrade {
    default upgrade;
    '' close;
  }
  map $http_destination $webdav_dest {
    ~*https://(.+) http://$1;
    default $http_destination;
  }
 server {
  server_name alist.jia.leiyanhui.com;
  listen 50443 ssl http2;
  ssl_certificate /home/nginxWebUI/.acme.sh/jia.leiyanhui.com/fullchain.cer;
  ssl_certificate_key /home/nginxWebUI/.acme.sh/jia.leiyanhui.com/jia.leiyanhui.com.key;
  ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
  listen 50080;
  if ($scheme = http) {
    return 301 https://$host:50443$request_uri;
  }
  error_page 497 https://$host:50443; #解决http到ssl端口的400错误;
  client_max_body_size 10G;
  location / {
    proxy_pass http://10.1.1.52/;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Host $http_host;
    proxy_set_header X-Forwarded-Port $server_port;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_redirect http:// https://;
    proxy_set_header Range $http_range;
    # 解决webdav不能创建文件夹问题
    if ($request_method = MKCOL) { rewrite ^(.*[^/])$ $1/ break; }

    # 解决webdav不能重命名问题
    if (-d $request_filename) {
        rewrite ^(.*[^/])$ $1/;
        set $webdav_dest $webdav_dest/;
        more_set_input_headers 'Destination: $webdav_dest';
    }

    # 解决webdav不能复制、移动文件问题
    if ($request_method ~ (MOVE|COPY))
    {  
        more_set_input_headers 'Destination: $webdav_dest';
    }
  }
}

}

```