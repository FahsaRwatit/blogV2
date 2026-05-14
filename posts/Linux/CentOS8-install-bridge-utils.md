---
title: CentOS8上安装CentOS8-install-bridge-utils
description: CentOS8上安装CentOS8-install-bridge-utils
date: 2022-04-10 16:10:00
slug: CentOS8-install-bridge-utils
image:
categories:
    - Linux
tags: ["Linux"]


---

bridge-utils安装包下载
https://mirrors.edge.kernel.org/pub/linux/utils/net/bridge-utils/bridge-utils-1.6.tar.xz

CentOS8上安装

```sh
rpm -ivh http://mirror.centos.org/centos/7/os/x86_64/Packages/bridge-utils-1.5-9.el7.x86_64.rpm
```

CentOS7安装

```sh
sudo yum install bridge-utils
```



#### References

[1] https://www.tecmint.com/create-network-bridge-in-rhel-centos-8/