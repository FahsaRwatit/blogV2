---
title: Kubernetes 入门：核心概念全梳理
date: 2024-01-02
tags: ["Kubernetes", "K8s", "容器编排"]
excerpt: 一篇文章理清 K8s 的核心概念：Pod、Deployment、Service、Namespace 都是什么，它们如何协作。
---

# Kubernetes 入门：核心概念全梳理

第一次接触 K8s 时，最大的困惑是概念太多 —— Pod、Deployment、Service、Ingress、ConfigMap…… 名词一堆，关系不清。

这篇文章用一条主线把核心概念串起来：**从一个容器，到一个生产可用的服务**。

## 一、为什么需要 Kubernetes

Docker 解决了「应用怎么打包运行」的问题，但当应用扩展到几十上百个容器后，新的问题出现了：

- 一个容器挂了，谁来重启？
- 流量大了，怎么自动扩容？
- 多副本如何负载均衡？
- 怎么平滑升级而不中断服务？
- 配置和密钥怎么管理？

**K8s 就是解决这些问题的容器编排系统。** 你只要声明「我想要什么状态」，K8s 负责让现实保持这个状态。

## 二、集群架构

K8s 集群分两部分：

```
┌──────────────────────────────────────┐
│      Control Plane (控制平面)        │
│  ┌─────────────┐  ┌──────────────┐  │
│  │ API Server  │  │  Scheduler   │  │
│  └─────────────┘  └──────────────┘  │
│  ┌─────────────┐  ┌──────────────┐  │
│  │    etcd     │  │  Controller  │  │
│  └─────────────┘  └──────────────┘  │
└──────────────────────────────────────┘
              │
        ┌─────┴─────┬─────────┐
        ▼           ▼         ▼
   ┌────────┐  ┌────────┐  ┌────────┐
   │ Node 1 │  │ Node 2 │  │ Node 3 │
   │  Pod   │  │  Pod   │  │  Pod   │
   │  Pod   │  │  Pod   │  │  Pod   │
   └────────┘  └────────┘  └────────┘
        Worker Nodes (工作节点)
```

- **Control Plane**：大脑，负责调度和管理
- **Worker Node**：干活的，实际跑容器

核心组件：

| 组件 | 作用 |
|------|------|
| API Server | 所有操作的入口，kubectl 就是和它对话 |
| etcd | 存储集群所有状态的数据库 |
| Scheduler | 决定 Pod 跑在哪个 Node 上 |
| Controller Manager | 各种控制器，确保实际状态等于期望状态 |
| kubelet | 每个 Node 上的代理，管理本机 Pod |

## 三、Pod —— 最小调度单位

**Pod 是 K8s 里最小的部署单元**，不是容器，是「一组共享网络和存储的容器」。

大部分情况下，一个 Pod 里就一个容器。少数场景会放多个，比如主容器 + Sidecar（日志收集、代理）。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
  - name: app
    image: nginx:1.25
    ports:
    - containerPort: 80
```

**为什么不直接用容器，要包一层 Pod？**

因为同一个 Pod 里的容器共享：
- 同一个 IP 地址（互相通过 `localhost` 通信）
- 同一个存储卷
- 同一个生命周期

这给「紧密耦合的辅助进程」提供了天然的容器。

⚠️ **Pod 是临时的**。Pod 重建后 IP 会变，所以业务不该直接连 Pod，而是连 Service（后面讲）。

## 四、Deployment —— 管 Pod 的「老板」

直接创建 Pod 在生产中几乎不用，因为 Pod 挂了不会自动重建。

**Deployment 替你管理一组 Pod**：

- 声明「我要 3 个副本」，K8s 保证永远有 3 个
- Pod 挂了自动重建
- 升级镜像时滚动更新，零停机
- 出问题可以一键回滚

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
spec:
  replicas: 3          # 我要 3 个副本
  selector:
    matchLabels:
      app: nginx
  template:            # Pod 模板
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
```

> Deployment 不直接管 Pod，中间还有个 ReplicaSet。Deployment → ReplicaSet → Pod。日常开发不用太关心 ReplicaSet。

## 五、Service —— 给 Pod 一个固定地址

Pod 的 IP 会变，怎么稳定访问？

**Service 给一组 Pod 提供固定的虚拟 IP 和 DNS 名字**，并自动负载均衡。

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
spec:
  selector:
    app: nginx        # 匹配带这个 label 的所有 Pod
  ports:
  - port: 80          # Service 暴露的端口
    targetPort: 80    # Pod 的端口
```

集群内其他 Pod 通过 `nginx-svc:80` 就能访问，背后自动负载均衡到 3 个 nginx Pod。

Service 有几种类型：

| 类型 | 用途 |
|------|------|
| ClusterIP | 默认，集群内访问 |
| NodePort | 通过 Node 的端口对外暴露（开发测试） |
| LoadBalancer | 云厂商负载均衡器（生产） |

## 六、Ingress —— HTTP 路由网关

`LoadBalancer` 每暴露一个服务就要一个公网 IP，太贵。

**Ingress 是 7 层 HTTP 路由**，一个 Ingress 可以根据域名和路径把流量分发到不同 Service：

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
  rules:
  - host: api.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api-svc
            port:
              number: 80
  - host: blog.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: blog-svc
            port:
              number: 80
```

Ingress 需要配合 Ingress Controller（nginx-ingress、traefik 等）一起用。

## 七、ConfigMap & Secret —— 配置管理

应用配置不应该写死在镜像里，K8s 提供两种存储：

- **ConfigMap**：普通配置（数据库 URL、日志级别等）
- **Secret**：敏感数据（密码、Token），base64 编码存储

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  LOG_LEVEL: "info"
  DB_HOST: "mysql.prod.svc"
```

在 Pod 里用：

```yaml
spec:
  containers:
  - name: app
    image: myapp:v1
    envFrom:
    - configMapRef:
        name: app-config
```

## 八、Namespace —— 逻辑隔离

Namespace 把集群切成多个逻辑空间，常用来分环境或团队：

```bash
kubectl create namespace dev
kubectl create namespace prod
```

每个 Namespace 内的资源名独立，可以有重名。但 Node、PersistentVolume 等是集群级别的，不分 Namespace。

## 九、Volume & PersistentVolume —— 持久化存储

容器是无状态的，删了数据就没了。要持久化数据，用 Volume。

- **Volume**：和 Pod 生命周期一样
- **PersistentVolume (PV)**：独立于 Pod 的存储资源
- **PersistentVolumeClaim (PVC)**：Pod 对 PV 的「申请单」

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data-pvc
spec:
  accessModes: [ReadWriteOnce]
  resources:
    requests:
      storage: 10Gi
```

## 十、StatefulSet —— 有状态应用

Deployment 适合无状态应用（Pod 之间完全等价）。

**StatefulSet 用于有状态应用**（数据库、消息队列），它保证：
- 每个 Pod 有固定的名字（`mysql-0`, `mysql-1`, `mysql-2`）
- 启动和删除按顺序
- 每个 Pod 绑定固定的存储

## 十一、概念全景图

```
                      Ingress
                         │
                         ▼
                      Service
                         │
              ┌──────────┼──────────┐
              ▼          ▼          ▼
            Pod        Pod        Pod        ← Deployment 管理
              │          │          │
              └──────────┼──────────┘
                         │
                    ConfigMap
                     Secret
                  (注入配置)
                         │
                         ▼
                    PersistentVolume
                     (持久化数据)
              ─────────────────────────
                     Namespace
                    (逻辑隔离)
```

## 十二、常用 kubectl 命令

```bash
# 查看资源
kubectl get pods                    # 看 Pod
kubectl get pods -A                 # 所有 Namespace
kubectl get deploy                  # 看 Deployment
kubectl get svc                     # 看 Service

# 详情和日志
kubectl describe pod <pod-name>     # 详细信息
kubectl logs <pod-name>             # 看日志
kubectl logs -f <pod-name>          # 实时跟踪日志

# 进容器
kubectl exec -it <pod-name> -- sh

# 应用 yaml
kubectl apply -f deploy.yaml
kubectl delete -f deploy.yaml

# 扩缩容
kubectl scale deploy nginx --replicas=5
```

## 学习建议

1. **本地搭一个 K8s** — `minikube` 或 `kind`，比看文档实际十倍
2. **从 Pod 开始一步步往上** — 先跑通一个 Pod，再换 Deployment，再加 Service
3. **多用 `kubectl describe`** — 出问题时它比 `get` 信息多得多
4. **理解声明式** — K8s 的核心思想是「描述期望状态」，不是「执行操作步骤」

> K8s 复杂度的天花板很高，但日常用到的就这十几个概念。把这些吃透，剩下的都是「为了解决特定问题加的工具」。
