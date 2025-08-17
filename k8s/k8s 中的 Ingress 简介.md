---
source: "https://www.cnblogs.com/hnzhengfy/p/18408742/k8s_ingress"
tags:
  - "k8s"
---
## 一、关于 Ingress

Ingress 是 K8s 中的一个 API 对象， **用于管理和配置 外部对集群内服务 的访问** 。它可定义 HTTP 和 HTTPS 路由规则，将请求从集群外部的负载均衡器引导到相应的服务。Ingress 的灵活性使得我们能够实现高级的应用程序路由、SSL 终端和负载均衡等功能。

通过 Ingress，我们可以将集群内的多个服务暴露到外部，并根据需要进行定制化的路由设置，这为应用的扩展和灵活部署提供了便利。

Ingress 有哪些用途：

- **统一入口控制。** Ingress 提供了一个统一的入口点，用于管理多个服务的访问和流量控制。
- **HTTP/HTTPS 路由。** 通过 Ingress，可以配置基于主机名和路径的路由规则，将外部请求定向到集群内部的服务。
- **SSL 终止。** Ingress 可以配置 SSL 证书，用于加密和解密外部流量，从而确保数据传输的安全性。
- **负载均衡。** 通过 Ingress 控制器，可以实现对多实例服务的负载均衡，将请求分发到多个后端实例。

Ingress 能正常工作，主要涉及两个关键组件： **Ingress 资源和控制器** 。

资源就是用户定义的 Kubernetes 资源， **描述了主机名、路径和后端服务之间的映射关系** 。

控制器是实现 Ingress 规则的实际组件。常见的 Ingress 控制器包括 Nginx、Traefik 和 HAProxy 等等。 **控制器监控 Ingress 资源的变化，并相应地配置其代理服务器，以实现路由和流量管理。**

Ingress 的工作流程：

- **定义 Ingress 资源。** 用户创建 Ingress 资源，定义了主机名、路径和后端服务的映射。
- **Ingress 控制器监控。** Ingress 控制器不断监控 Ingress 资源的变化。
- **配置代理服务器。** 根据 Ingress 资源的定义，Ingress 控制器配置其代理服务器（如 NGINX）以匹配请求。
- **处理请求。** 当外部请求到达集群时，Ingress 控制器的代理服务器根据 Ingress 规则进行路由，将请求转发到相应的服务。

Ingress 的应用场景：

- **多服务管理。** 适用于需要管理多个服务的环境，通过 Ingress 实现统一的入口控制。
- **基于域名的路由。** 在同一个集群中运行多个应用，通过不同的域名或子域名进行访问。
- **基于路径的路由。** 根据请求路径将流量路由到不同的服务，例如 /api 路径指向一个微服务，/web 路径指向另一个微服务。
- **SSL 终止。** 需要使用 HTTPS 加密流量的场景，通过 Ingress 配置 SSL 证书进行终止。
- **负载均衡。** 在多实例服务间进行负载均衡，以提高服务的可用性和扩展性。

![](https://img2024.cnblogs.com/blog/1868241/202409/1868241-20240913174921630-189234320.gif)

*参考： [https://blog.csdn.net/lwxvgdv/article/details/139505471](https://blog.csdn.net/lwxvgdv/article/details/139505471 "https://blog.csdn.net/lwxvgdv/article/details/139505471")*


## 二、Ingress 的结构和配置示例

### 2.1 Ingress 配置的基本结构

- **规则（Rules）** ：每个 Ingress 对象可以包含多个规则，每个规则定义了一组路径匹配规则和与之关联的后端服务。
- **后端服务（Backend Services）** ：规则中指定的后端服务是 Ingress 路由请求到达时的目标服务。这可以是集群中的 Service、Pod 或外部服务。
- **路径（Paths）** ：路径定义了应该如何将请求路由到后端服务。可以使用通配符和正则表达式进行路径匹配。
- **TLS（Transport Layer Security）** ：Ingress 还支持 TLS，用于启用 HTTPS。TLS 配置包括证书和密钥，确保数据在传输过程中的安全性。

下面是一个简单的 Ingress 对象的例子：

```yaml
apiVersion: networking.k8s.io/v1

kind: Ingress

metadata: # 包含了资源的元数据，如名称、标签等

  name: my-ingress

spec:

  rules: # 定义了一组规则，用于匹配传入的 HTTP 请求并将其路由到后端服务，可多个

  - host: mydomain.com # 指定了此规则适用于哪个主机名

    http: # 定义了HTTP相关的配置

      paths: # 定义了一系列路径和它们对应的后端服务

        # 指定了路径的前缀，这里是 /app

        # 这意味着所有以 /app 开头的请求都将被路由到这个路径下

      - path: /app 

        # 指定了路径的类型，这里是 Prefix，表示路径前缀匹配

        pathType: Prefix

        backend: # 定义了后端 Service 服务的详细信息

          service:

            name: my-app-service # 后端服务的名称 name

            port:

              number: 80

  tls: # 定义了 TLS 配置，用于加密传输

  - hosts: #  指定了哪些主机名需要使用 TLS 加密

    - mydomain.com

    # 指定了存储 TLS 证书和密钥的 Secret 的名称

    secretName: my-tls-secret
```

在这个例子中，我们定义了一个 Ingress 对象，它将 mydomain.com/app 的请求路由到名为 my-app-service 的 Service，并启用了 HTTPS。

### 2.2 Ingress 的使用实例

为了更好地理解 Ingress 的使用，下面例举一个具体的示例来演示。

假设我们有一个 Web 应用，包括前端（frontend）和后端（backend）服务。现在，我们希望通过 Ingress 将这两个服务暴露到外部，并在路径上进行定制化的路由。

首先，我们创建前端和后端的 Deployment 和 Service：

Deployment.yaml：

```yaml
apiVersion: apps/v1

kind: Deployment

metadata:

  name: frontend

spec:

  replicas: 3

  selector:

    matchLabels:

      app: frontend

  template:

    metadata:

      labels:

        app: frontend

    spec:

      containers:

      - name: web-server

        image: my-frontend-image:latest

        ports:

        - containerPort: 80
```

两个 Service：

```yaml
apiVersion: v1

kind: Service

metadata:

  name: frontend-service

spec:

  selector:

    app: frontend

  ports:

    - protocol: TCP

      port: 80

      targetPort: 80

yamlapiVersion: apps/v1

kind: Deployment

metadata:

  name: backend

spec:

  replicas: 3

  selector:

    matchLabels:

      app: backend

  template:

    metadata:

      labels:

        app: backend

    spec:

      containers:

      - name: api-server

        image: my-backend-image:latest

        ports:

        - containerPort: 8080
```
```yaml
apiVersion: v1

kind: Service

metadata:

  name: backend-service

spec:

  selector:

    app: backend

  ports:

    - protocol: TCP

      port: 8080

      targetPort: 8080
```

Ingress 对象：

```yaml
apiVersion: networking.k8s.io/v1

kind: Ingress

metadata:

  name: my-ingress

spec:

  rules:

  - host: mydomain.com

    http:

      paths:

      - path: /frontend

        pathType: Prefix

        backend:

          service:

            name: frontend-service

            port:

              number: 80

      - path: /backend

        pathType: Prefix

        backend:

          service:

            name: backend-service

            port:

              number: 8080

  tls:

  - hosts:

    - mydomain.com

    secretName: my-tls-secret
```

在这个示例中，我们定义了一个 Ingress 对象，它将 mydomain.com/frontend 的请求路由到前端服务，将 mydomain.com/backend 的请求路由到后端服务。此外，我们还启用了 HTTPS，并指定了 TLS 证书的 Secret。

### 2.3 动态更新 Ingress 规则

Ingress 的一个强大之处在于它支持动态更新。

**当路由规则需要调整时，可以直接修改 Ingress 对象，而不需要重启应用或重新创建服务。**

如下配置示例：

```yaml
apiVersion: networking.k8s.io/v1

kind: Ingress

metadata:

  name: my-ingress

spec:

  rules:

  - host: mydomain.com

    http:

      paths:

      - path: /frontend

        pathType: Prefix

        backend:

          service:

            name: frontend-service

            port:

              number: 80

  tls:

  - hosts:

    - mydomain.com

    secretName: my-tls-secret
```

在这个例子中，我们仅保留了前端服务的路由规则。然后，通过应用这个更新后的 Ingress 对象，K8s 将自动更新负载均衡器的配置，使得只有 mydomain.com/frontend 的请求能够到达前端服务。

*参考： [https://zhuanlan.zhihu.com/p/676245770](https://zhuanlan.zhihu.com/p/676245770 "https://zhuanlan.zhihu.com/p/676245770")*


## 三、外部请求的大概链路

![](https://img2024.cnblogs.com/blog/1868241/202409/1868241-20240913183102924-742701281.png)

如上图，大概的请求链路如下：

1）用户从 web/mobile/pc 等客户端发出 HTTP/HTTPS 请求。

2）由于应用服务通常是通过 **域名** 的形式对外暴露，所以请求将会先进行 **DNS 域名解析** ，得到对应的公网 IP 地址。

3）公网 IP 地址通常会绑定一个 **Load Balancer 负载均衡器** ，此时请求会进入此负载均衡器。

- 负载均衡器可以是硬件，也可以是软件，它 **通常会保持稳定（固定的公网 IP 地址），因为如果切换 IP 地址会因为 DNS 缓存的原因导致服务某段时间内不可达** 。
- 负载均衡器是一个 **重要的中间层** ，对外承接公网流量，对内进行流量的管理和转发。

4）Load Balancer 再将请求转发到 **k8s 集群的某个流量入口点，通常是 Ingress** 。

- Ingress 负责 **集群内部的路由转发** ，可以看成是集群内部的网关。
- Ingress 只是配置， **具体进行流量转发的是 Ingress-controller** ，后者有多种选择，比如 Nginx、HAProxy、Traefik、Kong 等等。

5）Ingress 根据用户自定义的路由规则进一步转发到 service。

- 比如 **根据请求的 Path 路径做转发** ：

![](https://img2024.cnblogs.com/blog/1868241/202409/1868241-20240913190431546-1542202036.png)

- **根据请求的 Host 做转发** ：

![](https://img2024.cnblogs.com/blog/1868241/202409/1868241-20240913190357231-382123905.png)

6）Service 根据 selector（匹配 label 标签）将请求转发到 Pod。

- Service 有多种类型，集群内部默认的类型就是 ClusterIP。
- Service 本质上也只是一种配置，这种配置最终会作用到 Node 节点上的 kube-proxy 组件，后者会通过设置 iptables/ipvs 来完成实际的请求转发。
- Service 可能会对应多个 Pod，但最终请求只会按照一定的规则转发到一个 Pod 上。

7）Pod 最后将请求发送给其中的 Container 容器。

- 同一个 Pod 内部可能有多个 Container，但是多个容器不能共用同一个端口，因此这里会根据具体的端口号将请求发给对应的 Container。

以上就是一种典型的集群外部 HTTP 请求如何达到 Pod 中的 Container 的全过程。

需要注意的是，由于网络配置灵活多变，以上请求流转过程并不是唯一的方式，例如：

- 如果是云服务环境，那么可以通过使用 LoadBalancer 类型的 Service 直接绑定一个云服务商提供的负载均衡器，然后再接 Ingress 或者其它 Service。
- 也可以通过 NodePort 类型的 Service 直接使用节点上的端口，通过这些节点自建负载均衡器。
- 如果要部署的服务特别简单，无序管理内部流量需要，这时不用 ingress 也是可以的。

*另外，关于 Linux 内核的 namespace。正是有了它才实现了资源的隔离。因为每个 Pod 有各自的 Linux namespace，所以不同的 Pod 是资源隔离的。namespace 有多种，包括 PID、IPC、Network、Mount、Time 等等。其中 PID namespace 实现了进程的隔离，因此 pod 内可以有自己的 1 号进程。而 Network namespace 则让每个 Pod 有了自己的网络。*

*参考： [https://segmentfault.com/a/1190000044517338](https://segmentfault.com/a/1190000044517338 "https://segmentfault.com/a/1190000044517338")*