---
title: Mac Clash Chrome无痕模式可以，正常模式不行的问题
date: 2022-01-10 14:26:54
tags: Clash Chrome
---


现象描述

```
安装了Clash，safari可以正常使用，chrome下无痕模式可以，正常模式就不行
```

解决方法

```
先chrome安装一个SwitchyOmega插件(可以离线安装，自行搜索方法)
然后在SwitchyOmega下的情景模式中配置代理服务器
代理服务器填127.0.0.1
端口填7890(如果改过Clash的端口，就填修改后的那个)
然后选择proxy模式就可以了
```