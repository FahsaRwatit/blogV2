---
title: Laravel + Redis 实现消息队列
date: 2021-03-17 18:00:00
description: Laravel + Redis 实现消息队列
slug: laravel-redis-queue1
image:
categories:
    - PHP
tags: ["PHP","Laravel"]
---

完整的消息队列由`消息`、`队列`、`处理程序`组成。

基本的流程就是由生产者（业务代码）将数据推送到队列中（此处使用的是Redis），然后由消费者（处理程序）从队列中取出数据进行加工处理。

消息队列主要解决异步处理、应用间耦合，流量削锋等问题，实现高性能，高可用，可伸缩和最终一致性架构。例如处理需要异步处理的比较耗时操作（邮件发送、文件上传下载），或者高并发业务（秒杀、消息推送）。

下面列举了一个例子，可以让你更好的理解消息队列是怎么样实现的？

本例是实现添加视频播放数的消息队列。

<!--more-->

ps：喜欢的朋友可以关注公众号：苏小怪的梦呓

### 队列

这种数据结构有先进先出（FIFO）的特点

首先在控制器中的方法内，根据自身业务情况将数据添加到队列中。

```php
class VideoController extends Controller
{
    public function addNum(Request $request){
        $vid = $request->input('vid');
        if(empty($vid)){
            return json_encode(['s'=>0,'d'=>'','m'=>'参数异常']);
        }
        // 将视频ID 添加到 video-playnum 队列中
        Redis::rpush('video-playnum',$vid);
        return json_encode(['s'=>1,'d'=>'','m'=>'成功']);
    }
}
```

### 消息

通常是指推送到队列中的数据，通常是一个字符串，如果是非字符串，可以通过序列化的形式转为字符串。

本例中的消息数据指的是视频ID

### 处理程序

从队列中取出消息数据进行处理，通常是一个或者多个常驻内存的进程。

为了简化流程直接使用命令`php artisan make:command PlayNumQueueWorker`生成一个 command 文件，并在 `handle` 方法内编写剩余业务代码。

记得修改 `$signature`属性的值为 `command:PlayNumQueueWorker` ，`name` 为你需要运行的命令名称。

```php
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use Illuminate\Support\Facades\Redis;
use App\Models\Video;
class PlayNumQueueWorker extends Command
{
    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'command:PlayNumQueueWorker';

    /**
     * The console command description.
     *
     * @var string
     */
    protected $description = '添加视频播放数';

    /**
     * Create a new command instance.
     *
     * @return void
     */
    public function __construct()
    {
        parent::__construct();
    }

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $this->info('监听消息队列：video-playnum:start');
        while(true){
            $vid = Redis::lpop('video-playnum');
            if(!empty($vid)){
                $video = new Video();
                $res = $video->where('v_id',$vid)->increment('v_playnum',1);
                $this->info($res);
                $this->info('监听消息队列：video-playnum：end');
            }
        }
    }
}
```

进入控制台运行命令：`php artisan make:command PlayNumQueueWorker`，可以看输出如下：

![image-20210317164738377](https://qnres.fahsa.cn/images/image-20210317164738377.png)

接着访问控制器，输出：

![image-20210317164859077](https://qnres.fahsa.cn/images/image-20210317164859077.png)

通过以上例子相信你对消息队列的基本实现原理有了大概的理解，但是 Laravel 中提供了更加优雅的队列系统，不需要我们手动去实现队列、消息和处理程序的实现代码，并且支持不同的队列驱动程序（database、beanstalkd、sqs、redis）。

接下来的内容是关于如何基于 Laravel 队列系统实现添加视频播放数量的消息队列。

### Laravel 的队列系统

队列配置文件存储在 ` config/queue.php`，在`.env`文件中，配置queue的连接为 Redis

```php
QUEUE_CONNECTION=redis
```

### 任务类

接下来使用命令 `php artisan make:job ProcessPlaynum` 生成任务类 ，任务类会放在`app/Jobs`目录下。

```php
<?php

namespace App\Jobs;

use Illuminate\Bus\Queueable;
use Illuminate\Queue\SerializesModels;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use App\Models\Video;

class ProcessPlaynum implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    protected $vid;

    /**
     * Create a new job instance.
     *
     * @return void
     */
    public function __construct($vid)
    {
        $this->vid = $vid;
    }
    /**
     * Execute the job.
     *
     * @return void
     */
    public function handle()
    {
        var_dump($this->vid);
        $video = new Video();
        $res = $video->where('v_id',$this->vid)->increment('v_playnum',1);
        var_dump($res);
        var_dump('ok...');
    }

    /**
     * @param null $exception
     * 执行失败的任务
     */
    public function failed($exception = null)
    {
    }
}

```

在`handle`方法内执行播放数加一操作。

### 分发任务

分发任务调用任务类的`dispatch`方法，`dispatch`方法中的参数就是你想要传递的数据，将会被传递给任务类的`__construct`方法。

```php
ProcessPlaynum::dispatch($vid)
    
public function __construct($vid)
{
    $this->vid = $vid;
}
```

同时你还可以使用 `onConnection`方法指定对应的连接驱动，使用 `onQueue`指定对应的队列，除此之外，有关于更多使用可以查看Laravel官方文档。

接下来在控制器中编写相关的业务代码：

```php
public function increasedNum(Request $request){
    $vid = $request->input('vid');
    if(empty($vid)){
        return json_encode(['s'=>0,'d'=>'','m'=>'参数异常']);
    }

    ProcessPlaynum::dispatch($vid)
        ->onConnection('redis') // 指定连接
        ->onQueue('addPlayNum'); // 指定队列
    return json_encode(['s'=>1,'d'=>'','m'=>'成功']);
}
```

### 监听

开启监听队列 `php artisan queue:work redis --queue=addPlayNum --tries=3`

 `tries`代表失败后最大尝试次数。

![image-20210317173115945](https://qnres.fahsa.cn/images/image-20210317173115945.png)

以上就是关于 Laravel 框架中如何使用消息队列的简单介绍，你学会了吗？