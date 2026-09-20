# 黑客 Linux 基础
## 模块 2：文本处理

---

## 概述

在 Linux 中，系统设置、日志、配置和用户数据几乎都以文本文件存在。掌握搜索和处理文本的方法后，你可以从数千行输出中在几秒内找出需要的内容。

## 这对黑客工作为什么重要

扫描工具通常会输出成千上万行内容，而不是一份整洁的摘要。你可以使用文本工具筛选结果，例如在扫描网络后用 `grep` 找出包含 “open” 的行，从而快速定位开放端口。`/etc` 下的系统配置也大多是纯文本文件，读写这些文件就能重新配置系统。

---

## 基本命令

### cat

`cat` 是 concatenate（连接）的缩写，是读取文件最直接的方式：它会把全部内容输出到屏幕。

```bash
ahegazy0@kali:~$ cat /etc/passwd
```

> **注意：** 不要对编译后的二进制文件运行 `cat`（例如 `cat /bin/ls`），否则原始字节会把终端弄得一团糟。若发生这种情况，盲打 `reset` 并按回车即可恢复。

也可以快速创建小文件：

```bash
ahegazy0@kali:~$ cat > targets.txt
```

逐行输入内容，完成后按 `Ctrl+D` 保存并退出。若要追加而不是覆盖，使用 `>>`：

```bash
ahegazy0@kali:~$ cat >> targets.txt
```

`>` 会覆盖，`>>` 会追加，这个区别非常重要。

### grep

`grep` 会搜索文件，只返回包含指定文字或模式的行。

```bash
ahegazy0@kali:~$ grep "password" logs.txt
```

常用选项：

| 选项 | 作用 |
|---|---|
| `-i` | 忽略大小写（同时匹配 `Password`、`PASSWORD` 等） |
| `-r` | 递归搜索文件夹中的所有文件 |
| `-n` | 同时显示行号 |
| `-v` | 反向匹配，显示不包含目标文字的行 |

示例：

```bash
ahegazy0@kali:~$ grep -i "admin" access.log
```

这会同时找出 `admin`、`Admin` 和 `ADMIN`。

### head 和 tail

文件很大、只想查看一小部分时，可以使用：

```bash
ahegazy0@kali:~$ head /etc/snort/snort.conf
ahegazy0@kali:~$ tail /var/log/syslog
```

默认分别显示前 10 行和后 10 行，也可以指定行数：

```bash
ahegazy0@kali:~$ head -n 20 file.txt     前 20 行
ahegazy0@kali:~$ tail -n 20 file.txt     后 20 行
```

`tail -f` 会实时跟踪文件新增内容，适合观察正在写入的日志：

```bash
ahegazy0@kali:~$ tail -f /var/log/syslog
```

### nl

给输出添加行号，便于引用具体行：

```bash
ahegazy0@kali:~$ nl /etc/snort/snort.conf
```

### less

文件太长时，不要用 `cat` 让内容一闪而过，使用 `less` 逐页查看：

```bash
ahegazy0@kali:~$ less /etc/snort/snort.conf
```

- `Space`：下一页
- `b`：上一页
- `/word`：搜索文字
- `q`：退出

### sed

`sed` 是流编辑器，可查找并替换文本：

```bash
ahegazy0@kali:~$ sed 's/mysql/MySQL/g' config.txt
```

其中 `s/` 表示替换，`mysql` 是查找内容，`/MySQL/` 是替换内容，`g` 表示替换每一处而不是只替换第一处。默认情况下 `sed` 只打印修改后的结果，不会修改文件。加上 `-i` 才会写回文件：

```bash
ahegazy0@kali:~$ cp config.txt config.txt.bak
ahegazy0@kali:~$ sed -i 's/old/new/g' config.txt
```

> 使用 `-i` 前先备份。它会永久修改文件，不能撤销。

---

## 管道：连接多个命令

管道符 `|` 会把一个命令的输出直接交给另一个命令。这是组合命令的关键能力。

![Linux 管道示意图](assets/linux_pipe_diagram_1789213616381.jpg)

```bash
ahegazy0@kali:~$ cat access.log | grep "failed"
ahegazy0@kali:~$ cat access.log | grep "failed" | tail -n 20
```

第二条命令先读取日志，再筛选 `failed`，最后只显示筛选结果中的最后 20 行。多个命令可以串成一条流水线。

---

## 命令速查

| 命令 | 作用 | 示例 |
|---|---|---|
| `cat file` | 将整个文件打印到屏幕 | `cat /etc/passwd` |
| `cat > file` | 创建文件并输入内容 | `cat > notes.txt` |
| `grep "word" file` | 查找包含指定文字的行 | `grep "root" passwd` |
| `grep -i` | 忽略大小写搜索 | `grep -i "admin" log` |
| `head -n 20 file` | 查看前 20 行 | `head -n 20 file.txt` |
| `tail -n 20 file` | 查看后 20 行 | `tail -n 20 file.txt` |
| `tail -f file` | 实时跟踪文件 | `tail -f syslog` |
| `nl file` | 显示带行号的文件 | `nl config.conf` |
| `less file` | 分页浏览文件 | `less bigfile.txt` |
| `sed 's/a/b/g' file` | 将所有 `a` 替换为 `b` | `sed 's/old/new/g' f` |
| `cmd1 \| cmd2` | 将 cmd1 输出交给 cmd2 | `cat log \| grep fail` |

---

## 练习

- [ ] 进入 `/etc/snort/`，用 `less` 打开 `snort.conf`，熟悉配置文件的样子
- [ ] 运行 `grep "output" /etc/snort/snort.conf`，观察返回的行
- [ ] 使用 `cat > targets.txt` 输入三个 IP 地址（每行一个），按 `Ctrl+D` 后用 `cat targets.txt` 读回
- [ ] 尝试组合：`cat /etc/snort/snort.conf | grep "output" | nl`

> 💡 *为了进行更深入的练习，也建议完成官方 **Linux Basics for Hackers** 书中每章末尾的练习。*
---

*下一篇：模块 3——网络管理*
