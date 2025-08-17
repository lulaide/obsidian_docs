---
source: "https://longhorn.io/docs/1.9.1/deploy/install/install-with-helm/"
tags:
  - "k8s"
---
在本节中，您将学习如何使用 Helm 安装 Longhorn。

### 先决条件

- Kubernetes 集群：确保每个节点满足 [安装要求](https://longhorn.io/docs/1.9.1/deploy/install/#installation-requirements) 。
- 您的工作站：安装 [Helm](https://helm.sh/docs/) v3.0 或更高版本。

> [Longhorn 命令行工具](https://longhorn.io/docs/1.9.1/advanced-resources/longhornctl/) 可用于检查 Longhorn 环境中的潜在问题。

### 安装 Longhorn

> **注意** ：
> 
> - Longhorn 的初始设置可以通过 [使用 Helm 选项自定义或编辑部署配置文件来找到。](https://longhorn.io/docs/1.9.1/advanced-resources/deploy/customizing-default-settings/#using-helm)
> - 对于 Kubernetes < v1.25，如果您的集群仍然启用 Pod 安全策略准入控制器，请将 helm 值 `enablePSP` 设置为 `true` 以安装 `longhorn-psp` PodSecurityPolicy 资源，这允许特权 Longhorn pod 启动。

1. 添加 Longhorn Helm 代码仓库：
	```shell
	helm repo add longhorn https://charts.longhorn.io
	```
2. 从代码仓库获取最新的图表：
	```shell
	helm repo update
	```
3. 在 `longhorn-system` 命名空间中安装 Longhorn。
	```shell
	helm install longhorn longhorn/longhorn --namespace longhorn-system --create-namespace --version 1.9.1
	```
4. 要确认部署成功，请运行：
	```bash
	kubectl -n longhorn-system get pod
	```
	结果应如下所示：
	```bash
	NAME                                                READY   STATUS    RESTARTS   AGE
	longhorn-ui-b7c844b49-w25g5                         1/1     Running   0          2m41s
	longhorn-manager-pzgsp                              1/1     Running   0          2m41s
	longhorn-driver-deployer-6bd59c9f76-lqczw           1/1     Running   0          2m41s
	longhorn-csi-plugin-mbwqz                           2/2     Running   0          100s
	csi-snapshotter-588457fcdf-22bqp                    1/1     Running   0          100s
	csi-snapshotter-588457fcdf-2wd6g                    1/1     Running   0          100s
	csi-provisioner-869bdc4b79-mzrwf                    1/1     Running   0          101s
	csi-provisioner-869bdc4b79-klgfm                    1/1     Running   0          101s
	csi-resizer-6d8cf5f99f-fd2ck                        1/1     Running   0          101s
	csi-provisioner-869bdc4b79-j46rx                    1/1     Running   0          101s
	csi-snapshotter-588457fcdf-bvjdt                    1/1     Running   0          100s
	csi-resizer-6d8cf5f99f-68cw7                        1/1     Running   0          101s
	csi-attacher-7bf4b7f996-df8v6                       1/1     Running   0          101s
	csi-attacher-7bf4b7f996-g9cwc                       1/1     Running   0          101s
	csi-attacher-7bf4b7f996-8l9sw                       1/1     Running   0          101s
	csi-resizer-6d8cf5f99f-smdjw                        1/1     Running   0          101s
	instance-manager-b34d5db1fe1e2d52bcfb308be3166cfc   1/1     Running   0          114s
	engine-image-ei-df38d2e5-cv6nc                      1/1     Running   0          114s
	```
5. 要启用对 Longhorn UI 的访问，您需要设置一个 Ingress 控制器。默认情况下，Longhorn UI 的认证未启用。有关使用基本认证创建 NGINX Ingress 控制器的信息，请参阅 [本节。](https://longhorn.io/docs/1.9.1/deploy/accessing-the-ui/longhorn-ingress)
6. 使用 [这些步骤](https://longhorn.io/docs/1.9.1/deploy/accessing-the-ui) 访问 Longhorn UI。