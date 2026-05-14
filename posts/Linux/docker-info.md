---
title: docker的一些信息
description: docker的一些信息
date: 2022-04-10 16:10:00
slug: docker-info
image:
categories:
    - Linux
tags: ["docker"]


---



```sh
sudo apt install -y docker.io #安装Docker Engine


sudo service docker start         #启动docker服务
sudo usermod -aG docker ${USER}   #当前用户加入docker组




sudo vim /etc/docker/daemon.json


```

dockerfile常用参数：
1.ARG：镜像层的环境变量
2.FROM：拉取基础镜像
3.COPY:拷贝文件
4.ADD：拷贝文件、URL、压缩文件等
5.EVN：镜像层和容器层参数
6.EXPOSE:暴露容器内部端口给外部使用
7.RUN：执行shell指令
8.CMD：构建完成时执行的指令

它们区别在于 ARG 创建的变量只在镜像构建过程中可见，容器运行时不可见，而 ENV 创建的变量不仅能够在构建镜像的过程中使用，在容器运行时也能够以环境变量的形式被应用程序使用。

docker 文档
https://docs.docker.com/reference/

```sh
Docker Hub
https://hub.docker.com/

docker login -u xxxx


docker tag 或者简单一点，直接用 docker build -t 在创建镜像的时候就起好名字。
docker tag ngx-app chronolaw/ngx-app:1.0


镜像发布 docker push 把这个镜像推上去
docker push chronolaw/ngx-app:1.0


Docker 提供了 save 和 load 这两个镜像归档命令，可以把镜像导出成压缩包，或者从压缩包导入 Docker，

docker save ngx-app:latest -o ngx.tar
docker load -i ngx.tar
```



```sh
# 074容器ID
# 将当前目录下的a.txt拷贝到容器里 的/tmp目录下
docker cp a.txt 074:/tmp
# 将容器下的/tmp/a.txt，拷贝到主机/home/mai/myfile/b.txt
docker cp 074:/tmp/a.txt /home/mai/myfile/b.txt
# 进入容器
docker exec -it 074 sh
#以 Redis 为例，启动容器，使用 -v 参数把本机的“/tmp”目录挂载到容器里的“/tmp”目录，也就是说让容器共享宿主机的“/tmp”目录：
docker run -d --rm -v /tmp:/tmp redis
```



```sh

Docker 提供了三种网络模式，分别是 null、host 和 bridge。

null 是最简单的模式，也就是没有网络，但允许其他的网络插件来自定义网络连接

host 的意思是直接使用宿主机网络，相当于去掉了容器的网络隔离（其他隔离依然保留），所有的容器会共享宿主机的 IP 地址和网卡。
host 模式需要在 docker run 时使用 --net=host 参数
docker run -d --rm --net=host nginx:alpine

第三种 bridge，也就是桥接模式，它有点类似现实世界里的交换机、路由器，只不过是由软件虚拟出来的，容器和宿主机再通过虚拟网卡接入这个网桥（图中的 docker0）

--net=bridge 来启用桥接模式
因为 Docker 默认的网络模式就是 bridge，所以一般不需要显式指定。

端口号映射需要使用 bridge 模式，并且在 docker run 启动容器时使用 -p 参数，形式和共享目录的 -v 参数很类似，用 : 分隔本机端口和容器端口
分隔本机端口:容器端口
docker run -d -p 80:80 --rm nginx:alpine
docker run -d -p 8080:80 --rm nginx:alpine
# 验证
curl 127.1:80 -I
curl 127.1:8080 -I
```

搭建 WordPress 网站

```sh
docker pull wordpress:5
docker pull mariadb:10
docker pull nginx:alpine


docker run -d --rm \
    --env MARIADB_DATABASE=db \
    --env MARIADB_USER=wp \
    --env MARIADB_PASSWORD=123 \
    --env MARIADB_ROOT_PASSWORD=123 \
    mariadb:10


docker exec -it d04 mysql -u wp -p
docker exec -it 9ac mysql -u wp -p

docker run -d --rm \
    --env WORDPRESS_DB_HOST=172.17.0.2 \
    --env WORDPRESS_DB_USER=wp \
    --env WORDPRESS_DB_PASSWORD=123 \
    --env WORDPRESS_DB_NAME=db \
    wordpress:5


docker inspect e78 | grep IPAddress

server {
  listen 80;
  default_type text/html;

  location / {
      proxy_http_version 1.1;
      proxy_set_header Host $host;
      proxy_pass http://172.17.0.6;
  }
}


docker run -d --rm \
    -p 80:80 \
    -v `pwd`/wp.conf:/etc/nginx/conf.d/default.conf \
    nginx:alpine

最后浏览器输入ip地址进行访问，例如：http://169.254.32.169/

# 数据库变化
use db
show tables;
# 查看容器运行日志
docker logs 024
```



快速搭建 Kubernetes 环境的工具
kind 和 minikube

kubernetes 文档
https://kubernetes.io/zh-cn/docs/tasks/tools/

minikube文档
https://minikube.sigs.k8s.io/docs/

```sh
快速搭建 Kubernetes 环境的工具
kind 和 minikube

kubernetes 文档
https://kubernetes.io/zh-cn/docs/tasks/tools/
https://kubernetes.io/zh-cn/

minikube文档
https://minikube.sigs.k8s.io/docs/


# 安装minikube

# Intel x86_64
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64

# Apple arm64
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-arm64

sudo install minikube /usr/local/bin/


# 查看minikube版本号
minikube version


不过 minikube 只能够搭建 Kubernetes 环境，要操作 Kubernetes，还需要另一个专门的客户端工具“kubectl”。

kubectl 的作用有点类似之前我们学习容器技术时候的工具“docker”，它也是一个命令行工具，作用也比较类似，
同样是与 Kubernetes 后台服务通信，把我们的命令转发给 Kubernetes，实现容器和集群的管理功能。

kubectl 是一个与 Kubernetes、minikube 彼此独立的项目，所以不包含在 minikube 里，
但 minikube 提供了安装它的简化方式，只需执行下面的这条命令：


minikube kubectl
它会把与当前 Kubernetes 版本匹配的 kubectl 下载下来，存放在内部目录（例如 .minikube/cache/linux/arm64/v1.23.3），然后我们就可以使用它来对 Kubernetes“发号施令”了。

所以，在 minikube 环境里，我们会用到两个客户端：minikube 管理 Kubernetes 集群环境，kubectl 操作实际的 Kubernetes 功能，和 Docker 比起来有点复杂。


使用命令 minikube start 会从 Docker Hub 上拉取镜像，以当前最新版本的 Kubernetes 启动集群。
不过为了保证实验环境的一致性，我们可以在后面再加上一个参数 --kubernetes-version，明确指定要使用 Kubernetes 版本。

kubectl version

minikube start --kubernetes-version=v1.23.3

minikube start --kubernetes-version=v1.23.3 --image-mirror-country='cn' --force

minikube delete --all --purge

minikube status
minikube node list

minikube kubectl -- version

alias kubectl="minikube kubectl --"

source <(kubectl completion bash)

kubectl version --short

kubectl version --client

kubectl run ngx --image=nginx:alpine
```

























