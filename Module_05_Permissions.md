---
layout: default
title: 模块 5：权限与特权
description: 读取权限、使用 chmod 和识别 SUID 风险。
permalink: /Module_05_Permissions.html
---

# 黑客 Linux 基础
## 模块 5：权限与特权

---

## 概述

Linux 是多用户系统。同一台机器上可以有多个账户，不是每个人都应该能读取、修改或运行所有内容。Linux 通过权限实现这种控制。本模块介绍如何读取和修改权限，以及权限何时会变成安全问题。

## 权限如何工作

Linux 中每个文件和文件夹都有三类权限主体：

- **所有者**：创建文件的用户
- **组**：共享访问权限的用户集合
- **其他人**：系统中的所有其他用户

每类主体都有三种权限：

| 权限 | 符号 | 含义 |
|---|---|---|
| 读取 | `r` | 查看文件内容 |
| 写入 | `w` | 修改或删除文件 |
| 执行 | `x` | 将文件作为程序运行 |

![Linux 文件权限](assets/file_permissions_diagram_1789213222578.jpg)

运行 `ls -l` 时，第一列显示每个文件的权限：

```bash
ahegazy0@kali:~$ ls -l
-rwxr-xr--  1  kali  kali  4096  Jan 1  file.sh
```

开头的字符就是权限块：

```
- rwx r-x r--
│  │   │   │
│  │   │   └── 其他人：只读
│  │   └────── 组：读取 + 执行
│  └────────── 所有者：读取 + 写入 + 执行
└───────────── 文件类型：- 表示普通文件，d 表示目录
```

下载的工具无法运行时，通常就是没有设置执行权限。

---

## chmod：修改权限

`chmod` 代表 change mode，用于设置谁可以对文件做什么。常见有数字法和符号法两种。

### 数字法

| 权限 | 数值 |
|---|---|
| 读取（r） | 4 |
| 写入（w） | 2 |
| 执行（x） | 1 |

将三类权限的数值相加，再按所有者、组、其他人的顺序写成三位数字：

| 数字 | 计算 | 权限 |
|---|---|---|
| 7 | 4+2+1 | 读取 + 写入 + 执行 |
| 6 | 4+2 | 读取 + 写入 |
| 5 | 4+1 | 读取 + 执行 |
| 4 | 4 | 只读 |
| 0 | 0 | 无权限 |

```bash
ahegazy0@kali:~$ chmod 755 script.sh
ahegazy0@kali:~$ chmod 644 notes.txt
ahegazy0@kali:~$ chmod 777 file.sh
```

`755` 让所有者拥有全部权限，组和其他人拥有读取与执行权限；`644` 是普通文本文件常见设置。

> **注意：** `chmod 777` 虽然能让脚本运行，但系统中的任何人都可以修改它。在渗透测试考试或真实环境中，盲目把敏感文件设为 777 是严重的安全错误。优先使用 `chmod +x`。

### 符号法

```bash
ahegazy0@kali:~$ chmod +x script.sh
ahegazy0@kali:~$ chmod u+x script.sh
ahegazy0@kali:~$ chmod g-w file.txt
```

`u` 表示所有者，`g` 表示组，`o` 表示其他人，`a` 表示三者全部。

---

## chown：修改所有权

`chown` 代表 change owner，用于重新指定文件所有者：

```bash
ahegazy0@kali:~$ chown kali file.txt
ahegazy0@kali:~$ chown kali:kali file.txt
ahegazy0@kali:~$ chown -R kali /home/kali/tools
```

`-R` 表示递归修改文件夹及其全部内容。修改不属于自己的文件所有权需要 root 权限。

---

## SUID：必须掌握的特殊权限

SUID（Set User ID）会改变文件的执行方式。通常程序以启动它的用户权限运行；如果 root 所有的文件设置了 SUID，它即使由普通用户运行，也会以 root 权限执行。

权限字符串中，SUID 会在所有者的执行位显示为 `s`：

```
-rwsr-xr-x   root   root   /usr/bin/passwd
```

`passwd` 需要 SUID，因为任何用户修改密码时都必须写入 root 所有的 `/etc/shadow`。SUID 本身是合法功能，但配置错误或编写不当的 SUID 程序可能被利用。

这就是**权限提升**：攻击者利用权限错误，从受限账户提升到 root。查找配置不当的 SUID 二进制文件是经典方法之一。

```bash
ahegazy0@kali:~$ find / -perm -u=s -type f 2>/dev/null
```

- `find /`：从根目录开始搜索
- `-perm -u=s`：查找设置了 SUID 的文件
- `-type f`：只查找文件，不查目录
- `2>/dev/null`：隐藏权限错误

在 Linux 系统上运行它并检查结果；有已知漏洞或错误配置的文件都可能成为提权路径。

---

## 关于权限提升

系统经常先以低权限用户身份被攻破，攻击者随后寻找获得 root 的方法。权限配置错误是最常见的途径之一：不应设置 SUID 的二进制文件、被错误用户写入的文件、root 所有但所有人可编辑的脚本等。

无论你是在攻击还是防御，理解权限都是发现这些问题的基础。

---

## 命令速查

| 命令 | 作用 | 示例 |
|---|---|---|
| `ls -l` | 显示文件权限 | `ls -l` |
| `chmod 755 file` | 用数字设置权限 | `chmod 755 script.sh` |
| `chmod +x file` | 给所有人添加执行权限 | `chmod +x tool.py` |
| `chmod u+x file` | 只给所有者添加执行权限 | `chmod u+x run.sh` |
| `chown user file` | 修改文件所有者 | `chown kali file.txt` |
| `chown user:group file` | 同时修改所有者和组 | `chown kali:kali file` |
| `chown -R user dir` | 递归修改所有权 | `chown -R kali /tools` |
| `find / -perm -u=s -type f 2>/dev/null` | 查找所有 SUID 文件 | - |

---

## 练习

- [ ] 使用 `touch test.txt` 创建文件，再运行 `ls -l` 读取默认权限
- [ ] 使用 `chmod 600 test.txt`，确认只有自己能读写
- [ ] 创建简单脚本，先直接运行观察失败，再用 `chmod +x` 运行
- [ ] 运行上面的 `find` 命令列出系统中的 SUID 文件并检查结果

> 💡 *为了进行更深入的练习，也建议完成官方 **Linux Basics for Hackers** 书中每章末尾的练习。*
---

*下一篇：模块 6——进程管理*
