---
source: "https://openebs.io/docs/quickstart-guide/installation"
tags:
  - "k8s"
---
本指南将帮助您设置、定制和安装 OpenEBS，并使用 OpenEBS 卷来运行您的 Kubernetes 有状态工作负载。如果您是首次在 Kubernetes 中运行有状态工作负载，您需要熟悉 [Kubernetes 存储概念](https://openebs.io/docs/concepts/basics) 。

## 如何设置和使用 OpenEBS？

OpenEBS 无缝集成到 Kubernetes 管理员和用户围绕 Kubernetes 的整体工作流工具中。

OpenEBS 工作流很好地融入了 Kubernetes 引入的协调模式，为下图所示的声明式存储控制平面铺平了道路：

![control plane overview](https://openebs.io/docs/assets/images/control-plane-overview-614ddee12776e70dfba057893807f080.svg)

## 通过 Helm 安装

验证 helm 是否已安装并且 helm 仓库已更新。您需要 helm 3.2 或更高版本。

1. 设置 helm 仓库。

```markdown
helm repo add openebs https://openebs.github.io/openebs
helm repo update
```

OpenEBS 在安装过程中提供了多个自定义选项，例如：

- 指定存储 hostpath 卷数据的目录或
- 指定应部署 OpenEBS 组件的节点等。
1. 使用默认值安装 OpenEBS helm chart。

```markdown
helm install openebs --namespace openebs openebs/openebs --create-namespace
```

上述命令将在 `openebs` 命名空间中安装 OpenEBS 本地 PV Hostpath、OpenEBS 本地 PV LVM、OpenEBS 本地 PV ZFS 和 OpenEBS 复制存储组件，图表名称为 `openebs` 。

如果您使用的是自定义的 Kubelet 位置或使用自定义 Kubelet 位置的 Kubernetes 发行版，则需要在安装时修改 Helm 值中的 Kubelet 目录。这可以通过在 Helm 安装命令中使用 `--set` 标志选项来实现，如下所示：

- 对于本地 PV LVM

```markdown
--set lvm-localpv.lvmNode.kubeletDir=<your-directory-path>
```

- 对于本地 PV ZFS

```markdown
--set zfs-localpv.zfsNode.kubeletDir=<your-directory-path>
```

- 对于复制的 PV Mayastor

```markdown
--set mayastor.csi.node.kubeletDir=<your-directory-path>
```

具体来说：

- 对于 **MicroK8s** ，Kubelet 目录必须更新为 `/var/snap/microk8s/common/var/lib/kubelet/` ，通过将默认的 `/var/lib/kubelet/` 替换为 `/var/snap/microk8s/common/var/lib/kubelet/` 。
- 对于 **k0s** ，默认的 Kubelet 目录（ `/var/lib/kubelet` ）必须更改为 `/var/lib/k0s/kubelet/` 。
- 对于 **RancherOS** ，默认的 Kubelet 目录(`/var/lib/kubelet`)必须更改为 `/opt/rke/var/lib/kubelet/` 。
1. 要查看图表并获取输出，请使用以下命令：

**命令**

```markdown
helm ls -n openebs
```

**示例输出**

```markdown
NAME     NAMESPACE   REVISION  UPDATED                                   STATUS     CHART           APP VERSION
openebs  openebs     1         2025-05-25 09:13:00.903321318 +0000 UTC   deployed   openebs-4.3.2   4.3.2
```

## 验证 OpenEBS 安装

### 验证 Pods

#### 默认安装

列出 `<openebs>` 命名空间中的 pods

```markdown
kubectl get pods -n openebs
```

在 OpenEBS 成功安装后，您应该看到如下示例输出：

```markdown
NAME                                              READY   STATUS    RESTARTS   AGE
openebs-agent-core-674f784df5-7szbm               2/2     Running   0          11m
openebs-agent-ha-node-nnkmv                       1/1     Running   0          11m
openebs-agent-ha-node-pvcrr                       1/1     Running   0          11m
openebs-agent-ha-node-rqkkk                       1/1     Running   0          11m
openebs-api-rest-79556897c8-b824j                 1/1     Running   0          11m
openebs-csi-controller-b5c47d49-5t5zd             6/6     Running   0          11m
openebs-csi-node-flq49                            2/2     Running   0          11m
openebs-csi-node-k8d7h                            2/2     Running   0          11m
openebs-csi-node-v7jfh                            2/2     Running   0          11m
openebs-etcd-0                                    1/1     Running   0          11m
openebs-etcd-1                                    1/1     Running   0          11m
openebs-etcd-2                                    1/1     Running   0          11m
openebs-io-engine-7t6tf                           2/2     Running   0          11m
openebs-io-engine-9df6r                           2/2     Running   0          11m
openebs-io-engine-rqph4                           2/2     Running   0          11m
openebs-localpv-provisioner-6ddf7c7978-4fkvs      1/1     Running   0          11m
openebs-loki-0                                    1/1     Running   0          11m
openebs-lvm-localpv-controller-7b6d6b4665-fk78q   5/5     Running   0          11m
openebs-lvm-localpv-node-mcch4                    2/2     Running   0          11m
openebs-lvm-localpv-node-pdt88                    2/2     Running   0          11m
openebs-lvm-localpv-node-r9jn2                    2/2     Running   0          11m
openebs-nats-0                                    3/3     Running   0          11m
openebs-nats-1                                    3/3     Running   0          11m
openebs-nats-2                                    3/3     Running   0          11m
openebs-obs-callhome-854bc967-5f879               2/2     Running   0          11m
openebs-operator-diskpool-5586b65c-cwpr8          1/1     Running   0          11m
openebs-promtail-2vrzk                            1/1     Running   0          11m
openebs-promtail-mwxk8                            1/1     Running   0          11m
openebs-promtail-w7b8k                            1/1     Running   0          11m
openebs-zfs-localpv-controller-f78f7467c-blr7q    5/5     Running   0          11m
openebs-zfs-localpv-node-h46m5                    2/2     Running   0          11m
openebs-zfs-localpv-node-svfgq                    2/2     Running   0          11m
openebs-zfs-localpv-node-wm9ks                    2/2     Running   0          11m
```

#### 禁用复制存储的安装

列出 `<openebs>` 命名空间中的 Pods

```markdown
kubectl get pods -n openebs
```

在成功安装 OpenEBS 后，您应该看到如下示例输出：

```markdown
NAME                                              READY   STATUS    RESTARTS   AGE
openebs-localpv-provisioner-6ddf7c7978-jsstg      1/1     Running   0          3m9s
openebs-lvm-localpv-controller-7b6d6b4665-wfw64   5/5     Running   0          3m9s
openebs-lvm-localpv-node-62lnq                    2/2     Running   0          3m9s
openebs-lvm-localpv-node-lhndx                    2/2     Running   0          3m9s
openebs-lvm-localpv-node-tlcqv                    2/2     Running   0          3m9s
openebs-zfs-localpv-controller-f78f7467c-k7ldb    5/5     Running   0          3m9s
openebs-zfs-localpv-node-5mwbz                    2/2     Running   0          3m9s
openebs-zfs-localpv-node-g45ft                    2/2     Running   0          3m9s
openebs-zfs-localpv-node-g77g6                    2/2     Running   0          3m9s
```

### 验证存储类

列出存储类以检查 OpenEBS 是否已安装默认的 StorageClasses。

```markdown
kubectl get sc
```

在成功安装后，您应该创建以下 StorageClasses：

```markdown
NAME                       PROVISIONER               RECLAIMPOLICY   VOLUMEBINDINGMODE    ALLOWVOLUMEEXPANSION 
mayastor-etcd-localpv      openebs.io/local          Delete          WaitForFirstConsumer false
mayastor-loki-localpv      openebs.io/local          Delete          WaitForFirstConsumer false
openebs-hostpath           openebs.io/local          Delete          WaitForFirstConsumer false
openebs-single-replica     io.openebs.csi-mayastor   Delete          Immediate            true
```

## 安装后注意事项

为了测试您的 OpenEBS 安装，您可以使用在 [本地存储用户指南](https://openebs.io/docs/user-guides/local-storage-user-guide/local-pv-hostpath/hostpath-overview) 中提到的 `openebs-hostpath` 来在主机路径上配置本地 PV。

您可以按照以下用户指南，使用每个引擎来利用节点上可用的存储设备，而不是使用 `/var/openebs` 目录来保存数据。

- [本地存储用户指南](https://openebs.io/docs/user-guides/local-storage-user-guide/local-pv-hostpath/hostpath-overview)
- [复制存储用户指南](https://openebs.io/docs/user-guides/replicated-storage-user-guide/replicated-pv-mayastor/rs-overview)

## 支持

如果您遇到问题或有疑问，请提交一个 [Github 问题](https://github.com/openebs/openebs/issues/new) ，或在 [#openebs Kubernetes Slack 频道](https://kubernetes.slack.com/messages/openebs/) 与我们联系。

## 另见

- [部署](https://openebs.io/docs/quickstart-guide/deployment)
- [OpenEBS 架构](https://openebs.io/docs/concepts/architecture)
- [OpenEBS 使用案例和示例](https://openebs.io/docs/introduction-to-openebs/usecases)
- [故障排除](https://openebs.io/docs/troubleshooting/)