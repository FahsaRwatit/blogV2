---
title: "修改Linux中vim的配色"
description: "修改Linux中vim的配色"
date: 2021-06-13 15:10:00
slug: set-my-color
image:
categories:
    - Linux
tags: ["Linux"]

---

- 确认是否存在文件夹 `/root/.vim/colors`，没有则创建一个`mkdir ~/.vim/colors`。

- 下载配色方案，例如：molokai.vim，放到以上创建的目录/root/.vim/colors下

- 找到`vimrc`文件，可以使用`find / -name vimrc`命令查找，也可以使用vim命令打开一个文件，然后命令行状态下输入`:version`，可以看到如下：

  ![image-20210726161418190](https://qnres.fahsa.cn/images/image-20210726161418190.png)

  vimrc 文件一般位于 `/etc/vimrc` 和 `~/.vim/vimrc`

- 修改 vimrc 文件，添加配置项：`colorscheme molokai`，molokai 为配色方案的名称，对应于 /root/.vim/colors 目录下的文件名。

## 配色方案

### molokai

https://github.com/tomasr/molokai

其他配色方案可参考：https://blog.csdn.net/u011596455/article/details/69381703
