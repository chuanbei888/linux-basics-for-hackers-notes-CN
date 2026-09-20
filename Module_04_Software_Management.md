# 黑客 Linux 基础
## 模块 4：软件管理

---

## 概述

Kali 已经预装了许多工具，但你仍会经常需要添加新工具、更新现有工具，或直接从开发者的 GitHub 获取项目。本模块介绍 Linux 如何安装软件，以及如何获取标准仓库中没有的工具。

## Linux 如何管理软件

Windows 通常是从网站下载安装程序、双击并按向导操作。Linux 使用的是**软件仓库**：由发行版维护和验证的大型在线软件库。你只需告诉系统软件名称，它就会自动查找、下载、安装，并同时处理依赖项。

Kali（以及基于 Debian 的系统）使用 `apt-get` 管理软件。仓库地址保存在 `/etc/apt/sources.list` 中。除非清楚自己在添加什么，否则不要修改它，因为非官方仓库可能包含恶意软件包。

---

## 基本命令

### apt-get update

安装前先运行此命令。它不会安装或升级软件，只会刷新本地可用软件包列表。跳过这一步可能安装旧版本或遇到错误。

```bash
ahegazy0@kali:~$ apt-get update
```

养成每次安装前先运行的习惯。

### apt-get install

下载并安装软件包，Kali 会自动处理依赖：

```bash
ahegazy0@kali:~$ apt-get install wireshark
ahegazy0@kali:~$ apt-get install -y wireshark
```

`-y` 会自动确认安装。

### apt-get remove

删除已安装的软件包。`remove` 会保留配置文件，`purge` 会连配置一起删除：

```bash
ahegazy0@kali:~$ apt-get remove wireshark
ahegazy0@kali:~$ apt-get purge wireshark
```

### apt-get upgrade

将所有已安装软件包升级到最新版本：

```bash
ahegazy0@kali:~$ apt-get upgrade
```

升级前始终先运行 `apt-get update`。

### apt-cache search

在本地仓库数据库中按关键词搜索软件包：

```bash
ahegazy0@kali:~$ apt-cache search wifi
ahegazy0@kali:~$ apt-cache search wireless
ahegazy0@kali:~$ apt-cache search password crack
```

### git clone

许多最新的黑客工具不在软件仓库中，而是托管在 GitHub。`git clone` 会将完整项目下载到本机：

```bash
ahegazy0@kali:~$ git clone https://github.com/username/toolname
ahegazy0@kali:~$ git clone https://github.com/example/tool
ahegazy0@kali:~$ cd tool
```

克隆后先阅读项目的 README，通常会给出 `pip install -r requirements.txt` 或直接运行 Python 脚本等安装步骤。

> 很多最新工具只存在于 GitHub。尽早学会 `git clone`，就不会受限于官方仓库中的软件。

---

## 关于 sources.list

`/etc/apt/sources.list` 包含系统下载软件的仓库地址：

```bash
ahegazy0@kali:~$ cat /etc/apt/sources.list
```

网上有时会建议你向其中添加第三方仓库。请谨慎处理：官方 Kali 仓库经过维护和验证，随机第三方仓库则不一定可靠。添加错误的软件源很容易安装不该安装的内容。

---

## 命令速查

| 操作 | 命令 |
|---|---|
| 刷新软件包列表 | `apt-get update` |
| 安装软件包 | `apt-get install [name]` |
| 安装时自动确认 | `apt-get install -y [name]` |
| 删除软件包 | `apt-get remove [name]` |
| 连同配置一起删除 | `apt-get purge [name]` |
| 更新所有软件包 | `apt-get upgrade` |
| 搜索软件包 | `apt-cache search [keyword]` |
| 从 GitHub 克隆项目 | `git clone [url]` |

---

## 练习

- [ ] 运行 `apt-get update` 刷新软件包列表
- [ ] 使用 `apt-cache search wifi` 查看搜索结果
- [ ] 从结果中选一个工具，用 `apt-get install` 安装
- [ ] 在 GitHub 找一个简单工具，使用 `git clone` 下载到本机

> 💡 *为了进行更深入的练习，也建议完成官方 **Linux Basics for Hackers** 书中每章末尾的练习。*
---

*下一篇：模块 5——文件和目录权限控制*
