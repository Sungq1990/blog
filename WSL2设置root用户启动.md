---
title: WSL2设置root用户启动
date: 2023-12-26 14:35:00
tags: WSL2
---

```
执行下面命令可以让wsl默认root启动，
好处就是再也不用关心权限问题，
比如原来我的phpstorm一直会因为权限不够导致编辑后无法保存的问题彻底得到解决

ubuntu config --default-user root
```
