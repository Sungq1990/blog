---
title: ssh: connect to host github.com port 22: Connection timed out
date: 2025-08-29 16:25:00
tags: github
---

发现一直能用的github项目git pull push等都超时了，报下面的错误：
```
ssh: connect to host github.com port 22: Connection timed out
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

最后发现是公司封锁了对外访问的 22 端口，解决方法就是改成走433端口

```
vi ~/.ssh/config

#在文件中添加以下内容
Host github.com
  Hostname ssh.github.com
  Port 443
  User git
```
