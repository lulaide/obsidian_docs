---
source: "https://docs.github.com/en/actions/concepts/workflows-and-actions/custom-actions"
tags:
  - "CICD"
---
# 关于自定义 action

## 关于自定义 action

您可以通过编写与您的代码仓库交互的自定义代码来创建操作，包括与 GitHub 的 API 以及任何公开可用的第三方 API 集成。例如，一个操作可以发布 npm 模块，在创建紧急问题时发送 SMS 警报，或部署生产就绪的代码。

您可以编写自己的操作以在工作流中使用，或与 GitHub 社区分享您构建的操作。要与所有人分享您构建的操作，您的代码仓库必须是公开的。

操作可以直接在机器上或在 Docker 容器中运行。您可以定义操作的输入、输出和环境变量。

## 操作类型

注意

您可以构建 Docker 容器、JavaScript 和复合操作。操作需要一个元数据文件来定义操作的输入、输出和运行配置。操作元数据文件使用 YAML 语法，元数据文件名必须是 `action.yml` 或 `action.yaml` 。首选格式是 `action.yml` 。

### Docker 容器操作

Docker 容器将环境与 GitHub Actions 代码打包在一起。这创建了一个更一致和可靠的工作单元，因为操作的消费者不需要担心工具或依赖项。

Docker 容器允许您使用特定版本的操作系统、依赖项、工具和代码。对于必须在特定环境配置中运行的操作，Docker 是一个理想的选择，因为您可以自定义操作系统和工具。由于构建和检索容器的延迟，Docker 容器操作的速度比 JavaScript 操作慢。

Docker 容器操作只能在运行 Linux 操作系统的运行器上执行。自托管运行器必须使用 Linux 操作系统并安装 Docker 才能运行 Docker 容器操作。有关自托管运行器要求的更多信息，请参见 [自托管运行器](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/about-self-hosted-runners#requirements-for-self-hosted-runner-machines) 。

### JavaScript 操作

JavaScript 操作可以直接在运行器机器上运行，并将操作代码与用于运行代码的环境分开。使用 JavaScript 操作简化了操作代码，并且比 Docker 容器操作执行得更快。

为了确保您的 JavaScript 操作与所有 GitHub 托管的运行器（Ubuntu、Windows 和 macOS）兼容，您编写的打包 JavaScript 代码应为纯 JavaScript，并且不依赖于其他二进制文件。JavaScript 操作直接在运行器上运行，并使用已经存在于运行器镜像中的二进制文件。

如果您正在开发 Node.js 项目，GitHub Actions 工具包提供了可以在您的项目中使用的包，以加快开发速度。有关更多信息，请参见 [actions/toolkit](https://github.com/actions/toolkit) 代码仓库。

### 复合操作

一个 *复合* 动作允许您将多个工作流步骤组合在一个动作中。例如，您可以使用此功能将多个运行命令打包成一个动作，然后有一个工作流使用该动作将打包的命令作为一个步骤执行。要查看示例，请查看 [创建复合动作](https://docs.github.com/en/actions/creating-actions/creating-a-composite-action) 。

## 下一步

要了解如何管理您的自定义动作，请参阅 [管理自定义动作](https://docs.github.com/en/actions/how-tos/administering-github-actions/managing-custom-actions) 。