---
title: Mysql严格模式
date: 2019-09-18 10:22:34
tags: mysql
---

MySQL5.7将默认的sql_mode 从“松散模式”更改为“严格模式”。
我们通过SQL语句

```
select @@sql_mode;
```

查询出 sql_mode ，发现其中有 NO_ZERO_IN_DATE,NO_ZERO_DATE。

```
sql_mode = ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
```

严格模式下使用GROUP BY语句做统计的话会报sql语法错误。

解决方法有两个

- 关闭mysql的严格模式，laravel框架的话只需要修改config下的database.php下mysql下strict改为false即可

- 临时关闭当前链接的严格模式，laravel框架的话在执行sql前

```
DB::connection()->select('set SESSION sql_mode=""');
```