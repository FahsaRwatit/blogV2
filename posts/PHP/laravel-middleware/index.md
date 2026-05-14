---
title: Laravel的中间件原理
date: 2021-03-01 18:17:00
description: Laravel的中间件原理
slug: laravel-middleware
image:
categories:
    - PHP
tags: ["PHP","Laravel"]
---

在上一篇文章中介绍过 [Laravel 的生命周期](https://www.fahsa.cn/php/laravel-life-cycle/)，这也算是对Laravel 框架入门的一些了解，最近几天也继续探究了下 Laravel 的中间件，出于好奇于是通过查看源码和阅读几篇深度好文，也是对 Laravel 的中间件有了深刻的印象，本篇文章比较长建议结合 IDE 参照源码进行理解。

### 目录

- 什么是中间件

- 创建中间件

- array_reduce 函数

- 中间件源码分析

  <!--more-->

### 什么是中间件

中间件在很多框架中有所应用，提供了一种机制方便过滤进入 HTTP 的一些请求。

在 Laravel 中：Request -> Middleware -> Middleware等 -> Response

### 创建中间件

1、生成中间键类：php artisan make:middleware XXX

2、编写代码，在中间件类的 handle 方法中实现前置中间件 `return $next($request)之前` 或者后置中间件`return $next($request)之后` 

3、注册中间件：在`app/Http/Kernel.php` 中注册一个中间件

### array_reduce 函数

因为在源码中用到了 array_reduce 函数，所以先了解它的具体用法，方便后续理解。

```php
// array_reduce — 用回调函数迭代地将数组简化为单一的值
mixed array_reduce( array $input, callable $function[, mixed $initial = NULL] )
// array_reduce() 将回调函数 function 迭代地作用到 input 数组中的每一个单元中，从而将数组简化为单一的值。 

## 参数
    input 需要迭代的数组
    
    $function 回调函数
    
    initial 如果指定了可选参数 initial，该参数将被当成是数组中的第一个值来处理，或者如果数组为空的话就作为最终返回值。    
```

array_reduce 函数举例1：

```php
$arr = ['aaa', 'bbb', 'ccc'];
$res = array_reduce($arr, function($carry, $piple){
    return $carry.$piple;
});
/*
结果：
1、第一次迭代：$carry=null,$piple='aaa'，返回'aaa'
2、第二次迭代：$carry='aaa'.$piple='bbb',返回'aaabbb'
2、第二次迭代：$carry='aaabbb'.$piple='ccc',返回'aaabbbccc'
*/
    
```

array_reduce 函数举例2（带初始值情况）：

```php
$arr = ['aaa', 'bbb', 'ccc'];
$res = array_reduce($arr, function($carry, $piple){
    return $carry.$piple;
}, '初始值-');
/*
结果：
1、第一次迭代：$carry='初始值-',$piple='aaa'，返回'初始值-aaa'
2、第二次迭代：$carry='初始值-aaa'.$piple='bbb',返回'初始值-aaabbb'
2、第二次迭代：$carry='初始值-aaabbb'.$piple='ccc',返回'初始值-aaabbbccc'
*/
```

### 源码分析

有关于中间件的执行逻辑是从 public/index.php 的 handle 开始，进入 handle 方法 的 

```php
// 处理请求
$response = $kernel->handle(
    // 创建请求实例
    $request = Illuminate\Http\Request::capture()
);
```

一个细节可以让你很好的理解为什么注册中间件需要在 app/Http/Kernel.php 中修改相关参数。

在 app/Http/Kernel.php  中查看源码可以看到  app/Http/Kernel.php 这个文件的类继承 Illuminate/Foundation/Http/Kernel.php ，所以在 app/Http/Kernel.php 中注册的中间件将会在父类中处理。

```php
// lluminate/Foundation/Http/Kernel.php 
protected $middleware = [];
protected $middlewareGroups = [];
protected $routeMiddleware = [];
```

接着查看，vendor/laravel/framework/src/Illuminate/Foundation/Http/Kernel.php 的 handle 方法的sendRequestThroughRouter

```php
// $response = $this->sendRequestThroughRouter($request);

protected function sendRequestThroughRouter($request)
{
    $this->app->instance('request', $request);

    Facade::clearResolvedInstance('request');

    $this->bootstrap();
    
    //这里之后就会去处理中间件
    return (new Pipeline($this->app))
        ->send($request)
        ->through($this->app->shouldSkipMiddleware() ? [] : $this->middleware)
        ->then($this->dispatchToRouter());
}

```

处理中间件：

1、构造 `Illuminate\Routing\Pipeline` 实例

2、将请求实例 `$request` 赋值给 Pipeline 的 passable 属性

3、在 through 方法中判断是否启用中间件（默认启用），启用的话则将全局中间件数组 `$middleware` 赋值到 `Pipeline` 的 `$pipes` 属性

4、调用 then 其中 dispatchToRouter 返回一个闭包函数，这个闭包函数将在处理完全局中间件逻辑后执行。

Illuminate/Pipeline/Pipeline.php  的send方法、through 方法

```php
public function send($passable)
{
    $this->passable = $passable;

    return $this;
}
public function through($pipes)
{
    $this->pipes = is_array($pipes) ? $pipes : func_get_args();

    return $this;
}
```

Illuminate/Pipeline/Pipeline.php  的 then方法

```php
public function then(Closure $destination)
{
    $pipeline = array_reduce(
        array_reverse($this->pipes), $this->carry(), $this->prepareDestination($destination)
    );

    return $pipeline($this->passable);
}
```

分析 then 方法：

调用 array_reduce 进行预处理，array_reverse($this->pipes) 指的是倒序的待处理的全局中间件数组。

$this->carry() : 返回的是回调函数。

```php
// Illuminate/Routing/Pipeline
protected function carry()
{
    return function ($stack, $pipe) {
        return function ($passable) use ($stack, $pipe) {
            try {
                $slice = parent::carry();
                $callable = $slice($stack, $pipe);
                return $callable($passable);
            } catch (Exception $e) {
                return $this->handleException($passable, $e);
            } catch (Throwable $e) {
                return $this->handleException($passable, new FatalThrowableError($e));
            }
        };
    };
}
```

`$this->prepareDestination($destination) ` 是 carry 函数中 `$stack` 的初始值，该函数返回的也是闭包函数，传入的闭包参数 `$destination` 为 `$this->dispatchToRouter()` 返回的闭包函数，其中包含了路由和中间件的匹配和执行 的相关逻辑。

```php
// Illuminate/Routing/Pipeline 文件中
protected function prepareDestination(Closure $destination)
{
    return function ($passable) use ($destination) {
        try {
            return $destination($passable);
        } catch (Exception $e) {
            return $this->handleException($passable, $e);
        } catch (Throwable $e) {
            return $this->handleException($passable, new FatalThrowableError($e));
        }
    };
}
```

当用 array_reduce 遍历完所有的 pipe 后，因为 carry 返回的是闭包函数，所以现在也未能执行中间件的 handle 方法 ，

以下代码是将所有全局中间件和路由处理通过统一的闭包函数进行迭代调用。最终返回的 `$pipeline` 是一个闭包函数：

```php
// carry 方法中返回
return function ($stack, $pipe) {
    return function ($passable) use ($stack, $pipe) {
        try {
            $slice = parent::carry();

            $callable = $slice($stack, $pipe);

            return $callable($passable);
        } catch (Exception $e) {
            return $this->handleException($passable, $e);
        } catch (Throwable $e) {
            return $this->handleException($passable, new FatalThrowableError($e));
        }
    };
};
```

`$pipe` 的值是全局中间件的第一个值，`$stack` 代表之前迭代的所有全局中间件和路由处理闭包

当代码执行到 then 方法的 `return $pipeline($this->passable);` 时才开始执行以下片段：

```php
return function ($passable) use ($stack, $pipe) {
    try {
        $slice = parent::carry();

        $callable = $slice($stack, $pipe);

        return $callable($passable);
    } catch (Exception $e) {
        return $this->handleException($passable, $e);
    } catch (Throwable $e) {
        return $this->handleException($passable, new FatalThrowableError($e));
    }
};
```

其中 `$passable` 为当前的请求实例 `$request` ，`$stack` 和 `$pipe` 则为 array_reduce 处理后的变量，上述代码中调用了父类的 carry 方法：

```php

protected function carry()
{
    return function ($stack, $pipe) {
        return function ($passable) use ($stack, $pipe) {
            if (is_callable($pipe)) {
                return $pipe($passable, $stack);
            } elseif (! is_object($pipe)) {
                [$name, $parameters] = $this->parsePipeString($pipe);
                $pipe = $this->getContainer()->make($name);
                $parameters = array_merge([$passable, $stack], $parameters);
            } else {
                $parameters = [$passable, $stack];
            }
            $response = method_exists($pipe, $this->method)
                ? $pipe->{$this->method}(...$parameters)
            : $pipe(...$parameters);
            return $response instanceof Responsable
                ? $response->toResponse($this->getContainer()->make(Request::class))
                : $response;
        };
    };
}
```

当前迭代的 `$pipe` 是闭包的话，直接执行并返回；如果`$pipe` 是对象的话，则实例化该对象并且解析出中间件参数，然后调用对象实例的 `handle` 方法处理相应的业务逻辑并且返回`$response` ，最后判断返回的`$response` 是否为Responsable实例，如果是的话直接返回HTTP响应到客户端，否则继续处理后续的中间件。

当迭代到最后一个全局中间件时，请求进入`$this->prepareDestination` 中返回的闭包函数 `return $destination($passable);`

```php
protected function prepareDestination(Closure $destination)
{
    return function ($passable) use ($destination) {
        return $destination($passable);
    };
}

prepareDestination 返回的是一个匿名函数，调用 prepareDestination 的时候并没有真正执行这个匿名函数( $destination($passable) )
```

通过代码来源发现最终执行的是：Illuminate\Foundation\Http\Kernel.php中的 dispatchToRouter方法返回的闭包函数：

```php
function ($request) {
    $this->app->instance('request', $request);

    return $this->router->dispatch($request);
};
```

其中 dispatch 方法如下：

```php
// Illuminate\Routing\Router.php 文件内
public function dispatch(Request $request)
{
    $this->currentRequest = $request;

    return $this->dispatchToRoute($request);
}
```

dispatchToRoute 执行的业务逻辑如下：

```php
// Illuminate\Routing\Router.php 文件内
public function dispatchToRoute(Request $request)
{
    return $this->runRoute($request, $this->findRoute($request));
}
```

首先根据 通过 findRoute 方法根据当前的请求匹配出对应的路由定义，路由定义中包含中间件等信息（若不存在路由定义，则返回404错误），接着执行 runRoute 方法：

```php
// 返回给定路由的响应。
protected function runRoute(Request $request, Route $route)
{
    $request->setRouteResolver(function () use ($route) {
        return $route;
    });

    $this->events->dispatch(new Events\RouteMatched($route, $request));

    return $this->prepareResponse($request,
                                  $this->runRouteWithinStack($route, $request)
                                 );
}
```

runRoute 中根据请求和路由定义的信息，去执行相应的闭包函数或者控制器方法。

核心函数 runRouteWithinStack：

```php
protected function runRouteWithinStack(Route $route, Request $request)
{
    $shouldSkipMiddleware = $this->container->bound('middleware.disable') &&
        $this->container->make('middleware.disable') === true;

    $middleware = $shouldSkipMiddleware ? [] : $this->gatherRouteMiddleware($route);

    return (new Pipeline($this->container))
        ->send($request)
        ->through($middleware)
        ->then(function ($request) use ($route) {
            return $this->prepareResponse(
                $request, $route->run()
            );
        });
}
```

通过 middleware.disable 判断是否禁用了 Laravel 中间件，如果禁用了则返回空数组，否则通过 gatherRouteMiddleware 从路由定义中收集中间件信息，并且按照优先级对中间件进行排序，然后调用 Kernel 中类似的代码对路由中间件进行处理，逻辑和前面的全局中间件类似，如果全部中间件校验通过，则传入闭包中的逻辑：

```php
return $this->prepareResponse(
    $request, $route->run()
);
```

准备好响应数据

接着来到 public/index.php 中的 `$response->send();` 将响应数据返回给客户端

最后是 terminate 逻辑的调用，Laravel 中间件会检查全局中间件和路由中间件中是否包含 trminate 方法，如果包含的话则认为改中间件为 终端中间件，并执行其中的 terminate 方法。

terminate 中间件对应的底层代码如下：

```php
protected function terminateMiddleware($request, $response)
{
    $middlewares = $this->app->shouldSkipMiddleware() ? [] : array_merge(
        $this->gatherRouteMiddleware($request),
        $this->middleware
    );

    foreach ($middlewares as $middleware) {
        if (! is_string($middleware)) {
            continue;
        }

        [$name] = $this->parseMiddleware($middleware);

        $instance = $this->app->make($name);

        if (method_exists($instance, 'terminate')) {
            $instance->terminate($request, $response);
        }
    }
}
```