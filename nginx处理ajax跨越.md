---
title: nginx处理ajax跨越
date: 2018-02-28 16:56:54
tags: nginx
---

新写个前后端分离的项目,前端的域名是aa.xxx.com, 接口域名是bb.xxx.com, 前端框架用的是vue,ajax包用的是axios。用axios发送请求到到后端接口是报跨域错误
<!--more-->
axios会先发送一个Request Method为OPTIONS的请求,浏览器自动在跨域的请求发送之前发送一个 OPTIONS 请求，以判断服务端是否允许这一域访问,我这边是改了nginx配置实现跨域的,废话不多说上代码和配置
##### vue前端代码
```
let Random = parseInt(Math.random() * 100000000).toString()
let params = {class:'SEARCH'}

console.log(JSON.stringify(params))
axios({
    method: "POST",
    url: 'https://housekeeper.health666.club',
    data: params,
    headers: {
      'Cache-Control': 'no-cache',
      'UTC-Timestamp': Math.round(new Date() / 1000).toString(),
      'Random': Random        
    }
})
.then(function(res) {
  console.log(res)
})
.catch(err => {
  console.log(err)
})
```

##### nginx配置(server部分)
```
server {
    listen       443;
    server_name  bb.xxx.com;
    ssl on;
    ssl_certificate 1_bb.xxx.com_bundle.crt;
    ssl_certificate_key 2_bb.xxx.com.key;
    ssl_session_timeout 5m;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2; #按照这个协议配置
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;#按照这个套件配置
    ssl_prefer_server_ciphers on;
    index index.html index.htm index.php;
    root /data/home/www/xxx/public;
    add_header 'Access-Control-Allow-Origin' https://aa.xxx.com;
    add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
    add_header 'Access-Control-Allow-Headers' 'DNT,X-CustomHeader,X-Requested-With,If-Modified-Since,Keep-Alive,User-Agent,Cache-Control,Content-Type,Connection,Content-length,Account-Key,UTC-Timestamp,Random,Signature';
    
    location / {
        if ($request_method = 'OPTIONS') {            
            add_header 'Access-Control-Max-Age' 1728000;
            add_header 'Content-Type' 'text/plain charset=UTF-8';
            add_header 'Content-Length' 0;
            add_header 'Access-Control-Allow-Origin' https://bb.xxx.com;
            add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
            add_header 'Access-Control-Allow-Headers' 'DNT,X-CustomHeader,X-Requested-With,If-Modified-Since,Keep-Alive,User-Agent,Cache-Control,Content-Type,Connection,Content-length,Account-Key,UTC-Timestamp,Random,Signature';
            return 204;
        }

        index  index.html index.htm index.php;
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {   
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include fastcgi_params;
    }      
}
```
我的是htpps的配置,端口监听的是443


