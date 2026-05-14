---
title: Laravel中使用Redis
date: 2019-03-02 18:17:00
description: Laravel中使用Redis
slug: laravel-redis1
image:
categories:
    - PHP
tags: ["PHP","Laravel"]

---

### Windows环境下安装Redis

1、下载：https://github.com/MicrosoftArchive/redis/releases

2、电脑新建 Redis 目录，将下载的 Redis-x64-3.0.504.zip 解压在该目录下

3、进入 Redis 目录，打开命令行 cmd ，启动 Redis ：`redis-server.exe redis.windows.conf` 这时候若看到 Redis 特有的标识则代表服务启动成功，但是使用上面的启动命令，只要关闭了cmd窗口，服务将立即停止，所以需要将 Redis 设置为 Windows 下的服务，应该使用 `redis-server --service-install redis.windows.conf` 命令，执行成功后将会在电脑任务管理器的服务下看到 Redis 服务，这时只要将它开启即可。

<!--more-->

### Laravel 框架中使用 Redis

1、通过 Composer 安装 predis

```php
composer require predis/predis
```

在安装的过程中我遇到了这个问题：

```git
 [Composer\Downloader\TransportException]                                                                          
  curl error 61 while downloading https://mirrors.aliyun.com/composer/p2/predis/predis~dev.json: Error while processi
  ng content unencoding: Unknown failure within decompression software.   
```

解决办法是执行命令：`composer config -g repo.packagist composer https://packagist.org`

然后重新安装即可。

2、配置。在 `config/database.php` 文件中

```php
'redis' => [

    'client' => 'predis',

    'default' => [
        'host' => env('REDIS_HOST', 'localhost'),
        'password' => env('REDIS_PASSWORD', null),
        'port' => env('REDIS_PORT', 6379),
        'database' => 0,
    ],

],
```

然后在 `.evn`文件中设置：

```php
REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
```

3、使用。在这里我新建 TestController 控制器

Redis 命令文档：http://doc.redisfans.com/index.html

```php
class TestController extends Controller
{
    public function index(){
        $data = [
            'username' => '香波',
            'sex' => '女',
            'age' => '25'
        ];
        Redis::set('user:chompoo',json_encode($data));
        $user = Redis::get('user:chompoo');


        $redis = app('redis.connection');
        $user = $redis->get('user:chompoo');

        Redis::set('str1','张三');
        $user = Redis::get('str1'); // 张三

        var_dump(Redis::del('str1')); // 删除 ，成功返回true 失败返回false
        $arr = [
            'str1'=> '张三1',
            'str2'=> '张三2',
            'str3'=> '张三3',
            'str4'=> '张三4',
        ];
        Redis::mset($arr); // 存储多个 key 对应的 value
        $users = Redis::mget(array_keys($arr));

        Redis::rpush('userlist','李四1');
        $userlist = Redis::lrange('userlist',0,-1);

//        var_dump(Redis::llen('userlist'));
        return $userlist;

    }

}
```

第一种连接方式：

```php
use Illuminate\Support\Facades\Redis;
Redis::set('user:chompoo',json_encode($data));
```

第二种连接方式：

```php
$redis = app('redis.connection');
$user = $redis->get('user:chompoo');
```