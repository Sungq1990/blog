---
title: WSL2的服务暴露给局域网里的其它机器访问
date: 2024-05-08 16:32:00
tags: WSL2
---

#### windows先开放端口

```txt
去防火墙那里配置一个入站规则，开放需要访问的端口
```

#### 修改wsl配置

```txt
用户目录 %USERPROFILE% 下面创建或修改配置文件 .wslconfig，加入以下内容
[experimental]
networkingMode=mirrored
dnsTunneling=true
firewall=true
autoProxy=true
```

#### 重启wsl
```txt
Windows cmd 执行 wsl --shutdown然后重启打开wsl就能访问了
```