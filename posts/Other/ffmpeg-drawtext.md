---
title: "ffmpeg的drawtext"
description: "ffmpeg的drawtext"
date: 2021-12-15 18:10:00
slug: ffmpeg-drawtext
image:
categories:
    - Other
tags: ["FFmpeg"]
---

官方文档：https://ffmpeg.org/ffmpeg-filters.html#drawtext-1

```sh
# 视频 水印居中
ffmpeg -i input.mp4 -i logo.png -filter_complex "overlay=(main_w-overlay_w)/2:(main_h-overlay_h)/2" -codec:a copy output.mp4

# 图片居中文本
ffmpeg -i r2.jpg -vf "drawtext=font='Impact':text='Text':fontcolor=white:borderw=3:fontsize-75:x=(w-tw)/2:y=((h-text_h)/2)" output1.jpg

```



```sh

ffmpeg -i r2.jpg -vf "drawtext=font='Impact':text='WithYou Music':fontcolor=yellow@0.2:box=1:boxcolor=red@0.2:borderw=3:fontsize-75:x=(w-tw)/2:y=((h-text_h)/2)" output2.jpg

ffmpeg -i r2.jpg -vf "drawtext=font='Impact':text='WithYou Music':fontcolor=yellow@0.8:box=1:boxcolor=red@0.2:borderw=3:fontsize-75:x=(w-tw)/2:y=((h-text_h)/2)" output2.jpg
```

![image-20211215103157656](https://qnres.fahsa.cn/images/image-20211215103157656.png)





```sh
ffmpeg -i input -filter_complex "subtitles=your_subtitles_file.srt:force_style='Outline=5,OutlineColour=&H000000&'" output

```



### 抽取视频中的一张图片：

```sh
ffmpeg -i r1.mp4 -f image2  -vf fps=fps=1/60 -qscale:v 2 r1.jpeg

ffmpeg -ss 0.5 -i r1.mp4 -vframes 1 -s 1080x720 -f image2 r2.jpg

```





