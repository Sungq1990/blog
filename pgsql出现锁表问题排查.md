---
title: pgsql出现锁表问题排查
date: 2024-09-24 15:41:00
tags: pgsql
---

同事线上想给a表加个字段，发现表被锁住了加不了字段，现在记录下排查和解决过程

知道了哪个表查起来就简单很多了，sql如下

```
SELECT
    pg_stat_activity.pid,
    pg_class.relname,
    pg_locks.mode,
    pg_locks.granted,
    pg_stat_activity.query,
    pg_stat_activity.state
FROM
    pg_locks
JOIN
    pg_class ON pg_locks.relation = pg_class.oid
JOIN
    pg_stat_activity ON pg_locks.pid = pg_stat_activity.pid
WHERE
    pg_class.relname = '目标表名';
```

发现是一句select语句开启了事务但是不知道为什么一直没提交
用下面的sql杀掉进程就可以了

```
SELECT pg_terminate_backend(PID)
```