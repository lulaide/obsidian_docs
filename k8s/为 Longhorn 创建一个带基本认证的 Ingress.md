---
source: "https://longhorn.io/docs/1.9.1/deploy/accessing-the-ui/longhorn-ingress/"
tags:
  - "k8s"
---
如果您在 Kubernetes 集群上使用 kubectl 或 Helm 安装 Longhorn，您需要创建一个 Ingress，以允许外部流量访问 Longhorn UI。

默认情况下，kubectl 和 Helm 安装未启用认证。在这些步骤中，您将学习如何使用注释为 nginx ingress 控制器创建一个带基本认证的 Ingress。

1. 创建一个基本认证文件 `auth` 。生成的文件名必须为 auth（实际上 - 该密钥必须为 `data.auth` ），否则 Ingress 将返回 503。
```fallback
USER=<USERNAME_HERE>; PASSWORD=<PASSWORD_HERE>; echo "${USER}:$(openssl passwd -stdin -apr1 <<< ${PASSWORD})" >> auth
```
2. 创建一个秘密：
```fallback
kubectl -n longhorn-system create secret generic basic-auth --from-file=auth
```
3. 创建一个 Ingress 清单 `longhorn-ingress.yml`:
	> 从 v1.2.0 开始，Longhorn 支持从 UI 上传后端镜像，因此请按如下方式指定 `nginx.ingress.kubernetes.io/proxy-body-size: 10000m` 以确保上传镜像按预期工作。
```fallback
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: longhorn-ingress
  namespace: longhorn-system
  annotations:
    # 认证类型
    nginx.ingress.kubernetes.io/auth-type: basic
    # 防止控制器重定向（308）到 HTTPS
    nginx.ingress.kubernetes.io/ssl-redirect: 'false'
    # 包含用户/密码定义的 Secret 名称
    nginx.ingress.kubernetes.io/auth-secret: basic-auth
    # 显示的提示信息，说明需要认证的原因
    nginx.ingress.kubernetes.io/auth-realm: 'Authentication Required '
    # 自定义请求体最大大小，用于上传文件（例如后端镜像上传）
    nginx.ingress.kubernetes.io/proxy-body-size: 10000m
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: longhorn-frontend
            port:
              number: 80
```
4. 创建 Ingress：
```fallback
kubectl -n longhorn-system apply -f longhorn-ingress.yml
```

例如：

```fallback
$ USER=foo; PASSWORD=bar; echo "${USER}:$(openssl passwd -stdin -apr1 <<< ${PASSWORD})" >> auth
$ cat auth
foo:$apr1$FnyKCYKb$6IP2C45fZxMcoLwkOwf7k0

$ kubectl -n longhorn-system create secret generic basic-auth --from-file=auth
secret/basic-auth created
$ kubectl -n longhorn-system get secret basic-auth -o yaml
apiVersion: v1
data:
  auth: Zm9vOiRhcHIxJEZueUtDWUtiJDZJUDJDNDVmWnhNY29Md2tPd2Y3azAK
kind: Secret
metadata:
  creationTimestamp: "2020-05-29T10:10:16Z"
  name: basic-auth
  namespace: longhorn-system
  resourceVersion: "2168509"
  selfLink: /api/v1/namespaces/longhorn-system/secrets/basic-auth
  uid: 9f66233f-b12f-4204-9c9d-5bcaca794bb7
type: Opaque

$ echo "
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: longhorn-ingress
  namespace: longhorn-system
  annotations:
    # type of authentication
    nginx.ingress.kubernetes.io/auth-type: basic
    # prevent the controller from redirecting (308) to HTTPS
    nginx.ingress.kubernetes.io/ssl-redirect: 'false'
    # name of the secret that contains the user/password definitions
    nginx.ingress.kubernetes.io/auth-secret: basic-auth
    # message to display with an appropriate context why the authentication is required
    nginx.ingress.kubernetes.io/auth-realm: 'Authentication Required '
spec:
  rules:
  - http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: longhorn-frontend
            port:
              number: 80
" | kubectl -n longhorn-system create -f -
ingress.networking.k8s.io/longhorn-ingress created

$ kubectl -n longhorn-system get ingress
NAME               HOSTS   ADDRESS                                     PORTS   AGE
longhorn-ingress   *       45.79.165.114,66.228.45.37,97.107.142.125   80      2m7s

$ curl -v http://97.107.142.125/
*   Trying 97.107.142.125...
* TCP_NODELAY set
* Connected to 97.107.142.125 (97.107.142.125) port 80 (#0)
> GET / HTTP/1.1
> Host: 97.107.142.125
> User-Agent: curl/7.64.1
> Accept: */*
>
< HTTP/1.1 401 Unauthorized
< Server: openresty/1.15.8.1
< Date: Fri, 29 May 2020 11:47:33 GMT
< Content-Type: text/html
< Content-Length: 185
< Connection: keep-alive
< WWW-Authenticate: Basic realm="Authentication Required"
<
<html>
<head><title>401 Authorization Required</title></head>
<body>
<center><h1>401 Authorization Required</h1></center>
<hr><center>openresty/1.15.8.1</center>
</body>
</html>
* Connection #0 to host 97.107.142.125 left intact
* Closing connection 0

$ curl -v http://97.107.142.125/ -u foo:bar
*   Trying 97.107.142.125...
* TCP_NODELAY set
* Connected to 97.107.142.125 (97.107.142.125) port 80 (#0)
* Server auth using Basic with user 'foo'
> GET / HTTP/1.1
> Host: 97.107.142.125
> Authorization: Basic Zm9vOmJhcg==
> User-Agent: curl/7.64.1
> Accept: */*
>
< HTTP/1.1 200 OK
< Date: Fri, 29 May 2020 11:51:27 GMT
< Content-Type: text/html
< Content-Length: 1118
< Last-Modified: Thu, 28 May 2020 00:39:41 GMT
< ETag: "5ecf084d-3fd"
< Cache-Control: max-age=0
<
<!DOCTYPE html>
<html lang="en">
......
```

## AWS EKS Kubernetes 集群的额外步骤

您需要创建一个 ELB（弹性负载均衡器）以将 nginx Ingress 控制器暴露到互联网。可能会产生额外费用。

1. 根据 [nginx ingress 控制器文档](https://kubernetes.github.io/ingress-nginx/deploy/#prerequisite-generic-deployment-command) 创建先决条件资源。
2. 按照 [这些步骤](https://kubernetes.github.io/ingress-nginx/deploy/#aws) 创建一个 ELB。

## 参考文献

[https://kubernetes.github.io/ingress-nginx/](https://kubernetes.github.io/ingress-nginx/)