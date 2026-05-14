---
title: "使用ffmpeg的循环"
description: "使用ffmpeg的循环"
date: 2021-12-08 18:10:00
slug: ffmpeg-loop
image:
categories:
    - Python
tags: ["MoviePy","FFmpeg"]

---

将新音频与视频重复视频合并，直到音频结束：

```sh
ffmpeg  -stream_loop -1 -i input.mp4 -i input.mp3 -shortest -map 0:v:0 -map 1:a:0 -y out.mp4

ffmpeg -y -stream_loop -1 -i video -i audio.mp3 -fflags +shortest -max_interleave_delta 50000 -c copy output.mp4

ffmpeg -y -stream_loop -1 -i video -i audio.mp3 -fflags +shortest -max_interleave_delta 50000 -c copy output.mp4

ffmpeg  -stream_loop -1 -i 1.mp4 -c copy -v 0 -f nut - | ffmpeg -thread_queue_size 10K -i - -i 1.mp3 -c copy -map 0:v -map 1:a -shortest -y out.mp4
```

使用 -stream_loop -1 表示无限循环 input.mp4，-shortest 表示在最短的输入流结束时完成编码。这里最短的输入流将是 input.mp3。

<!--more-->

如果您想将新音频与视频重复音频合并，直到视频结束，您可以尝试以下操作：

```sh
ffmpeg  -i input.mp4 -stream_loop -1 -i input.mp3 -shortest -map 0:v:0 -map 1:a:0 -y out.mp4
```

尝试`ffmpeg -i yourmovie.mp4 -loop 10`将输入循环 10 次



```
ffmpeg -ss 00:01:00 -i input.mp4 -to 00:02:00 -c copy output.mp4
```

**此命令可在几秒钟内修剪您的视频！**

命令的解释：

> **-i：**指定输入文件。在那种情况下，它是 (input.mp4)。
> **-ss：**与 -i 一起使用，这会在输入文件 (input.mp4) 中寻找位置。
> **00:01:00：**这是您修剪后的视频开始的时间。
> **-to：**指定从开始 (00:01:40) 到结束 (00:02:12) 的持续时间。
> **00:02:00：**这是您修剪后的视频结束的时间。
> **-c copy：**这是通过流复制进行修剪的选项。（注意：非常快）

计时格式为：*hh:mm:ss*

