---
source: "https://docs.github.com/en/actions/get-started/understand-github-actions"
tags:
  - "action"
---
# 理解 GitHub Actions

## 概述

GitHub Actions 是一个持续集成和持续交付 (CI/CD) 平台，允许您自动化构建、测试和部署流程。您可以创建工作流，以构建和测试每个对您的代码仓库的拉取请求，或将合并的拉取请求部署到生产环境。

GitHub Actions 超越了 DevOps，让您在代码仓库中发生其他事件时运行工作流。例如，您可以运行一个工作流，以便在有人在您的代码仓库中创建新问题时自动添加适当的标签。

GitHub 提供 Linux、Windows 和 macOS 虚拟机来运行您的工作流，或者您可以在自己的数据中心或云基础设施中托管自己的自托管运行器。

## GitHub Actions 的组件

您可以配置一个 GitHub Actions **工作流** ，以便在您的代码仓库中发生 **事件** 时触发，例如打开拉取请求或创建问题。您的工作流包含一个或多个 **作业** ，可以按顺序或并行运行。每个作业将在其自己的虚拟机 **运行器** 中运行，或在容器内运行，并具有一个或多个 **步骤** ，这些步骤要么运行您定义的脚本，要么运行 **操作** ，这是一种可重用的扩展，可以简化您的工作流。

![Diagram of an event triggering Runner 1 to run Job 1, which triggers Runner 2 to run Job 2. Each of the jobs is broken into multiple steps.](https://docs.github.com/assets/cb-25535/mw-1440/images/help/actions/overview-actions-simple.webp)

### 工作流

一个 **工作流** 是一个可配置的自动化过程，它将运行一个或多个作业。工作流由一个 YAML 文件定义，该文件被检查到你的代码仓库中，并将在你的代码仓库中由事件触发时运行，或者可以手动触发，或在定义的时间表上触发。

工作流在代码仓库中的 `.github/workflows` 目录中定义。一个代码仓库可以有多个工作流，每个工作流可以执行不同的一组任务，例如：

- 构建和测试拉取请求
- 每次创建发布时部署您的应用程序
- 每当打开新问题时添加标签

您可以在一个工作流中引用另一个工作流。有关更多信息，请参见 [重用工作流](https://docs.github.com/en/actions/using-workflows/reusing-workflows) 。

有关更多信息，请参见 [编写工作流](https://docs.github.com/en/actions/using-workflows) 。

### 事件

**事件** 是在代码仓库中触发 **工作流** 运行的特定活动。例如，当有人创建拉取请求、打开问题或向代码仓库推送提交时，活动可以源自 GitHub。您还可以通过 [计划](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#schedule) 触发工作流运行，通过 [向 REST API 发送请求](https://docs.github.com/en/rest/repos/repos#create-a-repository-dispatch-event) ，或手动触发。

要获取可以用来触发工作流的事件的完整列表，请参见 [触发工作流的事件](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows) 。

### 作业

**作业** 是一组在同一个 **运行器** 上执行的 **步骤** 。每个步骤要么是将要执行的 Shell 脚本，要么是将要运行的 **操作** 。步骤按顺序执行，并且相互依赖。由于每个步骤在同一个运行器上执行，您可以将数据从一个步骤共享到另一个步骤。例如，您可以有一个步骤构建您的应用程序，接着是一个测试已构建应用程序的步骤。

您可以配置作业与其他作业的依赖关系；默认情况下，作业没有依赖关系并且并行运行。当一个作业依赖于另一个作业时，它会在运行之前等待依赖作业完成。

您还可以使用 **矩阵** 多次运行相同的作业，每次使用不同的变量组合——例如操作系统或语言版本。

例如，您可以为不同的架构配置多个构建作业，而不需要任何作业依赖关系，并且有一个依赖于这些构建的打包作业。构建作业并行运行，一旦成功完成，打包作业就会运行。

有关更多信息，请参见 [选择您的工作流执行的内容](https://docs.github.com/en/actions/using-jobs) 。

### 操作

**操作** 是一组预定义的、可重用的作业或代码，在 **工作流** 中执行特定任务，从而减少您在工作流文件中编写的重复代码量。操作可以执行以下任务：

- 从 GitHub 拉取您的 Git 代码仓库
- 为您的构建环境设置正确的工具链
- 设置对云服务提供商的认证

您可以编写自己的操作，或者可以在 GitHub Marketplace 中找到可用于您的工作流的操作。

有关操作的更多信息，请参见 [重用自动化](https://docs.github.com/en/actions/creating-actions) 。

### Runners

一个 **runner** 是一个在触发时运行您的工作流的服务器。每个 runner 一次只能运行一个 **job** 。GitHub 提供 Ubuntu Linux、Microsoft Windows 和 macOS runner 来运行您的 **workflows** 。每次工作流运行都在一个全新、刚配置的虚拟机中执行。

GitHub 还提供更大的运行器，这些运行器可在更大的配置中使用。有关更多信息，请参见 [使用更大的运行器](https://docs.github.com/en/actions/using-github-hosted-runners/using-larger-runners) 。

如果您需要不同的操作系统或特定的硬件配置，您可以托管自己的运行器。

有关自托管运行器的更多信息，请参见 [管理自托管运行器](https://docs.github.com/en/actions/how-tos/managing-self-hosted-runners) 。

## 下一步

GitHub Actions 可以帮助您自动化应用程序开发过程的几乎每个方面。准备好开始了吗？以下是一些有用的资源，帮助您在 GitHub Actions 上迈出下一步：

- 要创建 GitHub Actions 工作流，请参阅 [使用工作流模板](https://docs.github.com/en/actions/learn-github-actions/using-starter-workflows) 。
- 有关持续集成 (CI) 工作流，请参阅 [构建和测试您的代码](https://docs.github.com/en/actions/automating-builds-and-tests) 。
- 有关构建和发布包，请参阅 [发布包](https://docs.github.com/en/actions/publishing-packages) 。
- 有关项目部署，请参见 [部署到第三方平台](https://docs.github.com/en/actions/deployment) 。
- 有关在 GitHub 上自动化任务和流程的信息，请参见 [使用 GitHub Actions 管理您的工作](https://docs.github.com/en/actions/managing-issues-and-pull-requests) 。
- 有关演示 GitHub Actions 更复杂功能的示例，请参见 [使用 GitHub Actions 管理您的工作](https://docs.github.com/en/actions/examples) 。这些详细示例解释了如何在运行器上测试代码、访问 GitHub CLI，以及使用并发和测试矩阵等高级功能。
- 要证明您在使用 GitHub Actions 自动化工作流和加速开发方面的熟练程度，请通过 GitHub 认证获得 GitHub Actions 证书。有关更多信息，请参见 [关于 GitHub 认证](https://docs.github.com/en/get-started/showcase-your-expertise-with-github-certifications/about-github-certifications) 。