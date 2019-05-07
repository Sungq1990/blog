---
title: php火焰图
date: 2019-05-07 14:46:14
tags: phpspy
---
早就想用火焰图来分析程序的瓶颈了，但是找了一圈并没有发现好用的php火焰图工具，最近发现了github上一个项目，可以生成火焰图，项目地址[phpspy](https://github.com/adsr/phpspy)，目前只能在Linux系统下使用，Mac和Windows都不支持

##### 我的环境 
```
PHP 7.2.14
CentOS Linux release 7.6.1810 (Core)
```
#### 使用
由于我使用的是php-fpm，所以需要监听php-fpm的进程来采集信息，命令行如下，其中-V72就是php版本，--time-limit-ms是采样时间，结束后就可以在tmp文件夹下看到svg文件，把svg文件拖进Chrome就可以看到火焰图了

```
pgrep php-fpm | xargs -P0 -I{} bash -c './phpspy -p {} -V72 --time-limit-ms 6000000 | ./stackcollapse-phpspy.pl | ./vendor/flamegraph.pl > ./tmp/phpspy-flamegraph.{}.svg'
```
效果如下
![](https://i.loli.net/2019/05/07/5cd13e5114593.jpg)