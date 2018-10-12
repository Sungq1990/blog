---
title: nginx日志切割和日志可视化
date: 2018-10-12 17:39:14
tags: nginx logrotate goaccess
---

logrotate程序是一个日志文件管理工具(linux自带)。用于分割日志文件，删除旧的日志文件，并创建新的日志文件，起到“转储”作用。可以节省磁盘空间
<!--more-->
#### 增加nignx日志按月分割
cd /etc/logrotate.d
vi nginx
插入以下内容
```
/path/accecc.log { # access.log路径
    notifempty
    monthly
    sharedscripts
    postrotate
    /bin/kill -USR1 `/bin/cat /path/nginx.pid` #nginx.pid路径 可在nginx.conf中查看
    endscript
}
```

```
#Debug验证配置文件
/usr/sbin/logrotate -d -f /etc/logrotate.d/nginx
#如果等不及cron自动执行日志轮转，想手动强制切割日志，需要加-f参数
/usr/sbin/logrotate -f /etc/logrotate.d/nginx
```

#### nginx日志可视化
这里使用goaccess作为可视化工具
###### 安装
```
yum install goaccess
```
###### 修改配置

```
vi /etc/goaccess.conf
删除time-format %H:%M:%S前面的#
删除date-format %d/%b/%Y前面的#
后台执行
goaccess -f /path/access.log -o /path/index.html --real-time-html --date-spec=hr --hour-spec=min --ws-url=domain&
```
参数说明
-f:access.log绝对路径
-o:goaccess生成的html路径
--real-time-html:实时刷新html
--date-spec=hr:按照小时分析独立访客
--hour-spec=min:设定为按每十分钟报告
--ws-url:此 URL 用于 WebSocket 服务器的回应。用于客户端侧的 WebSocket 构建器(ws:// 用于非加密连接, 以及 wss:// 用于加密连接)

###### 其它事项
我这里的time-format和date-format使用的是nginx默认的日志format格式，如果你自定义了nginx日志格式你可以参考[goaccess中文网](https://www.goaccess.cc)自行修改
WebSocket默认使用的是7890端口，所以记得防火墙开启这个端口
