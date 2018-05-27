---
title: php安装MongoDB扩展
date: 2018-05-27 21:56:00
tags: php MongoDB
---
安装php MongoDB扩展，我的php路径为/data/home/server/php，使用 pecl 命令来安装 命令为

```
/data/home/server/php/bin/pecl install mongodb
```
然后在php.ini 文件，添加 extension=mongodb.so,或者用命令行

```
echo "extension=mongodb.so" >> `/data/home/server/php/bin/php --ini | grep "Loaded Configuration" | sed -e "s|.*:\s*||"`
```
到这里一切顺利，重启php-fpm后用命令行php -m |grep mongo发现并没有安装成功
<!--more-->
折腾了好久没有进展,只能使用strace去追踪一下

```
strace  /data/home/server/php/bin/php -i 2> /tmp/1.log
grep 'php.ini' /tmp/1.log
```
发现报错了

```
open("/data/home/server/php-7.0.0/bin/php.ini", O_RDONLY) = -1 ENOENT (No such file or directory)  
open("/data/home/server/php-7.0.0/etc/php.ini", O_RDONLY) = -1 ENOENT (No such file or directory)
```
原来我的之前的php环境是用镜像直接拷过来的，不知道为什么php.ini跑到/etc/下去了, 把/etc/下的php.ini移动到/data/home/server/php-7.0.0/etc/下并重启php-fpm,MongoDB已经安装好了

