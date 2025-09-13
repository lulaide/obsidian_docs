---
source: "https://qdrant.tech/documentation/guides/installation/#docker"
tags:
  - "qdrant"
---
## 安装要求

以下部分描述了部署 Qdrant 的要求。

## CPU 和内存

您所需的 CPU 和 RAM 的首选大小取决于：

- 向量数量
- 向量维度
- [有效载荷](https://qdrant.tech/documentation/concepts/payload/) 及其索引
- 存储
- 复制
- 如何配置量化

我们的 [云定价计算器](https://cloud.qdrant.io/calculator?qdrant_tech_ajs_anonymous_id=f5040371-fc86-496b-9314-4ad1d57e1757&__hstc=265983056.859c79f1b3adaf4f7a8f578bbaea3db2.1757783523162.1757783523162.1757783523162.1&__hssc=265983056.3.1757783523162&__hsfp=1938414035) 可以帮助您在没有负载或索引数据的情况下估算所需资源。

### 支持的 CPU 架构：

**64位系统：**

- x86\_64/amd64
- AArch64/arm64

**32位系统：**

- 不支持

### 存储

对于持久存储，Qdrant 需要对具有 [POSIX 兼容文件系统](https://www.quobyte.com/storage-explained/posix-filesystem/) 的存储设备进行块级访问。提供块级访问的网络系统，如 [iSCSI](https://en.wikipedia.org/wiki/ISCSI) 也是可以接受的。Qdrant 不支持 [网络文件系统](https://en.wikipedia.org/wiki/File_system#Network_file_systems) （如 NFS）或 [对象存储](https://en.wikipedia.org/wiki/Object_storage) 系统（如 S3）。

如果您将向量卸载到本地磁盘，我们建议您使用固态硬盘（SSD 或 NVMe）。

### 网络

每个 Qdrant 实例需要三个开放端口：

- `6333` - 用于 HTTP API，针对 [监控](https://qdrant.tech/documentation/guides/monitoring/) 健康和指标端点
- `6334` - 用于 [gRPC](https://qdrant.tech/documentation/interfaces/#grpc-interface) API
- `6335` - 用于 [分布式部署](https://qdrant.tech/documentation/guides/distributed_deployment/)

集群中的所有 Qdrant 实例必须能够：

- 通过这些端口相互通信
- 允许来自使用 Qdrant 的客户端对端口 `6333` 和 `6334` 的传入连接。

### 安全性

Qdrant 的默认配置可能在某些情况下不够安全。有关更多信息，请参阅 [我们的安全文档](https://qdrant.tech/documentation/guides/security/) 。

## 安装选项

Qdrant 可以根据您的需求以不同方式安装：

对于生产环境，您可以使用我们的 Qdrant Cloud 在我们的基础设施中完全托管运行 Qdrant，或在您的基础设施中使用混合云。

如果您希望在自己的基础设施中运行 Qdrant，而不连接任何云，我们建议在 Kubernetes 集群中使用我们的 Qdrant Private Cloud Enterprise Operator 安装 Qdrant。

对于测试或开发环境，您可以运行 Qdrant 容器或作为二进制可执行文件。我们还提供了 Helm 图表，以便在 Kubernetes 中轻松安装。

## 生产环境

### Qdrant Cloud

您可以使用 [Qdrant Cloud](https://qdrant.to/cloud) 设置生产环境，该服务提供完全托管的 Qdrant 数据库。它提供水平和垂直扩展、一键安装和升级、监控、日志记录，以及备份和灾难恢复。有关更多信息，请参见 [Qdrant Cloud 文档](https://qdrant.tech/documentation/cloud/) 。

### Qdrant Kubernetes Operator

我们提供一个 Qdrant 企业操作器，用于 Kubernetes 安装，作为我们 [Qdrant 私有云](https://qdrant.tech/documentation/private-cloud/) 解决方案的一部分。欲了解更多信息，请 [使用此表单](https://qdrant.to/contact-us) 联系我们。

### Kubernetes

您可以使用现成的 [Helm Chart](https://helm.sh/docs/) 在您的 Kubernetes 集群中运行 Qdrant。虽然可以使用 Helm Chart 在分布式环境中部署 Qdrant，但它不具备与 Qdrant Cloud 解决方案或 Qdrant 私有云企业操作器相同级别的零停机升级、上下扩展、监控、日志记录以及备份和灾难恢复功能。相反，您必须 [自己](https://qdrant.tech/documentation/guides/distributed_deployment/) 管理和设置这些功能。对 Helm Chart 的支持仅限于社区支持。

下表为您提供了 Qdrant Cloud 与 Helm 图表之间功能差异的概述：

| 功能 | Qdrant Helm 图表 | Qdrant Cloud |
| --- | --- | --- |
| 开源 | ✅ |  |
| 仅限社区支持 | ✅ |  |
| 快速入门 | ✅ | ✅ |
| 垂直和水平扩展 | ✅ | ✅ |
| 具有细粒度访问控制的 API 密钥 | ✅ | ✅ |
| Qdrant 版本升级 | ✅ | ✅ |
| 支持传输和存储加密 | ✅ | ✅ |
| 零停机时间升级与优化的重启策略 |  | ✅ |
| 开箱即用，准备好投入生产 |  | ✅ |
| 降级时的数据丢失防护 |  | ✅ |
| 完整集群备份和灾难恢复 |  | ✅ |
| 自动分片重新平衡 |  | ✅ |
| 重新分片支持 |  | ✅ |
| 自动持久卷扩展 |  | ✅ |
| 高级遥测 |  | ✅ |
| 一键撤销 API 密钥 |  | ✅ |
| 在现有集群中使用新卷重新创建节点 |  | ✅ |
| 企业支持 |  | ✅ |

安装 helm 图表：

```bash
helm repo add qdrant https://qdrant.to/helm

helm install qdrant qdrant/qdrant
```

有关更多信息，请参见 [qdrant-helm](https://github.com/qdrant/qdrant-helm/tree/main/charts/qdrant) 的自述文件。

### Docker 和 Docker Compose

通常，我们建议在 Kubernetes 中运行 Qdrant，或在生产环境中使用 Qdrant Cloud。这使得设置高可用性和可扩展的 Qdrant 集群，以及备份和灾难恢复变得更加容易。

然而，您也可以使用 Docker 和 Docker Compose 在生产环境中运行 Qdrant，方法是按照 [Docker](https://qdrant.tech/documentation/guides/installation/#docker) 和 [Docker Compose](https://qdrant.tech/documentation/guides/installation/#docker-compose) 开发部分中的设置说明进行操作。此外，您还必须确保：

- 为您的数据使用高性能的 [持久存储](https://qdrant.tech/documentation/guides/installation/#storage)
- 为您的部署配置 [安全设置](https://qdrant.tech/documentation/guides/security/)
- 在多个节点上设置和配置 Qdrant，以实现高可用的 [分布式部署](https://qdrant.tech/documentation/guides/distributed_deployment/)
- 为您的 Qdrant 集群设置负载均衡器
- 为您的数据创建 [备份和灾难恢复策略](https://qdrant.tech/documentation/concepts/snapshots/)
- 将 Qdrant 与您的 [监控](https://qdrant.tech/documentation/guides/monitoring/) 和日志解决方案集成

## 开发

对于开发和测试，我们建议您在 Docker 中设置 Qdrant。我们还有不同的客户端库。

### Docker

开始使用 Qdrant 进行测试或开发的最简单方法是运行 Qdrant 容器镜像。最新版本始终可以在 [DockerHub](https://hub.docker.com/r/qdrant/qdrant/tags?page=1&ordering=last_updated) 上获取。

确保已安装并运行 [Docker](https://docs.docker.com/engine/install/) 、 [Podman](https://podman.io/docs/installation) 或您选择的容器运行时。以下说明使用 Docker。

拉取镜像：

```bash
docker pull qdrant/qdrant
```

在以下命令中，请根据您的 Docker 配置修改 `$(pwd)/path/to/data` 。然后使用更新后的命令来运行容器：

```bash
docker run -p 6333:6333 \
-v $(pwd)/path/to/data:/qdrant/storage \
qdrant/qdrant
```

使用此命令，您可以启动一个具有默认配置的 Qdrant 实例。它将所有数据存储在 `./path/to/data` 目录中。

默认情况下，Qdrant 使用端口 6333，因此在 [localhost:6333](http://localhost:6333/) 上您应该能看到欢迎消息。

要更改 Qdrant 配置，您可以覆盖生产配置：

```bash
docker run -p 6333:6333 \

    -v $(pwd)/path/to/data:/qdrant/storage \

    -v $(pwd)/path/to/custom_config.yaml:/qdrant/config/production.yaml \

    qdrant/qdrant
```

或者，您可以使用自己的 `custom_config.yaml` 配置文件：

```bash
docker run -p 6333:6333 \

    -v $(pwd)/path/to/data:/qdrant/storage \

    -v $(pwd)/path/to/custom_config.yaml:/qdrant/config/custom_config.yaml \

    qdrant/qdrant \

    ./qdrant --config-path config/custom_config.yaml
```

有关更多信息，请参见 [配置](https://qdrant.tech/documentation/guides/configuration/) 文档。

### Docker Compose

您也可以使用 [Docker Compose](https://docs.docker.com/compose/) 来运行 Qdrant。

这是一个单节点 Qdrant 集群的自定义 compose 文件示例：

```yaml
services:

  qdrant:

    image: qdrant/qdrant:latest

    restart: always

    container_name: qdrant

    ports:

      - 6333:6333

      - 6334:6334

    expose:

      - 6333

      - 6334

      - 6335

    configs:

      - source: qdrant_config

        target: /qdrant/config/production.yaml

    volumes:

      - ./qdrant_data:/qdrant/storage

configs:

  qdrant_config:

    content: |

      log_level: INFO
```

### 来自源头

Qdrant 是用 Rust 编写的，可以编译成二进制可执行文件。如果您想为特定的处理器架构编译 Qdrant，或者如果您不想使用 Docker，这种安装方法会很有帮助。

在编译之前，请确保已安装必要的库和 [Rust 工具链](https://www.rust-lang.org/tools/install) 。当前所需库的列表可以在 [Dockerfile](https://github.com/qdrant/qdrant/blob/master/Dockerfile) 中找到。

使用 Cargo 构建 Qdrant：

```bash
cargo build --release --bin qdrant
```

成功构建后，您可以在以下子目录中找到二进制文件 `./target/release/qdrant` 。

## 客户端库

除了服务，Qdrant 还提供了多种不同编程语言的客户端库。有关完整列表，请参阅我们的 [客户端库](https://qdrant.tech/documentation/interfaces/#client-libraries) 文档。