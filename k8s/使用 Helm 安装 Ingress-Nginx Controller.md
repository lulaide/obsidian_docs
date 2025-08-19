---
source: "https://kubernetes.github.io/ingress-nginx/deploy/#quick-start"
tags:
  - "k8s"
---
## 安装指南

安装 Ingress-Nginx Controller 有多种方法：

- 使用 [Helm](https://helm.sh/) ，通过项目代码仓库的图表；
- 使用 `kubectl apply` ，通过 YAML 清单；
- 使用特定的附加组件（例如 [minikube](https://kubernetes.github.io/ingress-nginx/deploy/#minikube) 或 [MicroK8s](https://kubernetes.github.io/ingress-nginx/deploy/#microk8s) ）。

在大多数 Kubernetes 集群上，ingress 控制器将无需额外配置即可工作。如果您想尽快开始，可以查看 [快速入门](https://kubernetes.github.io/ingress-nginx/deploy/#quick-start) 指南。然而，在许多环境中，您可以通过启用额外功能来提高性能或获得更好的日志。我们建议您查看 [特定环境的说明](https://kubernetes.github.io/ingress-nginx/deploy/#environment-specific-instructions) ，以获取有关为您的特定环境或云提供商优化 ingress 控制器的详细信息。


## 快速开始

**如果您有 Helm，** 您可以使用以下命令部署 ingress 控制器：

```js
helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace
```

它将在 `ingress-nginx` 命名空间中安装控制器，如果该命名空间尚不存在，则会创建它。

信息

此命令是 *幂等的* ：

- 如果未安装 ingress 控制器，它将安装它，
- 如果 ingress 控制器已经安装，它将升级它。

**如果您想要在使用 Helm 安装时可以设置的完整值列表，** 请运行：

```js
helm show values ingress-nginx --repo https://kubernetes.github.io/ingress-nginx
```

**如果您没有 Helm** ，或者如果您更喜欢使用 YAML 清单，您可以运行以下命令：

```js
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.13.1/deploy/static/provider/cloud/deploy.yaml
```

上面命令中的 YAML 清单是通过 `helm template` 生成的，因此您最终将获得几乎与使用 Helm 安装控制器相同的资源。

注意

如果您正在运行旧版本的 Kubernetes（1.18 或更早版本），请阅读 [本段](https://kubernetes.github.io/ingress-nginx/deploy/#running-on-kubernetes-versions-older-than-119) 以获取具体说明。由于 API 已被弃用，默认清单可能无法在您的集群上工作。每个提供者的子文件夹中提供了支持的 Kubernetes 版本的具体清单。

### 防火墙配置

要检查您的 ingress-nginx 安装使用了哪些端口，请查看 `kubectl -n ingress-nginx get pod -o yaml` 的输出。一般来说，您需要：

- 在所有运行 Kubernetes 节点的主机之间开放 8443 端口。这用于 ingress-nginx [准入控制器](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/) 。
- 在指向您应用程序的 DNS 的 Kubernetes 节点上，端口 80（用于 HTTP）和/或 443（用于 HTTPS）对公众开放。

### 预检

在 `ingress-nginx` 命名空间中应该启动几个 Pod：

```js
kubectl get pods --namespace=ingress-nginx
```

过一段时间后，它们应该都在运行。以下命令将等待入口控制器 pod 启动、运行并准备就绪：

```js
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=120s
```

### 本地测试

让我们创建一个简单的网络服务器及其相关服务：

```js
kubectl create deployment demo --image=httpd --port=80
kubectl expose deployment demo
```

然后创建一个 ingress 资源。以下示例使用一个映射到 `localhost` 的主机：

```js
kubectl create ingress demo-localhost --class=nginx \
  --rule="demo.localdev.me/*=demo:80"
```

现在，将本地端口转发到入口控制器：

```js
kubectl port-forward --namespace=ingress-nginx service/ingress-nginx-controller 8080:80
```

信息

关于 DNS 和网络连接的说明。本文件假设用户了解使用 ingress 时涉及的 DNS 和网络路由方面。上述提到的端口转发是演示 ingress 工作原理的最简单方法。上面的“kubectl port-forward...”命令将本地主机 tcp/ip 栈上的端口号 8080 转发到由 ingress-nginx 控制器安装创建的服务的端口号 80。因此，现在发送到本地主机端口号 8080 的流量将到达 ingress-controller 服务的端口号 80。端口转发不适用于生产环境的用例。但在这里，我们使用端口转发来模拟一个来自集群外部的 HTTP 请求，以到达暴露于接收来自集群外部流量的 ingress-nginx 控制器的服务。

[这个问题](https://github.com/kubernetes/ingress-nginx/issues/10014#issuecomment-1567791549described) 展示了一个典型的 DNS 问题及其解决方案。

此时，您可以使用 curl 访问您的部署；

```js
curl --resolve demo.localdev.me:8080:127.0.0.1 http://demo.localdev.me:8080
```

您应该看到一个包含文本的 HTML 响应，例如 **"它工作！"** 。

### 在线测试

如果您的 Kubernetes 集群是一个支持 `LoadBalancer` 类型服务的“真实”集群，它将为入口控制器分配一个外部 IP 地址或 FQDN。

您可以使用以下命令查看该 IP 地址或 FQDN：

```js
kubectl get service ingress-nginx-controller --namespace=ingress-nginx
```

它将是 `EXTERNAL-IP` 字段。如果该字段显示 `<pending>` ，这意味着您的 Kubernetes 集群无法配置负载均衡器（通常是因为它不支持 `LoadBalancer` 类型的服务）。

一旦您获得了外部 IP 地址（或 FQDN），请设置一个指向它的 DNS 记录。然后您可以创建一个 ingress 资源。以下示例假设您已为 `www.demo.io` 设置了 DNS 记录：

```js
kubectl create ingress demo --class=nginx \
  --rule="www.demo.io/*=demo:80"
```

或者，上述命令可以重写为以下形式，用于 `--rule` 命令及以下内容。

```js
kubectl create ingress demo --class=nginx \
  --rule www.demo.io/=demo:80
```

然后，当您连接到 [http://www.demo.io/](http://www.demo.io/) 时，您应该能够看到“它工作！”页面。恭喜您，您正在提供一个托管在 Kubernetes 集群上的公共网站！ 🎉

## 特定环境的说明

### 本地开发集群

#### minikube

可以通过 minikube 的插件系统安装 ingress 控制器：

```js
minikube addons enable ingress
```

#### MicroK8s

可以通过 MicroK8s 的插件系统安装 ingress 控制器：

```js
microk8s enable ingress
```

Please check the MicroK8s [documentation page](https://microk8s.io/docs/addon-ingress) for details.

#### Docker Desktop

Kubernetes is available in Docker Desktop:

- Mac, from [version 18.06.0-ce](https://docs.docker.com/docker-for-mac/release-notes/#stable-releases-of-2018)
- Windows, from [version 18.06.0-ce](https://docs.docker.com/docker-for-windows/release-notes/#docker-community-edition-18060-ce-win70-2018-07-25)

First, make sure that Kubernetes is enabled in the Docker settings. The command `kubectl get nodes` should show a single node called `docker-desktop`.

The ingress controller can be installed on Docker Desktop using the default [quick start](https://kubernetes.github.io/ingress-nginx/deploy/#quick-start) instructions.

On most systems, if you don't have any other service of type `LoadBalancer` bound to port 80, the ingress controller will be assigned the `EXTERNAL-IP` of `localhost`, which means that it will be reachable on localhost:80. If that doesn't work, you might have to fall back to the `kubectl port-forward` method described in the [local testing section](https://kubernetes.github.io/ingress-nginx/deploy/#local-testing).

#### Rancher Desktop

Rancher Desktop provides Kubernetes and Container Management on the desktop. Kubernetes is enabled by default in Rancher Desktop.

Rancher Desktop uses K3s under the hood, which in turn uses Traefik as the default ingress controller for the Kubernetes cluster. To use Ingress-Nginx Controller in place of the default Traefik, disable Traefik from Preference > Kubernetes menu.

Once traefik is disabled, the Ingress-Nginx Controller can be installed on Rancher Desktop using the default [quick start](https://kubernetes.github.io/ingress-nginx/deploy/#quick-start) instructions. Follow the instructions described in the [local testing section](https://kubernetes.github.io/ingress-nginx/deploy/#local-testing) to try a sample.

### Cloud deployments

If the load balancers of your cloud provider do active healthchecks on their backends (most do), you can change the `externalTrafficPolicy` of the ingress controller Service to `Local` (instead of the default `Cluster`) to save an extra hop in some cases. If you're installing with Helm, this can be done by adding `--set controller.service.externalTrafficPolicy=Local` to the `helm install` or `helm upgrade` command.

Furthermore, if the load balancers of your cloud provider support the PROXY protocol, you can enable it, and it will let the ingress controller see the real IP address of the clients. Otherwise, it will generally see the IP address of the upstream load balancer. This must be done both in the ingress controller (with e.g. `--set controller.config.use-proxy-protocol=true`) and in the cloud provider's load balancer configuration to function correctly.

In the following sections, we provide YAML manifests that enable these options when possible, using the specific options of various cloud providers.

#### AWS

In AWS, we use a Network load balancer (NLB) to expose the Ingress-Nginx Controller behind a Service of `Type=LoadBalancer`.

Info

The provided templates illustrate the setup for legacy in-tree service load balancer for AWS NLB. AWS provides the documentation on how to use [Network load balancing on Amazon EKS](https://docs.aws.amazon.com/eks/latest/userguide/network-load-balancing.html) with [AWS Load Balancer Controller](https://github.com/kubernetes-sigs/aws-load-balancer-controller).

##### Network Load Balancer (NLB)

```js
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.13.1/deploy/static/provider/aws/deploy.yaml
```

##### TLS termination in AWS Load Balancer (NLB)

By default, TLS is terminated in the ingress controller. But it is also possible to terminate TLS in the Load Balancer. This section explains how to do that on AWS using an NLB.

1. Download the [deploy.yaml](https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.13.1/deploy/static/provider/aws/nlb-with-tls-termination/deploy.yaml) template
	```js
	wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.13.1/deploy/static/provider/aws/nlb-with-tls-termination/deploy.yaml
	```
2. Edit the file and change the VPC CIDR in use for the Kubernetes cluster:
	```js
	proxy-real-ip-cidr: XXX.XXX.XXX/XX
	```
3. Change the AWS Certificate Manager (ACM) ID as well:
	```js
	arn:aws:acm:us-west-2:XXXXXXXX:certificate/XXXXXX-XXXXXXX-XXXXXXX-XXXXXXXX
	```
4. Deploy the manifest:
	```js
	kubectl apply -f deploy.yaml
	```

##### NLB Idle Timeouts

The default idle timeout value for TCP flows is 350 seconds and [can be modified to any value between 60-6000 seconds.](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/network-load-balancers.html#connection-idle-timeout)

For this reason, you need to ensure the [keepalive\_timeout](https://nginx.org/en/docs/http/ngx_http_core_module.html#keepalive_timeout) value is configured less than your configured idle timeout to work as expected.

By default, NGINX `keepalive_timeout` is set to `75s`.

More information with regard to timeouts can be found in the [official AWS documentation](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/network-load-balancers.html#connection-idle-timeout)

#### GCE-GKE

First, your user needs to have `cluster-admin` permissions on the cluster. This can be done with the following command:

```js
kubectl create clusterrolebinding cluster-admin-binding \
  --clusterrole cluster-admin \
  --user $(gcloud config get-value account)
```

Then, the ingress controller can be installed like this:

```js
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.13.1/deploy/static/provider/cloud/deploy.yaml
```

Warning

For private clusters, you will need to either add a firewall rule that allows master nodes access to port `8443/tcp` on worker nodes, or change the existing rule that allows access to port `80/tcp`, `443/tcp` and `10254/tcp` to also allow access to port `8443/tcp`. More information can be found in the [Official GCP Documentation](https://cloud.google.com/load-balancing/docs/tcp/setting-up-tcp#config-hc-firewall).

See the [GKE documentation](https://cloud.google.com/kubernetes-engine/docs/how-to/private-clusters#add_firewall_rules) on adding rules and the [Kubernetes issue](https://github.com/kubernetes/kubernetes/issues/79739) for more detail.

Proxy-protocol is supported in GCE check the [Official Documentations on how to enable.](https://cloud.google.com/load-balancing/docs/tcp/setting-up-tcp#proxy-protocol)

#### Azure

```js
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.13.1/deploy/static/provider/cloud/deploy.yaml
```

More information with regard to Azure annotations for ingress controller can be found in the [official AKS documentation](https://docs.microsoft.com/en-us/azure/aks/ingress-internal-ip#create-an-ingress-controller).

#### Digital Ocean

```js
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.13.1/deploy/static/provider/do/deploy.yaml
```
- By default the service object of the ingress-nginx-controller for Digital-Ocean, only configures one annotation. Its this one `service.beta.kubernetes.io/do-loadbalancer-enable-proxy-protocol: "true"`. While this makes the service functional, it was reported that the Digital-Ocean LoadBalancer graphs shows `no data`, unless a few other annotations are also configured. Some of these other annotations require values that can not be generic and hence not forced in a out-of-the-box installation. These annotations and a discussion on them is well documented in [this issue](https://github.com/kubernetes/ingress-nginx/issues/8965). Please refer to the issue to add annotations, with values specific to user, to get graphs of the DO-LB populated with data.

#### Scaleway

```js
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.13.1/deploy/static/provider/scw/deploy.yaml
```

Refer to the [dedicated tutorial](https://www.scaleway.com/en/docs/tutorials/proxy-protocol-v2-load-balancer/#configuring-proxy-protocol-for-ingress-nginx) in the Scaleway documentation for configuring the proxy protocol for ingress-nginx with the Scaleway load balancer.

#### Exoscale

```js
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/exoscale/deploy.yaml
```

The full list of annotations supported by Exoscale is available in the Exoscale Cloud Controller Manager [documentation](https://github.com/exoscale/exoscale-cloud-controller-manager/blob/master/docs/service-loadbalancer.md).

#### Oracle Cloud Infrastructure

```js
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.13.1/deploy/static/provider/cloud/deploy.yaml
```

可以在 [OCI Cloud Controller Manager](https://github.com/oracle/oci-cloud-controller-manager) 文档中找到 [Oracle Cloud Infrastructure 可用注释的完整列表](https://github.com/oracle/oci-cloud-controller-manager/blob/master/docs/load-balancer-annotations.md) 。

#### OVHcloud

```js
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm -n ingress-nginx install ingress-nginx ingress-nginx/ingress-nginx --create-namespace
```

您可以找到 [完整教程](https://docs.ovh.com/gb/en/kubernetes/installing-nginx-ingress/) 。

### 裸金属集群

本节适用于在裸金属服务器上部署的 Kubernetes 集群，以及手动安装 Kubernetes 的“原始”虚拟机，使用通用的 Linux 发行版（如 CentOS、Ubuntu 等）。

为了快速测试，您可以使用 [NodePort](https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport) 。这在几乎所有集群上都应该有效，但通常会使用 30000-32767 范围内的端口。

```js
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.13.1/deploy/static/provider/baremetal/deploy.yaml
```

有关裸金属部署的更多信息（以及如何使用 80 端口而不是 30000-32767 范围内的随机端口），请参见 [裸金属注意事项](https://kubernetes.github.io/ingress-nginx/deploy/baremetal/) 。

## 杂项

### 检查入口控制器版本

在 pod 内运行 `/nginx-ingress-controller --version` ，例如使用 `kubectl exec` ：

```js
POD_NAMESPACE=ingress-nginx
POD_NAME=$(kubectl get pods -n $POD_NAMESPACE -l app.kubernetes.io/name=ingress-nginx --field-selector=status.phase=Running -o name)
kubectl exec $POD_NAME -n $POD_NAMESPACE -- /nginx-ingress-controller --version
```

### 范围

默认情况下，控制器会监视所有命名空间中的 Ingress 对象。如果您想更改此行为，请使用标志 `--watch-namespace` 或检查 Helm 图表值 `controller.scope` 以将控制器限制为单个命名空间。尽管使用此标志并不常见，但需要注意一个重要事实，即包含 default-ssl-certificate 的密钥也需要存在于被监视的命名空间中。

有关更多详细信息，请参见 [“如何在同一集群中安装多个 Ingress 控制器”](https://kubernetes.github.io/ingress-nginx/user-guide/multiple-ingress/) 。

### Webhook 网络访问

警告

控制器使用 [准入 webhook](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/) 来验证 Ingress 定义。请确保您没有 [网络策略](https://kubernetes.io/docs/concepts/services-networking/network-policies/) 或其他防火墙阻止 API 服务器与 `ingress-nginx-controller-admission` 服务之间的连接。

### 证书生成

注意

当入口控制器第一次启动时，两个 [作业](https://kubernetes.io/docs/concepts/workloads/controllers/jobs-run-to-completion/) 会创建由入场 Webhook 使用的 SSL 证书。

这可能会导致初始延迟，最长可达两分钟，直到可以创建和验证 Ingress 定义。

您可以等待直到它准备好再运行下一个命令：

```js
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=120s
```

### 在 Kubernetes 1.19 之前运行

Ingress 资源随着时间的推移而演变。它们最初是 `apiVersion: extensions/v1beta1` ，然后转向 `apiVersion: networking.k8s.io/v1beta1` ，最近则是 `apiVersion: networking.k8s.io/v1` 。

以下是这些 Ingress 版本在 Kubernetes 中的支持情况：

- 在 Kubernetes 1.19 之前，仅支持 `v1beta1` Ingress 资源。
- 从 Kubernetes 1.19 到 1.21，支持 `v1beta1` 和 `v1` Ingress 资源
- 在 Kubernetes 1.22 及以上版本中，仅支持 `v1` Ingress 资源

以下是 Ingress-Nginx Controller 如何支持这些 Ingress 版本的方式：

- 在 1.0 版本之前，仅支持 `v1beta1` Ingress 资源
- 在 1.0 及以上版本中，仅支持 `v1` Ingress 资源。

因此，如果您正在运行 Kubernetes 1.19 或更高版本，您应该能够使用最新版本的 NGINX Ingress Controller；但如果您使用的是旧版本的 Kubernetes（1.18 或更早），则必须使用 Ingress-Nginx Controller 的 0.X 版本（例如版本 0.49）。

Ingress-Nginx Controller 的 Helm 图表在图表的第 4 版中切换到了版本 1。换句话说，如果您正在运行 Kubernetes 1.19 或更早版本，您应该使用该图表的 3.X 版本（这可以通过在 `helm install` 命令中添加 `--version='<4'` 来实现）。

Copied to clipboard