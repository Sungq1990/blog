---
title: Mysql8.0获取表自增ID不准确的问题
date: 2020-10-10 10:53:54
tags: Mysql
---

工作中遇到一个问题，就是mysql的自增id auto increment获取到的不是最新的，这个问题只会在mysql8.0版本遇到。
查了一下原来是mysql8.0做了很多改动，其中一点就是tables不再是某个引擎表，而是改造成了视图。auto_increment和update_time列均引用自mysql.table_stats表。那么tables视图的信息不准确，根本原因就是table_stats表的统计信息并没有实时更新。
可以使用analyze table命令去人为的触发表信息收集，tables视图的信息会更新至当前准确的状态。
稍微了解下改动的原理:
为了最小化磁盘IO，MySQL8.0增加了一个字典对象缓存（dictionary object cache）。同时为了提高information_schema的查询效率，statistics和tables字典表的数据缓存在字典对象缓存中，并且有一定的保留时间，如果没超过保留时间，即使是实例重启，缓存中的信息也不会更新，只有超过了保留时间，才会到存储引擎里抓取最新的数据。同时，字典对象缓存采用LRU的方式来管理缓存空间。
缓存时间就是information_schema_stats_expiry的值，默认是86400。
改动的方法就是把information_schema_stats_expiry设置成0

我这边只有一个地方需要用到auto increment，所以是在SESSION级别改的information_schema_stats_expiry
```
SET SESSION information_schema_stats_expiry=0
```