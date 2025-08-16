---
source: https://sealos.run/docs/k8s/quick-start/deploy-kubernetes
description: 使用Sealos快速部署Kubernetes集群，支持在线和离线安装，适用于amd64和arm64架构。轻松管理节点，安装分布式应用，支持Containerd和Docker运行时。
tags:
  - k8s
---
## 安装 K8s 集群

使用Sealos快速部署Kubernetes集群，支持在线和离线安装，适用于amd64和arm64架构。轻松管理节点，安装分布式应用，支持Containerd和Docker运行时。

分享到：

Sealos 支持安装 `amd64` 和 `arm64` 架构的 K8s 集群。

## 下载 sealos 命令行工具

```
echo "deb [trusted=yes] https://apt.fury.io/labring/ /" | sudo tee /etc/apt/sources.list.d/labring.list
sudo apt update
sudo apt install sealos -y
```

## 安装 K8s 单机版

```
sealos run registry.cn-shanghai.aliyuncs.com/labring/kubernetes:v1.29.9 registry.cn-shanghai.aliyuncs.com/labring/helm:v3.18.4 registry.cn-shanghai.aliyuncs.com/labring/cilium:v1.13.4 --single
```

==注：这里我更改了 helm 的版本号，官方文档上的 helm 版本过低，部署可能出现问题==

## 安装 K8s 集群

```
sealos run registry.cn-shanghai.aliyuncs.com/labring/kubernetes:v1.29.9 registry.cn-shanghai.aliyuncs.com/labring/helm:v3.18.4 registry.cn-shanghai.aliyuncs.com/labring/cilium:v1.16.1 \

     --masters 192.168.64.2,192.168.64.22,192.168.64.20 \

     --nodes 192.168.64.21,192.168.64.19 -p [your-ssh-passwd]
```

注意：labring/helm 应当在 labring/cilium 之前。

参数说明：

| 参数名 | 参数值示例 | 参数说明 |
| --- | --- | --- |
| \--masters | 192.168.0.2 | K8s master 节点地址列表 |
| \--nodes | 192.168.0.3 | K8s node 节点地址列表 |
| \--ssh-passwd | \[your-ssh-passwd\] | ssh 登录密码 |
| kubernetes | labring/kubernetes:v1.25.0 | K8s 集群镜像 |

在干净的服务器上直接执行上面命令，不要做任何多余操作即可安装一个高可用 K8s 集群。

## 安装各种分布式应用

```
sealos run registry.cn-shanghai.aliyuncs.com/labring/helm:v3.9.4 # install helm

sealos run registry.cn-shanghai.aliyuncs.com/labring/openebs:v3.9.0 # install openebs

sealos run registry.cn-shanghai.aliyuncs.com/labring/minio-operator:v4.5.5 registry.cn-shanghai.aliyuncs.com/labring/ingress-nginx:4.1.0
```

这样高可用的 Minio 等应用都有了，不用关心所有的依赖问题。

## 增加 K8s 节点

增加 node 节点：

```
sealos add --nodes 192.168.64.21,192.168.64.19
```

增加 master 节点：

```
sealos add --masters 192.168.64.21,192.168.64.19
```

## 删除 K8s 节点

删除 node 节点：

```
sealos delete --nodes 192.168.64.21,192.168.64.19
```

删除 master 节点：

```
sealos delete --masters 192.168.64.21,192.168.64.19
```

## 清理 K8s 集群

```
sealos reset
```

## 离线安装 K8s

离线环境只需要提前导入镜像，其它步骤与在线安装一致。

首先在有网络的环境中导出集群镜像：

```
sealos pull registry.cn-shanghai.aliyuncs.com/labring/kubernetes:v1.29.9

sealos save -o kubernetes.tar registry.cn-shanghai.aliyuncs.com/labring/kubernetes:v1.29.9
```

### 导入镜像并安装

将 kubernetes.tar 拷贝到离线环境，使用 load 命令导入镜像即可：

```
sealos load -i kubernetes.tar
```

剩下的安装方式与在线安装的步骤一致：

```
sealos images # 查看集群镜像是否导入成功

sealos run registry.cn-shanghai.aliyuncs.com/labring/kubernetes:v1.29.9  # 单机安装，集群安装同理
```

### 快速启动 K8s 集群

也可以不用 load 命令导入镜像，直接使用以下命令即可安装 K8s：

```
sealos run kubernetes.tar # 单机安装，集群安装同理
```

## 集群镜像版本支持说明

### 支持 Containerd 的 K8s

推荐使用 Containerd 作为容器运行时 (CRI) 的集群镜像版本，Containerd 是一种轻量级、高性能的容器运行时，与 Docker 兼容。使用 Containerd 的 Kubernetes 镜像可以提供更高的性能和资源利用率。以下是支持 Containerd 的集群镜像版本支持说明：

| K8s 版本 | Sealos 版本 | CRI 版本 | 集群镜像版本 |
| --- | --- | --- | --- |
| `<1.25` | `>=v4.0.0` | v1alpha2 | labring/kubernetes:v1.24.0 |
| `>=1.25` | `>=v4.1.0` | v1alpha2 | labring/kubernetes:v1.25.0 |
| `>=1.26` | `>=v4.1.4-rc3` | v1 | labring/kubernetes:v1.26.0 |
| `>=1.27` | `>=v4.2.0-alpha3` | v1 | labring/kubernetes:v1.27.0 |
| `>=1.28` | `>=v5.0.0` | v1 | labring/kubernetes:v1.28.0 |

根据 Kubernetes 版本的不同，您可以选择不同的 Sealos 版本和 CRI 版本。例如，如果您要使用 Kubernetes v1.26.0 版本，您可以选择 sealos v4.1.4-rc3 及更高版本，并使用 v1 CRI 版本。

### 支持 Docker 的 K8s

当然，你也可以选择使用 Docker 作为容器运行时，以下是支持 Docker 的集群镜像版本支持说明：

| K8s 版本 | Sealos 版本 | CRI 版本 | 集群镜像版本 |
| --- | --- | --- | --- |
| `<1.25` | `>=v4.0.0` | v1alpha2 | labring/kubernetes-docker:v1.24.0 |
| `>=1.25` | `>=v4.1.0` | v1alpha2 | labring/kubernetes-docker:v1.25.0 |
| `>=1.26` | `>=v4.1.4-rc3` | v1 | labring/kubernetes-docker:v1.26.0 |
| `>=1.27` | `>=v4.2.0-alpha3` | v1 | labring/kubernetes-docker:v1.27.0 |
| `>=1.28` | `>=v5.0.0` | v1 | labring/kubernetes-docker:v1.28.0 |

与支持 Containerd 的 Kubernetes 镜像类似，您可以根据 Kubernetes 版本的不同选择不同的 Sealos 版本和 CRI 版本。例如，如果您要使用 Kubernetes v1.26.0 版本，您可以选择 sealos v4.1.4-rc3 及更高版本，并使用 v1 CRI 版本。

### 支持 Containerd 的 k3s

| K3s 版本 | Sealos 版本 | 集群镜像版本 |
| --- | --- | --- |
| `>=1.24` | `>=v5.0.0` | labring/k3s:v1.24.0 |
