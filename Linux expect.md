---
title: Linux expect
date: 2019-05-09 10:20:14
tags: Linux expect
---
expect是一个免费的编程工具，用来实现自动的交互式任务，而无需人为干预。在实际工作中，我们运行命令、脚本或程序时，这些命令、脚本或程序都需要从终端输入某些继续运行的指令，而这些输入都需要人为的手工进行。而利用expect，则可以根据程序的提示，模拟标准输入提供给程序，从而实现自动化交互执行

#### expect基本只需要记住4个命令
| 命令 | 作用 |
| --- | --- |
| send | 用于向进程发送字符串 |
| expect | 从进程接收字符串 |
| spawn | 启动新的进程 |
| interact | 允许用户交互   |

#### 一个简单的Demo
```
#!/usr/tcl/bin/expect

set timeout 30
set host "101.200.241.109"
set username "root"
set password "123456"

spawn ssh $username@$host
expect "*password*" {send "$password\r"}
interact
```
其它还有模式-动作和传参等功能，具体可以问搜索引擎
