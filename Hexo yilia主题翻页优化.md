---
title: Hexo yilia主题翻页优化
date: 2021-08-13 11:30:54
tags: hexo yilia
---

#### 未优化前

![](https://i.loli.net/2021/08/13/NVctIzELo4y9eCP.png)


#### 优化后

![](https://i.loli.net/2021/08/13/4a7gRIslzymeU5p.png)

#### 改动过程

```
首先
/themes/yilia/layout/_partial/archive.ejs
全文搜索 &laquo; Prev和 Next &raquo 并改成 上一页和下一页
改js总共要改两处地方


最后
/themes/yilia/layout/_partial/script.ejs
删除该文件里的Next &raquo 和 &laquo; Prev
```