---
title: php解析pdf
date: 2018-06-03 20:33:00
tags: php
---
发现要爬虫的目标网址的数据在PDF上，一开始用"smalot/pdfparser"这个包去解析PDF文件,但是发现有的PDF可以解析，有的报TCPDF_PARSER ERROR: Invalid object reference: Array这个错，去issue上查了下别人说是PDF版本不是1.4造成的，但是我看了解析不了的pdf版本就是1.4的，相同版本有的可以解析有的不能解析，唯一的不同就是编码软件不一样，查了很多资料解决不了，逐放弃这个用这个包解析PDF
<!--more-->
php有很多PDF包，但是就pdfparser这个包简单好用，别的要么解析不了要么很麻烦，所以放弃用php解析了，转而寻找其他语言比如Python，java处理，但是想了一下又懒得部署一套其他语言的环境，最后想到用linux来解决这个问题
linux安装pdftotext,用这个把pdf转成TXT，在用php读取TXT达到曲线救国的目的，linux安装pdftotext很简单，命令为yum install xpdf,随后pdftotext -v查看是否安装完成，PDF转txt的命令为pdftotext test.pdf test.txt，在php里实现为

```
exec("pdftotext test.pdf text.txt", $output, $ret);

```
然后对text.txt的数据进行处理就行，处理的过程中发现一个奇怪的东西，如图
![avatar](https://ws2.sinaimg.cn/large/006tKfTcgy1fryaa8093pj31c20b0ag4.jpg)
两个大写的FF，一脸懵逼，突然想起来以前遇到过一个差不多的EOT（传输结束符），原来是ASCII码中的换页符号，用preg_replace("/\f/i", '', $str)去除即可，有空要好好的研究下ASCII码，免得看到一脸懵逼