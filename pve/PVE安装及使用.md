
## 一、Proxmox VE

### 1. 介绍

Proxmox Virtual Environment （通常简称为 Proxmox VE 或 PVE，下文统一简称为 PVE）是由 Proxmox 公司开发的一个基于 Debian Linux 的开源企业级虚拟化服务器管理平台。它将 KVM 虚拟机管理程序和 Linux 容器 (LXC)、软件定义存储以及网络功能集成在一个平台上，并且提供了 Web  用户界面，可以轻松管理虚拟机、容器和高可用性集群。

**特点**：自带Web管理界面，同时支持 QEMU KVM 虚拟机和 LXC 容器，并且支持在多台服务器上部署并组成集群。

一些可能有用的链接：

[官方网站](https://www.proxmox.com/en/products/proxmox-virtual-environment/overview)

[官方文档](https://pve.proxmox.com/pve-docs/)

[由志愿者自发翻译的中文手册](https://pve-doc-cn.readthedocs.io/zh-cn/latest/index.html)

[中文 Proxmox VE 管理员指南](http://104.194.87.108/pve/pve-admin-guide-cn.html)

[Proxmox 的 Github 页面](https://github.com/proxmox)

### 2. 安装

#### 2.1 获取安装镜像

因为 PVE 基于 Debian，所以我们可以直接在已有的 Debian 系统上安装 PVE。（这种方式常用于在云服务器上部署 PVE。）

不过，一个更常用、更便捷的方式是直接使用官方提供的 ISO 系统安装镜像进行安装。

官方提供的安装镜像可以在 [这里](https://www.proxmox.com/en/downloads) 下载。但因为其位于海外，下载速度通常不甚理想。所以，更推荐从第三方镜像站获取其安装镜像。

可以在 [这里](https://mirrors.cqupt.edu.cn/proxmox/iso/) 从我们自己的重邮开源软件镜像站下载 PVE 的 ISO 安装镜像，也可以到 [中国科学技术大学开源软件镜像站](https://mirrors.ustc.edu.cn/proxmox/iso/) 、[清华大学开源软件镜像站](https://mirrors.tuna.tsinghua.edu.cn/proxmox/iso/) 等镜像站下载。

本次课程使用当前最新的 **PVE 8.3 版本** 作为演示，部署在已有的 PVE 8.3 上。

生产级的 PVE 一般直接由镜像裸机安装，但考虑到 PVE 本身通常需要独占一块硬盘，如果你没有足够的硬件资源，可以在 VirtualBox、VMWare Workstation 等虚拟化软件或者 Hyper-V 甚至是 PVE 本身中嵌套安装。但在这种情况下，你**必须在虚拟化软件中启用嵌套虚拟化**。

#### 2.2 换源

更换 PVE 系统软件源和 Ceph 源：

```bash
# 备份原来的企业源
mv /etc/apt/sources.list.d/pve-enterprise.list /etc/apt/sources.list.d/pve-enterprise.list.bak
# 更换为国内源（以中科大源为例）
echo "deb https://mirrors.ustc.edu.cn/proxmox/debian/pve bookworm pve-no-subscription" > /etc/apt/sources.list.d/pve-no-subscription.list
# 更换 Ceph 源
echo "deb https://mirrors.ustc.edu.cn/proxmox/debian/ceph-quincy bookworm no-subscription" > /etc/apt/sources.list.d/ceph.list
sed -i.bak "s#http://download.proxmox.com/debian#https://mirrors.ustc.edu.cn/proxmox/debian#g" /usr/share/perl5/PVE/CLI/pveceph.pm
# 更新软件包索引并安装 https 证书
apt update && apt install -y apt-transport-https ca-certificates  --fix-missing
```

更换 CT 容器镜像源：

```bash
# 备份原来的 LXC 容器镜像源配置
cp /usr/share/perl5/PVE/APLInfo.pm /usr/share/perl5/PVE/APLInfo.pm_back
# 修改 LXC 容器镜像源
sed -i &#39;s|http://download.proxmox.com|https://mirrors.tuna.tsinghua.edu.cn/proxmox|g&#39; /usr/share/perl5/PVE/APLInfo.pm
# 重启相关服务
systemctl restart pvedaemon
```

取消无效订阅弹窗：

```bash
# 在 js 文件中删除相关内容
sed -Ezi.bak "s/(Ext.Msg.show\(\{\s+title: gettext\('No valid sub)/void\(\{ \/\/\1/g" /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js
# 重启相关服务
systemctl restart pveproxy
# 可能需要在浏览器中手动注销登录并在新标签页中打开 PVE 控制台以清除缓存
```

### 3. 创建虚拟机

#### 3.1 常规

1. **节点**：PVE 集群中的节点名称，通常为节点的主机名。如果没有加入 PVE 集群，则这里的节点默认为当前操作的 PVE 主机。

2. **资源池**：PVE 引入的一个用于管理用户和资源的抽象概念。在加入 PVE 集群后才可以选择资源池，否则默认不可选。

   > **资源池 **是一组虚拟机、容器和存储设备。在某些用户应该对一组特定资源具有受控访问权限的情况下，它对于权限处理非常有用，因为它允许将单个权限应用于一组元素，而不必基于每个资源对其进行管理。资源池通常与组串联使用，以便组的成员对一组计算机和存储具有权限。

3. **VM ID**：用于标识虚拟机或容器的唯一 ID。可以由 PVE 自动分配，也可以在创建时手动指定。在命令行环境中操作 PVE 创建的虚拟机或容器时，我们通常会使用 VM ID 来指定。

4. **名称**：显示在 PVE 管理界面中的虚拟机名称。

5. **开机自启动**：如果启用这个选项，那么在 PVE 主机开机后，会自动开启这个虚拟机。

   > **通常建议在第一次配置虚拟机时不要勾选开机自启动**。如果虚拟机的配置出现问题（例如 PCIE 直通配置不正确），那么在启用开机自启动的情况下，很可能会让 PVE 宿主机直接卡死，进而进入开机即卡死的死循环。

#### 3.2 操作系统

在这里选择已经上传到 PVE 镜像存储中的操作系统镜像。在选择好操作系统镜像后，右侧的「客户机操作系统 」部分需要与之对应。

> 对于 Windows 虚拟机，如果后续想要使用 SCSI 和半虚拟化网络，需要在这里勾选「为 VirtIO 驱动程序添加额外驱动器」。

#### 3.3 系统

1. **显卡**：通常在创建虚拟机时选择默认即可。如需配置 vGPU、显卡直通等，可以在安装系统后再配置。

2. **机型**：对于比较老的操作系统（如 Windows XP），可以选择 i440fx，否则建议优先选择 q35。

   > **i440fx** 是一个旧的主板模拟类型，它模拟的是 Intel 440FX 主板，该主板用于早期的 x86 架构计算机。i440fx 是一种较为传统和兼容性较高的选择，适用于需要与老旧操作系统或特定硬件环境兼容的情况。
   >
   > **q35** 是一种更现代的主板模拟类型，它模拟的是 Intel Q35 Express 芯片组，该芯片组用于较新的 x86 架构计算机。q35 提供了更多的功能和先进的虚拟化特性，如 PCI Express 设备的直通、高级电源管理等。它在性能和功能方面相对较好，适用于大多数现代虚拟化场景。

3. **BIOS**：这里可以选择 **SeaBIOS**（Legacy）和 **OVMF（UEFI）**。

4. **EFI 存储**：存储 UEFI 的配置信息，通常选择默认存储卷（local-lvm）即可。

5. **SCSI 控制器**：建议选择 **virtIO SCSI Single** 或 **virtIO SCSI**。不过，后者需要在虚拟机操作系统中单独安装驱动。

   > **virtIO SCSI** 是一种在虚拟化环境中常用的高性能虚拟存储控制器。它使用 QEMU 的 virtIO 驱动程序和 SCSI 协议实现，提供了较低的虚拟化开销和更好的性能。virtIO SCSI 可以与虚拟机中的客户操作系统进行直接通信，提供快速、可靠的存储访问。
   >
   > **virtIO SCSI Single** 是 virtIO SCSI 控制器的一个变种，它只有一个队列。相比于普通的 virtIO SCSI，它在一些场景中可能会有较低的性能，但对于某些特定的用例或旧版本的操作系统可能更适用。
   >
   > **MegaRAID SAS 8708EM2** 是一款物理存储控制器，用于管理和控制 SATA 和 SAS 硬盘驱动器。它是 LSI Logic（现在是 Broadcom）生产的硬件 RAID 控制器，支持多种 RAID 级别和高级功能，如热插拔、缓存和数据保护。
   >
   > **LSI 53C810** 是一款旧版的 SCSI 控制器，用于连接和管理 SCSI 设备。它是古老的硬件设备，过去通常用于连接磁盘驱动器、磁带机等 SCSI 设备。
   >
   > **LSI 53C895A** 是一款先进的 SCSI 控制器，也用于连接和管理 SCSI 设备。它提供更高的性能和更多的功能，支持广泛的 SCSI 设备和高级特性。

7. **QEMU 代理**：根据实际需求选择。

   > **QEMU 代理 **是一种轻量级的虚拟化代理程序，它可以将虚拟机的管理工作委派给其他 PVE 节点上的 QEMU 代理程序来处理。这样就可以实现多个节点共同承担虚拟机工作负载，从而提高整个系统的性能和可靠性。此外，QEMU 代理还可以帮助管理员更好地管理虚拟机资源。通过将虚拟机分配到多个节点上，可以更容易地管理虚拟机的资源使用情况，并且可以更快地响应虚拟机的请求。这对于需要高度可扩展性和灵活性的业务场景尤其重要。

8. **添加 TPM**：根据实际需求选择。一些操作系统（例如 Windows 11）可能会要求必须开启 TPM。

   > 需要注意的是，**虚拟机在开启 TPM 之后就不支持进行快照了**。 

#### 3.4 磁盘   

1. **磁盘大小**：这里建议先估计一下虚拟机可能占用的磁盘大小，然后留少量富余空间。不过，考虑到虚拟机的磁盘可以很容易地扩容但很难缩容，在存储空间吃紧的情况下，在创建虚拟机时可以少分配一些空间。

2. **总线/设备**：如果你的存储设备是 SATA 协议的机械硬盘或 SSD，则可以选择 **SCSI ** 总线。如果你的存储设备是 NVME 协议的固态硬盘，则应该选择 **virtio Block** 总线。如果你需要连接多个存储设备，则应该选择 **virtio Block** 总线，并并添加多个 virtio SCSI 设备。

3. **缓存**：在 PVE 宿主机没有 ECC 内存的情况下建议不要开启缓存（可能带来错误的数据写入）。如果虚拟机的系统盘位于机械硬盘上，并且配置了 swap，同样建议不要开启缓存。（会产生大量的磁盘 IO。）

   > 无缓存（No cache）：数据直接读写到磁盘，不使用任何缓存。这种方式可以保证数据的持久性和一致性，但读写性能较低。
   >
   > Direct sync（直接同步）：数据先被写入缓存，然后同步写入磁盘。在数据同步完成之前，系统会阻塞等待磁盘操作完成的确认信号。这种方式可以提高写入性能，同时保证了数据的持久性和一致性。
   >
   > Write through（写透）：数据先被写入缓存，然后立即被写入磁盘。写入缓存和写入磁盘是同时进行的，不需要等待磁盘操作的确认。这种方式可以提高写入性能，但对于读取操作，仍需要从磁盘中获取数据，可能降低读取性能。
   >
   > Write back（写回）：数据首先被写入缓存，然后根据一定的策略异步写入磁盘。在此期间，应用程序可以继续执行其他操作，而不需要等待磁盘操作完成的确认。这种方式可以显著提高写入性能，但存在数据丢失的风险，因为在数据写入磁盘之前发生系统崩溃或断电时，缓存中的数据会丢失。
   >
   > Writeback（不安全的写回）：与「Write back」相似，但没有提供有效的机制来保护数据免受系统崩溃或断电的影响。这种方式的优势是更高的写入性能，但风险更大，可能导致数据丢失或不一致。因此，它通常用于对数据完整性要求较低、但需要更高性能的应用场景。

4. **SSD 仿真**：如果为虚拟机分配的磁盘位于 SSD 上，则推荐勾选此项，这样能充分发挥 SSD 的性能。如果是机械硬盘，则不建议选择此项。

5. **丢弃**：用来控制虚拟机在删除文件时是否立即释放文件所占用的空间。如果勾选了“丢弃”，则当虚拟机删除文件时，磁盘空间会被立即释放，并可以被其他文件使用。如果没有勾选“丢弃”，则文件所占用的磁盘空间不会被立即释放，而是留作未分配空间。

   > 如果有足够的磁盘空间，并且不需要频繁的删除文件，可以不勾选「丢弃」选项，这样可以更快的读取文件。如果需要频繁删除文件并及时释放磁盘空间，则建议勾选「丢弃」。

6. **IO Thread**：如果 PVE 宿主机拥有多核处理器，并且为虚拟机分配了一个以上的 vCPU，则建议勾选此项。这样有助于充分发挥多核处理器的 IO 性能。

   > 较新版本的 QEMU 使用了 IO Thread 模型，它为每一个 vCPU 分配一个 QEMU 线程，以及一个专用的事件处理循环线程。各个 vCPU 线程可以并行的执行客户机指令，进而提供真正的 SMP 支持。IO Thread 负责运行事件处理循环。通过使用了一个全局的 mutex 互斥锁来维持线程同步。大多数时间里，vCPU 在运行客户机指令，IO Thread 则阻塞在 select(2) 上。这样使得 IO 处理能够完全脱离主线程，跑在多个不同的线程里面，以充分利用现代多核处理器的性能。

7. **异步 IO**：通常建议开启，有助于提高虚拟机的 IO 性能。

   > io_uring 是一个 Linux 内核的异步 I/O 框架，它提供了高性能的异步 I/O 操作，io_uring 的目标是通过减少系统调用和上下文切换的开销来提高 I/O 操作的性能。 

#### 3.5 CPU

1. **类别**：根据实际需求选择。可以直接使用 host 类型。这样相当于对 CPU 的完全模拟，即主机是什么 CPU 虚拟机就是什么 CPU。

   > 可以在 PVE 宿主机上使用命令  `qemu-system-x86_64 -cpu help`  来查看关于CPU类型的详细信息。

2. **启用 NUMA**：根据实际需求选择。需要 PVE 宿主机的 CPU 支持 NUMA 架构。

   > 可以在 PVE 宿主机上使用命令 `dmesg | grep -i numa` 来查看 CPU 是否支持 NUMA 架构。如果命令返回  `no NUMA configuration`  则代表不支持。
   >
   > NUMA 表示「非一致性存储访问」，是一种多处理器体系结构，其中每个 CPU 位于不同的内存区域。当服务器支持 NUMA 架构时，如果不正确地配置虚拟机设置，则可能会导致性能问题。在支持 NUMA 架构的服务器上启用 NUMA 可以提高性能，并确保虚拟机能够正确地使用可用的内存。

#### 3.6 内存

1. **内存**：根据实际需求选择设置。在虚拟机关机后可以自由调整为虚拟机分配的内存大小。
2. **Ballooning 设备**：即动态内存分配，能够让虚拟机未使用的内存在 PVE 宿主机上依然可用，同时在虚拟机用尽为其分配的内存时自动从宿主机上获取更多的内存，建议开启。 

#### 3.7 网络

1. **无网络设备**：如果需要为虚拟机启用网卡直通，则建议勾选。
2. **桥接**：选择 PVE 宿主机上的对应虚拟网卡进行桥接。如果想让虚拟机能够访问互联网，则需要桥接 PVE 宿主机上能够正常访问互联网的虚拟网卡。

3. **模型**：推荐选择 **virtio（半虚拟化）**以最大化虚拟网卡的性能。

4. **Multiqueue**： 允许虚拟机操作系统使用多个 vCPU 来处理网络数据包，从而增加处理数据包的总速率。

   > 当使用 Multiqueue 时，建议将其设置为等于虚拟机总 vCPU 数量的值。只有当虚拟机必须处理大量的传入连接时（例如作为软路由、网关或代理）才建议配置这个选项。

5. **MAC 地址**：可以自定义，也可以让 PVE 自动生成（Auto）。

### 4. 创建 LXC 容器

### 5. 创建和加入 PVE 集群

#### 4.1 PVE 集群简介

PVE 集群由多个 PVE 节点组成，节点之间通过网络连接，共享存储等资源。

相比之前我们创建的单节点 PVE 服务，多个节点组成的 PVE 集群有着如下优势：

- **高可用性**：当一个节点宕机时，其他节点可以继续提供服务。

- **资源共享**：多个节点之间可以共享虚拟机、容器等资源。

- **集中管理**：集群中的所有节点可以通过一个 Web 界面集中管理，便于监控和操作。

#### 4.2 创建 PVE 集群

#### 4.3 加入 PVE 集群

前提条件：

1、为了防止 VM ID 冲突，将要加入集群的节点中不能有虚拟机或容器，但创建 PVE 集群的节点可以有虚拟机或容器。

2、集群中 PVE 网络要互通，延迟需要尽可能低。

> 在生产环境中，通常会为 PVE 集群的每个节点配置至少两块物理网卡，一个用于交换网络数据，另一个用于集群内部不同节点之间的通信。为了隔离不同节点下的虚拟机网络，一般还会为不同节点分别配置 VLAN 并分配不同网段。当然，在这种情况下，为了实现不同网段中的 PVE 节点能够互访组成集群，还需要配置路由表等操作。

#### 4.4 在 PVE 集群中手动下线节点

在要下线的节点上：
```bash
# 停止节点上 PVE 集群的相关服务
systemctl stop pve-cluster corosync
# 在本地模式下重修启动 PVE 集群文件系统
pmxcfs -l
# 备份节点上原来的集群相关配置文件
cp -a /etc/corosync /etc/corosync.bak
cp /etc/pve/corosync.conf /etc/pve/corosync.conf.bak
# 删除节点上的集群相关配置文件
rm /etc/corosync/*
rm /etc/pve/corosync.conf
# 停止 PVE 集群文件系统进程
killall pmxcfs
# 重新启动 PVE 集群服务
systemctl start pve-cluster
```

在集群的其他任何一个节点上：

```bash
# 删除节点相关文件（这里假设需要下线的节点名称为 pve2）
cd /etc/pve/nodes
rm -rf pve2
# 使用 PVE 集群管理工具删除节点
pvecm delnode pve2
```