# 黑客 Linux 基础
## 模块 9：归档与压缩

---

## 概述

有时需要把许多文件作为一个包移动，有时需要在传输前缩小大文件；在取证或严肃侦察中，还可能需要制作整块磁盘的逐比特副本。本模块介绍三类操作：用 `tar` 归档、用 `gzip` 和 `bzip2` 压缩，以及用 `dd` 制作底层磁盘镜像。

## 归档和压缩不是一回事

**归档**是把多个文件组合成一个文件，不会减小体积，Linux 中使用 `tar`。**压缩**是通过更高效的编码让文件变小，使用 `gzip` 或 `bzip2`，通常作用于单个文件。

实践中常常先归档再压缩，因此会看到 `.tar.gz`：`.tar` 表示已归档，`.gz` 表示之后使用 gzip 压缩。

---

## tar：打包文件

`tar` 是 Tape Archive（磁带归档）的缩写，名称源于磁带备份，但工具至今仍非常常用。

创建归档：

```bash
ahegazy0@kali:~$ tar -cvf archive.tar file1.txt file2.txt file3.txt
```

- `-c`：创建归档
- `-v`：显示正在添加的内容
- `-f`：后一个参数是归档文件名

解包与查看内容：

```bash
ahegazy0@kali:~$ tar -xvf archive.tar
ahegazy0@kali:~$ tar -tvf archive.tar
```

`-x` 表示解包，`-t` 表示列出内容。查看时不需要实际解压。

## gzip 和 gunzip：缩小文件

`gzip` 会原地压缩单个文件，原文件被 `.gz` 文件替代：

```bash
ahegazy0@kali:~$ gzip archive.tar
ahegazy0@kali:~$ gunzip archive.tar.gz
ahegazy0@kali:~$ gzip -d archive.tar.gz
```

`gunzip` 和 `gzip -d` 都会恢复原文件。

## 一次完成归档和压缩：tar.gz

`-z` 表示同时使用 gzip：

```bash
ahegazy0@kali:~$ tar -cvzf archive.tar.gz file1.txt file2.txt file3.txt
ahegazy0@kali:~$ tar -xvzf archive.tar.gz
```

这是最常见的格式，“tar 包”通常指 `.tar.gz` 文件。

## bzip2：gzip 的另一种选择

`bzip2` 往往比 gzip 生成更小的文件，但压缩和解压更慢：

| | gzip | bzip2 |
|---|---|---|
| 速度 | 更快 | 更慢 |
| 压缩率 | 良好 | 更高 |
| 扩展名 | `.gz` | `.bz2` |
| tar 选项 | `-z` | `-j` |

```bash
ahegazy0@kali:~$ tar -cvjf archive.tar.bz2 files/
ahegazy0@kali:~$ tar -xvjf archive.tar.bz2
```

## dd：底层磁盘复制

`dd` 直接在原始磁盘层逐块复制数据，不关心文件系统结构：

```bash
ahegazy0@kali:~$ dd if=/dev/sda of=/dev/sdb
```

- `if`：输入文件，即源
- `of`：输出文件，即目标

这会把 `/dev/sda` 的每一位复制到 `/dev/sdb`，包括已删除文件和文件系统元数据，目标盘会成为源盘的精确克隆。也可以制作镜像文件：

```bash
ahegazy0@kali:~$ dd if=/dev/sda of=disk_image.img
ahegazy0@kali:~$ dd if=/dev/sda of=disk_image.img status=progress
```

`status=progress` 显示进度。

> **严重警告：** 如果误换 `if` 和 `of`，或指定了错误磁盘，`dd` 会直接覆盖真实磁盘，不会询问确认，也无法撤销。运行前至少三次核对输入和输出路径。

```bash
ahegazy0@kali:~$ dd if=/dev/sdb of=/dev/sda    ← 用 sdb 内容覆盖主磁盘
```

## 数字取证中的用途

调查人员通常先制作原始磁盘的 `dd` 镜像，再在镜像上分析，避免改动证据。由于按块复制，它能保留文件、已删除文件、文件片段、文件系统结构和空闲空间，Autopsy、Sleuth Kit 等工具可以进一步分析。

---

## 命令速查

| 命令 | 作用 |
|---|---|
| `tar -cvf archive.tar files` | 创建 tar 归档 |
| `tar -xvf archive.tar` | 解包 tar 归档 |
| `tar -tvf archive.tar` | 列出 tar 内容 |
| `tar -cvzf archive.tar.gz files` | 创建 gzip 压缩归档 |
| `tar -xvzf archive.tar.gz` | 解压 gzip 归档 |
| `tar -cvjf archive.tar.bz2 files` | 创建 bzip2 压缩归档 |
| `tar -xvjf archive.tar.bz2` | 解压 bzip2 归档 |
| `gzip filename` | 使用 gzip 压缩文件 |
| `gunzip filename.gz` | 解压 `.gz` 文件 |
| `dd if=source of=destination` | 复制原始磁盘块 |
| `dd if=/dev/sda of=image.img status=progress` | 制作带进度显示的镜像 |

| 选项 | 含义 |
|---|---|
| `-c` | 创建归档 |
| `-x` | 从归档中解包 |
| `-t` | 列出内容 |
| `-v` | 详细输出 |
| `-f` | 指定文件名 |
| `-z` | 使用 gzip |
| `-j` | 使用 bzip2 |

---

## 练习

- [ ] 用 `touch file1.txt file2.txt file3.txt` 创建三个空文件
- [ ] 用 `tar -cvf bundle.tar file1.txt file2.txt file3.txt` 打包
- [ ] 运行 `gzip bundle.tar`，再用 `ls -lh` 比较大小
- [ ] 用 `tar -xvzf bundle.tar.gz` 解压并确认文件存在
- [ ] 使用 bzip2 重复实验，比较 `.tar.gz` 和 `.tar.bz2` 的大小

> 💡 *为了进行更深入的练习，也建议完成官方 **Linux Basics for Hackers** 书中每章末尾的练习。*
---

*下一篇：模块 10——文件系统与存储设备*
