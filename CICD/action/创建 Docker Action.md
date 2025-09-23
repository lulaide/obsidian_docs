---
source: "https://docs.github.com/en/actions/tutorials/use-containerized-services/create-a-docker-container-action"
tags:
  - "CICD"
---
# 创建 Docker Action

## 介绍

在本指南中，您将了解创建和使用打包的 Docker 容器操作所需的基本组件。为了将本指南集中在打包操作所需的组件上，操作代码的功能是最小的。该操作在日志中打印“Hello World”，或者如果您提供自定义名称，则打印“Hello \[who-to-greet\]”。

完成此项目后，您应该了解如何构建自己的 Docker 容器操作并在工作流中测试它。

自托管运行器必须使用 Linux 操作系统并安装 Docker 才能运行 Docker 容器操作。有关自托管运行器要求的更多信息，请参见 [自托管运行器](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/about-self-hosted-runners#requirements-for-self-hosted-runner-machines) 。

## 先决条件

- 您必须在 GitHub 上创建一个代码仓库并将其克隆到您的工作站。有关更多信息，请参见 [创建新代码仓库](https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-new-repository) 和 [克隆代码仓库](https://docs.github.com/en/repositories/creating-and-managing-repositories/cloning-a-repository) 。
- 如果您的代码仓库使用 Git LFS，您必须将对象包含在代码仓库的归档中。有关更多信息，请参见 [在代码仓库的归档中管理 Git LFS 对象](https://docs.github.com/en/enterprise-cloud@latest/repositories/managing-your-repositorys-settings-and-features/managing-repository-settings/managing-git-lfs-objects-in-archives-of-your-repository) 。
- 您可能会发现，基本了解 GitHub Actions、环境变量和 Docker 容器文件系统是有帮助的。有关更多信息，请参见 [将信息存储在变量中](https://docs.github.com/en/actions/learn-github-actions/variables) 和 [GitHub 托管的运行器](https://docs.github.com/en/enterprise-cloud@latest/actions/using-github-hosted-runners/about-github-hosted-runners#docker-container-filesystem) 。

## 创建 Dockerfile

在您的新 `hello-world-docker-action` 目录中，创建一个新的 `Dockerfile` 文件。如果您遇到问题，请确保文件名的大小写正确（使用大写的 `D` ，但不使用大写的 `f` ）。有关更多信息，请参见 [GitHub Actions 的 Dockerfile 支持](https://docs.github.com/en/actions/creating-actions/dockerfile-support-for-github-actions) 。

**Dockerfile**

```dockerfile
# Container image that runs your code
FROM alpine:3.10

# Copies your code file from your action repository to the filesystem path \`/\` of the container
COPY entrypoint.sh /entrypoint.sh

# Code file to execute when the docker container starts up (\`entrypoint.sh\`)
ENTRYPOINT ["/entrypoint.sh"]
```

在您上面创建的 `hello-world-docker-action` 目录中创建一个新的 `action.yml` 文件。有关更多信息，请参见 [元数据语法参考](https://docs.github.com/en/actions/creating-actions/metadata-syntax-for-github-actions) 。

**action.yml**

```yaml
# action.yml
name: 'Hello World'
description: 'Greet someone and record the time'
inputs:
  who-to-greet:  # id of input
    description: 'Who to greet'
    required: true
    default: 'World'
outputs:
  time: # id of output
    description: 'The time we greeted you'
runs:
  using: 'docker'
  image: 'Dockerfile'
  args:
    - ${{ inputs.who-to-greet }}
```

此元数据定义了一个 `who-to-greet` 输入和一个 `time` 输出参数。要将输入传递给 Docker 容器，您应该使用 `inputs` 声明输入，并在 `args` 关键字中传递输入。您在 `args` 中包含的所有内容都会传递给容器，但为了更好地让您的操作用户发现，我们建议使用输入。

GitHub 将从您的 `Dockerfile` 构建一个镜像，并使用该镜像在新容器中运行命令。

## 编写操作代码

您可以选择任何基础 Docker 镜像，因此可以为您的操作选择任何语言。以下 shell 脚本示例使用 `who-to-greet` 输入变量在日志文件中打印 "Hello \[who-to-greet\]"。

接下来，脚本获取当前时间并将其设置为输出变量，以便在作业中后续运行的操作可以使用。为了让 GitHub 识别输出变量，您必须将它们写入 `$GITHUB_OUTPUT` 环境文件： `echo "<output name>=<value>" >> $GITHUB_OUTPUT` 。有关更多信息，请参见 [GitHub Actions 的工作流命令](https://docs.github.com/en/actions/using-workflows/workflow-commands-for-github-actions#setting-an-output-parameter) 。

1. 在 `hello-world-docker-action` 目录中创建一个新的 `entrypoint.sh` 文件。
2. 将以下代码添加到您的 `entrypoint.sh` 文件中。
	**entrypoint.sh**
	```shell
	#!/bin/sh -l
	echo "Hello $1"
	time=$(date)
	echo "time=$time" >> $GITHUB_OUTPUT
	```
	如果 `entrypoint.sh` 执行没有任何错误，则操作的状态设置为 `success` 。您还可以在操作的代码中显式设置退出代码以提供操作的状态。有关更多信息，请参见 [设置操作的退出代码](https://docs.github.com/en/actions/creating-actions/setting-exit-codes-for-actions) 。
3. 使您的 `entrypoint.sh` 文件可执行。Git 提供了一种显式更改文件权限模式的方法，以便在每次克隆/分叉时不会重置。
	```shell
	git add entrypoint.sh
	git update-index --chmod=+x entrypoint.sh
	```
4. 可选地，要检查 git 索引中文件的权限模式，请运行以下命令。
	```shell
	git ls-files --stage entrypoint.sh
	```
	像 `100755 e69de29bb2d1d6434b8b29ae775ad8c2e48c5391 0       entrypoint.sh` 这样的输出表示该文件具有可执行权限。在此示例中， `755` 表示可执行权限。

## 创建 README

为了让人们知道如何使用你的操作，你可以创建一个 README 文件。当你计划公开分享你的操作时，README 最为有用，但它也是提醒你或你的团队如何使用该操作的好方法。

在你的 `hello-world-docker-action` 目录中，创建一个 `README.md` 文件，指定以下信息：

- 该操作的详细描述。
- 必需的输入和输出参数。
- 可选的输入和输出参数。
- 操作使用的机密。
- 操作使用的环境变量。
- 在工作流中使用您的操作的示例。

**README.md**

```markdown
# Hello world docker action

This action prints "Hello World" or "Hello" + the name of a person to greet to the log.

## Inputs

## \`who-to-greet\`

**Required** The name of the person to greet. Default \`"World"\`.

## Outputs

## \`time\`

The time we greeted you.

## Example usage

uses: actions/hello-world-docker-action@v2
with:
  who-to-greet: 'Mona the Octocat'
```

## 提交、标记并推送你的操作

从你的终端，提交你的 `action.yml` 、 `entrypoint.sh` 、 `Dockerfile` 和 `README.md` 文件。

为你的操作添加版本标签是最佳实践。有关版本控制的更多信息，请参见 [关于自定义操作](https://docs.github.com/en/actions/creating-actions/about-custom-actions#using-release-management-for-actions) 。

```shell
git add action.yml entrypoint.sh Dockerfile README.md
git commit -m "My first action is ready"
git tag -a -m "My first action release" v1
git push --follow-tags
```

## 在工作流中测试你的操作

现在你准备在工作流中测试你的操作了。

- 当一个动作位于私有代码仓库时，您可以控制谁可以访问它。有关更多信息，请参见 [管理代码仓库的 GitHub Actions 设置](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/enabling-features-for-your-repository/managing-github-actions-settings-for-a-repository#allowing-access-to-components-in-a-private-repository) 。
- 当一个动作位于内部代码仓库时，该动作只能在同一代码仓库的工作流中使用。
- 公共动作可以被任何代码仓库中的工作流使用。

### 使用公共操作的示例

以下工作流代码在公共 [`actions/hello-world-docker-action`](https://github.com/actions/hello-world-docker-action) 代码仓库中使用了已完成的 *hello world* 动作。将以下工作流示例代码复制到 `.github/workflows/main.yml` 文件中，但请将 `actions/hello-world-docker-action` 替换为您的代码仓库和动作名称。您还可以将 `who-to-greet` 输入替换为您的名字。即使公共动作未发布到 GitHub Marketplace，也可以使用。有关更多信息，请参见 [在 GitHub Marketplace 发布动作](https://docs.github.com/en/actions/creating-actions/publishing-actions-in-github-marketplace#publishing-an-action) 。

**.github/workflows/main.yml**

```yaml
on: [push]

jobs:
  hello_world_job:
    runs-on: ubuntu-latest
    name: A job to say hello
    steps:
      - name: Hello world action step
        id: hello
        uses: actions/hello-world-docker-action@v2
        with:
          who-to-greet: 'Mona the Octocat'
      # Use the output from the \`hello\` step
      - name: Get the output time
        run: echo "The time was ${{ steps.hello.outputs.time }}"
```

### 使用私有操作的示例

将以下示例工作流代码复制到您操作的代码仓库中的 `.github/workflows/main.yml` 文件中。您还可以将 `who-to-greet` 输入替换为您的名字。此私有操作无法发布到 GitHub Marketplace，并且只能在此代码仓库中使用。

**.github/workflows/main.yml**

```yaml
on: [push]

jobs:
  hello_world_job:
    runs-on: ubuntu-latest
    name: A job to say hello
    steps:
      # To use this repository's private action,
      # you must check out the repository
      - name: Checkout
        uses: actions/checkout@v5
      - name: Hello world action step
        uses: ./ # Uses an action in the root directory
        id: hello
        with:
          who-to-greet: 'Mona the Octocat'
      # Use the output from the \`hello\` step
      - name: Get the output time
        run: echo "The time was ${{ steps.hello.outputs.time }}"
```

在您的代码仓库中，点击 **操作** 选项卡，然后选择最新的工作流运行。在 **作业** 下或在可视化图表中，点击 **一个问候的作业** 。

点击 **你好，世界操作步骤** ，您应该在日志中看到“Hello Mona the Octocat”或您用于 `who-to-greet` 输入的名称。要查看时间戳，请点击 **获取输出时间** 。

## 访问容器操作创建的文件

当容器操作运行时，它会自动将运行器上的默认工作目录（ `GITHUB_WORKSPACE` ）映射到容器上的 `/github/workspace` 目录。添加到容器此目录中的任何文件都将对同一作业中的后续步骤可用。例如，如果您有一个构建项目的容器操作，并且您想将构建输出作为工件上传，您可以使用以下步骤。

**workflow.yml**

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v5

      # Output build artifacts to /github/workspace on the container.
      - name: Containerized Build
        uses: ./.github/actions/my-container-action

      - name: Upload Build Artifacts
        uses: actions/upload-artifact@v4
        with:
          name: workspace_artifacts
          path: ${{ github.workspace }}
```

有关将构建输出作为工件上传的更多信息，请参见 [使用工作流工件存储和共享数据](https://docs.github.com/en/actions/using-workflows/storing-workflow-data-as-artifacts) 。

## GitHub.com 上的示例 Docker 容器操作

您可以在 GitHub.com 上找到许多 Docker 容器操作的示例。

- [github/issue-metrics](https://github.com/github/issue-metrics)
- [microsoft/infersharpaction](https://github.com/microsoft/infersharpaction)
- [microsoft/ps-docs](https://github.com/microsoft/ps-docs)