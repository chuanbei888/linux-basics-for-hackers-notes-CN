# 黑客 Linux 基础
## 模块 17：Python 脚本

---

## 概述

Bash 脚本适合简单自动化和快速单行命令；任务变复杂时，例如解析数据、处理网络、构建需要决策的工具，或阅读别人编写的漏洞利用代码，就应该使用 Python。本模块介绍安全工作所需的 Python 基础。

## 为什么安全工作常用 Python

- 语法清晰易读，可以快速理解脚本
- 标准库覆盖网络、文件 I/O、加密等功能
- 有大量面向安全任务的第三方库
- 许多漏洞利用、工具和概念验证代码都用 Python 编写
- Linux 支持 Python 运行

即使还不会编写工具，能够阅读 Python 也非常有用。看不懂漏洞脚本，就等于在盲目工作。

## Python 2 与 Python 3

Python 2 已于 2020 年正式停止支持。新代码应使用 Python 3，Kali 默认也提供 Python 3。旧教程中的 `print "hello"` 是 Python 2 语法，Python 3 应写成 `print("hello")`。

```bash
ahegazy0@kali:~$ python3 --version
ahegazy0@kali:~$ python3
```

第二条命令会进入交互式解释器，适合测试小片段；用 `exit()` 或 `Ctrl+D` 退出。

## 第一个脚本

创建 `hello.py`：

```python
#!/usr/bin/env python3

print("Hello, world")
```

`/usr/bin/env python3` 会查找系统中的 Python 3，比写死解释器路径更具可移植性。

```bash
ahegazy0@kali:~$ python3 hello.py
ahegazy0@kali:~$ chmod 755 hello.py
ahegazy0@kali:~$ ./hello.py
```

## 变量和数据类型

Python 是动态类型语言，不需要声明类型：

```python
name = "Alice"           # 字符串
port = 80                # 整数
pi = 3.14                # 浮点数
active = True            # 布尔值
```

字符串、整数、列表和字典示例：

```python
target = "192.168.1.1"
print("Scanning: " + target)
print(f"Scanning: {target}")    # f-string

port = 22
print(port + 1)    # 23

ports = [22, 80, 443, 3306]
print(ports[0])      # 22，索引从 0 开始
print(ports[-1])     # 3306，从末尾倒数

user = {"username": "admin", "password": "password123", "role": "root"}
print(user["username"])    # admin
```

字典是安全脚本中常见的数据结构，可保存解析结果、HTTP 请求头和配置值。

## 接收用户输入

```python
target = input("Enter target IP: ")
print("Scanning " + target)
port = int(input("Enter port: "))
```

`input()` 返回字符串；需要数字时用 `int()` 转换，否则字符串 `"80"` 不能直接与数字 1 相加。

## 条件语句

```python
password = input("Enter password: ")

if password == "secretpass":
    print("Access granted")
elif password == "admin":
    print("Admin access")
else:
    print("Wrong password")
```

Python 使用缩进而不是花括号定义代码块，标准缩进为 4 个空格。

| 运算符 | 含义 |
|---|---|
| `==` | 等于 |
| `!=` | 不等于 |
| `>` `<` | 大于/小于 |
| `>=` `<=` | 大于等于/小于等于 |
| `in` | 检查值是否存在于列表或字符串 |

## 循环

`for` 循环遍历列表或范围：

```python
ports = [22, 80, 443]
for port in ports:
    print(f"Checking port {port}")

for i in range(1, 256):
    print(f"192.168.1.{i}")
```

`range(1, 256)` 生成 1 到 255，可用于构建 IP 地址范围。

`while` 循环在条件为真时继续：

```python
attempts = 0
while attempts < 3:
    password = input("Password: ")
    if password == "secret":
        print("Access granted")
        break
    attempts += 1
print("Too many attempts")
```

`break` 立即退出循环，`continue` 跳到下一轮。

## 函数

函数让你只编写一次代码，再按名称调用：

```python
def scan_port(ip, port):
    print(f"Scanning {ip}:{port}")

scan_port("192.168.1.1", 80)
scan_port("192.168.1.1", 443)

def add(a, b):
    return a + b

result = add(3, 4)
print(result)    # 7
```

结构良好的脚本会把逻辑放进函数，在文件底部调用它们，便于阅读和复用。

## 导入库

```python
import socket
import os
import sys
```

`socket` 用于网络连接：

```python
import socket

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(("google.com", 80))
print("Connected")
s.close()
```

`os` 用于操作系统交互：

```python
import os

os.system("ls -la")           # 运行 shell 命令
cwd = os.getcwd()              # 获取当前目录
files = os.listdir(".")        # 列出当前目录文件
```

`sys` 可读取命令行参数：

```python
import sys

print(sys.argv)
# python3 script.py 192.168.1.1 80
# sys.argv = ['script.py', '192.168.1.1', '80']
```

## 安装第三方库

```bash
ahegazy0@kali:~$ pip3 install requests
ahegazy0@kali:~$ pip3 install scapy
ahegazy0@kali:~$ pip3 install paramiko
```

- `requests`：比直接使用 socket 更方便地发 HTTP 请求
- `scapy`：构造和分析数据包
- `paramiko`：在 Python 中建立 SSH 连接

使用示例：

```python
import requests

response = requests.get("http://example.com")
print(response.status_code)
print(response.text)
```

## 实例：基础端口扫描器

```python
#!/usr/bin/env python3
import socket
import sys

def scan_port(ip, port):
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.settimeout(1)
    result = s.connect_ex((ip, port))   # 连接成功返回 0
    s.close()
    return result == 0

target = input("Enter target IP: ")
print(f"\nScanning {target}...\n")

for port in range(1, 1025):
    if scan_port(target, port):
        print(f"Port {port} is OPEN")

print("\nScan complete.")
```

`connect_ex` 返回错误码而不是抛出异常，0 表示端口开放；`settimeout(1)` 让每个端口最多等待 1 秒。这是 nmap 的简化版本，只应扫描自己或得到明确授权的主机。

## 错误处理

网络连接会失败，文件可能不存在，用户也可能输入错误。使用 `try/except` 优雅处理：

```python
try:
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect(("192.168.1.1", 80))
    print("Connected")
except socket.error as e:
    print(f"Connection failed: {e}")
finally:
    s.close()
```

`try` 尝试执行，`except` 在失败时处理，`finally` 无论结果如何都会执行清理代码。

---

## 命令速查

| 命令 | 作用 |
|---|---|
| `python3 script.py` | 运行 Python 脚本 |
| `python3` | 打开交互式 Python |
| `pip3 install [package]` | 安装第三方库 |
| `pip3 list` | 查看已安装包 |
| `pip3 show [package]` | 查看软件包详情 |

## Python 速查表

| 概念 | 语法 |
|---|---|
| 输出 | `print("text")` |
| 变量 | `name = "value"` |
| 用户输入 | `x = input("prompt: ")` |
| 字符串格式化 | `f"Hello {name}"` |
| If/else | `if x == y:` / `else:` |
| For 循环 | `for item in list:` |
| While 循环 | `while condition:` |
| 函数 | `def name(params):` |
| 导入 | `import socket` |
| 列表 | `items = [1, 2, 3]` |
| 字典 | `d = {"key": "value"}` |
| Try/except | `try:` / `except Error as e:` |

## 练习

- [ ] 写一个询问姓名并用 f-string 打招呼的脚本
- [ ] 循环遍历 20–25 端口并打印每个端口
- [ ] 构建上面的端口扫描器，用 `127.0.0.1` 测试自己的机器
- [ ] 写一个询问密码并根据硬编码值输出正确/错误的脚本
- [ ] 为端口扫描器加入错误处理，确保网络错误不会中断整个扫描

> 💡 *为了进行更深入的练习，也建议完成官方 **Linux Basics for Hackers** 书中每章末尾的练习。*
---

## 后续方向

- **Scapy**：构造和发送自定义数据包，编写数据包级扫描器和嗅探器
- **Paramiko**：自动化 SSH 连接
- **Requests + BeautifulSoup**：网页抓取和 HTTP 交互
- **Subprocess**：从 Python 调用系统命令并获取输出
- 阅读现有漏洞利用代码：GitHub 上许多 CVE 概念验证都是 Python，能够阅读和修改它们是非常实用的技能

学习方法始终相同：理解概念、运行命令、在虚拟机中安全地实验，遇到问题再查资料。

---

*《Linux Basics for Hackers》结束——17 个模块全部完成。*
