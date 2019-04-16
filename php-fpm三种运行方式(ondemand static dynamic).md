---
title: php-fpm三种运行方式(ondemand static dynamic)
date: 2018-10-04 09:23:14
tags: php-fpm
---

php-fpm有3种对子进程的管理方式

#### ① pm = static
```
始终保持一个固定数量的子进程，这个数由pm.max_children定义，这种方式很不灵活，也通常不是默认的
```
#### ② pm = dynamic
```
启动时，会产生固定数量的子进程（由pm.start_servers控制）可以理解成最小子进程数，而最大子进程数则由pm.max_children去控制，OK，这样的话，子进程数会在最大和最小数范围中变化，还没有完，闲置的子进程数还可以由另2个配置控制，分别是pm.min_spare_servers和pm.max_spare_servers，也就是闲置的子进程也可以有最小和最大的数目，而如果闲置的子进程超出了pm.max_spare_servers，则会被杀掉
dynamic模式为了最大化地优化服务器响应，会造成更多内存使用，因为这种模式只会杀掉超出最大闲置进程数（pm.max_spare_servers）的闲置进程，比如最大闲置进程数是30，最大进程数是50，然后网站经历了一次访问高峰，此时50个进程全部忙碌，0个闲置进程数，接着过了高峰期，可能没有一个请求，于是会有50个闲置进程，但是此时php-fpm只会杀掉20个子进程，始终剩下30个进程继续作为闲置进程来等待请求，这可能就是为什么过了高峰期后即便请求数大量减少服务器内存使用却也没有大量减少，也可能是为什么有些时候重启下服务器情况就会好很多，因为重启后，php-fpm的子进程数会变成最小闲置进程数，而不是之前的最大闲置进程数 
```
#### ③ pm = ondemand
```
和pm= dynamic相反，把内存放在第一位，他的工作模式很简单，每个闲置进程，在持续闲置了pm.process_idle_timeout秒后就会被杀掉，有了这个模式，到了服务器低峰期内存自然会降下来，如果服务器长时间没有请求，就只会有一个php-fpm主进程，当然弊端是，遇到高峰期或者如果pm.process_idle_timeout的值太短的话，无法避免服务器频繁创建进程的问题 
```
<!--more-->

## php-fpm和nginx状态监控
#### 新增nginx站点
```
server {
    listen       80;
    server_name  localhost;

    index index.php index.html;
    
    location /nginx_status {
          stub_status on;
          access_log off;
          allow 127.0.0.1;
    }

    location ~ /php_fpm_status$ {
            allow 127.0.0.1;
            fastcgi_param SCRIPT_FILENAME $fastcgi_script_name;
            include fastcgi_params;
            fastcgi_pass 127.0.0.1:9000;
    }
}
```
#### 修改php-fpm的配置
```
vi www.conf
去掉 pm.status_path前面的注释，并改成pm.status_path = /php_fpm_status
重启php-fpm
```

#### 使用curl指令查看php-fpm和nginx的status
```
curl localhost/nginx_status
curl localhost/php_fpm_status
```
#### nginx status的含义
- active connections – 活跃的连接数量
- server accepts handled requests — 总共处理了11989个连接 , 成功创建11989次握手, 总共处理了11991个请求
- reading — 读取客户端的连接数.
- writing — 响应数据到客户端的数量
- waiting — 开启 keep-alive 的情况下,这个值等于 active – (reading+writing), 意思就是 Nginx 已经处理完正在等候下一次请求指令的驻留连接

#### php-fpm status的含义
- start time – 启动日期,如果reload了php-fpm，时间会更新
- start since – 运行时长
- accepted conn – 当前池子接受的请求数
- listen queue – 请求等待队列，如果这个值不为0，那么要增加FPM的进程数量
- max listen queue – 请求等待队列最高的数量
- listen queue len – socket等待队列长度
- idle processes – 空闲进程数量
- active processes – 活跃进程数量
- total processes – 总进程数量
- max active processes – 最大的活跃进程数量（FPM启动开始算）
- max children reached - 大道进程最大数量限制的次数，如果这个数量不为0，那说明你的最大进程数量太小了，请改大一点。
- slow requests – 启用了php-fpm slow-log，缓慢请求的数量