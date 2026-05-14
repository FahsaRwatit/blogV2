---
title: virtualbox装ubuntu-22.04.1
description: virtualbox装ubuntu-22.04.1
date: 2022-04-10 16:10:00
slug: virtualbox-ubuntu
image:
categories:
    - Linux
tags: ["ubuntu"]

---







 CPU、内存、硬盘、网络都配置好之后，再加载上 Ubuntu 22.04 的光盘镜像

在安装的过程中，为了节约时间，建议选择“最小安装”，同时物理断网，避免下载升级包。

首先我们需要用 Ctrl + Alt + T 打开命令行窗口，然后用 apt 从 Ubuntu 的官方软件仓库安装 git、vim、curl 等常用工具：

```shell

sudo apt update
sudo apt install -y git vim curl jq
```

Ubuntu 桌面版默认是不支持远程登录的，所以为了让后续的实验更加便利，我们还需要安装“openssh-server”，再使用命令 ip addr ，查看虚拟机的 IP 地址，然后就可以在宿主机上使用 ssh 命令登录虚拟机：

```shell

sudo apt install -y openssh-server
ip addr
```

配置共享文件方式：https://blog.51cto.com/u_15309736/5424668



配置网络：

```shell
# 网卡1  仅主机host only网络

# 网卡2 网络地址转换NAT

# VirtualBox > 管理 > 主机网络管理器 > 
DHCP服务器启用，手动配置网卡，DHCP服务器(D),启用服务器(E)
```

















安装docker

https://www.51cto.com/article/715086.html

https://developer.aliyun.com/article/855513?userCode=okjhlpr5

```shell

Command 'docker' not found, but can be installed with:
sudo apt install docker.io


sudo apt install -y docker.io #安装Docker Engine


sudo service docker start         #启动docker服务
sudo usermod -aG docker ${USER}   #当前用户加入docker组

# 上面的三条命令执行完之后，我们还需要退出系统（命令 exit ），再重新登录一次，这样才能让修改用户组的命令 usermod 生效。

```

安装docker-compose：

https://ost.51cto.com/posts/13053

```shell
sudo curl -L https://get.daocloud.io/docker/compose/releases/download/v2.11.2/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

# 高速下载
sudo curl -L "https://get.daocloud.io/docker/compose/releases/download/v2.11.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose

sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```





问题：

```shell
failed to solve: rpc error: code = Unknown desc = failed to solve with frontend dockerfile.v0: failed to create LLB definition:
 failed to copy: httpReadSeeker: failed open: failed to do request: Get "https://production.cloudflare.docker.com/registry-v2/docker/registry/v2/blobs/sha256/7c/7c33e9293aaa033edbe2e15952fcf3df27a2dccc27875b655f67b20e324add9f/data?verify=1672966838-0FB79FGpNxYva9ykx0r%2F9GSWIoU%3D": dial tcp 104.18.123.25:443: i/o timeou


#获取链接
https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors
# 配置阿里镜像源
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://suqqyvt2.mirror.aliyuncs.com", "http://f1361db2.m.daocloud.io"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker


{"registry-mirrors": ["http://f1361db2.m.daocloud.io"]}


{
  "registry-mirrors": [
    "https://registry.docker-cn.com",
    "http://hub-mirror.c.163.com",
    "https://docker.mirrors.ustc.edu.cn"
  ]
}

```













