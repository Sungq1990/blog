---
title: Laravel的TrimStrings中间件会把请求数据的首尾空格去掉
date: 2021-10-11 17:18:00
tags: Laravel TrimStrings
---

```
今天业务方反馈一个运行很久了的接口报签名错误，经过定位发现是post提交的
参数中某个字段尾部有个空格，业务方在生成签名的时候是带上空格的，到我这边
的时候发现没有空格了，经过搜索引擎的帮助发现是Larave的TrimStrings中
间件造成的。


两种解决办法：
一种是业务方对数据进行处理去掉首尾空格；
二是注释掉TrimStrings这个中间件
```