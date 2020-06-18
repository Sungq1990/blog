---
title: php冷门知识
date: 2020-06-18 20:06:54
tags: php
---

* 在响应用户请求后继续运行php脚本

```
ignore_user_abort(true);
set_time_limit(0);

ob_start();
// do initial processing here
echo $response; // send the response
header('Connection: close');
header('Content-Length: '.ob_get_length());
ob_end_flush();
ob_flush();
flush();

// now the request is sent to the browser, but the script is still running
// so, you can continue...

上面的是Apache服务器的写法，如果是php-fpm的话可以用fastcgi_finish_request
```