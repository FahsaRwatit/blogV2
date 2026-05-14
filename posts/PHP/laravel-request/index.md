---
title: Laravel的Request请求类分析
date: 2021-03-05 18:00:00
description: Laravel基于Composer实现自动加载原理分析
slug: laravel-request
image:
categories:
    - PHP
tags: ["PHP","Laravel"]
---

### 目录

- 通过Request对象获取用户请求数据
- Laravel 的 Request 实现

<!--more-->

### 通过Request对象获取用户请求数据

为了对 Laravel 的 Request 请求有所了解，我们先熟悉一下在 Laravel 中是如何接收 HTTP 请求获取用户的请求数据的，分为2块内容：获取请求对象和获取请求数据。

ps：喜欢的朋友可以关注公众号：苏小怪的梦呓

#### 获取请求对象

**1、在控制器方法内，通过依赖注入的方式获取**

```php
<?php
namespace App\Http\Controllers;
use Illuminate\Http\Request;

class TestController extends Controller
{
    public function test01(Request $request)
    {
        $username = $request->input('username');
    }
}
```

**2、通过全局辅助函数`request()`**

```php
$request = request();
$username = $request->input('username');
```

1 和 2  这两种方式最终都是通过解析服务容器的方式获取请求实例

**3、通过解析服务容器获取request实例**

以下2种方式都是解析的同一请求实例，`request`是 `\Illuminate\Http\Request::class`类 的别名，在`Illuminate\Foundation\Application`的方法`registerCoreContainerAliases`中有设置：

```php
'request'              => [\Illuminate\Http\Request::class, \Symfony\Component\HttpFoundation\Request::class],
```

解析服务容器获取request请求实例：

```php
$request = app('request');   
$request = app(\Illuminate\Http\Request::class);
```

#### 获取请求数据

**1、获取所有请求数据**

```php
$input = $request->all();
```

**2、获取单个请求数据**

`input` 方法：无论哪种 HTTP 请求，都可以获取到。

`get` 方法：获取通过 GET 请求传递的参数。

`post` 方法：获取通过 POST 请求传递的参数。

```php

$id = $request->input('id');	// 获取指定参数的值
$id = $request->input('id',233);	// 给指定参数设置默认值

// 获取数组参数中的指定值
$name = $request->input('products.0.name');	
$request->input('books.0');

$request->input(); // 未指定参数时获取请求中的所有输入值

$id = $request->get('id');
$id = $request->post('id');
```

**3·、获取请求中的 JSON 数据**

将请求头 ` Content-Type` 设置为 `application/json`，可以通过 `input`方法用`.`的方式访问数据。

```php
$name = $request->input('user.name');
```

**4、通过动态属性进行获取**

```php
$name = $request->input('user.name');
```

**5、其他获取方法**

```php
$request->has('name'); // 判断请求参数是否存在，返回布尔值
$request->filled('name') // 判断请求参数是否存在,并且不为空

$username = $request->old('username'); // 获取旧数据
$value = $request->cookie('name'); // 从cookies中进行获取

$request->except('id'); // 排除指定字段
$request->only(['name', 'site', 'domain']); // 获取指定字段

$request->flash(); // 将当前的输入刷入session中
// 将部分值刷入
$request->flashOnly('username', 'email');
$request->flashExcept('password');
```

### Laravel 的 Request 实现

为了便于理解附上一张简图（如下）：图片来源于 Laravel 学院。

![img](https://qnres.fahsa.cn/images/2ccfcabe332102d1953742c2bee4551a.png)

**在 `public/index.php`**

HTTP 请求入口位于 `public/index.php` 文件，以下是处理 HTTP 请求的相关代码：

```php
// 这个相当于我们创建了Kernel::class的服务提供者
$kernel = $app->make(Illuminate\Contracts\Http\Kernel::class);
// 获取一个 Request ，返回一个 Response。以把该内核想象作一个代表整个应用的大黑盒子，输入 HTTP 请求，返回 HTTP 响应
// 处理请求
$response = $kernel->handle(
    // 创建请求实例
    $request = Illuminate\Http\Request::capture()
);
// 就是把我们服务器的结果返回给浏览器。
$response->send();
// 这个就是执行我们比较耗时的请求，
$kernel->terminate($request, $response);
```

**创建请求实例**

`capture` 方法（捕获用户请求）会调用 SymfonyRequest 的一些底层方法进行初始化工作，并将用户请求数据赋值给 Request 请求对象实例。

```php
// 创建请求实例
$request = Illuminate\Http\Request::capture()
```

客户端的请求实例对象：`Illuminate\Http\Request`

```php
use Closure;	// 匿名函数
use ArrayAccess; //spl  array
use RuntimeException;	// 运行异常类
use Illuminate\Support\Arr;	// 框架基础支持的数组类
use Illuminate\Support\Str; // 框架基础支持的字符串类
use Illuminate\Support\Traits\Macroable; // 框架基础的宏指令类
use Illuminate\Contracts\Support\Arrayable; // 框架基础的数组比较类
use Symfony\Component\HttpFoundation\ParameterBag; // Symfony 基础请求包类
use Symfony\Component\HttpFoundation\Request as SymfonyRequest; // Symfony 基础请求类

class Request extends SymfonyRequest implements Arrayable, ArrayAccess
{
    use Concerns\InteractsWithContentTypes,
        Concerns\InteractsWithFlashData,
        Concerns\InteractsWithInput,
        Macroable;
    
    public static function capture()
    {
        static::enableHttpMethodParameterOverride();

        return static::createFromBase(SymfonyRequest::createFromGlobals());
    }
    
}
```

通过以上代码可知 `Request` 继承 SymfonyRequest：`Symfony\Component\HttpFoundation\Request`

**`SymfonyRequest` 的 `createFromGlobals` 方法：**

```php
// SymfonyRequest：Symfony\Component\HttpFoundation\Request 文件中
// 根据PHP提供的超级全局数组来创建Smyfony Request实例
public static function createFromGlobals()
{
    $request = self::createRequestFromFactory($_GET, $_POST, [], $_COOKIE, $_FILES, $_SERVER);

    if (0 === strpos($request->headers->get('CONTENT_TYPE'), 'application/x-www-form-urlencoded')
        && \in_array(strtoupper($request->server->get('REQUEST_METHOD', 'GET')), ['PUT', 'DELETE', 'PATCH'])
       ) {
        parse_str($request->getContent(), $data);
        $request->request = new ParameterBag($data);
    }
    return $request;
}
```

通过 PHP 提供的全局数组：`$_GET` `$_POST` `$_COOKIE` `$_FILES` `$_SERVER` ，来创建 `SymfonyRequest` 实例，同时在创建 `SymfonyRequest` 实例时，也会根据这些全局数组创建`ParameterBag `：Symfony 基础请求类，如下：

```php
public function initialize(array $query = [], array $request = [], array $attributes = [], array $cookies = [], array $files = [], array $server = [], $content = null)
{
    $this->request = new ParameterBag($request);
    $this->query = new ParameterBag($query);
    $this->attributes = new ParameterBag($attributes);
    $this->cookies = new ParameterBag($cookies);
    $this->files = new FileBag($files);
    $this->server = new ServerBag($server);
    $this->headers = new HeaderBag($this->server->getHeaders());

    $this->content = $content;
    $this->languages = null;
    $this->charsets = null;
    $this->encodings = null;
    $this->acceptableContentTypes = null;
    $this->pathInfo = null;
    $this->requestUri = null;
    $this->baseUrl = null;
    $this->basePath = null;
    $this->method = null;
    $this->format = null;
}
```

继续执行代码，Laravel 框架中的 Request 是在`SymfonyRequest` 的基础上进行封装和实现的，通过 `SymfonyRequest::createFromGlobals()` 拿到`SymfonyRequest` 实例后，在这个实例的基础上再进行创建 `Request` 实例

```php
return static::createFromBase(SymfonyRequest::createFromGlobals());
```

```php
// 在SymfonyRequest实例的基础上创建Request实例 
// duplicate 克隆请求对象并覆盖其中的某些参数。
public static function createFromBase(SymfonyRequest $request)
 {
     if ($request instanceof static) {
         return $request;
     }
	// duplicate 克隆请求对象并覆盖其中的某些参数。
     $newRequest = (new static)->duplicate(
         $request->query->all(), $request->request->all(), $request->attributes->all(),
         $request->cookies->all(), $request->files->all(), $request->server->all()
     );

     $newRequest->headers->replace($request->headers->all());

     $newRequest->content = $request->content;

     $newRequest->request = $newRequest->getInputSource();

     return $newRequest;
 }

public function duplicate(array $query = null, array $request = null, array $attributes = null, array $cookies = null, array $files = null, array $server = null)
{
    return parent::duplicate($query, $request, $attributes, $cookies, $this->filterFiles($files), $server);
}
```

**Symfony 中Request 类的 duplicate 方法**

Symfony 的 HttpFoundation 组件文档：http://www.symfonychina.com/doc/current/components/http_foundation.html

`Symfony\Component\HttpFoundation\Request`

```php
// 克隆请求对象并覆盖其中的某些参数。
public function duplicate(array $query = null, array $request = null, array $attributes = null, array $cookies = null, array $files = null, array $server = null)
{
    $dup = clone $this;
    if (null !== $query) {
        $dup->query = new ParameterBag($query);
    }
    if (null !== $request) {
        $dup->request = new ParameterBag($request);
    }
    if (null !== $attributes) {
        $dup->attributes = new ParameterBag($attributes);
    }
    if (null !== $cookies) {
        $dup->cookies = new ParameterBag($cookies);
    }
    if (null !== $files) {
        $dup->files = new FileBag($files);
    }
    if (null !== $server) {
        $dup->server = new ServerBag($server);
        $dup->headers = new HeaderBag($dup->server->getHeaders());
    }
    $dup->languages = null;
    $dup->charsets = null;
    $dup->encodings = null;
    $dup->acceptableContentTypes = null;
    $dup->pathInfo = null;
    $dup->requestUri = null;
    $dup->baseUrl = null;
    $dup->basePath = null;
    $dup->method = null;
    $dup->format = null;

    if (!$dup->get('_format') && $this->get('_format')) {
        $dup->attributes->set('_format', $this->get('_format'));
    }

    if (!$dup->getRequestFormat(null)) {
        $dup->setRequestFormat($this->getRequestFormat(null));
    }

    return $dup;
}
```

Request 对象创建完成后，Laravel 框架便会将这个请求绑定到服务容器中，以便我们在运用过程中通过解析该实例获取用户的请求。

```php
// 处理传入的HTTP请求。
$response = $kernel->handle(
    // 创建请求实例
    $request = Illuminate\Http\Request::capture()
);
```

**handle 方法处理 HTTP 请求**

将 HTTP 请求抽象成 `Laravel Request请求实例`后，请求实例会被传导进入到 HTTP 内核的 `handle` 方法内部，请求的处理就是由 `handle` 方法来完成的。

```php
public function handle($request)
{
    try {
        $request->enableHttpMethodParameterOverride();

        $response = $this->sendRequestThroughRouter($request);
    } catch (Exception $e) {
        $this->reportException($e);

        $response = $this->renderException($request, $e);
    } catch (Throwable $e) {
        $this->reportException($e = new FatalThrowableError($e));

        $response = $this->renderException($request, $e);
    }

    $this->app['events']->dispatch(
        new Events\RequestHandled($request, $response)
    );

    return $response;
}
```

handle 方法接收了一个 request 请求对象 并返回 response 响应对象，有关于`sendRequestThroughRouter`的介绍在[Laravel的生命周期]('https://fahsa.cn/php/laravel-life-cycle/#more') 中介绍过。

在 `sendRequestThroughRouter` 方法中，它会加载内核中定义的引导程序，来引导启动应用，并通过 `Pipeline` 管道对象传输 HTTP 请求对象，流经 Laravel 框架中定义的 HTTP 中间件们 和 路由 中间件们，通过它们来完成过滤请求的目的，最终将 HTTP 请求传递给应用处理程序（控制器方法或者路由中的闭包）并返回相应的响应，

**`sendRequestThroughRouter` 方法：**

```php
protected function sendRequestThroughRouter($request)
{
    $this->app->instance('request', $request);

    Facade::clearResolvedInstance('request');

    $this->bootstrap();

    return (new Pipeline($this->app))
        ->send($request)
        ->through($this->app->shouldSkipMiddleware() ? [] : $this->middleware)
        ->then($this->dispatchToRouter());
}
```

Request 请求对象到达应用处理程序后，它的使命也就完成了，最后就是将我们服务器的结果返回给浏览器。

以上就是关于 Laravel 框架中的 Request 请求类分析。