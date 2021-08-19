---
title: Laravel ORM操作大量数据时慢
date: 2021-08-19 13:58:00
tags: Laravel ORM
---

```
今天越到一个问题，一个接口执行了将近15s，接口逻辑很简单就执行一条sql然后把结果返回，
但是通过debug软件跟踪发现瓶颈不在数据库。后面通过研究发现瓶颈在框架的ORM，因为这个sql
返回了上万条数据，当数据量巨大时ORM就会变得非常慢，这个时候需要直接用原生方法。
```

#### 使用orm

![](https://i.loli.net/2021/08/19/93NZCUlsjzVStxu.png)

#### 使用原生sql

![](https://i.loli.net/2021/08/19/bTX49RaV7KMFv5G.png)


```
可以看到使用原生sql后TTFB从15s缩减到不到1s，整个接口的速度有了质的提升
所以处理大数据量的时候建议直接用原生sql，不要用orm
```