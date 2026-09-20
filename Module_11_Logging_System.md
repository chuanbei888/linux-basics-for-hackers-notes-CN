---
layout: default
title: 模块 11：日志系统
description: 读取系统日志、理解轮换并从防御角度审计事件。
permalink: /Module_11_Logging_System.html
---

# 黑客 Linux 基础
## 模块 11：日志系统

---

## 概述

Linux 会记录系统中几乎发生的一切：登录、失败登录、硬件事件、服务活动和错误。对防守者来说，日志能显示是否有人在探查系统；对攻击者来说，日志又记录了自己的活动，因此必须理解它们如何被记录和审计。所有操作只应在获得授权的实验环境中进行。

## 日志存放在哪里

几乎所有日志都在 `/var/log`：

```bash
ahegazy0@kali:~$ ls /var/log
```

| 日志文件 | 记录内容 |
|---|---|
| `/var/log/auth.log` | 登录尝试、sudo 使用、SSH 连接 |
| `/var/log/syslog` | 通用系统消息 |
| `/var/log/kern.log` | 内核消息、硬件事件、驱动错误 |
| `/var/log/messages` | 与 syslog 类似，部分发行版使用 |
| `/var/log/mail.log` | 邮件服务器活动 |
| `/var/log/apache2/` | Web 服务器访问和错误日志 |
| `/var/log/mysql/` | 数据库活动 |

最常查看的是 `auth.log`，它记录每次登录尝试以及网络登录时的来源 IP。

## 读取日志

```bash
ahegazy0@kali:~$ tail /var/log/auth.log
ahegazy0@kali:~$ tail -f /var/log/auth.log
ahegazy0@kali:~$ grep "Failed" /var/log/auth.log
ahegazy0@kali:~$ grep "Failed" /var/log/auth.log | grep "192.168.1.50"
```

`tail -f` 会实时跟踪新增内容，`grep` 可以筛选失败登录或特定 IP 的记录。

## rsyslog：实际写入日志的服务

**rsyslog** 负责收集内核和程序消息并写入 `/var/log`：

```bash
ahegazy0@kali:~$ service rsyslog status
ahegazy0@kali:~$ service rsyslog stop
```

停止它会让新日志暂时不再写入，但日志突然中断本身就是明显的告警信号。不要在未经授权的系统上停用日志服务。

## logrotate：防止日志占满磁盘

日志持续增长，`logrotate` 通常每天自动执行：

- 重命名当前日志（如 `auth.log` 变为 `auth.log.1`）
- 创建新的空日志
- 压缩旧日志（如 `auth.log.2.gz`）
- 删除超过保留期限的日志

配置位置：

```
/etc/logrotate.conf          ← 主配置
/etc/logrotate.d/            ← 各服务配置
```

## shred：安全销毁文件

普通 `rm` 只会标记空间可复用，数据可能仍能被取证工具恢复。`shred` 会在删除前多次覆盖文件：

```bash
ahegazy0@kali:~$ shred -vzu filename.txt
ahegazy0@kali:~$ shred -n 10 -vzu filename.txt
```

- `-v`：显示进度
- `-z`：最后用零覆盖，隐藏已执行 shred 的迹象
- `-u`：覆盖后删除
- `-n 10`：覆盖 10 次

> 现代 SSD、日志聚合系统和写时复制文件系统可能保留副本，`shred` 不能保证所有介质上的绝对不可恢复。

## 日志完整性与防御视角

从防守角度：

- 怀疑入侵时，可用 `tail -f /var/log/auth.log` 实时监控
- 同一 IP 重复出现 “Failed password” 可能是暴力破解，应使用 `iptables`、`ufw` 或 `fail2ban` 处理
- 日志为空、消失或近期被覆盖本身就是篡改证据
- 应把日志转发到集中式、只读或远程存储，避免攻击者仅修改本机日志

在授权取证或实验中，可以比较多个日志源（认证日志、Web 访问日志、数据库日志、IDS 记录），因为单一文件无法代表完整时间线。

---

## 命令速查

| 命令 | 作用 |
|---|---|
| `ls /var/log` | 查看全部日志文件 |
| `tail /var/log/auth.log` | 查看认证日志最后 10 行 |
| `tail -f /var/log/auth.log` | 实时跟踪认证日志 |
| `grep "term" /var/log/file` | 搜索特定文字 |
| `service rsyslog status` | 检查日志服务状态 |
| `service rsyslog stop` | 停止日志服务（仅限授权实验） |
| `shred -vzu filename` | 覆盖并删除文件 |
| `shred -n 10 -vzu filename` | 覆盖 10 次后删除 |
| `history -c` | 清除当前 Bash 历史（按组织审计政策操作） |
| `cat /etc/logrotate.conf` | 查看日志轮换配置 |

---

## 练习

- [ ] 进入 `/var/log` 并运行 `ls`
- [ ] 运行 `tail /var/log/auth.log` 查看最近记录
- [ ] 在一个终端运行 `tail -f /var/log/auth.log`，在另一个终端执行 `ssh localhost`，观察实时日志
- [ ] 创建测试文件 `echo "sensitive data" > test.txt`，再在自己的实验目录运行 `shred -vzu test.txt`
- [ ] 打开 `/etc/logrotate.conf`，找出旧日志保留数量的配置

> 💡 *为了进行更深入的练习，也建议完成官方 **Linux Basics for Hackers** 书中每章末尾的练习。*
---

*下一篇：模块 12——服务的使用与滥用*
