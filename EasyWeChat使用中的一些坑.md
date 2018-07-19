---
title: EasyWeChat使用中的一些坑
date: 2018-03-28 16:56:54
tags: php EasyWeChat
---

```
遇到百度解决不了的问题多去issue上找找
```

1. 现在开发一般都使用框架开发，例如Laravel、Symfony、Lumen等，框架都有个debug模式，在配置微信回调通知URL时记得把框架debug模式关闭，不然会出现微信公众平台开发者中心配置一直报token验证失败
2. Laravel5.1版本 EasyWeChat4.0版本时会有包冲突 报Symfony\Component\Cache\Simple\FilesystemCache 这个类找不到，解决方法就是把Laravel升级到5.1以上
3. EasyWeChat4以上才支持多个公众号管理
4. EasyWeChat实现多公众号管理的方法

```
use EasyWeChat\Factory;
class Test
{
    public function application($id)
    {
        //不再从配置文件wechat.php读取配置,直接传入，这里可以用数据库来维护各个账户
        $config = [
          'app_id' => '',
          'secret' => '',
          'response_type' => 'array',
        ];
        $app = Factory::officialAccount($config);
        return $app;
    }
}
```