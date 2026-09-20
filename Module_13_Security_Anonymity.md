---
layout: default
title: 模块 13：安全与匿名性
description: 认识代理、Tor、VPN 和操作安全的边界。
permalink: /Module_13_Security_Anonymity.html
---

# 黑客 Linux 基础
## 模块 13：安全与匿名性

---

## 概述

你在网上做的每件事都会留下痕迹。网站和服务器通常都能看到你的真实 IP。本模块介绍代理、Tor、VPN 和 proxychains 如何降低暴露面，也说明它们的局限。请只在合法、获得授权的环境中进行测试。

## 为什么 IP 地址重要

IP 地址由 ISP 分配，每次连接互联网服务器都会携带它。目标服务器、ISP 以及所在网络都可能观察到你的连接和流量。匿名工具的作用，是在你和目标之间增加中间层，让目标看到另一个地址。

## 代理

代理服务器位于你和目标之间：

```
你 → 代理 → 网站
```

网站看到代理的 IP，ISP 看到你连接了代理，但代理本身知道你的真实 IP 和目标。免费代理常常不适合敏感活动，可能是专门记录流量的蜜罐。

## Tor：洋葱路由

Tor 将流量依次经过入口、中间和出口三个节点，每个节点只知道前后一步：

```
你 → 入口节点 → 中间节点 → 出口节点 → 网站
```

流量被包裹成三层加密，每个节点剥开一层来决定下一跳，但无法看到完整路径。网站看到出口节点 IP，入口节点知道你的 IP 但不知道目的地，中间节点两者都不知道。

在 Kali 中：

```bash
ahegazy0@kali:~$ apt install tor
ahegazy0@kali:~$ service tor start
```

Tor 会增加延迟，不适合高带宽或强时效活动；如果在 Tor 中登录个人账户，应用层身份仍会暴露。

## Proxychains：让工具通过代理

浏览器流量经过 Tor 或 VPN，并不意味着终端工具也会经过。`proxychains` 会拦截程序的网络调用并强制它们经过配置的代理链：

```bash
ahegazy0@kali:~$ proxychains nmap -sT 192.168.1.1
ahegazy0@kali:~$ nano /etc/proxychains.conf
```

配置示例：

```
socks5  127.0.0.1  9050    ← Tor 本地端口
socks4  10.0.0.1   1080
http    203.0.113.5  3128
```

- `strict_chain`：必须按顺序经过所有代理，任何一个失效都会失败
- `dynamic_chain`：自动跳过失效代理，使用可用代理

大多数场景下 `dynamic_chain` 更实用。

## VPN

VPN 在你和 VPN 服务器之间建立加密隧道：

```
你 → [加密隧道] → VPN 服务器 → 互联网
```

与 Tor 相比，VPN 更快，但服务商可以看到流量并知道你的真实 IP。应选择有审计支持的无日志政策的可信服务商。也有人组合使用 VPN + Tor：

```
你 → VPN → Tor 网络 → 网站
```

这样会增加更多延迟，且不能替代良好的操作安全。

## 加密通信

**ProtonMail** 提供端到端加密邮件，**Signal** 默认提供端到端加密并尽量减少元数据。记者、律师和处理敏感信息的人都可以使用这些工具；使用加密通信本身并不等于可疑。

## 真正会破坏匿名性的因素

- 使用 Tor 时登录 Google 或社交媒体等个人账户
- 在匿名和非匿名活动中复用用户名
- 浏览器指纹暴露操作系统、分辨率、字体和时区
- 文件元数据暴露创建设备信息
- 将敏感流量交给免费或未知代理

匿名性是一种操作习惯，而不是安装一个工具就能获得的属性。

---

## 命令速查

| 命令 | 作用 |
|---|---|
| `service tor start` | 启动 Tor 服务 |
| `proxychains [command]` | 让命令通过代理链运行 |
| `nano /etc/proxychains.conf` | 编辑 proxychains 配置 |
| `curl ifconfig.me` | 查看互联网看到的 IP |
| `proxychains curl ifconfig.me` | 通过代理链查看表面 IP |

## 练习

- [ ] 在实验环境安装并启动 Tor，运行 `proxychains curl ifconfig.me`
- [ ] 打开 `/etc/proxychains.conf`，了解 `socks5 127.0.0.1 9050` 配置
- [ ] 直接访问 IP 查询网站，再用 Tor Browser 访问并比较
- [ ] 了解 ProtonMail 或其他端到端加密通信工具

> 💡 *为了进行更深入的练习，也建议完成官方 **Linux Basics for Hackers** 书中每章末尾的练习。*
---

*下一篇：模块 14——无线网络*
