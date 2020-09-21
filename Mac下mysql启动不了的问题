---
title: Mac下mysql启动不了的问题
date: 2020-09-21 14:28:54
tags: mysql
---

最近估计强制重启了一下电脑，Mac本地的mysql服务挂了，这里记录一下解决方法。

先说下环境:

```
Macbook Pro 2013款
brew安装的mysql 版本为5.6
mysql的WorkingDirectory是/usr/local/var/mysql
```

错误:

```
执行sudo mysql.server start报错

. ERROR! The server quit without updating PID file (/usr/local/var/mysql/sgq.local.pid).

详细的错误可引去mysql的WorkingDirectory里看，我的是/usr/local/var/mysql/sgq.local.err，里面是启动失败的具体信息

2020-09-21 06:47:16 16682 [ERROR] Fatal error: Please read "Security" section of the manual to find out how to run mysqld as root!

2020-09-21 06:47:16 16682 [ERROR] Aborting

```

通过报错发现是mysql不让通过root启动，最简单的方式就是强制使用root启动，后面追加--user=root即可

```
sudo mysql.server start --user=root

```