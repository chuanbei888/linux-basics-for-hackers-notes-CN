# 黑客 Linux 基础
## 模块 8：Bash 脚本

---

## 概述

重复输入相同命令是在浪费时间。脚本可以把命令写入文件，之后直接运行文件。本模块从简单的“Hello World”开始，介绍如何编写会接收输入并完成实际工作的 Bash 脚本。

## 什么是脚本

脚本是包含一系列命令的纯文本文件。运行它时，Bash 会按顺序读取并执行每一行，就像你手动输入一样，只是速度更快，也不必重复输入。终端中能运行的命令（`ls`、`ping`、`nmap`、`grep` 等）都可以放入脚本，还能用逻辑判断组合它们。

## shebang 行

每个 Bash 脚本的第一行都应是：

```bash
#!/bin/bash
```

这叫 **shebang**（或 hashbang），告诉系统用哪个程序解释文件。必须放在第一行，前面不能有空行。

## 创建第一个脚本

创建 `myscript.sh`：

```bash
#!/bin/bash
echo "Hello, world"
```

赋予执行权限并运行：

```bash
ahegazy0@kali:~$ chmod 755 myscript.sh
ahegazy0@kali:~$ ./myscript.sh
Hello, world
```

`./` 表示从当前目录运行文件，因为当前目录通常不在 PATH 中。

## echo：向屏幕输出

```bash
ahegazy0@kali:~$ echo "Scan starting..."
ahegazy0@kali:~$ echo "Done."
ahegazy0@kali:~$ echo "Your username is: $USER"
Your username is: kali
```

## 脚本中的变量

变量用于保存和复用数据，赋值时等号两边不能有空格：

```bash
ahegazy0@kali:~$ name="Bob"
ahegazy0@kali:~$ echo "Hello, $name"
Hello, Bob
```

引用变量值时在名称前加 `$`，赋值时不要加。

## read：接收用户输入

`read` 会暂停脚本，等待用户输入并保存到变量：

```bash
#!/bin/bash
echo "What is your name?"
read name
echo "Hello, $name"
```

也可以用 `-p` 在一行中显示提示：

```bash
ahegazy0@kali:~$ read -p "Enter your name: " name
```

## 实例：简单的 ping 扫描器

```bash
#!/bin/bash
read -p "Enter IP address to ping: " target
echo "Pinging $target..."
ping -c 4 $target
```

`-c 4` 让 ping 只发送 4 个数据包。保存为 `pinger.sh`，运行 `chmod 755 pinger.sh` 后执行 `./pinger.sh`。

## 注释

以 `#` 开头的行是注释，Bash 会忽略它们：

```bash
#!/bin/bash
# This script pings a target IP address
# Written for practice - Module 8

read -p "Enter target IP: " target
ping -c 4 $target   # send 4 packets only
```

注释用于帮助未来的自己和其他读者理解代码。

## 条件逻辑：if/else

```bash
#!/bin/bash
read -p "Enter a number: " num

if [ $num -gt 10 ]; then
echo "That number is greater than 10"
else
echo "That number is 10 or less"
fi
```

`fi` 用于结束 `if` 块，`-gt` 表示“大于”。

| 运算符 | 含义 |
|---|---|
| `-eq` | 等于 |
| `-ne` | 不等于 |
| `-gt` | 大于 |
| `-lt` | 小于 |
| `-ge` | 大于或等于 |
| `-le` | 小于或等于 |

比较字符串时，在方括号内使用 `=` 和 `!=`。

## 循环：重复执行

```bash
#!/bin/bash
for i in 1 2 3 4 5; do
echo "Pinging 192.168.1.$i"
ping -c 1 192.168.1.$i
done
```

这会依次 ping 五个 IP 地址，是一个简易网络扫描器。nmap 的原理类似，只是功能更丰富。

## 实用脚本的推荐结构

```bash
#!/bin/bash
# Script name and what it does
# Your name, date

# --- Variables ---
target=""

# --- Input ---
read -p "Enter target: " target

# --- Main logic ---
echo "Running scan on $target..."
nmap -sV $target

# --- Done ---
echo "Scan complete."
```

按“变量、输入、主要逻辑、完成”组织内容，并为不明显的部分写注释，脚本会更易读。

## 文件权限回顾：chmod

脚本创建时默认只是普通文本文件，必须添加执行权限：

```bash
ahegazy0@kali:~$ chmod 755 myscript.sh
```

- `7`：所有者可读、可写、可执行
- `5`：组可读、可执行
- `5`：其他人可读、可执行

只供自己运行的脚本可以使用 `chmod 700`。

---

## 命令速查

| 命令/概念 | 作用 |
|---|---|
| `#!/bin/bash` | shebang，必须是脚本第一行 |
| `echo "text"` | 向屏幕输出文字 |
| `read var` | 接收输入并保存到变量 |
| `read -p "prompt" var` | 接收输入并显示提示 |
| `name="value"` | 给变量赋值 |
| `$name` | 使用变量值 |
| `chmod 755 file.sh` | 使脚本可执行 |
| `./script.sh` | 运行当前目录中的脚本 |
| `# comment` | Bash 忽略的注释行 |
| `if [ ] then / fi` | 条件逻辑 |
| `for x in ... do / done` | 遍历列表的循环 |

---

## 练习

- [ ] 写一个询问姓名并输出 “Hello, [name]” 的脚本
- [ ] 用 `chmod 755` 添加执行权限并用 `./` 运行
- [ ] 写第二个脚本，询问 IP 地址并执行 `ping -c 4`
- [ ] 修改它，循环 ping `192.168.1.1` 到 `192.168.1.5`

认真完成 ping 循环练习。这是实际会用到的技巧，也能帮助你理解 nmap 输出。

> 💡 *为了进行更深入的练习，也建议完成官方 **Linux Basics for Hackers** 书中每章末尾的练习。*
---

*下一篇：模块 9——归档与压缩*
