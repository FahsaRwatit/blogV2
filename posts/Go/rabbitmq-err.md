---
title: "Application rabbit exited with reason: {{cannot_log_to_file"
description: "Application rabbit exited with reason: {{cannot_log_to_file"
date: 2022-05-10 18:10:00
slug: rabbitmq-err
image:
categories:
    - Go
tags: ["Go", "RabbitMQ"]
---

关于一不小心删除了RabbitMQ日志后，然后进行`systemctl start rabbitmq-server`总是报以下错误：

```shell
Job for rabbitmq-server.service failed because the control process exited with error code.
See "systemctl status rabbitmq-server.service" and "journalctl -xe" for details.
```

按照提示执行`journalctl -xe`，发现以下错误：

```shell
Application rabbit exited with reason: {{cannot_log_to_file
```

创建该日志文件，并修改其权限，然后重新执行`systemctl start rabbitmq-server`即可启动成功。

总结：学会根据提示信息，执行相关命令找到发生错误的原因并解决它。