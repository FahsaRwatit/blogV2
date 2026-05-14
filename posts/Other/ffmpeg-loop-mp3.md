---
title: "ffmpeg的循环音频"
description: "ffmpeg的循环音频"
date: 2021-12-15 18:10:00
slug: ffmpeg-loop-mp3
image:
categories:
    - Other
tags: ["FFmpeg"]

---

官方文档：https://ffmpeg.org/ffmpeg-filters.htm

使用 ffmpeg 和 concat[enate] 过滤器设法做到了。以下是如何循环三遍的示例：

```sh
ffmpeg -i audio.wav -filter_complex "[0:a]afifo[a0];[0:a]afifo[a1];[0:a]afifo[a2];[a0][a1][a2]concat=n=3:v=0:a=1[a]" -map "[a]" out.wav
```

**已编辑 (23/12/2020)**

除了上述之外，还有另外一种超级简单的2种方法：

**第一种方法：定义音频输出 SIZE**。

含义：它会重复/连接“MyAudio.mp3”，直到达到 10M 的大小并停止（您需要自己计算最终大小）

```sh
ffmpeg -stream_loop -1 -i "MyAudio.mp3" -fs 10M -c copy "MyRepeatingAudio.mp4"
```

**第二种方法：没有定义输出大小（一旦达到所需大小，您将需要停止进程，CTRL+C**

```sh
ffmpeg -stream_loop -1 -i "MyAudio.mp3" -c copy "MyRepeatingAudio.mp4"
```

**请注意，上述方法也适用于重复视频**

**第三方编辑 (15/09/2021)**

此命令只需一步即可将重复音频添加到视频中：

```sh
ffmpeg -i input.mp4 -stream_loop -1 -i audio.mp4 -shortest \
    -map 0:v:0 -map 1:a:0 -c:v copy output.mp4
```



这是一个直接的cmd： `ffmpeg -lavfi "amovie=audio.wav:loop=3" out.wav`









