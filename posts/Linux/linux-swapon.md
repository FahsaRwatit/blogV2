---
title: 在 Ubuntu、Linux Mint 和衍生产品中创建 SWAP 分区
description: 在 Ubuntu、Linux Mint 和衍生产品中创建 SWAP 分区
date: 2021-06-13 15:10:00
slug: linux-swapon
image:
categories:
    - Linux
tags: ["Linux"]

---

## 在 Ubuntu、Linux Mint 和衍生产品中创建 SWAP 分区

由于服务器内存大小的限制，导致出现错误：

1、OSError: [Errno 12] Cannot allocate memory

2、dd: failed to open ‘/swapfile’: Text file busy

<!--more-->

转载于：https://askubuntu.com/questions/920595/fallocate-fallocate-failed-text-file-busy-in-ubuntu-17-04

### 方法 1：来自终端的命令行方式（最快的方式！）

**第 1**步**：**第一步是检查您的 PC 中是否已经创建了任何 SWAP 分区：

```
sudo swapon --show
```

输入您的根密码。如果没有看到输出，则表示 SWAP 不存在。

**STEP 2：**接下来，让我们看看你电脑硬盘的当前分区结构：

```
df -h
```

**第 3**[步](https://askubuntu.com/users/4272/heynnema)**：**正如[heynnema](https://askubuntu.com/users/4272/heynnema)评论的那样，在开始更改之前，请禁用交换：

```
sudo swapoff -a
```

**第 4 步：**现在是创建 SWAP 文件的时候了。确保硬盘上有足够的空间。您需要多少 SWAP 大小是一个偏好问题。

我的建议是：如果您有最多 4GB 的 RAM，我建议为 SWAP 放置两倍的 RAM（用于 SWAP 为 8GB）。对于超过 4GB 的 PC，我建议为 SWAP 使用相同数量的 RAM 加上 2GB。示例：就我而言，它是 8GB，我放了 8GB + 2GB，总共 10GB 用于 SWAP。但您可以随意做出选择。

```
sudo dd if=/dev/zero of=/swapfile bs=1G count=10 status=progress
```

**第 5**步：现在创建 SWAP 文件。让我们为其授予仅 root 权限。

```
sudo chmod 600 /swapfile
```

**第 6**步：将文件标记为 SWAP 空间：

```
sudo mkswap /swapfile
```

**步骤 7：**最后启用 SWAP。

```
sudo swapon /swapfile
```

**步骤 8：**您现在可以使用相同的 swapon 命令检查是否创建了 SWAP。

```
sudo swapon --show
```

**STEP 9** : 还要再次检查最终的分区结构。

```
free -h
```

**STEP 10** : 一切设置好后，你必须将 SWAP 文件设置为永久，否则重启后你将丢失 SWAP。运行此命令：

```
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

完成，现在退出终端！

您可以在**系统监视器**实用程序上检查 SWAP 状态。

------

### 方法二：GUI方式使用GParted

如果您想直接通过图形界面，请输入下面有很好解释的参考链接。

### 交换空间

[交换空间](https://www.digitalocean.com/community/tutorials/how-to-add-swap-space-on-ubuntu-16-04)是系统硬盘驱动器中的内存空间，已被指定为[操作系统](https://stackoverflow.com/questions/tagged/os)临时存储它不能再保存在 RAM 中的数据。这使您能够增加程序可以保持其工作的数据量[记忆](https://stackoverflow.com/questions/tagged/memory). 硬盘驱动器上的交换空间将主要在 RAM 中不再有足够空间来保存正在使用的应用程序数据时使用。然而，写入 I/O 的信息将比保存在 RAM 中的信息慢得多，但操作系统更愿意继续在内存中运行应用程序数据，并为旧数据使用交换空间。部署交换空间作为系统 RAM 耗尽时的后备方案，是一种安全措施，可防止在具有非 SSD 存储可用的系统上出现[内存](https://stackoverflow.com/questions/59715649/how-to-set-memory-limit-for-oom-killer-for-chrome/59750032#59750032)不足问题。