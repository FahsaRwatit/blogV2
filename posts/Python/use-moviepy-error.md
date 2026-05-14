---
title: "使用moviepy时遇到的错误"
description: "使用moviepy时遇到的错误"
date: 2021-10-08 18:10:00
slug: use-moviepy-error
image:
categories:
    - Python
tags: ["MoviePy","FFmpeg"]

---





### OSError: convert: error while loading shared libraries: libMagickCore-7.Q16HDRI.so.10: cannot open shared object file: No such file or directory

```shell
# 执行
ldconfig /usr/local/lib
```







