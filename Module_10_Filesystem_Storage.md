# 黑客 Linux 基础
## 模块 10：文件系统与存储

---

## 概述

Linux 对存储设备的处理方式与 Windows 完全不同：没有盘符，插入设备时也不会自动弹出窗口。一切都是文件，每个设备都位于文件系统树中的某处；要使用磁盘，必须显式挂载。本模块解释这些机制。

## 一切都是文件

“一切都是文件”在 Linux 中不是比喻，而是系统的实际设计。硬盘、键盘和网卡都以特殊文件形式存在，设备文件位于 `/dev`，内核知道如何与它们通信。

| 设备 | 含义 |
|---|---|
| `/dev/sda` | 第一块硬盘 |
| `/dev/sdb` | 第二块硬盘 |
| `/dev/sdc` | 第三块硬盘（或 USB） |
| `/dev/sda1` | 第一块硬盘的第一个分区 |
| `/dev/sda2` | 第一块硬盘的第二个分区 |
| `/dev/sr0` | CD/DVD 光驱 |

`sd` 源于 SCSI disk，现代 SATA 和 USB 磁盘也沿用该命名。字母表示磁盘顺序（`a` 第一块，`b` 第二块），数字表示分区号。因此 `/dev/sdb3` 就是第二块磁盘的第三个分区。

## 挂载：Linux 如何连接磁盘

Windows 会给 USB 分配 `D:` 等盘符，Linux 则要手动**挂载**。挂载就是把设备连接到文件树中的某个文件夹，该文件夹叫**挂载点**。

```
/
└── mnt/
    └── usb/    ← 挂载后 USB 出现在这里
```

临时手动挂载通常使用 `/mnt`，桌面环境自动挂载设备时常用 `/media`。

```bash
ahegazy0@kali:~$ mkdir /mnt/usb
ahegazy0@kali:~$ mount /dev/sdb1 /mnt/usb
ahegazy0@kali:~$ umount /mnt/usb
```

注意命令拼写是 `umount` 而不是 `unmount`。拔出设备前必须先卸载，否则可能损坏文件系统。

## fdisk：查看磁盘

`fdisk -l` 会列出所有磁盘和分区，需要 root 权限：

```bash
ahegazy0@kali:~$ fdisk -l
```

示例输出：

```
Disk /dev/sda: 500 GB, 500107862016 bytes
Device     Boot    Start      End  Sectors  Size  Type
/dev/sda1  *        2048  1026047  1024000  500M  EFI System
/dev/sda2        1026048 97654784 96628737 46.1G  Linux filesystem

Disk /dev/sdb: 16 GB, 16013942784 bytes
Device     Boot  Start     End  Sectors  Size  Type
/dev/sdb1  *      2048 31252479 31250432 14.9G  Microsoft basic data
```

可以看到连接的磁盘、分区、大小和文件系统类型。处理未知机器时，这是确认存储设备的第一步。

## df：查看剩余空间

`df`（disk free）显示每个已挂载文件系统的使用和可用空间：

```bash
ahegazy0@kali:~$ df -h
```

`-h` 以 GB、MB 等易读单位显示：

```
Filesystem      Size  Used Avail Use%  Mounted on
/dev/sda2        46G   12G   32G  27%  /
/dev/sdb1        15G  8.1G  6.9G  54%  /mnt/usb
tmpfs           2.0G  1.2M  2.0G   1%  /run
```

`Use%` 可快速看出哪个分区接近已满；`tmpfs` 是位于内存中的虚拟文件系统。

## /etc/fstab：持久化挂载配置

用 `mount` 的挂载默认只持续到重启。要在启动时自动挂载，把条目写入 `/etc/fstab`：

```bash
ahegazy0@kali:~$ cat /etc/fstab
```

```
# device          mountpoint    fstype   options    dump  pass
/dev/sda1         /             ext4     defaults    0     1
/dev/sda2         /home         ext4     defaults    0     2
UUID=1234-ABCD    /mnt/data     ntfs     defaults    0     0
```

从侦察角度看，`fstab` 会暴露启动时挂载的网络共享（NFS、SMB）、加密卷和外部磁盘等信息。

## fsck：检查和修复磁盘

`fsck`（filesystem check）扫描文件系统错误并尝试修复：

```bash
ahegazy0@kali:~$ umount /dev/sdb1
ahegazy0@kali:~$ fsck /dev/sdb1
```

**绝不能对已挂载的文件系统运行 fsck。** 挂载状态下运行可能让损坏更严重。检查根分区时无法在运行中的系统中卸载，需要从 Live USB 启动后执行。

## 文件系统类型

| 文件系统 | 常见场景 |
|---|---|
| ext4 | 标准 Linux 文件系统 |
| ext3 / ext2 | 较老的 Linux 文件系统 |
| NTFS | Windows 磁盘 |
| FAT32 / exFAT | USB、SD 卡，跨平台 |
| XFS | 高性能 Linux，常见于服务器 |

挂载非 Linux 磁盘时可以指定类型：

```bash
ahegazy0@kali:~$ mount -t ntfs /dev/sdb1 /mnt/windows
```

## 挂载 USB 的完整流程

```bash
# 1. 查看刚连接的设备
ahegazy0@kali:~$ fdisk -l

# 2. 创建挂载点
ahegazy0@kali:~$ mkdir /mnt/usb

# 3. 挂载分区
ahegazy0@kali:~$ mount /dev/sdb1 /mnt/usb

# 4. 确认成功
ahegazy0@kali:~$ ls /mnt/usb

# 5. 执行需要的操作

# 6. 完成后卸载
ahegazy0@kali:~$ umount /mnt/usb
```

## 命令速查

| 命令 | 作用 |
|---|---|
| `fdisk -l` | 列出所有磁盘和分区 |
| `mount /dev/sdb1 /mnt/point` | 将分区挂载到文件夹 |
| `mount -t ntfs /dev/sdb1 /mnt/point` | 指定文件系统类型挂载 |
| `umount /mnt/point` | 安全卸载磁盘 |
| `df -h` | 以易读格式显示空间使用情况 |
| `fsck /dev/sdb1` | 检查并修复文件系统（先卸载） |
| `cat /etc/fstab` | 查看持久化挂载配置 |
| `mkdir /mnt/point` | 创建挂载点目录 |
| `lsblk` | 以树状结构显示磁盘和分区 |

`lsblk` 比 `fdisk -l` 更适合快速查看布局：

```bash
ahegazy0@kali:~$ lsblk
NAME   MAJ:MIN  SIZE  TYPE  MOUNTPOINT
sda    8:0      500G  disk
├─sda1 8:1      500M  part  /boot
└─sda2 8:2      499G  part  /
sdb    8:16      16G  disk
└─sdb1 8:17     16G  part  /mnt/usb
```

## 练习

- [ ] 运行 `fdisk -l`，识别虚拟机连接的所有磁盘和分区
- [ ] 运行 `df -h`，找出最接近已满的分区
- [ ] 用 `mkdir` 创建 `/mnt/test`，添加虚拟磁盘，找到设备名并挂载
- [ ] 运行 `ls /mnt/test` 确认成功，完成后用 `umount /mnt/test`
- [ ] 阅读 `/etc/fstab`，理解每一行配置

> 💡 *为了进行更深入的练习，也建议完成官方 **Linux Basics for Hackers** 书中每章末尾的练习。*
---

*下一篇：模块 11——日志与日志文件*
