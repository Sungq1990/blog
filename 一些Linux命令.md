---
title: 一些Linux命令
date: 2018-07-03 15:02:00
tags: linux
---

记录一些常用的linux命令，方便查询
<!--more-->

##### 文件夹操作类
> 1.mv或者cp: 复制 exp: mv file1 file2
> 2.mkdir: 新建文件夹 exp: mkdir name
> 3.vi: 新建或者编辑文件 exp: vi name
> 4.more: 一页一页查看文件(按空格换页) exp:more filename
> 5.less: 和more相似,less 在查看之前不会加载整个文件
> 6.head && tail: head看头部内容,tail看底部内容
##### 文件查找命令
> 1.which: 查看可执行文件的位置 exp:which php
> 2.whereis: 只能用于程序名的搜索
> 3.find:在目录结构中搜索文件 exp:find path -name filename
##### linux文件权限设置
##### 磁盘存储相关
##### 性能监控和优化命令
> 1.ps -q PID 查看对应pid的进程
> 2.lsof -i tcp:80 查看端口占用进程
##### 网络命令
##### 其他命令
> 1.& 放在启动参数后面表示设置此进程为后台进程
> 2. 批量杀进程
ps aux|grep 进程名|grep -v grep|awk '{print $2}'|xargs kill -9
> 3.kill -USR1 和 kill -USR2
kill -USR1 等于 nginx -s reopen 
这个信号量本来就是用于重新读取日志文件的 
kill -USR2 等于 nginx -s reload 
reload 和 reopen 的行为相差很大，reopen 仅仅检查日志文件，reload 会重载配置，并启动新 worker，关闭旧 worker 
##### 查看机器负载
> ①uptime或者w 
exp:12:22:02 up 44 days, 21:48,  2 users,  load average: 3.96, 6.28,5.16  
load average分别对应于过去1分钟，5分钟，15分钟的负载平均值
> ②top  实时的监控，按q退出
(1) 在top下按“1”查看CPU核心数量，shift+"c"按cpu使用率大小排序，shif+"p"按内存使用率高低排序
(2) 使用iostat -x 命令来监控io的输入输出是否过大
> ③安装htop，使用htop命令



