---
title: Laravel中操作Redis的各种数据类型
date: 2019-03-05 18:17:00
description: Laravel中操作Redis的各种数据类型
slug: laravel-redis-datatype
image:
categories:
    - PHP
tags: ["PHP","Laravel"]

---

首先我在 Laravel 框架中用`composer require predis/predis`安装了redis。

然后我直接在框架中使用redis，因为报了个错误`Please make sure the PHP Redis extension is installed and enabled`，所以我了解到：

> php连接redis常用的有两种驱动，phpredis和predis。
> phpredis是以php扩展方式安装的，安装步骤比较繁琐，执行效率稍高。
> predis是用php代码写的，安装非常简单，执行效率稍差。

因为我在之前的文章中写过使用`redis`，现在改为`predis`试试。

<!--more-->

配置文件中redis的client改为：`'client' => env('REDIS_CLIENT', 'predis'),`

然后接下来就是有关于Laravel中操作各种Redis类型的一些方式。

## 字符串String

```php
// string
Redis::set("name","Mai");
$name = Redis::get("name");
echo $name; // Mai

Redis::set("name","Kim");
$name = Redis::get("name");
echo $name; // Kim
```

## 列表List

```php
// List: 存数据到列表
Redis::lpush("fruits","apple");
Redis::lpush("fruits","banana");
Redis::lpush("fruits","pear");

// 获取列表中的值
$fruits = Redis::lrange("fruits",0,-1);
var_dump($fruits);// array(3) { [0]=> string(4) "pear" [1]=> string(6) "banana" [2]=> string(5) "apple" }

// 往右侧添加数据
Redis::rpush("fruits","strawberry");
$fruits = Redis::lrange("fruits",0,-1);
var_dump($fruits); // array(4) { [0]=> string(4) "pear" [1]=> string(6) "banana" [2]=> string(5) "apple" [3]=> string(10) "strawberry" }

// 左侧弹出一个
Redis::lpop("fruits");
$fruits = Redis::lrange("fruits",0,-1);
var_dump($fruits);

// 右侧弹出
Redis::rpop("fruits");
$fruits = Redis::lrange("fruits",0,-1);
var_dump($fruits);
```

## 哈希字典Hash

```php
// Hash(哈希字典)
// 如果没有则设置成功,返回1,如果存在会替换原有的值,返回0,失败返回0
$res = Redis::hset("animals","dog","dog1");
var_dump($res); // 1

$res = Redis::hset("animals","dog","dog1");
var_dump($res); // 0

$res = Redis::hset("animals","dog","dog2");
var_dump($res); // 0

$res = Redis::hset("animals","cat","cat1");
var_dump($res); // 1

$res = Redis::hset("animals","monkey","monkey1");
var_dump($res); // 1

// 获取值
$dog = Redis::hget("animals","dog");
var_dump($dog); // string(4) "dog2"

//        // 获取所有key
$keys = Redis::hkeys("animals");
var_dump($keys); // array(3) { [0]=> string(3) "dog" [1]=> string(3) "cat" [2]=> string(6) "monkey" }

// 获取所有值，顺序是随机的
$val = Redis::hvals("animals");
var_dump($val); // array(3) { [0]=> string(4) "dog2" [1]=> string(4) "cat1" [2]=> string(7) "monkey1" }

// 获取所有key和value ，顺序随机
$animals = Redis::hgetall("animals");
var_dump($animals); // array(3) { ["dog"]=> string(4) "dog2" ["cat"]=> string(4) "cat1" ["monkey"]=> string(7) "monkey1" }

```

## 集合Set

```php
// Set
Redis::sadd("set","cat"); //1
Redis::sadd("set","cat"); // 0
Redis::sadd("set","dog");
Redis::sadd("set","bear");

// // 查看集合中所有的元素
$set = Redis::smembers("set");
var_dump($set); // array(3) { [0]=> string(3) "dog" [1]=> string(4) "bear" [2]=> string(3) "cat" }

// 删除集合中元素
$res = Redis::srem("set","cat");
var_dump($res); // 1


// 判断元素是否是set的成员
$res = Redis::sismember("set","cat");
var_dump($res); // 0

// 查看集合中成员的数量
$res = Redis::scard("set");
var_dump($res); // 2

// 移除并返回集合中的一个随机元素(返回被移除的元素)
$res = Redis::spop("set");
var_dump($res); // string(3) "dog"


Redis::sadd("set1","horse");
Redis::sadd("set1","cat");
Redis::sadd("set1","dog");
Redis::sadd("set1","bird");

Redis::sadd("set2","fish");
Redis::sadd("set2","dog");
Redis::sadd("set2","bird");

$res1 = Redis::smembers("set1");
var_dump($res1); // array(4) { [0]=> string(3) "dog" [1]=> string(5) "horse" [2]=> string(4) "bird" [3]=> string(3) "cat" }
$res2 = Redis::smembers("set2");
var_dump($res2); // array(3) { [0]=> string(3) "dog" [1]=> string(4) "bird" [2]=> string(4) "fish" }


//返回集合的交集
$res = Redis::sinter("set1","set2");
var_dump($res); // array(2) { [0]=> string(3) "dog" [1]=> string(4) "bird" }

// 执行交集操作 并结果放到一个集合中
Redis::sinterstore("set3","set1","set2");
$res = Redis::smembers("set3");
var_dump($res); // array(2) { [0]=> string(3) "dog" [1]=> string(4) "bird" }

//返回集合的并集
$res = Redis::sunion("set1","set2");
var_dump($res); //array(5) { [0]=> string(4) "bird" [1]=> string(3) "cat" [2]=> string(3) "dog" [3]=> string(5) "horse" [4]=> string(4) "fish" }

// 执行并集操作 并结果放到一个集合中
Redis::sunionstore("set4","set1","set2");
$res = Redis::smembers("set4");
var_dump($res); //array(5) { [0]=> string(4) "bird" [1]=> string(3) "cat" [2]=> string(3) "dog" [3]=> string(5) "horse" [4]=> string(4) "fish" }

// 返回集合的差集
$res = Redis::sdiff("set1","set2");
var_dump($res); //array(2) { [0]=> string(5) "horse" [1]=> string(3) "cat" }

// 执行差集操作 并结果放到一个集合中
Redis::sdiffstore("set5","set1","set2");
$res = Redis::smembers("set5");
var_dump($res); // array(2) { [0]=> string(5) "horse" [1]=> string(3) "cat" }
```

## 有序集合Sorted Set

```php
// Sorted Set (有序集合)
Redis::zadd("zset",1,"dog");
Redis::zadd("zset",2,"cat");
Redis::zadd("zset",3,"fish");
Redis::zadd("zset",4,"dog");
Redis::zadd("zset",5,"bird");

// 返回集合中所有元素
$res = Redis::zrange("zset",0,-1);
var_dump($res); // array(4) { [0]=> string(3) "cat" [1]=> string(4) "fish" [2]=> string(3) "dog" [3]=> string(4) "bird" }

// 返回元素的score值
$res = Redis::zscore("zset","dog");
var_dump($res); // string(1) "4"

//返回存储的个数
$res = Redis::zcard("zset");
var_dump($res); // int(4)


// 删除指定成员
Redis::zrem("zset","cat");
$res = Redis::zrange("zset",0,-1);
var_dump($res); // array(3) { [0]=> string(4) "fish" [1]=> string(3) "dog" [2]=> string(4) "bird" }

//返回集合中介于min和max之间的值的个数
$res = Redis::zcount("zset",3,5);
var_dump($res); // var_dump($res);

// 返回有序集合中score介于min和max之间的值
$res = Redis::zrangebyscore("zset",3,5);
var_dump($res); // array(3) { [0]=> string(4) "fish" [1]=> string(3) "dog" [2]=> string(4) "bird" }

//        // 返回集合中指定区间内所有的值 倒叙
$res = Redis::zrevrange("zset",1,2);
var_dump($res); // array(2) { [0]=> string(3) "dog" [1]=> string(4) "fish" }

// 有序集合中指定值的socre增加
$res = Redis::zscore("zset","dog");
var_dump($res); // string(1) "4"
Redis::zincrby("zset",2,"dog");
$res = Redis::zscore("zset","dog");
var_dump($res); //  string(1) "6"


//移除score值介于min和max之间的元素
$res = Redis::zrange("zset",0,-1);
var_dump($res); //array(3) { [0]=> string(4) "fish" [1]=> string(4) "bird" [2]=> string(3) "dog" }

$res = Redis::zremrangebyscore("zset",3,4);
var_dump($res); // int(1)

$res = Redis::zrange("zset",0,-1);
var_dump($res); // array(2) { [0]=> string(4) "bird" [1]=> string(3) "dog" }
```