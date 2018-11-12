---
title: nginx从gzip换成brotli
date: 2018-11-12 17:39:14
tags: nginx gzip brotli
---
原来nginx一直用的是gzip,偶然听说brotli可以让网站速度更快，就换成了brotli,brotli要求nginx版本在1.9以上，我服务器上的nginx版本是1.7,折腾了一会儿直接升级到1.14了
<!--more-->
#### 升级nginx
先说一下升级nginx,网上搜了一大推平滑升级的但是没几个能用的，所以我直接
```
wegt 
./configure
make && make install
```
编译安装了个新的，把旧的直接删了

#### 安装brotli
①安装libbrotli

```
cd /usr/local/src/
git clone https://github.com/bagder/libbrotli
cd libbrotli
./autogen.sh
./configure
make && make install
```
②安装ngx_brotli

```
cd /usr/local/src/
git clone https://github.com/google/ngx_brotli
cd ngx_brotli && git submodule update --init
```

③获取Nginx Arguments

```
nginx -V
```
在最后加上 --add-module=/usr/local/src/ngx_brotli
编译安装nginx

```
./configure xxxxxxx --add-module=/usr/local/src/ngx_brotli
make && make install
```
④修改nginx.conf
http块中加入

```
#Brotli Compression
brotli on;
brotli_comp_level 6;
brotli_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript application/javascript image/svg+xml;
```
⑤重启Nginx

这时候用浏览器访问可以看到content-encoding:br替换了原来的content-encoding:gzip了

