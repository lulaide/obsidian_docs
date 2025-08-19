---
source: https://cert-manager.io/docs/configuration/acme/dns01/cloudflare/#api-tokens
tags:
  - cert-manager
  - k8s
---
要使用 Cloudflare，您可以使用两种类型的令牌之一。 **API 令牌** 允许与特定区域和权限绑定的应用程序范围的密钥，而 **API 密钥** 是具有与您的帐户相同权限的全局范围密钥。

**API 令牌** 被推荐用于更高的安全性，因为它们具有更严格的权限，并且更容易被撤销。

## API 令牌

令牌可以在 **用户个人资料 > API 令牌 > API 令牌** 中创建。建议使用以下设置：

- 权限：
	- `Zone - DNS - Edit`
	- `Zone - Zone - Read`
- 区域资源：
	- `Include - All Zones`

要创建一个新的 `Issuer` ，首先需要创建一个包含新 API 令牌的 Kubernetes 秘密：

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: cloudflare-api-token-secret
type: Opaque
stringData:
  api-token: <API Token>
```

然后在您的 `Issuer` 清单中：

```yaml
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: example-issuer
spec:
  acme:
    ...
    solvers:
    - dns01:
        cloudflare:
          apiTokenSecretRef:
            name: cloudflare-api-token-secret
            key: api-token
```

## API 密钥

API 密钥可以在 **用户个人资料 > API 令牌 > API 密钥 > 全局 API 密钥 > 查看** 中获取。

要创建一个新的 `Issuer` ，首先需要创建一个包含您的 API 密钥的 Kubernetes 秘密：

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: cloudflare-api-key-secret
type: Opaque
stringData:
  api-key: <API Key>
```

然后在您的 `Issuer` 清单中：

```yaml
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: example-issuer
spec:
  acme:
    ...
    solvers:
    - dns01:
        cloudflare:
          email: my-cloudflare-acc@example.com
          apiKeySecretRef:
            name: cloudflare-api-key-secret
            key: api-key
```

## 故障排除

如果您收到错误提示，表示您的令牌没有正确的权限来列出区域，可能有两个原因。

1. 该令牌缺少 `  区域 - 区域 - 读取  ` 权限。
2. 由于 DNS 问题，cert-manager 识别了域名的错误区域名称。

在第二个问题的情况下，您将看到如下错误：

```
Events:
  Type     Reason        Age              From          Message
  ----     ------        ----             ----          -------
  Normal   Started       6s               cert-manager  Challenge scheduled for processing
  Warning  PresentError  3s (x2 over 3s)  cert-manager  Error presenting challenge: Cloudflare API Error for GET "/zones?name=<TLD>" 
            Error: 0: Actor 'com.cloudflare.api.token.xxxx' requires permission 'com.cloudflare.api.account.zone.list' to list zones
```

在这种情况下，我们建议 [更改您的 DNS01 自检名称服务器](https://cert-manager.io/docs/configuration/acme/dns01/#setting-nameservers-for-dns01-self-check) 。

您可能遇到这个问题，因为 Cloudflare 阻止使用 API 更新以下顶级域名的 DNS 记录：`.cf` 、`.ga` 、`.gq` 、`.ml` 和 `.tk` 。有关此问题的讨论可以在 [Cloudflare 社区](https://community.cloudflare.com/t/unable-to-update-ddns-using-api-for-some-tlds/167228) 中找到。我们建议在使用这些顶级域名时使用其他 DNS 提供商。