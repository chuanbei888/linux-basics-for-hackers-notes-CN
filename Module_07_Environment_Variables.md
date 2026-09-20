# 黑客 Linux 基础
## 模块 7：环境变量

---

## 概述

Linux 依靠一组后台设置控制系统行为：去哪里查找程序、当前用户是谁、主目录在哪里、终端提示符长什么样。这些设置叫作**环境变量**。它们一直存在，只是你还没有查看过。

## 环境变量究竟是什么

环境变量就是用户登录期间由系统保存在内存中的命名值。程序和 shell 会不断读取它们来决定行为。变量名通常使用大写字母，引用时在名称前加 `$`：

```bash
ahegazy0@kali:~$ echo $HOME
/home/kali
```

---

## 需要认识的重要变量

| 变量 | 保存的内容 |
|---|---|
| `$PATH` | 输入命令时系统搜索的文件夹列表 |
| `$HOME` | 主目录路径 |
| `$USER` | 当前用户名 |
| `$SHELL` | 正在运行的 shell（通常是 /bin/bash） |
| `$HISTSIZE` | 历史文件保存的命令数量 |
| `$PS1` | 命令提示符的样式 |

---

## PATH：最重要的变量

`$PATH` 是由冒号分隔的一组文件夹路径。输入 `ls` 时，系统会逐个搜索 PATH 中的文件夹，直到找到同名程序：

```bash
ahegazy0@kali:~$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

因此，安装工具后若其目录不在 PATH 中，就会出现“command not found”。修复方法是把目录加入 PATH。

**不要破坏 PATH。** 如果覆盖了原有值，系统会找不到几乎所有命令。编辑时务必小心。

## 查看全部环境变量

```bash
ahegazy0@kali:~$ env
```

这会打印当前会话设置的全部环境变量。

## 修改和创建变量

只对当前终端会话创建或修改变量：

```bash
ahegazy0@kali:~$ MYVAR="hello"
ahegazy0@kali:~$ echo $MYVAR
hello
```

关闭终端后这种变量会消失。使用 `export` 后，当前终端启动的子进程也能读取它：

```bash
ahegazy0@kali:~$ export MYVAR="hello"
ahegazy0@kali:~$ export PATH=$PATH:/new/folder/here
```

开头的 `$PATH` 会保留已有目录，只在末尾追加新路径。忘记它会覆盖整个 PATH。

## 让修改永久生效

`export` 只对当前会话有效。要跨重启和新终端保留设置，把它写入 Bash 配置文件 `~/.bashrc`：

```bash
ahegazy0@kali:~$ export PATH=$PATH:/your/new/tool/folder
ahegazy0@kali:~$ source ~/.bashrc
```

`source` 会重新加载配置，不需要重启终端。

## 修改提示符：PS1

`$PS1` 控制命令提示符的显示方式：

```bash
ahegazy0@kali:~$ export PS1="Hacker-Level-99: # "
```

提示符会变成：

```
Hacker-Level-99: # 
```

它主要是外观设置，也可以配置为显示当前目录、Git 分支或时间。

## 黑客使用的历史记录技巧

`$HISTSIZE` 控制历史文件保存多少条命令。设为 0 就不会保存：

```bash
ahegazy0@kali:~$ export HISTSIZE=0
```

这是一种基础的操作安全措施，可以减少命令痕迹。但在实际系统中应遵守授权和审计要求，不能把它当作消除日志的手段。

---

## 命令速查

| 命令 | 作用 |
|---|---|
| `env` | 打印当前所有环境变量 |
| `echo $VAR` | 打印指定变量的值 |
| `MYVAR="value"` | 创建当前会话变量 |
| `export MYVAR="value"` | 创建对子进程可见的变量 |
| `export PATH=$PATH:/folder` | 在不破坏 PATH 的前提下追加目录 |
| `source ~/.bashrc` | 不重启终端就重新加载 Bash 配置 |

---

## 练习

- [ ] 运行 `env` 找到 `$SHELL`，确认当前 shell
- [ ] 运行 `echo $PATH` 查看系统搜索程序的所有目录
- [ ] 尝试 `export HISTSIZE=0`，再按向上箭头观察本会话历史
- [ ] 修改 PS1 只显示你的名字或自定义内容，观察提示符即时变化

> 💡 *为了进行更深入的练习，也建议完成官方 **Linux Basics for Hackers** 书中每章末尾的练习。*
---

*下一篇：模块 8——Bash 脚本*
