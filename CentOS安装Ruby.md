---
title: CentOS安装Ruby
date: 2023-01-04 18:45:00
tags: Ruby
---
最近一个项目是用Ruby写的，需要自己搭建Ruby环境，这里记录一下搭建过程
安装的是2.6.5版本的Ruby，编译安装

```
下载ruby安装包
wget https://cache.ruby-china.com/pub/ruby/ruby-2.6.5.tar.xz


解压安装包
xz -d ruby-2.6.5.tar.xz
tar -xvf ruby-2.6.5.tar


编译安装 
cd /ruby-2.6.5
./configure
make
make install
```

ruby依赖zlib、openssl-devel、readline等，需要是用yum安装相应的软件，然后分别进/ruby-2.6.5/ext/zlib、
/ruby-2.6.5/ext/openssl

```
ruby ./extconf.rb
make
make install
```

这样ruby就会安上相应的扩展

安装扩展的时候可能会报make:*** No rule to make target `/include/ruby.h', needed by `zlib.o'.Stop.

这个报错是指找不到/include/ruby.h这个文件，只要修改各自对应的Makefile脚本文件最上面加上

```
top_srcdir = ../..
```