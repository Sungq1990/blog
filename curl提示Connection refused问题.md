---
title: curl提示Connection refused问题
date: 2022-01-24 15:31:00
tags: Deployer php
---

装了clash后发现终端执行curl不行了，会报下面的错误

```
curl: (7) Failed to connect to 127.0.0.1 port 1080: Connection refused
```

解决方法
```
修改curl的配置文件
配置文件一般位于 ~/.curlrc
发现里面有一行代理配置 socks5 = "127.0.0.1:1080"
注释掉就行
```
