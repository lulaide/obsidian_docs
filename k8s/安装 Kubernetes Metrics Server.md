---
source: "https://kubernetes-sigs.github.io/metrics-server/?utm_source=chatgpt.com"
tags:
  - "k8s"
---
## Kubernetes 监控服务器

Metrics Server 是一个可扩展、高效的容器资源指标来源，专为 Kubernetes 内置的自动扩缩容管道而构建。

Metrics Server 从 Kubelet 收集资源指标，并通过 Metrics API 在 Kubernetes apiserver 中公开，以供水平 Pod 自动扩缩容器和垂直 Pod 自动扩缩容器使用。Metrics API 也可以被 `kubectl top` 访问，从而更容易调试自动扩缩容管道。

Metrics Server 并不适用于非自动扩缩容目的。例如，不要将其用于将指标转发到监控解决方案，或作为监控解决方案指标的来源。在这种情况下，请直接从 Kubelet `/metrics/resource` 端点收集指标。

Metrics Server 提供：

- 一个适用于大多数集群的单一部署（请参阅要求）
- 快速自动扩展，每 15 秒收集一次指标。
- 资源效率，每个集群节点使用 1 毫核心的 CPU 和 2MB 的内存。
- 可扩展支持最多 5000 个节点的集群。

## 用例

您可以使用 Metrics Server 来：

- 基于 CPU/内存的水平自动扩展（了解更多关于水平自动扩展的信息）
- 自动调整/建议容器所需的资源（了解更多关于垂直自动扩展的信息）

当您需要以下情况时，请不要使用 Metrics Server：

- 非 Kubernetes 集群
- 准确的资源使用指标来源
- 基于 CPU/内存以外的其他资源的水平自动扩展

对于不支持的用例，请查看完整的监控解决方案，如 Prometheus。

## 要求

Metrics Server 对集群和网络配置有特定要求。这些要求并不是所有集群发行版的默认设置。在使用 Metrics Server 之前，请确保您的集群发行版支持这些要求：

- Metrics Server 必须能够通过容器 IP 地址（如果启用了 hostNetwork，则为节点 IP）从 kube-apiserver 访问。
- kube-apiserver 必须启用聚合层。
- 节点必须启用 Webhook 认证和授权。
- Kubelet 证书需要由集群证书授权机构签名（或者通过将 `--kubelet-insecure-tls` 传递给 Metrics Server 来禁用证书验证）
- 容器运行时必须实现容器指标 RPC（或者支持 cAdvisor）

## 安装

可以通过运行以下命令安装最新的 Metrics Server 版本：

```shell
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

以前版本的安装说明可以在 Metrics Server 发布页面找到。

兼容性矩阵：

| 度量服务器 | 度量 API 组/版本 | 支持的 Kubernetes 版本 |
| --- | --- | --- |
| 0.6.x | `metrics.k8s.io/v1beta1` | \*1.19+ |
| 0.5.x | `metrics.k8s.io/v1beta1` | \*1.8+ |
| 0.4.x | `metrics.k8s.io/v1beta1` | \*1.8+ |
| 0.3.x | `metrics.k8s.io/v1beta1` | 1.8-1.21 |

\*对于 <1.16 需要传递 `--authorization-always-allow-paths=/livez,/readyz` 命令行标志

### 高可用性

最新的 Metrics Server 版本可以通过运行以下命令以高可用模式安装：

```shell
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/high-availability.yaml
```

请注意，此配置要求集群至少有 2 个节点，以便可以调度 Metrics Server。

此外，为了最大化此高可用配置的效率，建议将 `--enable-aggregator-routing=true` CLI 标志添加到 kube-apiserver，以便将发送到 Metrics Server 的请求在两个实例之间进行负载均衡。

## 安全上下文

Metrics Server 需要 `CAP_NET_BIND_SERVICE` 能力，以便以非根用户身份绑定到特权端口。如果您在使用 PSP 或其他机制限制 Pod 能力的环境中运行 Metrics Server，请确保允许 Metrics Server 使用此能力。即使您使用 `--secure-port` 标志将 Metrics Server 绑定的端口更改为非特权端口，这一点也适用。

## 扩展

从 v0.5.0 开始，Metrics Server 提供了默认的资源请求，这应该能保证大多数集群配置（最多 100 个节点）的良好性能：

- 100m 核心 CPU
- 200MiB 内存

Metrics Server 的资源使用依赖于多个独立的维度，形成了一个可扩展性范围。默认的 Metrics Server 配置应适用于不超过以下任何阈值的集群：

| 数量 | 命名空间阈值 | 集群阈值 |
| --- | --- | --- |
| #Nodes | 不适用 | 100 |
| #每个节点的 Pods | 70 | 70 |
| \# 带有 HPA 的部署 | 100 | 100 |

资源可以根据集群中的节点数量成比例地进行调整。对于超过 100 个节点的集群，额外分配：

- 每个节点 1m 核心
- 每个节点 2MiB 内存

您可以使用相同的方法来降低资源请求，但在某个边界上，这可能会影响其他可扩展性维度，例如每个节点的最大 Pod 数量。

### 配置

根据您的集群设置，您可能还需要更改传递给 Metrics Server 容器的标志。最有用的标志：

- `--kubelet-preferred-address-types` - 在确定连接到特定节点的地址时使用的节点地址类型的优先级（默认值 \[Hostname,InternalDNS,InternalIP,ExternalDNS,ExternalIP\]）
- `--kubelet-insecure-tls` - 不验证 Kubelet 提供的服务证书的 CA。仅用于测试目的。
- `--requestheader-client-ca-file` - 指定一个根证书包，用于验证传入请求的客户端证书。

您可以通过运行以下命令获取 Metrics Server 配置标志的完整列表：

```shell
docker run --rm k8s.gcr.io/metrics-server/metrics-server:v0.5.0 --help
```

#### Helm Chart

此 Helm 图表可以在您的集群中部署 metric-server 服务。

## 设计

Metrics Server 是 Kubernetes 监控架构中核心指标管道的一个组件。

有关更多信息，请参见：

- [指标 API 设计](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/instrumentation/resource-metrics-api.md)
- [Metrics Server 设计](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/instrumentation/metrics-server.md)

## 有问题吗？

在发布问题之前，请先查看常见问题和已知问题。

了解如何在社区页面与 Kubernetes 社区互动。

您可以通过以下方式联系该项目的维护者：

- [Slack 频道](https://kubernetes.slack.com/messages/sig-instrumentation)
- [邮件列表](https://groups.google.com/forum/#!forum/kubernetes-sig-instrumentation)

该项目由 SIG 仪器组维护

### 行为准则

参与 Kubernetes 社区受 Kubernetes 行为准则的管理。