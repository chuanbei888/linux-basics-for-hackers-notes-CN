# 黑客 Linux 基础
## 模块 12：服务的使用与滥用

---

## 概述

服务是在后台持续运行、等待提供功能的程序，例如网页、数据库查询或 SSH 连接。理解服务既有助于搭建自己的工具，也有助于识别目标机器上运行的内容和潜在入口。所有测试必须针对自己的设备或已获书面授权的系统。

## 什么是服务

服务也叫守护进程（daemon），通常在启动时运行并持续在后台监听某个网络端口。

- Apache 监听 80（HTTP）和 443（HTTPS）并提供网页
- SSH 监听 22，接受远程终端连接
- MySQL 监听 3306，处理数据库查询
- PostgreSQL 监听 5432

端口号帮助你判断远程连接对应的服务；nmap 扫描时，每个开放端口都代表一个服务。

![服务器后台服务](assets/background_services_diagram_1789213348325.jpg)

## 启动和停止服务

`service` 是旧式工具，`systemctl` 是现代标准，两者在 Kali 中都可用：

```bash
ahegazy0@kali:~$ service apache2 start
ahegazy0@kali:~$ service apache2 stop
ahegazy0@kali:~$ service apache2 restart
ahegazy0@kali:~$ service apache2 status

ahegazy0@kali:~$ systemctl start apache2
ahegazy0@kali:~$ systemctl stop apache2
ahegazy0@kali:~$ systemctl restart apache2
ahegazy0@kali:~$ systemctl status apache2
ahegazy0@kali:~$ systemctl enable apache2
ahegazy0@kali:~$ systemctl disable apache2
```

`enable` 让服务开机自动启动，`disable` 取消自动启动。

## Apache：Web 服务器

```bash
ahegazy0@kali:~$ service apache2 start
ahegazy0@kali:~$ echo "<h1>My custom page</h1>" > /var/www/html/index.html
```

启动后访问 `http://localhost`，默认页面位于 `/var/www/html/index.html`。Apache 日志保存在 `/var/log/apache2/`，访问日志记录连接者和请求内容，错误日志记录故障。

未打补丁或配置不当的 Web 服务器可能成为入口。看到 80 或 443 端口开放时，需要确认 Web 服务器及版本，并在授权范围内检查已知漏洞。

## SSH：远程终端访问

SSH 允许通过加密网络连接控制另一台电脑：

```bash
ahegazy0@kali:~$ ssh username@192.168.1.50
ahegazy0@kali:~$ service ssh start
ahegazy0@kali:~$ ssh -p 2222 username@192.168.1.50
ahegazy0@kali:~$ scp file.txt username@192.168.1.50:/home/username/
```

弱密码或默认凭据是常见风险。启用 SSH 后应立即修改默认密码，并配置密钥认证、限速和 fail2ban。

## MySQL：数据库

MySQL 使用表和 SQL 查询存储 Web 应用数据：

```bash
ahegazy0@kali:~$ service mysql start
ahegazy0@kali:~$ mysql -u root -p
```

进入提示符后，可执行：

```sql
SHOW DATABASES;           -- 列出数据库
USE database_name;        -- 切换数据库
SHOW TABLES;              -- 列出表
SELECT * FROM users;      -- 查询 users 表
```

Web 应用若未正确处理输入，可能受到 SQL 注入；数据库弱密码也会直接暴露用户数据。只应在授权测试中验证这些问题。

## PostgreSQL

Metasploit 使用 PostgreSQL 保存扫描结果和会话数据：

```bash
ahegazy0@kali:~$ service postgresql start
```

启动 Metasploit 前若数据库未运行，通常会提示数据库未连接。

## 查看正在运行的服务

```bash
ahegazy0@kali:~$ systemctl list-units --type=service --state=running
ahegazy0@kali:~$ ss -tlnp
ahegazy0@kali:~$ netstat -tlnp
```

输出会显示端口、协议和监听进程。检查自己的机器可以了解暴露面，检查目标则必须拥有相应授权。

## 默认凭据是现实风险

路由器、摄像头、数据库和服务器经常保留安装时的默认密码，例如 `admin/admin`、`root/root`。大量真实入侵不是依靠零日漏洞，而是因为默认凭据未被修改。部署服务后应立即更换默认凭据；授权评估时，检查默认凭据也是常见的早期步骤。

---

## 命令速查

| 命令 | 作用 |
|---|---|
| `service name start/stop/restart` | 使用旧式语法管理服务 |
| `systemctl start/stop/restart name` | 使用现代语法管理服务 |
| `systemctl status name` | 查看服务状态 |
| `systemctl enable name` | 设置开机自动启动 |
| `systemctl disable name` | 取消开机自动启动 |
| `systemctl list-units --type=service` | 列出服务 |
| `ssh user@ip` | 通过 SSH 连接远程机器 |
| `scp file user@ip:/path` | 复制文件到远程机器 |
| `mysql -u root -p` | 登录 MySQL |
| `ss -tlnp` | 查看开放端口和监听服务 |

| 服务 | 默认端口 | 作用 |
|---|---:|---|
| SSH | 22 | 远程终端 |
| HTTP（Apache） | 80 | Web 服务器 |
| HTTPS | 443 | 加密 Web 服务器 |
| MySQL | 3306 | 数据库 |
| PostgreSQL | 5432 | 数据库 |
| FTP | 21 | 文件传输 |
| SMTP | 25 | 发送邮件 |

## 练习

- [ ] 在自己的实验机上启动 Apache，访问 `http://localhost`
- [ ] 修改 `/var/www/html/index.html` 并刷新页面
- [ ] 启动 MySQL，使用 `mysql -u root -p` 登录并运行 `SHOW DATABASES;`
- [ ] 运行 `ss -tlnp`，观察这些服务打开的端口
- [ ] 完成后停止不再需要的服务

> 💡 *为了进行更深入的练习，也建议完成官方 **Linux Basics for Hackers** 书中每章末尾的练习。*
---

*下一篇：模块 13——安全与匿名性*
