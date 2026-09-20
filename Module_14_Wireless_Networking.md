---
layout: default
title: 模块 14：无线网络
description: 在授权环境中检查 Wi-Fi 与 Bluetooth 网络。
permalink: /Module_14_Wireless_Networking.html
---

# 黑客 Linux 基础
## 模块 14：无线网络

---

## 概述

Wi-Fi 普遍存在且经常配置不当，是常见的攻击面。本模块介绍无线网络的工作方式、用于检查和审计的工具，以及 Bluetooth 的基本操作。所有内容只应在自己拥有或明确获书面授权的网络和设备上使用。

## Wi-Fi 的工作方式

笔记本连接路由器时，会在特定频率交换无线电信号。默认情况下，网卡只关注发给自己的数据包，这叫**管理模式**（managed mode）。无线分析需要接收范围内所有设备的数据包，这叫**监听模式**（monitor mode）。

并非所有网卡都支持监听模式或数据包注入，通常需要兼容的 USB 无线适配器。

## aircrack-ng 工具套件

| 工具 | 作用 |
|---|---|
| `airmon-ng` | 启用或关闭监听模式 |
| `airodump-ng` | 捕获数据包并显示附近网络 |
| `aireplay-ng` | 向网络注入数据包（仅限授权测试） |
| `aircrack-ng` | 尝试破解捕获的 WPA 握手 |

它们通常组合成工作流，而不是单独使用。

## 将网卡切换到监听模式

```bash
ahegazy0@kali:~$ iwconfig
ahegazy0@kali:~$ airmon-ng check kill
ahegazy0@kali:~$ airmon-ng start wlan0
ahegazy0@kali:~$ airmon-ng stop wlan0mon
```

接口通常会从 `wlan0` 改名为 `wlan0mon`。`check kill` 会停止可能干扰监听模式的 NetworkManager 等进程，完成后用 `airmon-ng stop` 恢复普通模式。

## 使用 airodump-ng 扫描附近网络

```bash
ahegazy0@kali:~$ airodump-ng wlan0mon
```

示例输出：

```
BSSID              PWR  Beacons  #Data  CH   MB   ENC   ESSID
AA:BB:CC:DD:EE:FF  -45      120     34   6  130   WPA2  HomeNetwork
11:22:33:44:55:66  -72       80     12  11   54   WPA2  CoffeeShop_WiFi
```

| 列 | 含义 |
|---|---|
| BSSID | 路由器的 MAC 地址 |
| PWR | 信号强度，越负越弱 |
| CH | 网络使用的信道 |
| ENC | 加密类型（WPA2、WPA3、WEP、OPN） |
| ESSID | 连接时看到的网络名称 |

在授权网络上聚焦特定路由器：

```bash
ahegazy0@kali:~$ airodump-ng --bssid AA:BB:CC:DD:EE:FF --channel 6 -w capture wlan0mon
```

## 无线术语

- **SSID**：网络名称
- **BSSID**：路由器 MAC 地址
- **信道**：2.4 GHz 或 5 GHz 频段中的无线信道
- **WPA2/WPA3**：现代 Wi-Fi 加密标准
- **WEP**：已经被攻破的旧标准，不应继续使用
- **握手**：设备连接 WPA2 时交换的四次握手，授权测试中可用于离线验证密码强度

## 使用 iwlist 扫描

只想查看附近网络而不切换监听模式时：

```bash
ahegazy0@kali:~$ iwlist wlan0 scan
```

它会显示 SSID、BSSID、信道、信号强度和加密类型，信息不如 airodump-ng 详细，但适合基础侦察。

## 使用 BlueZ 操作 Bluetooth

Linux 的 Bluetooth 工具来自 **BlueZ**：

```bash
ahegazy0@kali:~$ hcitool scan
Scanning...
    AA:BB:CC:DD:EE:FF    John's iPhone
    11:22:33:44:55:66    Sony WH-1000XM4

ahegazy0@kali:~$ l2ping AA:BB:CC:DD:EE:FF
ahegazy0@kali:~$ hcitool info AA:BB:CC:DD:EE:FF
ahegazy0@kali:~$ hcitool lescan
```

`hcitool scan` 查找可发现设备，`l2ping` 检查设备是否可达，`hcitool info` 获取设备信息，`lescan` 用于 Bluetooth Low Energy（BLE）设备。

## 无线侦察中要关注什么

- **OPN 开放网络**：无加密，流量可能被直接读取
- **WEP 网络**：加密已失效
- **弱密码的 WPA2 网络**：握手可被捕获并离线验证
- **隐藏 SSID**：名称不广播，但 BSSID 仍可能出现
- **处于可发现模式的 Bluetooth 设备**：尤其是本不应公开的设备

---

## 命令速查

| 命令 | 作用 |
|---|---|
| `iwconfig` | 查看无线接口和当前模式 |
| `iwlist wlan0 scan` | 扫描附近 Wi-Fi |
| `airmon-ng check kill` | 停止干扰监听模式的进程 |
| `airmon-ng start wlan0` | 启用监听模式 |
| `airmon-ng stop wlan0mon` | 关闭监听模式 |
| `airodump-ng wlan0mon` | 捕获数据包并显示网络 |
| `airodump-ng --bssid [MAC] --channel [CH] -w out wlan0mon` | 捕获指定网络 |
| `hcitool scan` | 扫描可发现的 Bluetooth 设备 |
| `hcitool lescan` | 扫描 BLE 设备 |
| `l2ping [MAC]` | Ping Bluetooth 设备 |
| `hcitool info [MAC]` | 获取设备详情 |

## 练习

- [ ] 在自己的网络上运行 `iwlist wlan0 scan`，记录信道和加密类型
- [ ] 在自己的设备附近运行 `hcitool scan`，观察可发现设备
- [ ] 只有在拥有兼容适配器且获得授权时，才运行监听模式和 `airodump-ng`

不要捕获不属于你的网络流量；在多数国家，即使不使用数据，未经授权的捕获也可能违法。

> 💡 *为了进行更深入的练习，也建议完成官方 **Linux Basics for Hackers** 书中每章末尾的练习。*
---

*下一篇：模块 15——Linux 内核与可加载内核模块*
