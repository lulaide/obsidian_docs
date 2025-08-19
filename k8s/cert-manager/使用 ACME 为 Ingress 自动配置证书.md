---
source: "https://cert-manager.io/docs/configuration/acme/"
tags:
  - "cert-manager"
---
ACME Issuer 类型代表与自动证书管理环境 (ACME) 证书授权服务器注册的单个账户。当您创建一个新的 ACME `Issuer` 时，cert-manager 将生成一个私钥，用于在 ACME 服务器上识别您。

由公共 ACME 服务器颁发的证书通常默认被客户端计算机信任。这意味着，例如，访问一个由为该 URL 颁发的 ACME 证书支持的网站，通常会被大多数客户端网页浏览器默认信任。ACME 证书通常是免费的。

## 解决挑战

为了让 ACME CA 服务器验证客户端是否拥有请求证书的域名，客户端必须完成“挑战”。这确保了客户端无法为他们不拥有的域名请求证书，从而防止欺诈性地冒充他人的网站。如 [RFC8555](https://tools.ietf.org/html/rfc8555) 中详细说明，cert-manager 提供了两种挑战验证 - HTTP01 和 DNS01 挑战。

[HTTP01](https://cert-manager.io/docs/configuration/acme/http01/) 挑战通过在 HTTP URL 端点上呈现一个计算出的密钥来完成，该密钥应可在互联网上路由。此 URL 将使用请求证书的域名。一旦 ACME 服务器能够通过互联网从此 URL 获取该密钥，ACME 服务器就可以验证您是该域名的所有者。当创建 HTTP01 挑战时，cert-manager 将自动配置您的集群入口，以将该 URL 的流量路由到一个小型 Web 服务器，该服务器呈现此密钥。

[DNS01](https://cert-manager.io/docs/configuration/acme/dns01/) 挑战通过提供一个存在于 DNS TXT 记录中的计算密钥来完成。一旦该 TXT 记录在互联网上传播，ACME 服务器就可以通过 DNS 查询成功检索到该密钥，并验证客户端是否拥有请求证书的域名。在正确的权限下，cert-manager 将自动为您指定的 DNS 提供商呈现此 TXT 记录。

## 配置

### 创建基本的 ACME 颁发者

所有 ACME `  颁发者  ` 都遵循类似的配置结构 - 包括客户端的 `  电子邮件  ` 、 `  服务器  ` URL、 `privateKeySecretRef` 和一个或多个 `  求解器  ` 。以下是一个简单 ACME 颁发者的示例：


```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    # 必须将此电子邮件地址替换为您自己的地址。
    # Let's Encrypt 会使用此地址来就证书即将到期与您联系。
    email: user@example.com
    # 如果 ACME 服务器支持配置文件，可在此指定配置文件名称。
    # 参见下方的 #acme-certificate-profiles。
    profile: tlsserver
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      # 将用于存储账户私钥的 Secret 资源。
      # 这是您在 ACME 提供者处的身份。可以选择任意 Secret 名称。
      # 此 Secret 会被自动填充数据，通常无需进一步操作。
      # 如果您丢失此身份/Secret，您可以生成一个新的身份并为
      # 以前账户管理的任意/所有域名生成证书，但您将无法撤销使用先前账户生成的证书。
      name: example-issuer-account-key
    # 添加单个挑战求解器，使用 nginx 的 HTTP01
    solvers:
    - http01:
        ingress:
          ingressClassName: nginx
```

解算器以 [`dns01`](https://cert-manager.io/docs/configuration/acme/dns01/) 和 [`http01`](https://cert-manager.io/docs/configuration/acme/http01/) 形式出现。有关如何配置 这些解算器类型的更多信息，请访问它们各自的文档 - [DNS01](https://cert-manager.io/docs/configuration/acme/dns01/) ， [HTTP01](https://cert-manager.io/docs/configuration/acme/http01/) 
### ACME 证书配置文件

> ℹ️ 此功能在 cert-manager `>= v1.18.0` 中可用。

ACME 服务器 *可能* 为 ACME 客户端提供不同证书配置文件的选择。

在 `Issuer` 或 `ClusterIssuer` 中使用可选的 `profile` 字段来为您的 ACME 订单选择一个配置文件。

Let's Encrypt 已经提供了 [一系列配置文件](https://letsencrypt.org/docs/profiles/) 。其他 ACME 服务器可能尚不支持配置文件，或者它们可能提供不同的配置文件，因此请查看您的 ACME 服务器文档以了解可用的配置文件。

您可以通过下载目录对象来查找您的 ACME 服务器是否支持配置文件。例如：

```bash
curl -fsSL https://acme-staging-v02.api.letsencrypt.org/directory
```

如果支持配置文件，您将在 JSON 对象的字段中看到“profiles”。

如果您未指定配置文件，ACME 服务器将使用其默认配置文件，在 Let's Encrypt 的情况下，默认配置文件是 `classic` 配置文件。

> ⚠️ 如果您指定了一个配置文件并连接到一个尚不支持 [ACME 配置文件扩展](https://datatracker.ietf.org/doc/draft-aaron-acme-profiles/) 的 ACME 服务器，cert-manager 将在 CertificateRequest 资源上报告错误。
> 
> ℹ️ 如果您指定了一个 ACME 服务器不识别的配置文件，cert-manager 将在 CertificateRequest 资源上报告错误。
> 
> 📖 阅读 [证书配置文件的 ACME 协议扩展（IETF 草案）](https://datatracker.ietf.org/doc/draft-aaron-acme-profiles/) 以了解更多信息。

### 外部账户绑定

cert-manager 支持使用外部账户绑定与您的 ACME 账户。外部账户绑定用于将您的 ACME 账户与外部账户（例如 CA 自定义数据库）关联。对于大多数 cert-manager 用户来说，这通常不是必需的，除非您明确知道需要这样做。

外部账户绑定需要在 ACME `Issuer` 上提供两个字段，这些字段代表您的 ACME 账户。这些字段是：

- `keyID` - 外部账户管理器索引的外部账户绑定的密钥 ID 或账户 ID
- `keySecretRef` - 包含您外部账户对称 MAC 密钥的 base 64 编码 URL 字符串的秘密的名称和密钥

> 注意：在 *大多数* 情况下，MAC 密钥必须以 `base64URL` 编码。以下命令将对密钥进行 base64 编码并转换为 `base64URL` ：
> 
> ```
> $ echo 'my-secret-key' | base64 -w0 | sed -e 's/+/-/g' -e 's/\//_/g' -e 's/=//g'
> ```
> 
> 然后您可以使用以下命令创建 Secret 资源：
> 
> ```
> $ kubectl create secret generic eab-secret --from-literal \
>   secret={base64 encoded secret key}
> ```

带有外部账户绑定的 ACME 颁发者示例如下。

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: my-acme-server-with-eab
spec:
  acme:
    email: user@example.com
    server: https://my-acme-server-with-eab.com/directory
    externalAccountBinding:
      keyID: my-keyID-1
      keySecretRef:
        name: eab-secret
        key: secret
    privateKeySecretRef:
      name: example-issuer-account-key
    solvers:
    - http01:
        ingress:
          ingressClassName: nginx
```

> 注意：cert-manager 版本在 `v1.3.0` 之前也要求用户指定 EAB 的 MAC 算法，通过设置 `Issuer.spec.acme.externalAccountBinding.keyAlgorithm` 字段。该字段现在已被弃用，因为上游 Go `x/crypto` 库将算法硬编码为 `HS256` 。（请参见上游相关讨论 [`CL#41430`](https://github.com/golang/go/issues/41430) ）。

### 重用 ACME 账户

您可能希望在多个集群中重用一个 ACME 账户。这在使用 EAB 时尤其有用。如果 `disableAccountKeyGeneration` 字段被设置，cert-manager 将不会创建新的 ACME 账户，而是使用 在 `privateKeySecretRef` 中指定的现有密钥。请注意， `Issuer` / `ClusterIssuer` 将不会准备好，并将继续重试，直到 提供了 `Secret` 。

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: my-acme-server-with-existing-acme-account
spec:
  acme:
    email: user@example.com
    disableAccountKeyGeneration: true
    privateKeySecretRef:
      name: example-issuer-account-key
```

### 添加多个解题器类型

您可能希望为不同的入口控制器使用不同类型的挑战解题器配置，例如，如果您想使用 `DNS01` 签发通配符证书，同时还要签发使用 `HTTP01` 验证的其他证书。

`solvers` 段落有一个可选的 `selector` 字段，可以用来指定哪些 `Certificates` ，以及进一步指定在这些 `Certificates` 上应使用哪些 DNS 名称来解决挑战。

可以使用三种选择器类型来形成对某个对象的要求 `证书 ` 必须满足以便被选中用于求解器 - `  匹配标签  ` ， `dnsNames` 和 `dnsZones` 。您可以在单个解算器上使用任意数量的这三个选择器。

`matchLabel` 选择器要求所有 `Certificates` 必须匹配该段落字符串映射列表中定义的所有标签。例如，以下 `Issuer` 仅会匹配具有标签 `"user-cloudflare-solver": "true"` 和 `"email": "user@example.com"` 的 `Certificates` 。

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    ...
    solvers:
    - dns01:
        cloudflare:
          email: user@example.com
          apiKeySecretRef:
            name: cloudflare-apikey-secret
            key: apikey
      selector:
        matchLabels:
          "use-cloudflare-solver": "true"
          "email": "user@example.com"
```

#### DNS 名称

`dnsNames` 选择器是一个确切的 DNS 名称列表，这些名称应该映射到一个解算器。这意味着包含这些 DNS 名称的 `Certificates` 将被选中。如果找到匹配项， `dnsNames` 选择器将优先于 [`dnsZones`](https://cert-manager.io/docs/configuration/acme/#dns-zones) 选择器。如果多个解算器与相同的 `dnsNames` 值匹配，将选择在 [`matchLabels`](https://cert-manager.io/docs/configuration/acme/#match-labels) 中具有最多匹配标签的解算器。如果两者都没有更多匹配项，将选择列表中定义较早的解算器。

以下示例将为具有 DNS 名称的 `Certificates` 解决挑战 `example.com` 和 `*.example.com` 这两个域。

> 注意： `dnsNames` 需要完全匹配，不支持通配符，这意味着以下的 `Issuer` *将不会* 解决像 `foo.example.com` 这样的 DNS 名称。使用 [`dnsZones`](https://cert-manager.io/docs/configuration/acme/#dns-zones) 选择器类型可以匹配一个区域内的所有子域。

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    ...
    solvers:
    - dns01:
        cloudflare:
          email: user@example.com
          apiKeySecretRef:
            name: cloudflare-apikey-secret
            key: apikey
      selector:
        dnsNames:
        - 'example.com'
        - '*.example.com'
```

#### DNS 区域

`dnsZones` 段定义了可以由此解决的 DNS 区域列表 求解器。如果 DNS 名称完全匹配，或者是任何指定域名的子域名 `dnsZones` ，将使用此解析器，除非有更具体的 [`dnsNames`](https://cert-manager.io/docs/configuration/acme/#dns-names) 匹配已配置。这意味着 `sys.example.com` 将会被选择，而不是指定 `example.com` 的域名 `www.sys.example.com` 。如果多个解算器与相同的 `dnsZones` 值匹配，则将选择在 [`matchLabels`](https://cert-manager.io/docs/configuration/acme/#match-labels) 中具有最多匹配标签的解算器。如果两者都没有更多匹配，则将选择列表中定义较早的解算器。

在以下示例中，此求解器将为域名 `example.com` 及其所有子域名 `*.example.com` 解决挑战。

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    ...
    solvers:
    - dns01:
        cloudflare:
          email: user@example.com
          apiKeySecretRef:
            name: cloudflare-apikey-secret
            key: apikey
      selector:
        dnsZones:
        - 'example.com'
```

#### 全部在一起

每个解算器可以定义任意数量的三种选择器类型。在以下示例中，将使用 `DNS01` 解算器来为包含 DNS 名称的 `  证书  ` 解决域名的挑战 `a.example.com` 和 `b.example.com` 。将使用 Google CloudDNS 的 `DNS01` 解算器来解决 DNS 名称匹配区域 `test.example.com` 及其所有子域名（例如 `foo.test.example.com` ）的 `  证书  ` 的挑战。

对于所有其他挑战， `HTTP01` 解算器将仅在 `证书 ` 还包含标签 `"use-http01-solver": "true"` 时使用。

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    ...
    solvers:
    - http01:
        ingress:
          ingressClassName: nginx
      selector:
        matchLabels:
          "use-http01-solver": "true"
    - dns01:
        cloudflare:
          email: user@example.com
          apiKeySecretRef:
            name: cloudflare-apikey-secret
            key: apikey
      selector:
        dnsNames:
        - 'a.example.com'
        - 'b.example.com'
    - dns01:
        cloudDNS:
          project: my-project-id
          hostedZoneName: 'test-example.com'
          serviceAccountSecretRef:
            key: sa
            name: gcp-sa-secret
      selector:
        dnsZones:
        - 'test.example.com' # This should be the DNS name of the zone
```

每个单独的选择器块可以包含多种选择器类型，例如：

```yaml
solvers:
- dns01:
  cloudflare:
    email: user@example.com
    apiKeySecretRef:
      name: cloudflare-apikey-secret
      key: apikey
  selector:
    matchLabels:
     'email': 'user@example.com'
     'solver': 'cloudflare'
    dnsZones:
      - 'test.example.com'
      - 'example.dev'
```

在这种情况下，Cloudflare 的 `DNS01` 解析器仅在 `Certificate` 具有来自 `matchLabels` *并且* DNS 名称与 `dnsZones` 中的一个区域匹配时，才会用于解决挑战。

## 私有 ACME 服务器

cert-manager 也应该可以与私有或自托管的 ACME 服务器一起使用，只要它们遵循 ACME 规范。

如果您的 ACME 服务器不使用公共信任的证书，您可以在创建颁发者时传递一个受信任的 CA，从 cert-manager 1.11 开始：

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: my-acme-server-issuer
spec:
  acme:
    server: https://my-acme-server.example.com
    caBundle: <base64 encoded CA Bundle in PEM format>
    ...
```

## 替代证书链

在从 ACME 服务器获取证书时，可以选择替代证书链。这允许颁发者在过渡期间优雅地将用户切换到新的根证书；最著名的例子是 Let's Encrypt ["ISRG Root" 的更换](https://community.letsencrypt.org/t/transition-to-isrgs-root-delayed-until-jan-11-2021/125516) 。

此功能并非仅限于 Let's Encrypt；如果您的 ACME 服务器支持多个 CA 的签名，您可以在证书的颁发者部分使用 `preferredChain` ，其值为您想要的链的通用名称。如果通用名称与不同的链匹配，服务器可以选择使用并返回该新链。

如果 `preferredChain` 与证书不匹配，服务器将返回其认为的默认证书。

作为一个例子，下面是用户在（现已完成的）“ISRG Root”更换之前如何请求替代链的，但请注意，由于此更换已经发生，因此现在使用 Let's Encrypt 不再需要这样做：

```yaml
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: letsencrypt
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    preferredChain: "ISRG Root X1"
```