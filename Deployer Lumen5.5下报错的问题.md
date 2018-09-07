---
title: Deployer Lumen5.5下报错的问题
date: 2018-08-20 18:19:00
tags: Deployer php
---

用Deployer作为lumen的代码部署工具，dep init后报

```
php artisan storage:link failed
```
在改了默认的deploy.php后可以正常部署了,附上代码
<!--more-->
```
<?php
namespace Deployer;

require 'recipe/laravel.php';

// Project name
set('application', 'xxxxx');
set('default_stage', 'production');
set('keep_releases', 5);
set('default_timeout', 6000);
// Project repository
set('repository', 'git@xxxxxx.git');
set('ssh_type', 'native');
set('ssh_multiplexing', false);

add('shared_files', [
    '.env'
]);
add('shared_dirs', [
    'storage'
]);

// Writable dirs by web server
add('writable_dirs', [
    'bootstrap/cache',
    'storage',
    'storage/app',
    'storage/app/public',
    'storage/framework',
    'storage/framework/cache',
    'storage/framework/sessions',
    'storage/framework/views',
    'storage/logs',
]);

// [Optional] Allocate tty for git clone. Default value is false.
set('git_tty', true); 

// Hosts

host('xxx.com')
    ->user('root')
    ->identityFile('/Users/mc/.ssh/deployerkey')
    ->set('deploy_path', '/data/home/www/xxx')
    ->stage('production');
// [Optional] if deploy fails automatically unlock.
after('deploy:failed', 'deploy:unlock');

task('deploy', [
    'deploy:prepare',
    'deploy:lock',
    'deploy:release',
    'deploy:update_code',
    'deploy:shared',
    'deploy:vendors',
    'deploy:writable',
    'artisan:cache:clear',
    'artisan:optimize',
    'deploy:symlink',
    'deploy:unlock',
    'cleanup',
]);
```

附上Deployer的详细安装教程[教程](https://laravel-china.org/articles/13242/another-introduction-to-the-use-of-deployer)

这里要注意deploy task会默认执行artisan:cache:clear，每次发布会把redis清空，这样会产生严重的生产问题，解决方法有两个，一个是重写task，直接忽略这个命令

```
task('artisan:cache:clear', function () {
    return true;
});
```

另外一个处理方式，那就是Cache,Queue,Session链接不同的redis片区

```
'redis' => [

   'cluster' => false,

   'cahce' => [
       'host'     => env('REDIS_HOST', 'localhost'),
       'password' => env('REDIS_PASSWORD', null),
       'port'     => env('REDIS_PORT', 6379),
       'database' => 0,
   ],

   'session' => [
         'host'     => env('REDIS_HOST', 'localhost'),
         'password' => env('REDIS_PASSWORD', null),
         'port'     => env('REDIS_PORT', 6379),
         'database' => 1,
   ],
   
   'queue' => [
         'host'     => env('REDIS_HOST', 'localhost'),
         'password' => env('REDIS_PASSWORD', null),
         'port'     => env('REDIS_PORT', 6379),
         'database' => 2,
   ],
],
```

总的来说，还是不要再生产环境运行artisan:cache:clear命令比较好

