# 黑客 Linux 基础
## 模块 16：自动化与计划任务

---

## 概述

每天手动输入相同命令很浪费时间。Linux 内置的 cron 调度系统可以按指定时间和周期运行脚本或命令。本模块介绍 crontab、开机启动和自动化系统的工作方式。

## Cron：Linux 调度器

Cron 是后台服务，每分钟检查一次是否有到期任务并执行。每个用户都有自己的 **crontab**（cron table），root 也有一份；它们相互独立，并以所属用户的权限运行。

## crontab 语法

```
MIN  HOUR  DAY  MONTH  WEEKDAY  command
```

| 字段 | 范围 | 含义 |
|---|---|---|
| MIN | 0–59 | 小时中的分钟 |
| HOUR | 0–23 | 小时 |
| DAY | 1–31 | 月中的日期 |
| MONTH | 1–12 | 月份 |
| WEEKDAY | 0–7 | 星期，0 和 7 都表示星期日 |

任意字段中的 `*` 表示“每一个”。示例：

```bash
# 每分钟
* * * * *  /path/to/script.sh

# 每天凌晨 3 点
0 3 * * *  /path/to/script.sh

# 每周三凌晨 3 点
0 3 * * 3  /path/to/script.sh

# 每月 15 日中午
0 12 15 * *  /path/to/script.sh

# 周一至周五上午 8 点
0 8 * * 1-5  /path/to/script.sh

# 每小时整点
0 * * * *  /path/to/script.sh
```

从左到右读取即可，例如 `0 3 * * 3` 表示星期三凌晨 3:00。

## 编辑 crontab

```bash
ahegazy0@kali:~$ crontab -e
ahegazy0@kali:~$ crontab -l
ahegazy0@kali:~$ crontab -r
ahegazy0@kali:~$ sudo crontab -e
```

`-e` 编辑，`-l` 查看，`-r` 删除当前用户全部任务且不会确认，使用前要谨慎。root 的任务以 root 权限运行。

## 特殊简写

| 简写 | 等价于 | 运行时间 |
|---|---|---|
| `@reboot` | - | 每次系统启动一次 |
| `@hourly` | `0 * * * *` | 每小时 |
| `@daily` | `0 0 * * *` | 每天午夜 |
| `@weekly` | `0 0 * * 0` | 每周日午夜 |
| `@monthly` | `0 0 1 * *` | 每月第一天 |

例如：

```bash
@reboot  /home/kali/scripts/startup.sh
```

## 实例：每分钟记录时间

```bash
#!/bin/bash
echo "System check: $(date)" >> /home/kali/check.log
```

保存为 `check.sh` 并赋予权限：

```bash
ahegazy0@kali:~$ chmod 755 /home/kali/check.sh
ahegazy0@kali:~$ crontab -e
```

加入：

```
* * * * *  /home/kali/check.sh
```

等待几分钟后查看：

```bash
ahegazy0@kali:~$ cat /home/kali/check.log
```

## Cron 输出和日志

默认情况下，cron 会通过邮件发送任务输出。更可靠的方式是重定向到日志：

```bash
* * * * *  /home/kali/check.sh >> /home/kali/cron.log 2>&1
* * * * *  /home/kali/check.sh > /dev/null 2>&1
```

`>>` 追加标准输出，`2>&1` 将错误输出重定向到同一位置，`/dev/null` 丢弃所有输出。

## 让服务开机启动

```bash
ahegazy0@kali:~$ systemctl enable mysql
ahegazy0@kali:~$ systemctl enable apache2
ahegazy0@kali:~$ systemctl enable ssh
ahegazy0@kali:~$ systemctl list-unit-files --type=service | grep enabled
```

现代系统使用 `systemctl enable`；旧系统可能使用：

```bash
ahegazy0@kali:~$ update-rc.d mysql defaults
```

## /etc/cron 目录

系统还提供按周期运行脚本的目录：

```
/etc/cron.hourly/     每小时
/etc/cron.daily/      每天
/etc/cron.weekly/     每周
/etc/cron.monthly/    每月
```

将可执行脚本放入相应目录即可，无需额外 crontab 条目。

---

## 命令速查

| 命令 | 作用 |
|---|---|
| `crontab -e` | 编辑个人计划 |
| `crontab -l` | 查看当前任务 |
| `crontab -r` | 删除全部个人任务 |
| `sudo crontab -e` | 编辑 root 计划 |
| `systemctl enable [service]` | 设置服务开机启动 |
| `systemctl disable [service]` | 取消服务开机启动 |
| `systemctl list-unit-files --type=service` | 查看服务启动状态 |

## 练习

- [ ] 添加每分钟记录日期的任务，验证两分钟后运行，再删除任务
- [ ] 添加 `@reboot` 任务并在重启后验证
- [ ] 查看 root 的 crontab，了解已有任务
- [ ] 用 `systemctl enable mysql` 设置 MySQL 开机启动并检查状态
- [ ] 写出“工作日每天 6:30”的 cron 表达式：`30 6 * * 1-5`

> 💡 *为了进行更深入的练习，也建议完成官方 **Linux Basics for Hackers** 书中每章末尾的练习。*
---

*下一篇：模块 17——Python 脚本*
