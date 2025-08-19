---
source: https://cert-manager.io/docs/installation/helm/
tags:
  - cert-manager
  - k8s
---
## 使用 Helm 安装

请务必不要将 cert-manager 嵌入为其他 Helm 图表的子图表；cert-manager 管理您集群中的非命名空间资源，必须小心确保它仅安装一次。

==注：个人感觉使用旧版 helm 仓库安装更快一点==
### 先决条件

- 安装 Helm 版本 3 或更高版本。
- 安装支持的 Kubernetes 或 OpenShift 版本。
- 如果您在云平台上使用 Kubernetes，请阅读与 Kubernetes 平台提供商的兼容性。

### 使用 Helm 安装 cert-manager

cert-manager 作为 OCI Helm 图表和 Helm 仓库提供。我们建议对任何最近版本的 cert-manager 使用 OCI Helm 图表。

非常旧版本的 cert-manager（早于 v1.12）仅在遗留的 Helm 仓库中正式提供。本文档的其余部分假设使用 OCI 注册表。

#### 从 OCI 注册表安装

为了简化，cert-manager Helm 图表与 cert-manager 容器镜像发布在同一个 OCI 注册表中，位于 `quay.io/jetstack` 。

最新的 cert-manager 图表可在以下位置获取：

```bash
oci://quay.io/jetstack/charts/cert-manager:v1.18.2
```

您可以直接使用 Helm 安装命令安装 cert-manager，无需其他设置：

```bash
helm install \
  cert-manager oci://quay.io/jetstack/charts/cert-manager \
  --version v1.18.2 \
  --namespace cert-manager \
  --create-namespace \
  --set crds.enabled=true
```

（可选）安装时验证图表上的签名，这需要先从该网站下载 GPG 密钥环。

```bash
curl -LO https://cert-manager.io/public-keys/cert-manager-keyring-2021-09-20-1020CF3C033D4F35BAE1C19E1226061C665DF13E.gpg

helm install \
  cert-manager oci://quay.io/jetstack/charts/cert-manager \
  --version v1.18.2 \
  --namespace cert-manager \
  --create-namespace \
  --verify \
  --keyring ./cert-manager-keyring-2021-09-20-1020CF3C033D4F35BAE1C19E1226061C665DF13E.gpg \
  --set crds.enabled=true
```

#### 从旧版 Helm 仓库安装

cert-manager 的 Helm 图表历史上已发布到 Jetstack 仓库，地址为 `https://charts.jetstack.io` 。

该仓库仍然可用，目前没有更改的计划，但建议使用 OCI Helm 图表来获取 cert-manager 的最新版本。

要使用旧版仓库而不是 OCI 注册表，您需要将 Jetstack Helm 仓库添加到本地 Helm 客户端，并使用稍微不同的 Helm 安装命令。以下提供了两者的示例。

```bash
helm repo add jetstack https://charts.jetstack.io --force-update

helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.18.2 \
  --set crds.enabled=true
```
