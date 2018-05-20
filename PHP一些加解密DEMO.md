---
title: PHP一些加解密DEMO
date: 2018-04-12 15:56:54
tags: php
---

由于工作中要和各个第三方平台对接API，接口之间通信就涉及到了加密或者验签，网上有很多关于php加解密的DEMO，但是混乱不堪，很多甚至还是PHP5的例子，在PHP7环境下根本无法运行，下面就放一些常见加解密的DEMO，包括RSA非对称加密，DES对称加密
<!--more-->
#### 常用的对称加密算法有DES与AES
1.对称密码中加密和解密使用的是同一个密码
2.对称密码中加密后的密文只有该对称密码才能解密

|  | DES | AES |
| --- | --- | --- |
| 密钥长度 | 56位 | 128, 192, 256 位 |
| 加密方式 | 对称分组密码 | 对称分组密码 |
| 加密轮数 | 16轮 | 128位10轮，192位12轮，256位14轮 |
| 安全性 | 被攻破 | 安全 |
| 速度 | 较慢 | 较慢 |

#### 非对称密码（也称为公钥密码）与对称密码的不同点是非对称密码包含一个公钥和一个私钥
1.公钥和私钥是严格符合数学关系一一对应且不可互换的
2.公钥可以公开发布，私钥必须自己保管，不可以泄露
3.使用公钥加密的内容只有对应的私钥才能解密，使用私钥签名的内容只有对应的公钥才能解密


关于密码学的介绍可以看这篇文章[TLS协议分析与现代加密通信协议设计](https://blog.helong.info/blog/2015/09/07/tls-protocol-analysis-and-crypto-protocol-design)，写的很详细了

#### RSA私钥加密，公钥验签
```
    /**
     * RSA签名验证
     * @param string $sign
     * @param array $data
     * @return bool
     */
     function checkSign($sign, $data)
    {
        //字典序排序
        ksort($data);
        $str = '';
        //分割
        foreach ($data as $k => $v) {
            if ($k != 'sign' && $v != '' && !is_array($v) || $v == '0') {
                $str .= $k . '=' . $v . '&';
            }
        }
        //将解密的结果和传输的数据进行比较
        $str = trim($str,'&');
        //读取公钥文件
        $aKey = file_get_contents('公钥');
        //转换为openssl格式密钥
        $res = openssl_get_publickey($aKey);

        $ret = (bool)openssl_verify($str,base64_decode($sign),$res);

        return $ret;
    }

    /**
     * 生成RSA签名
     * @param array $data
     * @return string
     */
     function getSign($data)
    {
        //字典序排序
        ksort($data);
        $str = '';
        //分割
        foreach ($data as $k => $v) {
            if ($k != 'sign' && $v != '' && !is_array($v) || $v == '0') {
                $str .= $k . '=' . $v . '&';
            }
        }
        $source = trim($str,'&');
        //读取私钥文件
        $aKey = file_get_contents('私钥');
        //转换为openssl格式密钥
        $res = openssl_pkey_get_private($aKey);

        openssl_sign($source,$output,$res);

        $sign = base64_encode($output);

        return $sign;
    }
```
#### RSA私钥分段加密，公钥解密
```
//分段长度
define("ENCRYPT_LEN", 32);

 // 私钥加密
function encryptedByPrivateKey($data_content,$private_key){
    $data_content = base64_encode($data_content);
     $encrypted = "";
     $totalLen = strlen($data_content);
     $encryptPos = 0;
     while ($encryptPos < $totalLen){
         openssl_private_encrypt(substr($data_content, $encryptPos, ENCRYPT_LEN), $encryptData,$private_key);
         $encrypted .= bin2hex($encryptData);
         $encryptPos += ENCRYPT_LEN;
         }
     return $encrypted;
}
    
// 公钥解密
function decryptByPublicKey($encrypted,$public_key){
      $decrypt = "";
      $totalLen = strlen($encrypted);
      var_dump($encrypted);
      $decryptPos = 0;
      while ($decryptPos < $totalLen) {
          openssl_public_decrypt(hex2bin(substr($encrypted, $decryptPos, ENCRYPT_LEN * 8)), $decryptData, $public_key);
          $decrypt .= $decryptData;
          $decryptPos += ENCRYPT_LEN * 8;
      }
      //openssl_public_decrypt($encrypted, $decryptData, $this->public_key);
      $decrypt = base64_decode($decrypt);
      return $decrypt;
}
```
#### DES加密(3des加密CBC模式,PKCS5Padding填充)
```
    /**
     * des加密
     * @desc 3des加密CBC模式,PKCS5Padding填充
     * @param $jsonData
     * @return string
     */
    function encrypt($jsonData)
    {
        $jsonData = urlencode($jsonData);
        $str = pkcs5_pad($jsonData, 8);
        if (strlen($str) % 8) {
            $str = str_pad($str,
                strlen($str) + 8 - strlen($str) % 8, "\0");
        }
        return bin2hex(openssl_encrypt($str, 'des-ede3-cbc', substr($this->desKey, 0, 8), OPENSSL_RAW_DATA | OPENSSL_NO_PADDING, substr($this->desKey, 0, 8)));
    }

    /**
     * des解密
     * @param $str
     * @return string
     */
    function decrypt($str)
    {
        $str = hex2bin($str);

        return openssl_decrypt($str, 'des-ede3-cbc', substr($this->desKey, 0, 8), OPENSSL_RAW_DATA | OPENSSL_NO_PADDING, substr($this->desKey, 0, 8));
    }
    
        /**
     * PKCS5Padding填充
     * @param $text
     * @param $blocksize
     * @return string
     */
    function pkcs5_pad($text, $blocksize)
    {
        $pad = $blocksize - (strlen($text) % $blocksize);
        return $text . str_repeat(chr($pad), $pad);
    }
```


