# 黑客 Linux 基础
## 模块 3：网络管理

---

## 概述

黑客活动几乎总是发生在网络上。在网络中进行任何操作前，你需要了解自己的网络身份：IP 地址和 MAC 地址，并知道如何管理它们。本模块还会介绍 DNS 的基础知识。

![网络路由示意图](assets/network_routing_diagram_1789213191969.jpg)

## 每台设备都有的两个地址

**IP 地址**是设备在网络上的逻辑地址，由路由器分配，也可以手动设置。它像街道地址，告诉网络你在哪里，但可能发生变化。

**MAC 地址**由网卡制造商写入硬件，是由 12 个十六进制字符组成的硬件级标识符，每个网络接口都不同，例如 `00:1A:2B:3C:4D:5E`。它不像 IP 地址那样容易永久修改，但可以临时伪装。

理解二者很重要，因为它们共同标识了你在网络上的设备。能够读取并修改它们是基本技能。

---

## 基本命令

### ifconfig

用于查看和管理网络接口。不带参数运行时，会显示当前网络配置：

```bash
ahegazy0@kali:~$ ifconfig
```

常见接口：

| 接口 | 含义 |
|---|---|
| `eth0` | 有线以太网连接 |
| `wlan0` | 无线 Wi-Fi 连接 |
| `lo` | 回环接口，系统与自身通信，始终为 127.0.0.1 |

输出中的 `inet` 是当前 IP 地址，`ether` 是 MAC 地址。

```bash
ahegazy0@kali:~$ ifconfig eth0
ahegazy0@kali:~$ ifconfig eth0 192.168.1.100
ahegazy0@kali:~$ ifconfig eth0 192.168.1.100 netmask 255.255.255.0
ahegazy0@kali:~$ ifconfig eth0 up
ahegazy0@kali:~$ ifconfig eth0 down
```

关闭再重新启用接口，有时是应用配置变更所必需的。

### iwconfig

类似 `ifconfig`，但专门用于无线接口，会显示 ESSID、信号强度和传输速率等 Wi-Fi 信息：

```bash
ahegazy0@kali:~$ iwconfig
```

它主要用于查看信息，也适合确认当前连接的无线网络和信号情况。

### dhclient

使用 `ifconfig` 手动设置 IP 后，就不会再从路由器自动获取地址。要恢复自动获取，请使用 `dhclient`：

```bash
ahegazy0@kali:~$ dhclient eth0
ahegazy0@kali:~$ dhclient -r eth0     释放当前 IP
ahegazy0@kali:~$ dhclient eth0        请求新的 IP
```

如果手动修改 IP 后失去网络连接，通常运行它即可修复。

### 修改 MAC 地址

MAC 地址在硬件层面应当固定，但可以在软件层临时伪装。重启后真实 MAC 会恢复。

```bash
ahegazy0@kali:~$ ifconfig eth0 down
ahegazy0@kali:~$ ifconfig eth0 hw ether 00:11:22:33:44:55
ahegazy0@kali:~$ ifconfig eth0 up
```

运行 `ifconfig eth0` 确认 `ether` 行是否显示新地址。

> 这种修改只存在于内存中，不会跨越重启。需要持久化时，可以使用 `macchanger` 等工具。

### dig

`dig` 用于查询 DNS，即把 `google.com` 等域名转换成 IP 地址的系统。它提供的信息比简单地 ping 域名更详细：

```bash
ahegazy0@kali:~$ dig google.com
ahegazy0@kali:~$ dig google.com mx
ahegazy0@kali:~$ dig google.com ns
```

`ANSWER SECTION` 会显示域名解析到的 IP。`mx` 代表 Mail Exchange，用于查找处理域名邮件的服务器，常用于侦察。

| 记录类型 | 内容 |
|---|---|
| `a` | 域名的 IPv4 地址 |
| `aaaa` | IPv6 地址 |
| `mx` | 邮件服务器 |
| `ns` | 权威名称服务器 |
| `txt` | 文本记录，常包含验证数据 |

### DNS：为什么值得关注

DNS（Domain Name System，域名系统）是互联网的地址簿。输入 `google.com` 时，电脑会询问 DNS 服务器它对应的 IP。没有 DNS，你就必须记住每个网站的 IP 地址。

黑客关注 DNS 有两个原因：DNS 查询可以暴露目标的邮件服务器、子域名和名称服务器等基础设施信息；DNS 响应也可能被操纵，把用户重定向到错误的服务器，这称为 DNS 中毒或 DNS 欺骗。

---

## 命令速查

| 任务 | 命令 |
|---|---|
| 查看所有网络接口 | `ifconfig` |
| 查看一个接口 | `ifconfig eth0` |
| 设置 IP 地址 | `ifconfig eth0 192.168.1.100` |
| 关闭/启用接口 | `ifconfig eth0 down / up` |
| 查看无线信息 | `iwconfig` |
| 从 DHCP 请求 IP | `dhclient eth0` |
| 释放当前 IP | `dhclient -r eth0` |
| 伪装 MAC 地址 | `ifconfig eth0 hw ether 00:11:22:33:44:55` |
| DNS 查询 | `dig google.com` |
| 查找邮件服务器 | `dig google.com mx` |

---

## 练习

- [ ] 运行 `ifconfig`，找出自己的 IP 和 MAC 地址
- [ ] 运行 `iwconfig`，查看无线连接信息
- [ ] 用 `dig google.com` 查看 DNS 响应，再用 `dig google.com mx` 查找邮件服务器
- [ ] 在自己的实验机上关闭接口、为其设置临时 IP，然后用 `dhclient` 恢复自动配置

> 💡 *为了进行更深入的练习，也建议完成官方 **Linux Basics for Hackers** 书中每章末尾的练习。*
---

*下一篇：模块 4——软件管理*
