# 黑客 Linux 基础
## 模块 15：内核与可加载内核模块

---

## 概述

内核是操作系统的核心，终端、程序和文件系统都运行在它之上。本模块介绍内核的职责、如何查看和调整设置，以及如何在不重启的情况下使用模块扩展内核功能。

## 内核究竟做什么

软件需要读文件、发网络数据或绘制屏幕时，不会直接操作硬件，而是请求内核完成。内核运行在**内核空间**，与程序所在的**用户空间**隔离；用户空间进程只能执行内核允许的操作，而内核可以访问任意内存和硬件，因此内核代码拥有系统最高权限。

![Linux 内核架构](assets/linux_kernel_architecture_1789213639316.jpg)

## 查看内核版本

```bash
ahegazy0@kali:~$ uname -a
Linux kali 6.1.0-kali9-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.27-1kali1 x86_64 GNU/Linux
```

- `Linux`：操作系统
- `kali`：主机名
- `6.1.0-kali9-amd64`：内核版本和架构
- `x86_64`：64 位内核

## 可加载内核模块（LKM）

每增加一种硬件都重新编译内核并不现实，因此 Linux 支持在运行期间插入和移除的内核代码块。驱动就是常见例子：插入 USB Wi-Fi 适配器时，内核加载驱动；拔出后可以移除模块。

模块运行在内核空间，拥有与内核相同的权限。恶意模块可能拦截系统调用、隐藏进程或文件，因此必须只从可信来源加载模块，并审计系统中的异常模块。

## 列出已加载模块

```bash
ahegazy0@kali:~$ lsmod
```

```
Module                  Size  Used by
bluetooth             663552  2 btusb
snd_hda_intel         57344  3
usbcore              286720  5 btusb,xhci_hcd
```

- `Module`：模块名
- `Size`：占用内存
- `Used by`：依赖它的模块或进程

## 查看、加载和移除模块

```bash
ahegazy0@kali:~$ modinfo bluetooth
ahegazy0@kali:~$ modprobe bluetooth
ahegazy0@kali:~$ modprobe -r bluetooth
```

`modinfo` 会显示文件路径、描述、作者、依赖和适用内核版本。`modprobe` 会自动处理依赖，`-r` 用于移除模块。

## sysctl：运行时调整内核设置

内核通过 `/proc/sys` 暴露许多参数，`sysctl` 可以读取和修改：

```bash
ahegazy0@kali:~$ sysctl -a
ahegazy0@kali:~$ sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 0
ahegazy0@kali:~$ sysctl -w net.ipv4.ip_forward=1
ahegazy0@kali:~$ sysctl -p
```

`net.ipv4.ip_forward` 控制系统是否转发网络包，默认关闭。`sysctl -w` 的修改只持续到重启；要永久保存，可在 `/etc/sysctl.conf` 写入：

```
net.ipv4.ip_forward = 1
```

错误的内核参数可能造成网络故障或系统不稳定，修改前必须确认含义，并在授权实验环境中操作。

## /proc 文件系统

`/proc` 是只存在于内存中的虚拟文件系统，内核把自身和进程信息以文件形式暴露出来：

```bash
ahegazy0@kali:~$ ls /proc
ahegazy0@kali:~$ cat /proc/cpuinfo
ahegazy0@kali:~$ cat /proc/meminfo
ahegazy0@kali:~$ cat /proc/version
```

其中的数字目录对应进程 PID，`cpuinfo`、`meminfo` 等文件由内核实时生成，并非磁盘上的普通文件。

---

## 命令速查

| 命令 | 作用 |
|---|---|
| `uname -a` | 显示完整内核版本和架构 |
| `lsmod` | 列出已加载模块 |
| `modinfo [name]` | 查看模块详情 |
| `modprobe [name]` | 加载模块及依赖 |
| `modprobe -r [name]` | 移除模块 |
| `sysctl -a` | 显示全部内核参数 |
| `sysctl [parameter]` | 查看指定参数 |
| `sysctl -w [param=value]` | 运行时修改参数 |
| `sysctl -p` | 应用 `/etc/sysctl.conf` |
| `cat /proc/cpuinfo` | 查看 CPU 信息 |
| `cat /proc/meminfo` | 查看内存信息 |

## 练习

- [ ] 运行 `uname -a`，识别内核版本和 32/64 位架构
- [ ] 运行 `lsmod`，大致统计加载的模块数量
- [ ] 运行 `modinfo bluetooth`，查看 `depends` 字段
- [ ] 运行 `sysctl net.ipv4.ip_forward`，记录当前值并解释修改影响

> 💡 *为了进行更深入的练习，也建议完成官方 **Linux Basics for Hackers** 书中每章末尾的练习。*
---

*下一篇：模块 16——自动化与计划任务*
