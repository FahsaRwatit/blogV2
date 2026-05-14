---
title: 如何在 Linux 中清理磁盘空间
description: 如何在 Linux 中清理磁盘空间
date: 2022-03-10 15:10:00
slug: how-to-clean-up-disk-space-in-linux
image:
categories:
    - Linux
tags: ["Linux"]

---









/var/log/messages      绝大多数的系统日志都记录到该文件
/var/log/secure          所有跟安全和认证授权等日志都会记录到此文件
/var/log/maillog       邮件服务的日志
/var/log/cron             crond 计划任务的日志
/var/log/boot.log       系统启动的相关日志



只保留近一周的日志
journalctl --vacuum-time=1w

只保留 500MB 的日志
journalctl --vacuum-size=500M

用 echo 命令，将空字符串内容重定向到指定文件中
echo "" > system.journal



