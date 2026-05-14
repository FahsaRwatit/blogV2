---
title: "Linux安装docker和docker-compose"
description:  "Linux安装docker和docker-compose"
date: 2022-06-08 18:10:00
slug: linux-install-docker-and-docker-compose
image:
categories:
    - Go
tags: ["docker"]

---

### Linux安装docker

```shell
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun

docker --version
# 设置开机启动
systemctl enable docker
# 查看是否启动
docker ps -a
ps -ef | grep docker

# 启动docker
systemctl start docker

```

### 配置镜像加速器

```shell
针对Docker客户端版本大于 1.10.0 的用户

您可以通过修改daemon配置文件/etc/docker/daemon.json来使用加速器

sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://suqqyvt2.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

### Linux安装docker-compose

```shell

curl -L "https://github.com/docker/compose/releases/download/1.25.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose


chmod +x /usr/local/bin/docker-compose

docker-compose --version

sudo systemctl restart docker
```