---
title: Mysql主从配置
date: 2017-09-21 11:23:14
tags: mysql
---

记录一下mysql的主从复制

```
主服务器：
开启二进制日志
    配置唯一的 server-id
    获得master二进制日志文件名及位置
    创建一个用于slave和master通信的用户账号
从服务器：
    配置唯一的 server-id
    使用master分配的用户账号读取master二进制日志
    启用slave服务
```

<!--more-->
### 主数据库 master 配置
* 修改 my.cnf 文件 在 [mysqld] 加上如下的配置

```
[mysqld]
log-bin=mysql-bin     #开启二进制日志
server-id=1           #设置server-id
character_set_server=utf8
init_connect='SET NAMES utf8'
```
* 重启mysql
* 登陆Mysql，创建用于同步的用户账号

```
CREATE USER 'repl'@'139.199.***.***' IDENTIFIED BY 'YourPassword9#';
GRANT REPLICATION SLAVE ON *.* TO 'repl'@'139.199.***.***';
FLUSH PRIVILEGES;
139.199.***.***  这里填上自己从服务器的 ip 
```
* 查看master状态，记录二进制文件名 mysql-bin.000001 和位置 2930

```
mysql>  SHOW MASTER STATUS;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000001 |     2930 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```

### 从数据库 slave 设置
* 修改 my.cnf 文件 在 [mysqld] 加上如下的配置

```
[mysqld]
server-id=2           #设置server-id
character_set_server=utf8
init_connect='SET NAMES utf8'
```
* 重启 mysql
* 登陆Mysql，并执行同步SQL语句

```
mysql> CHANGE MASTER TO
MASTER_HOST='116.196.***.***',             # 主服务器ip
MASTER_USER='repl',                        # 主服务器登陆名
MASTER_PASSWORD='YourPassword9#',          # 主服务器登陆密码
MASTER_LOG_FILE='mysql-bin.000001',        # 二进制文件的名称
MASTER_LOG_POS=2930;                       # 二进制文件的位置
```
* 启动 slave 同步进程

```
mysql> start slave;
Query OK, 0 rows affected (0.00 sec)
```
* 查看 slave 状态

```
mysql> show slave status\G;
如果以下两项都是 yes 就表示主从同步设置成功了
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
```
![最终效果](https://i.loli.net/2019/04/29/5cc6b7d0b72fc.gif)

