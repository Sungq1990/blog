---
title: Mysql5.6 慢日志和BINLOG和数据备份
date: 2018-09-07 09:51:00
tags: mysql
---

虽然开启BINLOG据说会有1%的性能损耗，但是为了以防万一还是开启吧-.-
<!--more-->

开启慢日志

```
vi /etc/my.cnf
在mysqld下面加入
slow_query_log=1  //开启慢日志
long_query_time=2  //判断时间阀值
log_queries_not_using_indexes=1  //没走索引的sql
slow-query-log-file=/var/log/mysql/slow.log  //慢日志记录位置
保存退出
```

开启BINLOG

```
vi /etc/my.cnf
在mysqld下面加入
log-bin=mysql-bin  //开启binlog
expire_logs_days=5  //binlog过期时间
保存退出
```

定时备份数据库

```
#!/bin/sh
 
#备份目录
dic="/data/home/db_bak/data"
y=$(date +%Y)
m=$(date +%m)
d=$(date +%d)

dbname="test"
 
#mysql备份文件名
filename=$dic/$dbname$y$m$d.sql
 
echo "bakup name:"$filename
 
#删除7天前的备份文件
for file in `ls -a $dic`
do
 find -mtime 7 -name "*.sql" -exec rm -rf {} \;
done
 
#备份数据库
echo "备份数据库"
#这里要用mysqldump的绝对路径
/usr/local/mysql/bin/mysqldump -uroot -p123456 $dbname > $filename
```

查看binlog

```
//查看binlog文件列表
show binary logs;
//具体查看某个binlog
show binlog events in 'mysql-bin.000004';
```
    

