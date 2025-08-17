---
source: "https://yan-jian.com/pve%E8%99%9A%E6%8B%9F%E6%9C%BA%E6%89%A9%E5%AE%B9%E7%A1%AC%E7%9B%98.html?utm_source=chatgpt.com"
tags:
  - "pve"
---
### pve虚拟机扩容硬盘

**场景**

- 最初安装 centos 7 的时候，有一块 100GB 硬盘（非系统盘、整盘格式化，挂载目录 `/data` ），现在需要扩到 200G

**详细步骤**

- 第1步，将要扩容的虚拟机关机
- 第2步，在pve控制台 `Hardware` 标签页中选中要扩容的硬盘，依次选择 `Disk Action` - `Resize` ，然后将 `Size Increment` 设为 100（注意 **不是200** ），即再增加 100 GB

[![](https://static.sqlfans.cn/file/20230323/pve-hdd-200g.png)](https://static.sqlfans.cn/file/20230323/pve-hdd-200g.png)

- 第3步，将虚拟机开机，执行 `fdisk -l` 看到 `/dev/sdb` 已经扩容到 214.7G，但文件系统 `/dev/sdb` 还是 99G 没有变

```
[root\@localhost \~]# df -Th | grep dev

devtmpfs                devtmpfs  7.9G     0  7.9G   0% /dev

tmpfs                   tmpfs     7.9G     0  7.9G   0% /dev/shm

/dev/mapper/centos-root xfs        36G   22G   14G  63% /

/dev/sda1               xfs      1014M  184M  831M  19% /boot

/dev/sdb                ext4       99G   30G   64G  32% /data

[root\@localhost \~]# fdisk -l | grep dev

Disk /dev/sda: 42.9 GB, 42949672960 bytes, 83886080 sectors

/dev/sda1   \*        2048     2099199     1048576   83  Linux

/dev/sda2         2099200    83886079    40893440   8e  Linux LVM

Disk /dev/sdb: 214.7 GB, 214748364800 bytes, 419430400 sectors

Disk /dev/mapper/centos-root: 37.7 GB, 37706792960 bytes, 73646080 sectors

Disk /dev/mapper/centos-swap: 4160 MB, 4160749568 bytes, 8126464 sectors

[root\@localhost \~]# lsblk

NAME            MAJ\:MIN RM  SIZE RO TYPE MOUNTPOINT

sdb               8:16   0  200G  0 disk /data

sr0              11:0    1  4.4G  0 rom\

sda               8:0    0   40G  0 disk

├─sda2            8:2    0   39G  0 part

│ ├─centos-swap 253:1    0  3.9G  0 lvm\

│ └─centos-root 253:0    0 35.1G  0 lvm  /

└─sda1            8:1    0    1G  0 part /boot
```

- 第4步，针对ext4格式的磁盘，执行 `resize2fs /dev/sdb` 将文件系统进行扩容

```
[root\@localhost \~]# resize2fs /dev/sdb

resize2fs 1.42.9 (28-Dec-2013)

Filesystem at /dev/sdb is mounted on /data; on-line resizing required

old\_desc\_blocks = 13, new\_desc\_blocks = 25

The filesystem on /dev/sdb is now 52428800 blocks long.
```

- 第5步，再次查看系统分区情况， `/data` 已经扩容成功了

```
[root\@localhost \~]# df -Th | grep dev

devtmpfs                devtmpfs  7.9G     0  7.9G   0% /dev

tmpfs                   tmpfs     7.9G     0  7.9G   0% /dev/shm

/dev/mapper/centos-root xfs        36G   22G   14G  63% /

/dev/sda1               xfs      1014M  184M  831M  19% /boot

/dev/sdb                ext4      197G   30G  159G  16% /data
```