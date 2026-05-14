---
title: PHP 关于字符串的处理
date: 2019-02-10 19:17:00
description: PHP 关于字符串的处理
slug: php-string
image:
categories:
    - PHP
tags: ["PHP","Laravel"]
---

### 长度

```php
int strlen ( string $string )	// 返回给定的字符串 string 的长度。
```

### 顺序

```php
string str_shuffle ( string $str ) // str_shuffle 随机打乱一个字符串
string strrev ( string $string )   // strrev 反转字符串
```

<!--more-->

### 空白处理

```php
trim // 去掉字符串首尾的空白字符  （或者其他字符）
ltrim // 删除字符串开头的空白字符   （或者其他字符）
rtrim/chop：删除字符串末端的的空白字符   （或者其他字符）   第二个参数为要去除的其他字符。   
```

### 分割拼接

```php
// 使用一个字符串分割另一个字符串，将字符串按照给定字符转为数组
array explode ( string $delimiter , string $string [, int $limit ] )

implode/join // 按照指定的字符串进行拼接成字符串（将一个一维数组的值转化为字符串）
    
array str_split ( string $string [, int $split_length = 1 ] ) // 将一个字符串转化为数组

void parse_str ( string $str [, array &$arr ] ) // 将字符串解析成多个变量
    $str = "first=value&arr[]=foo+bar&arr[]=baz";
    parse_str($str);
    echo $first;  // value
    echo $arr[0]; // foo bar
    echo $arr[1]; // baz

string str_repeat ( string $input , int $multiplier ) // 重复一个字符串

```

### 比较

```php
int strcmp ( string $str1 , string $str2 ) // strcmp 二进制比较字符串
int strcasecmp ( string $str1 , string $str2 ) // strcasecmp   二进制安全比较字符串（不区分大小写） j
string bin2hex ( string $str ) // 将二进制字符串转为16进制字符串
string hex2bin ( string $data )  // 将16进制字符串转为二进制(可见字符)字符串      
```

### 查找定位

```php
strstr // 查找字符串的首次出现(区分大小写)
stristr // 查找字符串的首次出现 (不区分大小写)

strrchr // 返回最后一次出现到结尾的内容

strpos // 返回首次出现的位置。返回值是数字或者false
stripos // 查找字符串首次出现的位置（不区分大小写）

strrpos // 返回最后一次出现的位置.返回值是数字或者false.
strripos // strrpos忽略大小写的版本    
    
string substr ( string $string , int $start [, int $length ] ) // substr: 返回字符串的子串

strpbrk // 返回(字符列表中任意字符)首次出现到结尾的内容
    $text = 'This is a Simple text.';

    // 输出 "is is a Simple text."，因为 'i' 先被匹配
    echo strpbrk($text, 'mi');

    // 输出 "Simple text."，因为字符区分大小写
    echo strpbrk($text, 'S');
    
```

### 修改

```php
// 用指定的字符串替换指定的字符串
mixed str_replace ( mixed $search , mixed $replace , mixed $subject [, int &$count ] ) 
str_ireplace：str_replace的忽略大小写版本
```

### ip 转换

```php
long2ip: 将整型数字转化为ip
ip2long: 将ip转化为整型数字    
```

### 加密

```php
// md5 加密
string md5 ( string $str [, bool $raw_output = false ] )
```