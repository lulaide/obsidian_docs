---
source: "https://docs.min.io/community/minio-object-store/operations/deployments/baremetal-deploy-minio-as-a-container.html"
tags:
  - "minio"
---
## 将 MinIO 部署为容器

本页面记录了如何在支持容器化进程的任何操作系统上部署 MinIO 作为容器。

本文件假设已安装 Docker、Podman 或其他支持标准容器镜像格式的运行时。MinIO 镜像使用 [Red Hat Universal Base Image 9 Micro](https://catalog.redhat.com/software/container-stacks/detail/609560d9e2b160d361d24f98) 。

MinIO 容器的功能和性能可能会受到基础操作系统的限制。

该程序包括有关部署单节点多驱动（SNMD）和单节点单驱动（SNSD）拓扑的指导，以支持早期开发和评估环境。

重要

MinIO 通过 MinIO Kubernetes Operator 正式支持在 Kubernetes 基础设施上容器化的多节点多驱动（MNMD）“分布式”配置。

MinIO 不支持也不提供使用 Docker Swarm、Docker Compose 或任何其他编排容器环境部署分布式集群的说明。

### 审核检查表

在尝试此过程之前，请确保您已查看我们发布的硬件、软件和安全检查清单。

### 擦除编码奇偶校验

MinIO 会根据拓扑中节点和驱动器的总数自动确定集群的默认 [擦除编码](https://docs.min.io/community/minio-object-store/operations/concepts/erasure-coding.html#minio-erasure-coding) 配置。您可以在设置集群时配置每个对象的 [奇偶校验](https://docs.min.io/community/minio-object-store/glossary.html#term-parity) 设置 *或* 让 MinIO 选择默认值（对于生产级集群为 `EC:4` ）。

奇偶校验控制对象可用性与磁盘存储之间的关系。使用 MinIO [擦除码计算器](https://min.io/product/erasure-code-calculator) 来指导您选择适合您集群的擦除码奇偶校验级别。

虽然您可以随时更改纠删码奇偶校验设置，但使用给定奇偶校验写入的对象 **不会** 自动更新为新的奇偶校验设置。

### 容器存储

此过程假设您将一个或多个专用存储设备挂载到容器上，以作为 MinIO 的持久存储。

存储在临时容器路径上的数据在容器重启或被删除时会丢失。使用任何此类路径需自行承担风险。

1. 启动容器

本程序提供了在 rootfull 模式下使用 Podman 和 Docker 的说明。对于无根部署，请参考每个运行时的文档以进行配置和容器启动。

对于所有其他容器运行时，请遵循该运行时的文档并指定等效的选项、参数或配置。

#### Podman

以下命令在您的主目录中创建一个文件夹，然后使用 Podman 启动 MinIO 容器：

```shell
mkdir -p ~/minio/data

podman run \
   -p 9000:9000 \
   -p 9001:9001 \
   --name minio \
   -v ~/minio/data:/data \
   -e "MINIO_ROOT_USER=ROOTNAME" \
   -e "MINIO_ROOT_PASSWORD=CHANGEME123" \
   quay.io/minio/minio server /data --console-address ":9001"
```

该命令将端口 `9000` 和 `9001` 分别绑定到 MinIO S3 API 和 Web 控制台。

本地驱动器 `~/minio/data` 被挂载到容器上的 `/data` 文件夹。您可以修改 [`MINIO_ROOT_USER`](https://docs.min.io/community/minio-object-store/reference/minio-server/settings/root-credentials.html#envvar.MINIO_ROOT_USER "envvar.MINIO_ROOT_USER") 和 [`MINIO_ROOT_PASSWORD`](https://docs.min.io/community/minio-object-store/reference/minio-server/settings/root-credentials.html#envvar.MINIO_ROOT_PASSWORD "envvar.MINIO_ROOT_PASSWORD") 变量以根据需要更改根登录。

对于多驱动器部署，依次将每个本地驱动器或文件夹绑定到远程的顺序编号路径。然后，您可以修改 [`minio server`](https://docs.min.io/community/minio-object-store/reference/minio-server/minio-server.html#command-minio.server "minio.server") 启动以指定这些路径：

```shell
mkdir -p ~/minio/data-{1..4}

podman run \
   -p 9000:9000 \
   -p 9001:9001 \
   --name minio \
   -v /mnt/drive-1:/mnt/drive-1 \
   -v /mnt/drive-2:/mnt/drive-2 \
   -v /mnt/drive-3:/mnt/drive-3 \
   -v /mnt/drive-4:/mnt/drive-4 \
   -e "MINIO_ROOT_USER=ROOTNAME" \
   -e "MINIO_ROOT_PASSWORD=CHANGEME123" \
   quay.io/minio/minio server http://localhost:9000/mnt/drive-{1...4} --console-address ":9001"
```

对于 Windows 主机，使用 Windows 文件系统语义指定本地文件夹路径 `C:\minio\:/data` 。
#### Docker

以下命令在您的主目录中创建一个文件夹，然后使用 Docker 启动 MinIO 容器：

```shell
mkdir -p ~/minio/data

docker run \
   -p 9000:9000 \
   -p 9001:9001 \
   --name minio \
   -v ~/minio/data:/data \
   -e "MINIO_ROOT_USER=ROOTNAME" \
   -e "MINIO_ROOT_PASSWORD=CHANGEME123" \
   quay.io/minio/minio server /data --console-address ":9001"
```

该命令将端口 `9000` 和 `9001` 分别绑定到 MinIO S3 API 和 Web 控制台。

本地驱动器 `~/minio/data` 被挂载到容器上的 `/data` 文件夹。您可以修改 [`MINIO_ROOT_USER`](https://docs.min.io/community/minio-object-store/reference/minio-server/settings/root-credentials.html#envvar.MINIO_ROOT_USER "envvar.MINIO_ROOT_USER") 和 [`MINIO_ROOT_PASSWORD`](https://docs.min.io/community/minio-object-store/reference/minio-server/settings/root-credentials.html#envvar.MINIO_ROOT_PASSWORD "envvar.MINIO_ROOT_PASSWORD") 变量以根据需要更改根登录。

对于多驱动器部署，依次将每个本地驱动器或文件夹绑定到远程的顺序编号路径。然后，您可以修改 [`minio server`](https://docs.min.io/community/minio-object-store/reference/minio-server/minio-server.html#command-minio.server "minio.server") 启动以指定这些路径：

```shell
mkdir -p ~/minio/data-{1..4}

docker run \
   -p 9000:9000 \
   -p 9001:9001 \
   --name minio \
   -v /mnt/drive-1:/mnt/drive-1 \
   -v /mnt/drive-2:/mnt/drive-2 \
   -v /mnt/drive-3:/mnt/drive-3 \
   -v /mnt/drive-4:/mnt/drive-4 \
   -e "MINIO_ROOT_USER=ROOTNAME" \
   -e "MINIO_ROOT_PASSWORD=CHANGEME123" \
   quay.io/minio/minio server http://localhost:9000/mnt/drive-{1...4} --console-address ":9001"
```

对于 Windows 主机，使用 Windows 文件系统语义指定本地文件夹路径 `C:\minio\:/data` 。

### 2\. 连接到部署

打开浏览器访问 [http://localhost:9000](http://localhost:9000/) 以打开 [MinIO 控制台](https://docs.min.io/community/minio-object-store/administration/minio-console.html#minio-console) 登录页面。

使用 MINIO\_ROOT\_USER 和 MINIO\_ROOT\_PASSWORD 登录 来自上一步。

[![MinIO Console Login Page](https://docs.min.io/community/minio-object-store/_images/console-login1.png)](https://docs.min.io/community/minio-object-store/_images/console-login1.png)

您可以使用 MinIO 控制台进行一般管理任务，如身份和访问管理、指标和日志监控或服务器配置。每个 MinIO 服务器都包含其自己的嵌入式 MinIO 控制台。

按照 [安装说明](https://docs.min.io/community/minio-object-store/reference/minio-mc.html#mc-install) 在本地主机上安装 `mc` 。 运行 `mc --version` 来验证安装。

安装完成后，为 MinIO 部署创建一个别名：

```shell
mc alias set myminio http://localhost:9000 USERNAME PASSWORD
```

更改主机名、用户名和密码以反映您的部署。