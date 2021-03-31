---
title: 记录下小程序直传阿里云oss
date: 2021-03-31 17:30:23
tags: oss 阿里云
---

### 阿里云文档如下

```
https://help.aliyun.com/document_detail/92883.html
```
### 必要的前置操作

阿里云后台配置oss的操作就不复述了，可以参考上面的文档进行，小程序使用的话肯定是外网访问域名了。将阿里云后台的这个域名添加到小程序->开发设置的上传、下载域名里去，注意需要加上http或者https头。

### 下面说下具体的代码实现

* 服务端主要是生成signature和policy,至于是走阿里云上传成功回调还是小程序回调，自行选择
* php代码

```
<?php
  function gmt_iso8601($time) {
    $dtStr = date("c", $time);
    $mydatetime = new DateTime($dtStr);
    $expiration = $mydatetime->format(DateTime::ISO8601);
    $pos = strpos($expiration, '+');
    $expiration = substr($expiration, 0, $pos);
    return $expiration."Z";
  }

  $id= '';      // 请填写您的AccessKeyId。
  $key= '';     // 请填写您的AccessKeySecret。
  // $host的格式为 bucketname.endpoint，请替换为您的真实信息，加上http或者https头。
  $host = '';  
  // $callbackUrl为上传回调服务器的URL，不用不填。
  $callbackUrl = '';
  $dir = 'test';    //上传文件的目标文件夹路径

  $callback_param = array('callbackUrl'=>$callbackUrl, 'callbackBody'=>'filename=${object}&size=${size}&mimeType=${mimeType}&height=${imageInfo.height}&width=${imageInfo.width}','callbackBodyType'=>"application/x-www-form-urlencoded");
  $callback_string = json_encode($callback_param);

  $base64_callback_body = base64_encode($callback_string);
  $now = time();
  $expire = 60;    //设置该policy超时时间是10s. 即这个policy过了这个有效时间，将不能访问。
  $end = $now + $expire;
  $expiration = gmt_iso8601($end);


  //最大文件大小.用户可以自己设置
  $condition = array(0=>'content-length-range', 1=>0, 2=>1048576000);
  $conditions[] = $condition; 

  // 表示用户上传的数据，必须是以$dir开始，不然上传会失败，这一步不是必须项，只是为了安全起见，防止用户通过policy上传到别人的目录。
  $start = array(0=>'starts-with', 1=>'$key', 2=>$dir);
  $conditions[] = $start;

  $arr = array('expiration'=>$expiration,'conditions'=>$conditions);
  $policy = json_encode($arr);
  $base64_policy = base64_encode($policy);
  $string_to_sign = $base64_policy;
  $signature = base64_encode(hash_hmac('sha1', $string_to_sign, $key, true));

  $response = array();
  $response['accessid'] = $id;
  $response['host'] = $host;
  $response['policy'] = $base64_policy;
  $response['signature'] = $signature;
  $response['expire'] = $end;
  $response['callback'] = $base64_callback_body;
  $response['dir'] = $dir;  // 这个参数是设置用户上传文件时指定的目录。
  echo json_encode($response);
?>

```

* 小程序核心上传代码

```
const host = '<host>';
const signature = '<signatureString>';
const ossAccessKeyId = '<accessKey>';
const policy = '<policyBase64Str>';
const key = '<object name>'; //就是服务端的dir+'/'+文件名(带文件类型后缀)
const filePath = '<filePath>'; // 待上传文件的文件路径。
wx.uploadFile({
  url: host, // 开发者服务器的URL。
  filePath: filePath,
  name: 'file', // 必须填file。
  formData: {
    key,
    policy,
    OSSAccessKeyId: ossAccessKeyId,
    signature
  },
  success: (res) => {
    if (res.statusCode === 204) {
      console.log('上传成功');
    }
  },
  fail: err => {
    console.log(err);
  }
});
```