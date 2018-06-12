---
title: php和java的sha1
date: 2018-06-12 19:31:00
tags: php
---

对接一个java的api,加密的方法很简单,3次sha1后用BASE64算法生成字符串
<!--more-->
java代码如下

```
package com.xxxxx.common.utils;

import java.security.MessageDigest;

import org.apache.commons.codec.binary.Base64;

public class SHAUtils {

  public static final String SHA1 = "SHA-1";

  public static final int HASH_ITERATIONS = 3;

  public static final String DEF_CHARSET = "UTF-8";

  public static String shaDigest(String str, String algorithm, int iterations) {
    try {
      MessageDigest crypt = MessageDigest.getInstance(algorithm);
      byte[] result = crypt.digest(str.getBytes(DEF_CHARSET));

      for (int i = 1; i < iterations; i++) {
        crypt.reset();
        result = crypt.digest(result);
      }
      return Base64.encodeBase64URLSafeString(result);
    } catch (Exception e) {
      e.printStackTrace();
    }
    return null;
  }

  public static void main(String[] args) {
    String code = SHAUtils.shaDigest("12001JIE18611111111ANwSYoBbT2poXZF6ARs52w", SHAUtils.SHA1,
        SHAUtils.HASH_ITERATIONS);
    System.out.println(code);
  }

}
```

php代码如下

```
<?php
function base64url_encode($string) {
  return rtrim(strtr(base64_encode($string),'+/','-_'),'=');
}

$str = '12001JIE18611111111ANwSYoBbT2poXZF6ARs52w';
$result = sha1($str,true);

for ($i=1; $i < 3; $i++) {
  $result = sha1($result,true);
}

echo base64url_encode($result).PHP_EOL;
```

需要注意一个地方，那就是php端sha1函数第二个参数，默认是FALSE，输出格式是40字符十六进制数，这里需要输出原始20字符二进制格式


