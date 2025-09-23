---
source: "https://docs.github.com/en/actions/tutorials/publish-packages/publish-docker-images#publishing-images-to-github-packages"
tags:
  - "CICD"
---
# 发布 Docker 镜像

## 介绍

本指南向您展示如何创建一个工作流，该工作流执行 Docker 构建，然后将 Docker 镜像发布到 Docker Hub 或 GitHub Packages。通过一个工作流，您可以将镜像发布到单个注册表或多个注册表。

## 先决条件

我们建议您对工作流配置选项和如何创建工作流文件有基本了解。有关更多信息，请参见 [编写工作流](https://docs.github.com/en/actions/learn-github-actions) 。

您可能还会发现对以下内容有基本了解是有帮助的：

- [在 GitHub Actions 中使用机密](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions)
- [在工作流中使用 GITHUB\_TOKEN 进行认证](https://docs.github.com/en/actions/security-guides/automatic-token-authentication)
- [使用容器注册表](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry)

## 关于镜像配置

本指南假设您在 GitHub 代码仓库中有一个完整的 Docker 镜像定义。例如，您的代码仓库必须包含一个 *Dockerfile* ，以及执行 Docker 构建以创建镜像所需的其他文件。

您可以使用预定义的注释键为您的容器镜像添加元数据，包括描述、许可证和源代码仓库。有关更多信息，请参见 [使用容器注册表](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry#labelling-container-images) 。

在本指南中，我们将使用 Docker `build-push-action` 操作来构建 Docker 镜像并将其推送到一个或多个 Docker 注册表。有关更多信息，请参见 [`build-push-action`](https://github.com/marketplace/actions/build-and-push-docker-images) 。

## 将镜像发布到 Docker Hub

每次在 GitHub 上创建新版本时，您都可以触发一个工作流来发布您的镜像。下面示例中的工作流在 `release` 事件以 `published` 活动类型触发时运行。

在下面的示例工作流中，我们使用 Docker `login-action` 和 `build-push-action` 操作来构建 Docker 镜像，并在构建成功时将构建的镜像推送到 Docker Hub。

要推送到 Docker Hub，您需要拥有一个 Docker Hub 账户，并创建一个 Docker Hub 代码仓库。有关更多信息，请参见 Docker 文档中的 [将 Docker 容器镜像推送到 Docker Hub](https://docs.docker.com/docker-hub/quickstart/#step-3-build-and-push-an-image-to-docker-hub) 。

推送到 Docker Hub 所需的 `login-action` 选项包括：

- `username` 和 `password` ：这是您的 Docker Hub 用户名和密码。我们建议将您的 Docker Hub 用户名和密码存储为机密，以便它们不会在工作流文件中暴露。有关更多信息，请参见 [在 GitHub Actions 中使用机密](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions) 。

Docker Hub 所需的 `metadata-action` 选项为：

- `images` ：您正在构建/推送到 Docker Hub 的 Docker 镜像的命名空间和名称。

Docker Hub 所需的 `build-push-action` 选项为：

- `tags` ：您新镜像的标签，格式为 `DOCKER-HUB-NAMESPACE/DOCKER-HUB-REPOSITORY:VERSION` 。您可以按照下面的示例设置单个标签，或在列表中指定多个标签。
- `push` ：如果设置为 `true` ，则在成功构建后，镜像将被推送到注册表。

```yaml
# This workflow uses actions that are not certified by GitHub.
# They are provided by a third-party and are governed by
# separate terms of service, privacy policy, and support
# documentation.

# GitHub recommends pinning actions to a commit SHA.
# To get a newer version, you will need to update the SHA.
# You can also reference a tag or branch, but the action may change without warning.

name: Publish Docker image

on:
  release:
    types: [published]

jobs:
  push_to_registry:
    name: Push Docker image to Docker Hub
    runs-on: ubuntu-latest
    permissions:
      packages: write
      contents: read
      attestations: write
      id-token: write
    steps:
      - name: Check out the repo
        uses: actions/checkout@v5

      - name: Log in to Docker Hub
        uses: docker/login-action@f4ef78c080cd8ba55a85445d5b36e214a81df20a
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Extract metadata (tags, labels) for Docker
        id: meta
        uses: docker/metadata-action@9ec57ed1fcdbf14dcef7dfbe97b2010124a938b7
        with:
          images: my-docker-hub-namespace/my-docker-hub-repository

      - name: Build and push Docker image
        id: push
        uses: docker/build-push-action@3b5e8027fcad23fda98b2e3ac259d8d67585f671
        with:
          context: .
          file: ./Dockerfile
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}

      - name: Generate artifact attestation
        uses: actions/attest-build-provenance@v3
        with:
          subject-name: index.docker.io/my-docker-hub-namespace/my-docker-hub-repository
          subject-digest: ${{ steps.push.outputs.digest }}
          push-to-registry: true
```

上述工作流检出 GitHub 代码仓库，使用 `login-action` 登录到注册表，然后使用 `build-push-action` 操作：基于您代码仓库的 `Dockerfile` 构建 Docker 镜像；将镜像推送到 Docker Hub，并为镜像应用标签。

在最后一步，它为镜像生成一个工件证明，这增加了供应链安全性。有关更多信息，请参见 [使用工件证明来建立构建的来源](https://docs.github.com/en/actions/security-guides/using-artifact-attestations-to-establish-provenance-for-builds) 。

## 将镜像发布到 GitHub Packages

每次在 GitHub 上创建新版本时，您都可以触发一个工作流来发布您的镜像。下面示例中的工作流在对 `release` 分支进行更改时运行。

在下面的示例工作流中，我们使用 Docker 的 `login-action` 、 `metadata-action` 和 `build-push-action` 动作来构建 Docker 镜像，如果构建成功，则将构建的镜像推送到 GitHub Packages。

GitHub Packages 所需的 `login-action` 选项是：

- `registry` ：必须设置为 `ghcr.io` 。
- `username` ：您可以使用 `${{ github.actor }}` 上下文自动使用触发工作流运行的用户的用户名。有关更多信息，请参见 [上下文参考](https://docs.github.com/en/actions/learn-github-actions/contexts#github-context) 。
- `password` ：您可以使用自动生成的 `GITHUB_TOKEN` 密钥作为密码。有关更多信息，请参见 [在工作流中使用 GITHUB\_TOKEN 进行认证](https://docs.github.com/en/actions/security-guides/automatic-token-authentication) 。

GitHub Packages 所需的 `metadata-action` 选项是：

- `images` ：您正在构建的 Docker 镜像的命名空间和名称。

GitHub Packages 所需的 `build-push-action` 选项为：

- `context` ：将构建的上下文定义为位于指定路径中的文件集。
- `push` ：如果设置为 `true` ，则在成功构建后，镜像将被推送到注册表。
- `tags` 和 `labels` ：这些由 `metadata-action` 的输出填充。

```yaml
name: Create and publish a Docker image
```

```yaml
on:
  push:
    branches: ['release']
```

配置此工作流，以便每次向名为 `release` 的分支推送更改时运行。

```yaml
env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}
```

为工作流定义两个自定义环境变量。这些变量用于容器注册域和此工作流构建的 Docker 镜像名称。

```yaml
jobs:
  build-and-push-image:
    runs-on: ubuntu-latest
```

此工作流中有一个单独的作业。它被配置为在最新可用版本的 Ubuntu 上运行。

```yaml
permissions:
      contents: read
      packages: write
      attestations: write
      id-token: write
```

设置授予此作业中操作的 `GITHUB_TOKEN` 的权限。

```yaml
steps:
      - name: Checkout repository
        uses: actions/checkout@v5
```

```yaml
- name: Extract metadata (tags, labels) for Docker
        id: meta
        uses: docker/metadata-action@9ec57ed1fcdbf14dcef7dfbe97b2010124a938b7
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
```

此步骤使用 [docker/metadata-action](https://github.com/docker/metadata-action#about) 提取将应用于指定镜像的标签和标记。 `id` "meta" 允许在后续步骤中引用此步骤的输出。 `images` 值提供了标签和标记的基本名称。

```yaml
- name: Build and push Docker image
        id: push
        uses: docker/build-push-action@f2a1d5e99d037542a71f64918e516c093c6f3fc4
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
```

此步骤使用 `docker/build-push-action` 动作根据您的代码仓库中的 `Dockerfile` 构建镜像。如果构建成功，它会将镜像推送到 GitHub Packages。它使用 `context` 参数将构建的上下文定义为位于指定路径中的文件集。有关更多信息，请参见 [docker/build-push-action](https://github.com/docker/build-push-action#usage) 代码仓库的 README 中的 [用法](https://github.com/docker/build-push-action#usage) 。它使用 `tags` 和 `labels` 参数根据 "meta" 步骤的输出为镜像打标签和标记。

```yaml
- name: Generate artifact attestation
        uses: actions/attest-build-provenance@v3
        with:
          subject-name: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME}}
          subject-digest: ${{ steps.push.outputs.digest }}
          push-to-registry: true
```

此步骤为镜像生成一个工件证明，这是一个关于镜像构建位置和方式的不可伪造声明。它提高了使用该镜像的人的供应链安全性。有关更多信息，请参见 [使用工件证明来建立构建的来源](https://docs.github.com/actions/security-guides/using-artifact-attestations-to-establish-provenance-for-builds) 。

```yaml
#
name: Create and publish a Docker image

# Configures this workflow to run every time a change is pushed to the branch called \`release\`.
on:
  push:
    branches: ['release']

# Defines two custom environment variables for the workflow. These are used for the Container registry domain, and a name for the Docker image that this workflow builds.
env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

# There is a single job in this workflow. It's configured to run on the latest available version of Ubuntu.
jobs:
  build-and-push-image:
    runs-on: ubuntu-latest
    # Sets the permissions granted to the \`GITHUB_TOKEN\` for the actions in this job.
    permissions:
      contents: read
      packages: write
      attestations: write
      id-token: write
      #
    steps:
      - name: Checkout repository
        uses: actions/checkout@v5
      # Uses the \`docker/login-action\` action to log in to the Container registry registry using the account and password that will publish the packages. Once published, the packages are scoped to the account defined here.
      - name: Log in to the Container registry
        uses: docker/login-action@65b78e6e13532edd9afa3aa52ac7964289d1a9c1
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      # This step uses [docker/metadata-action](https://github.com/docker/metadata-action#about) to extract tags and labels that will be applied to the specified image. The \`id\` "meta" allows the output of this step to be referenced in a subsequent step. The \`images\` value provides the base name for the tags and labels.
      - name: Extract metadata (tags, labels) for Docker
        id: meta
        uses: docker/metadata-action@9ec57ed1fcdbf14dcef7dfbe97b2010124a938b7
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
      # This step uses the \`docker/build-push-action\` action to build the image, based on your repository's \`Dockerfile\`. If the build succeeds, it pushes the image to GitHub Packages.
      # It uses the \`context\` parameter to define the build's context as the set of files located in the specified path. For more information, see [Usage](https://github.com/docker/build-push-action#usage) in the README of the \`docker/build-push-action\` repository.
      # It uses the \`tags\` and \`labels\` parameters to tag and label the image with the output from the "meta" step.
      - name: Build and push Docker image
        id: push
        uses: docker/build-push-action@f2a1d5e99d037542a71f64918e516c093c6f3fc4
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
      
      # This step generates an artifact attestation for the image, which is an unforgeable statement about where and how it was built. It increases supply chain security for people who consume the image. For more information, see [Using artifact attestations to establish provenance for builds](/actions/security-guides/using-artifact-attestations-to-establish-provenance-for-builds).
      - name: Generate artifact attestation
        uses: actions/attest-build-provenance@v3
        with:
          subject-name: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME}}
          subject-digest: ${{ steps.push.outputs.digest }}
          push-to-registry: true
```

上述工作流是通过对“release”分支的推送触发的。它检出 GitHub 代码仓库，并使用 `login-action` 登录到容器注册表。然后，它提取 Docker 镜像的标签和标记。最后，它使用 `build-push-action` 动作构建镜像并将其发布到容器注册表。

## 将镜像发布到 Docker Hub 和 GitHub Packages

在单个工作流中，您可以通过为每个注册表使用 `login-action` 和 `build-push-action` 动作，将您的 Docker 镜像发布到多个注册表。

以下示例工作流使用前面部分的步骤（ [将镜像发布到 Docker Hub](https://docs.github.com/en/actions/tutorials/publish-packages/#publishing-images-to-docker-hub) 和 [将镜像发布到 GitHub Packages](https://docs.github.com/en/actions/tutorials/publish-packages/#publishing-images-to-github-packages) ）创建一个单一的工作流，推送到两个注册表。

```yaml
# This workflow uses actions that are not certified by GitHub.
# They are provided by a third-party and are governed by
# separate terms of service, privacy policy, and support
# documentation.

# GitHub recommends pinning actions to a commit SHA.
# To get a newer version, you will need to update the SHA.
# You can also reference a tag or branch, but the action may change without warning.

name: Publish Docker image

on:
  release:
    types: [published]

jobs:
  push_to_registries:
    name: Push Docker image to multiple registries
    runs-on: ubuntu-latest
    permissions:
      packages: write
      contents: read
      attestations: write
      id-token: write
    steps:
      - name: Check out the repo
        uses: actions/checkout@v5

      - name: Log in to Docker Hub
        uses: docker/login-action@f4ef78c080cd8ba55a85445d5b36e214a81df20a
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Log in to the Container registry
        uses: docker/login-action@65b78e6e13532edd9afa3aa52ac7964289d1a9c1
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata (tags, labels) for Docker
        id: meta
        uses: docker/metadata-action@9ec57ed1fcdbf14dcef7dfbe97b2010124a938b7
        with:
          images: |
            my-docker-hub-namespace/my-docker-hub-repository
            ghcr.io/${{ github.repository }}

      - name: Build and push Docker images
        id: push
        uses: docker/build-push-action@3b5e8027fcad23fda98b2e3ac259d8d67585f671
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}

      - name: Generate artifact attestation
        uses: actions/attest-build-provenance@v3
        with:
          subject-name: ghcr.io/${{ github.repository }}
          subject-digest: ${{ steps.push.outputs.digest }}
          push-to-registry: true
```

上述工作流检出 GitHub 代码仓库，使用 `login-action` 两次登录到两个注册表，并使用 `metadata-action` 动作生成标签和标签。然后， `build-push-action` 动作构建并推送 Docker 镜像到 Docker Hub 和容器注册表。

在最后一步，它为镜像生成一个工件证明，这增加了供应链安全性。有关更多信息，请参见 [使用工件证明来建立构建的来源](https://docs.github.com/en/actions/security-guides/using-artifact-attestations-to-establish-provenance-for-builds) 。