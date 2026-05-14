---
title: Laravel的HTTP响应Response
date: 2021-03-16 18:00:00
description: Laravel的HTTP响应Response
slug: laravel-response
image:
categories:
    - PHP
tags: ["PHP","Laravel"]

---

前几天分析了 Laravel 框架的相关内容：

- [Laravel的Request请求类分析]('https://fahsa.cn/php/laravel-request/#more')
- [Laravel基于Composer实现自动加载原理分析]('https://fahsa.cn/php/laravel-composer/')
- [Laravel的中间件原理]('https://fahsa.cn/php/laravel-middleware/#more')
- [Laravel的生命周期]('https://fahsa.cn/php/laravel-life-cycle/#more')

今天我们来看看 Laravel中是怎么处理 HTTP 响应的，也就是关于 Response 的代码分析。

## 找到返回 Response 的代码块

### 入口文件 `public/index.php` 

首先进入 Laravel 框架的入口文件中可以看到 `handle`函数中 返回了 `$response`实例。

<!--more-->

`public/index.php`文件中：

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
```

### Kernel 类

进入`Illuminate\Foundation\Http\Kernel`类中，改方法在 [Laravel中的生命周期]('https://fahsa.cn/php/laravel-life-cycle/#more') 中分析过，在这就不进行详细讲解，这里我们找到返回 `$response` 的 `sendRequestThroughRouter` 。

### `handle` 函数：

```php
// Illuminate\Foundation\Http\Kernel.php文件中
public function handle($request)
{
    try {
        $request->enableHttpMethodParameterOverride();

        $response = $this->sendRequestThroughRouter($request);
    } catch (Exception $e) {
        // 根据应用配置向不同的渠道报告异常；
        $this->reportException($e);

        // 将异常转化为可渲染的格式并以响应的方式返回给终端用户。
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

### `sendRequestThroughRouter` 函数：

通过中间件/路由器发送给定的请求。

`$response = $this->sendRequestThroughRouter($request)`

```php
// Illuminate\Foundation\Http\Kernel.php文件中
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

该方法的程序执行如下：

1、将 request 请求实例注册到 app 容器当中

2、清除之前的 request 实例缓存

3、启动引导程序：加载内核中定义的引导程序来引导启动应用

4、将请求发送到路由：通过 Pipeline 对象传输 HTTP请求对象流经框架中定义的HTTP中间件和路由器来完成过滤请求，最终将请求传递给处理程序（控制器方法或者路由中的闭包）由处理程序返回相应的响应。

此处的 `dispatchToRouter`就是我们今天要讲解的返回 Response 响应的代码块的初始位置。

## Laravel 之 HTTP 响应 分析（Response）

### `dispatchToRouter`函数：

```php
// Illuminate\Foundation\Http\Kernel.php文件中
// 获取路由调度器回调。
protected function dispatchToRouter()
{
    return function ($request) {
        $this->app->instance('request', $request);

        return $this->router->dispatch($request);
    };
}
```

### `dispatch`函数：

`Illuminate\Routing\Router.php`文件中 `dispatch`函数：

```php
// 将请求发送到应用程序。
public function dispatch(Request $request)
{
    $this->currentRequest = $request;

    return $this->dispatchToRoute($request);
}
```

### `dispatchToRoute` 函数：

```php
// 将请求分派到路由并返回响应
public function dispatchToRoute(Request $request)
{
    return $this->runRoute($request, $this->findRoute($request));
}
```

### `runRoute` 函数中创建 Response 对象：

此处将创建 Response 对象。通过给定的路由和请求返回响应对象并进行预处理。

```php
// 返回给定路由的响应。
protected function runRoute(Request $request, Route $route)
{
    // 设置路由解析程序回调
    $request->setRouteResolver(function () use ($route) {
        return $route;
    });
    $this->events->dispatch(new Events\RouteMatched($route, $request));
    return $this->prepareResponse($request,
                                  $this->runRouteWithinStack($route, $request)
                                 );
}
```

### `runRouteWithinStack` 函数：

从堆栈中运行给定的路由，该方法是最终执行路由处理程序 (控制器方法或者闭包处理程序) 的地方，通过以下代码发现在该方法中通过 `prepareResponse`方法返回了 Reqponse 实例，然后当程序返回到`runRoute`中又执行了一遍 `prepareResponse`，最后得到要返回给客户端的 Response 对象，那么`prepareResponse`中到底做了些什么事情呢？

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

### `prepareResponse` 函数：

```php
// 从给定值创建响应实例。
public function prepareResponse($request, $response)
{
    return static::toResponse($request, $response);
}
```

### `toResponse`函数：

```php
public static function toResponse($request, $response)
{
    if ($response instanceof Responsable) {
        $response = $response->toResponse($request);
    }

    if ($response instanceof PsrResponseInterface) {
        $response = (new HttpFoundationFactory)->createResponse($response);
    } elseif ($response instanceof Model && $response->wasRecentlyCreated) {
        $response = new JsonResponse($response, 201);
    } elseif (! $response instanceof SymfonyResponse &&
              ($response instanceof Arrayable ||
               $response instanceof Jsonable ||
               $response instanceof ArrayObject ||
               $response instanceof JsonSerializable ||
               is_array($response))) {
        $response = new JsonResponse($response);
    } elseif (! $response instanceof SymfonyResponse) {
        $response = new Response($response);
    }

    if ($response->getStatusCode() === Response::HTTP_NOT_MODIFIED) {
        $response->setNotModified();
    }

    return $response->prepare($request);
}
```

上面返回了三种类型的 Response  对象：

- PsrResponseInterface (Psr\Http\Message\ResponseInterface 的别名)

  Psr 规范中对服务端响应的定义

- Illuminate\Http\JsonResponse  

  Laravel 中对服务端 JSON 响应的定义

- Illuminate\Http\Response

  Laravel 中对普通的非 JSON 响应的定义

通过以上可知，`$response`实例都是`Symfony\Component\HttpFoundation\Response`类或者其子类的对象，它和 Request 一样底层都是使用 `Symfony` 框架 中的 `HttpFoundation` 组件来实现的，接下来分析 Symfony 中的 Response 类。

### Symfony 中的 Response 类

`Symfony\Component\HttpFoundation\Response`类中：

```php
class Response
{
    public function __construct($content = '', int $status = 200, array $headers = [])
    {
        $this->headers = new ResponseHeaderBag($headers);
        $this->setContent($content);
        $this->setStatusCode($status);
        $this->setProtocolVersion('1.0');
    }
}
```

```php
public function setContent($content)
{
    if (null !== $content && !\is_string($content) && !is_numeric($content) && !\is_callable([$content, '__toString'])) {
        throw new \UnexpectedValueException(sprintf('The Response content must be a string or object implementing __toString(), "%s" given.', \gettype($content)));
    }

    $this->content = (string) $content;

    return $this;
}
```

在 Symfony 中的 Response 类的构造函数中将路由处理程序的返回值设置到 content 属性中，该值为返回到客户端的响应内容。

生成 Response 对象后我们再回到` Illuminate\Routing\Router`类的 `toResponse`函数中，接下来执行`prepare` 函数。

### `prepare` 函数：

其主要目的是对 Response 进行微调使其能够遵从 HTTP/1.1 协议（RFC 2616）。

函数中根据各种情况设置了响应头中的`Content-Type`、`Content-Length`等首部字段。

```php
// Symfony\Component\HttpFoundation\Response 类中
public function prepare(Request $request)
{
    $headers = $this->headers;

    if ($this->isInformational() || $this->isEmpty()) {
        $this->setContent(null);
        $headers->remove('Content-Type');
        $headers->remove('Content-Length');
        // prevent PHP from sending the Content-Type header based on default_mimetype
        ini_set('default_mimetype', '');
    } else {
        // Content-type based on the Request
        if (!$headers->has('Content-Type')) {
            $format = $request->getRequestFormat(null);
            if (null !== $format && $mimeType = $request->getMimeType($format)) {
                $headers->set('Content-Type', $mimeType);
            }
        }

        // Fix Content-Type
        $charset = $this->charset ?: 'UTF-8';
        if (!$headers->has('Content-Type')) {
            $headers->set('Content-Type', 'text/html; charset='.$charset);
        } elseif (0 === stripos($headers->get('Content-Type'), 'text/') && false === stripos($headers->get('Content-Type'), 'charset')) {
            // add the charset
            $headers->set('Content-Type', $headers->get('Content-Type').'; charset='.$charset);
        }

        // Fix Content-Length
        if ($headers->has('Transfer-Encoding')) {
            $headers->remove('Content-Length');
        }

        if ($request->isMethod('HEAD')) {
            // cf. RFC2616 14.13
            $length = $headers->get('Content-Length');
            $this->setContent(null);
            if ($length) {
                $headers->set('Content-Length', $length);
            }
        }
    }

    // Fix protocol
    if ('HTTP/1.0' != $request->server->get('SERVER_PROTOCOL')) {
        $this->setProtocolVersion('1.1');
    }

    // Check if we need to send extra expire info headers
    if ('1.0' == $this->getProtocolVersion() && false !== strpos($headers->get('Cache-Control'), 'no-cache')) {
        $headers->set('pragma', 'no-cache');
        $headers->set('expires', -1);
    }

    $this->ensureIEOverSSLCompatibility($request);

    if ($request->isSecure()) {
        foreach ($headers->getCookies() as $cookie) {
            $cookie->setSecureDefault(true);
        }
    }

    return $this;
}
```

程序执行到这时已经完成了 Response的设置，之后它会流经路由和框架中的中间件的后置操作，这些后置操作都是对 Response 进一步加工，最后将 Response 返回给客户端。

回到 `public/index.php`中

```php
$response->send();
```

### `send`函数：

```php
// Symfony\Component\HttpFoundation\Response 类中
public function send()
{
    // 发送headers到客户端
    $this->sendHeaders();
    // 发送响应内容到客户端
    $this->sendContent();

    if (\function_exists('fastcgi_finish_request')) {
        fastcgi_finish_request();
    } elseif (!\in_array(\PHP_SAPI, ['cli', 'phpdbg'], true)) {
        static::closeOutputBuffers(0, true);
    }
    return $this;
}
```

### `sendHeaders`函数：

```php
public function sendHeaders()
{
    // headers have already been sent by the developer
    if (headers_sent()) {
        return $this;
    }

    // headers
    foreach ($this->headers->allPreserveCaseWithoutCookies() as $name => $values) {
        $replace = 0 === strcasecmp($name, 'Content-Type');
        foreach ($values as $value) {
            header($name.': '.$value, $replace, $this->statusCode);
        }
    }

    // cookies
    foreach ($this->headers->getCookies() as $cookie) {
        header('Set-Cookie: '.$cookie, false, $this->statusCode);
    }

    // status
    header(sprintf('HTTP/%s %s %s', $this->version, $this->statusCode, $this->statusText), true, $this->statusCode);

    return $this;
}
```

### `sendContent`函数：

```php
public function sendContent()
{
    echo $this->content;

    return $this;
}
```

`send`中将设置好的那些 headers 设置到 HTTP 的头部字段中，然后将 Content 输出到 HTTP 的实体中，最后通过 PHP 将完整的 HTTP 响应发送给客户端。

源码分析就是代码比较多，看着枯燥无味，不过结合IDE和源码分析下来，相信你也会有所收获。

道阻且长，行则将至，做则必成。