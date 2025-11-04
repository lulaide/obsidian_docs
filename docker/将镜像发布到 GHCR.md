---
source: "https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry"
tags:
  - "docker"
---
## 关于容器注册表

容器注册表在您的组织或个人账户中存储容器镜像，并允许您将镜像与一个仓库关联。您可以选择是否从仓库继承权限，或独立于仓库设置细粒度权限。您还可以匿名访问公共容器镜像。

## 关于容器注册表支持

容器注册表目前支持以下容器镜像格式：

- [Docker 镜像清单 V2，模式 2](https://docs.docker.com/registry/spec/manifest-v2-2/)
- [开放容器倡议 (OCI) 规范](https://github.com/opencontainers/image-spec)

在安装或发布 Docker 镜像时，容器注册表支持外部层，例如 Windows 镜像。

## 对容器注册表进行身份验证

您需要一个访问令牌才能发布、安装和删除私有、内部和公共包。

您可以使用个人访问令牌（经典）对 GitHub Packages 或 GitHub API 进行身份验证。当您创建个人访问令牌（经典）时，可以根据需要为令牌分配不同的作用域。有关个人访问令牌（经典）相关的包作用域的更多信息，请参见 [关于 GitHub Packages 的权限](https://docs.github.com/en/packages/learn-github-packages/about-permissions-for-github-packages#about-scopes-and-permissions-for-package-registries) 。

要在 GitHub Actions 工作流中对 GitHub Packages 注册表进行身份验证，您可以使用：

- `GITHUB_TOKEN` 用于发布与工作流仓库相关联的包。
- 一个具有至少 `read:packages` 范围的个人访问令牌（经典），用于安装与其他私有仓库相关联的包（如果仓库被授予对包的读取权限，可以使用 `GITHUB_TOKEN` 。有关详细信息，请参见 [配置包的访问控制和可见性](https://docs.github.com/en/packages/learn-github-packages/configuring-a-packages-access-control-and-visibility) ）。

### 在 GitHub Actions 工作流中进行身份验证

此注册表支持细粒度权限。对于支持细粒度权限的注册表，如果您的 GitHub Actions 工作流使用个人访问令牌进行身份验证，我们强烈建议您更新工作流以使用 `GITHUB_TOKEN` 。有关更新使用个人访问令牌进行身份验证的注册表的工作流的指导，请参见 [使用 GitHub Actions 发布和安装包](https://docs.github.com/en/packages/managing-github-packages-using-github-actions-workflows/publishing-and-installing-a-package-with-github-actions#upgrading-a-workflow-that-accesses-a-registry-using-a-personal-access-token) 。

您可以在 GitHub Actions 工作流中使用 `GITHUB_TOKEN` 通过 REST API 删除或恢复包，前提是该令牌对包具有 `admin` 权限。使用工作流发布包的仓库，以及您明确连接到包的仓库，自动获得对该仓库中包的 `admin` 权限。

有关 `GITHUB_TOKEN` 的更多信息，请参见 [在工作流中使用 GITHUB\_TOKEN 进行身份验证](https://docs.github.com/en/actions/security-guides/automatic-token-authentication#using-the-github_token-in-a-workflow) 。有关在操作中使用注册表时的最佳实践的更多信息，请参见 [安全使用参考](https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions#considering-cross-repository-access) 。

您还可以选择为 GitHub Codespaces 和 GitHub Actions 独立授予包的访问权限。有关更多信息，请参见 [配置包的访问控制和可见性](https://docs.github.com/en/packages/learn-github-packages/configuring-a-packages-access-control-and-visibility#ensuring-codespaces-access-to-your-package) 和 [配置包的访问控制和可见性](https://docs.github.com/en/packages/learn-github-packages/configuring-a-packages-access-control-and-visibility#ensuring-workflow-access-to-your-package) 。

### 使用个人访问令牌（经典）进行身份验证

1. 创建一个新的个人访问令牌（经典），并为您想要完成的任务选择适当的范围。如果您的组织要求单点登录（SSO），您必须为您的新令牌启用 SSO。
	- 选择 `read:packages` 范围以下载容器镜像并读取其元数据。
	- 选择 `write:packages` 范围以下载和上传容器镜像，并读取和写入它们的元数据。
	- 选择 `delete:packages` 范围以删除容器镜像。
	有关更多信息，请参见 [管理您的个人访问令牌](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token) 。
2. 保存您的个人访问令牌（经典版）。我们建议将您的令牌保存为环境变量。
	```shell
	export CR_PAT=YOUR_TOKEN
	```
3. 使用您的容器类型的命令行界面，登录到容器注册服务 `ghcr.io` 。
	```shell
	$ echo $CR_PAT | docker login ghcr.io -u USERNAME --password-stdin
	> Login Succeeded
	```

## 推送容器镜像

此示例推送 `IMAGE_NAME` 的最新版本。

```shell
docker push ghcr.io/NAMESPACE/IMAGE_NAME:latest
```

将 `NAMESPACE` 替换为您希望将镜像作用于的个人账户或组织的名称。

此示例推送镜像的 `2.5` 版本。

```shell
docker push ghcr.io/NAMESPACE/IMAGE_NAME:2.5
```

当您首次发布一个包时，默认可见性为私有。要更改可见性或设置访问权限，请参见 [配置包的访问控制和可见性](https://docs.github.com/en/packages/learn-github-packages/configuring-a-packages-access-control-and-visibility) 。您可以使用用户界面或命令行将已发布的包链接到一个仓库。有关更多信息，请参见 [将仓库连接到包](https://docs.github.com/en/packages/learn-github-packages/connecting-a-repository-to-a-package) 。

当您从命令行推送容器镜像时，镜像默认情况下并未链接到任何仓库。即使您使用与仓库名称匹配的命名空间对镜像进行标记，例如 `ghcr.io/octocat/my-repo:latest` ，情况也是如此。

将仓库与容器包连接的最简单方法是通过使用 `${{secrets.GITHUB_TOKEN}}` 的工作流发布包，因为包含工作流的仓库会自动链接。请注意，如果您之前已将包推送到同一命名空间，但未将包连接到仓库，则 `GITHUB_TOKEN` 将没有权限推送该包。

要在命令行发布镜像时连接一个仓库，并确保您的 `GITHUB_TOKEN` 在使用 GitHub Actions 工作流时具有适当的权限，我们建议在您的 `Dockerfile` 中添加标签 `org.opencontainers.image.source` 。有关更多信息，请参阅本文中的“ [标记容器镜像](https://docs.github.com/en/packages/working-with-a-github-packages-registry/#labelling-container-images) ”和“ [使用 GitHub Actions 发布和安装包](https://docs.github.com/en/packages/managing-github-packages-using-github-actions-workflows/publishing-and-installing-a-package-with-github-actions) 。”

## 拉取容器镜像

### 按摘要拉取

为了确保您始终使用相同的镜像，您可以通过 `digest` SHA 值指定要拉取的确切容器镜像版本。

1. 要找到摘要 SHA 值，请使用 `docker inspect` 或 `docker pull` ，并复制 `Digest:` 后面的 SHA 值
	```shell
	docker inspect ghcr.io/NAMESPACE/IMAGE_NAME
	```
	将 `NAMESPACE` 替换为图像所属的个人账户或组织的名称。
2. 根据需要在本地删除图像。
	```shell
	docker rmi ghcr.io/NAMESPACE/IMAGE_NAME:latest
	```
3. 在图像名称后使用 `@YOUR_SHA_VALUE` 拉取容器图像。
	```shell
	docker pull ghcr.io/NAMESPACE/IMAGE_NAME@sha256:82jf9a84u29hiasldj289498uhois8498hjs29hkuhs
	```

### 按名称拉取

```shell
docker pull ghcr.io/NAMESPACE/IMAGE_NAME
```

将 `NAMESPACE` 替换为图像所属的个人账户或组织的名称。

### 按名称和版本拉取

Docker CLI 示例，显示通过其名称和 `1.14.1` 版本标签拉取的图像：

```shell
$ docker pull ghcr.io/NAMESPACE/IMAGE_NAME:1.14.1
> 5e35bd43cf78: Pull complete
> 0c48c2209aab: Pull complete
> fd45dd1aad5a: Pull complete
> db6eb50c2d36: Pull complete
> Digest: sha256:ae3b135f133155b3824d8b1f62959ff8a72e9cf9e884d88db7895d8544010d8e
> Status: Downloaded newer image for ghcr.io/NAMESPACE/IMAGE_NAME/release:1.14.1
> ghcr.io/NAMESPACE/IMAGE_NAME/release:1.14.1
```

将 `NAMESPACE` 替换为图像所归属的个人账户或组织的名称。

### 按名称和最新版本拉取

```shell
$ docker pull ghcr.io/NAMESPACE/IMAGE_NAME:latest
> latest: Pulling from NAMESPACE/IMAGE_NAME
> Digest: sha256:b3d3e366b55f9a54599220198b3db5da8f53592acbbb7dc7e4e9878762fc5344
> Status: Downloaded newer image for ghcr.io/NAMESPACE/IMAGE_NAME:latest
> ghcr.io/NAMESPACE/IMAGE_NAME:latest
```

将 `NAMESPACE` 替换为图像所属的个人账户或组织的名称。

## 构建容器镜像

此示例构建 `hello_docker` 镜像：

```shell
docker build -t hello_docker .
```

## 标记容器镜像

1. 找到您想要标记的 Docker 镜像的 ID。
	```shell
	$ docker images
	> REPOSITORY                                            TAG                 IMAGE ID            CREATED             SIZE
	> ghcr.io/my-org/hello_docker         latest            38f737a91f39        47 hours ago        91.7MB
	> hello-world                                           latest              fce289e99eb9        16 months ago       1.84kB
	```
2. 使用镜像 ID 和您所需的镜像名称及托管目的地为您的 Docker 镜像打标签。
	```shell
	docker tag 38f737a91f39 ghcr.io/NAMESPACE/NEW_IMAGE_NAME:latest
	```

将 `NAMESPACE` 替换为您希望将镜像作用于的个人账户或组织的名称。

## 标记容器镜像

您可以使用预定义的注释键为您的容器镜像添加元数据，包括描述、许可证和源代码库。支持的键的值将出现在镜像的包页面上。

对于大多数镜像，您可以使用 Docker 标签将注释键添加到镜像中。有关更多信息，请参见官方 Docker 文档中的 [LABEL](https://docs.docker.com/engine/reference/builder/#label) 和 `opencontainers/image-spec` 仓库中的 [预定义注释键](https://github.com/opencontainers/image-spec/blob/main/annotations.md#pre-defined-annotation-keys) 。

对于多架构镜像，您可以通过将适当的注释键添加到镜像清单中的 `annotations` 字段来为镜像添加描述。有关更多信息，请参见 [为多架构镜像添加描述](https://docs.github.com/en/packages/working-with-a-github-packages-registry/#adding-a-description-to-multi-arch-images) 。

以下注释键在容器注册表中受支持。

| 关键 | 描述 |
| --- | --- |
| `org.opencontainers.image.source` | 与该包关联的仓库的 URL。有关更多信息，请参见 [将仓库连接到包](https://docs.github.com/en/packages/learn-github-packages/connecting-a-repository-to-a-package#connecting-a-repository-to-a-container-image-using-the-command-line) 。 |
| `org.opencontainers.image.description` | 仅限 512 个字符的文本描述。此描述将出现在包页面上，位于包名称下方。 |
| `org.opencontainers.image.licenses` | 一个 SPDX 许可证标识符，例如 "MIT"，限制为 256 个字符。许可证将出现在包页面的 "详细信息" 侧边栏中。有关更多信息，请参见 [SPDX 许可证列表](https://spdx.org/licenses/) 。 |

要将键作为 Docker 标签添加，我们建议在您的 `Dockerfile` 中使用 `LABEL` 指令。例如，如果您是用户 `octocat` 并且拥有 `my-repo` ，并且您的镜像是在 MIT 许可证条款下分发的，您可以在 `Dockerfile` 中添加以下行：

```dockerfile
LABEL org.opencontainers.image.source=https://github.com/octocat/my-repo
LABEL org.opencontainers.image.description="My container image"
LABEL org.opencontainers.image.licenses=MIT
```

或者，您可以在构建时使用 `docker build` 命令向镜像添加标签。

```shell
$ docker build \
 --label "org.opencontainers.image.source=https://github.com/octocat/my-repo" \
 --label "org.opencontainers.image.description=My container image" \
 --label "org.opencontainers.image.licenses=MIT"
```

### 为多架构镜像添加描述

多架构镜像是支持多种架构的镜像。它通过在单个清单中引用一系列镜像来工作，每个镜像支持不同的架构。

多架构镜像的包页面上显示的描述是从镜像清单中的 `annotations` 字段获取的。与 Docker 标签类似，注释提供了一种将元数据与镜像关联的方法，并支持预定义的注释键。有关更多信息，请参见 `opencontainers/image-spec` 存储库中的 [注释](https://github.com/opencontainers/image-spec/blob/main/annotations.md) 。

要为多架构镜像提供描述，请在清单的 `annotations` 字段中为 `org.opencontainers.image.description` 键设置一个值，如下所示。

```json
"annotations": {
  "org.opencontainers.image.description": "My multi-arch image"
}
```

例如，以下 GitHub Actions 工作流步骤构建并推送一个多架构镜像。 `outputs` 参数设置镜像的描述。

```yaml
# This workflow uses actions that are not certified by GitHub.
# They are provided by a third-party and are governed by
# separate terms of service, privacy policy, and support
# documentation.

- name: Build and push Docker image
  uses: docker/build-push-action@f2a1d5e99d037542a71f64918e516c093c6f3fc4
  with:
    context: .
    file: ./Dockerfile
    platforms: ${{ matrix.platforms }}
    push: true
    outputs: type=image,name=target,annotation-index.org.opencontainers.image.description=My multi-arch image
```

## 故障排除

- 容器注册表对每个层的大小限制为 10 GB。
- 容器注册表对上传的超时时间限制为10分钟。