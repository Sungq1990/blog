---
title: test
date: 2018-03-28 16:56:54
tags: php EasyWeChat
---


```flow
st=>start: 开始
e=>end: 结束

op1=>operation: 推送用户注册基本信息
cond1=>condition: 是否为已注册用户？
op2=>operation: 更新用户主表,将变更前信息录入快照表(只允许姓名和手机号变更)
cond2=>condition: 用户信息是否有变动？
io=>inputoutput: 返回openId
op3=>operation: 用户信息入库

st->op1->cond1
cond1(yes)->cond2
cond1(no)->op3->io->cond2
cond2(yes)->op2->io->e
cond2(no)->io->e
```