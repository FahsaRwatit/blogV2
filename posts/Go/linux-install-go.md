---
title: "Linux上搭建Go环境"
description: "Linux上搭建Go环境"
date: 2021-10-08 18:10:00
slug: linux-install-go
image:
categories:
    - Go
tags: ["Go"]
---

下载安装包，选择 Linux 版本：https://golang.org/dl/

```shell
# wget 下载
wget https://golang.org/dl/go1.17.3.linux-amd64.tar.gz
# 解压到指定目录 /usr/local/
tar -zxvf go1.17.3.linux-amd64.tar.gz -C /usr/local/
# 进入/usr/local/，创建 Go 项目存放路径 gocode
cd /usr/local/
mkdir gocode
# 配置环境变量
vim /etc/profile
# 在文件末尾添加以下内容
export GOROOT=/usr/local/go
export GOPATH=/usr/local/gocode
export PATH=$GOPATH/bin:$GOROOT/bin:$PATH
# 使环境变量 生效
source /etc/profile
# 最后查看 Go 版本 和 配置信息
go version
go env

# 如下
GOROOT="/usr/local/go"
GOPATH="/usr/local/gocode"

```



## yum 安装

```shell
yum install epel-release
yum install golang
go version
```
