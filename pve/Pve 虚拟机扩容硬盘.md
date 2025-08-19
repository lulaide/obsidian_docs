---
source: "https://yan-jian.com/pve%E8%99%9A%E6%8B%9F%E6%9C%BA%E6%89%A9%E5%AE%B9%E7%A1%AC%E7%9B%98.html?utm_source=chatgpt.com"
tags:
  - "pve"
---
## pve控制台增大硬盘

在pve控制台 `Hardware` 标签页中选中要扩容的硬盘，依次选择 `Disk Action` - `Resize` ，然后将 `Size Increment` 设为 100（注意 **不是200** ），即再增加 100 GB

[![](https://static.sqlfans.cn/file/20230323/pve-hdd-200g.png)](https://static.sqlfans.cn/file/20230323/pve-hdd-200g.png)

- 将虚拟机开机，执行 `lsblk` 看到 `sda` 已经扩容，但文件系统 `/var/lib/containers/storage/overlay` 还是没有变

```
# lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                         8:0    0  112G  0 disk 
├─sda1                      8:1    0    1M  0 part 
├─sda2                      8:2    0    2G  0 part /boot
└─sda3                      8:3    0   30G  0 part 
  └─ubuntu--vg-ubuntu--lv 252:0    0   30G  0 lvm  /var/lib/containers/storage/overlay
                                                   /
sr0                        11:0    1    3G  0 rom  
```

## 扩展分区

先安装 `cloud-guest-utils`：

```bash
sudo apt update && sudo apt install cloud-guest-utils -y
```

然后扩展 sda3：

```bash
sudo growpart /dev/sda 3
```

扩展后确认：

```bash
lsblk
```

`sda3` 应该已经变大。


再次查看系统分区情况， `/data` 已经扩容成功了

## 扩展逻辑卷 (LV)

把多出来的空间分配给系统盘（ubuntu-vg/ubuntu-lv）：

```bash
sudo lvextend -l +100%FREE /dev/ubuntu-vg/ubuntu-lv
```

检查：

```bash
sudo lvs
```

---

## 扩展文件系统

最后扩展 ext4 文件系统：

```bash
sudo resize2fs /dev/ubuntu-vg/ubuntu-lv
```

检查：

```bash
df -h /
```

应该能看到 `/` 从 30G 变成接近 112G。