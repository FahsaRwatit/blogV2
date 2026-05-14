---
title: Laravel中包含多个路由文件
date: 2019-03-05 18:17:00
description: Laravel基于Composer实现自动加载原理分析
slug: laravel-router
image:
categories:
    - PHP
tags: ["PHP","Laravel"]

---

因为接口比较多，想按照模块给route做个区分，分成多个路由文件。

这里有2中方法：

1、用`include_once`包含

2、根据Laravel的加载方式引入

<!--more-->

## 用`include_once`包含

因为我引入的是关于接口的路由，所以在 `api.php`文件中添加：

```php
include_once '../routes/api/test.php'; // test.php 路由文件文件名
```

## 根据Laravel的加载方式引入

找到`RouteServiceProvider`类，在类中添加如下代码：

```php
 public function map()
 {
     $this->mapApiRoutes();

     $this->mapWebRoutes();
     $this->mapOtherApiRoutes(); // 新加的
     //
 }
// 新加的
protected function mapOtherApiRoutes(){
    /**
      * routes/api分组
      */
    foreach(glob(base_path("routes/api/")."*.php")as $file){
        Route::prefix('api')
            ->middleware('api')
            ->namespace($this->namespace)
            ->group($file);
    }
}
```

