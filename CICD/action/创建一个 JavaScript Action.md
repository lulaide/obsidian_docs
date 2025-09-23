---
source: "https://docs.github.com/en/actions/tutorials/create-actions/create-a-javascript-action"
tags:
  - "CICD"
---
# 创建一个 JavaScript Action

## 介绍

在本指南中，您将了解创建和使用打包 JavaScript 操作所需的基本组件。为了将本指南的重点放在打包操作所需的组件上，操作代码的功能是最小的。该操作在日志中打印“Hello World”，或者如果您提供自定义名称，则打印“Hello \[who-to-greet\]”。

本指南使用 GitHub Actions Toolkit Node.js 模块来加速开发。有关更多信息，请参见 [actions/toolkit](https://github.com/actions/toolkit) 代码仓库。

完成此项目后，您应该了解如何构建自己的 JavaScript 操作并在工作流中进行测试。

为了确保您的 JavaScript 操作与所有 GitHub 托管的运行器（Ubuntu、Windows 和 macOS）兼容，您编写的打包 JavaScript 代码应为纯 JavaScript，并且不依赖于其他二进制文件。JavaScript 操作直接在运行器上运行，并使用已经存在于运行器镜像中的二进制文件。

## 先决条件

在开始之前，您需要下载 Node.js 并创建一个公共 GitHub 代码仓库。

1. 下载并安装 Node.js 20.x，其中包括 npm。
	[https://nodejs.org/en/download/](https://nodejs.org/en/download/)
2. 在 GitHub 上创建一个新的公共代码仓库，并将其命名为 "hello-world-javascript-action"。有关更多信息，请参见 [创建新代码仓库](https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-new-repository) 。
3. 将您的代码仓库克隆到您的计算机上。有关更多信息，请参见 [克隆代码仓库](https://docs.github.com/en/repositories/creating-and-managing-repositories/cloning-a-repository) 。
4. 在终端中，切换到您的新代码仓库目录。
	```shell
	cd hello-world-javascript-action
	```
5. 从你的终端中，使用 npm 初始化目录以生成一个 `package.json` 文件。
	```shell
	npm init -y
	```

在 `hello-world-javascript-action` 目录中创建一个名为 `action.yml` 的新文件，并包含以下示例代码。有关更多信息，请参见 [元数据语法参考](https://docs.github.com/en/actions/creating-actions/metadata-syntax-for-github-actions) 。

```yaml
name: Hello World
description: Greet someone and record the time

inputs:
  who-to-greet: # id of input
    description: Who to greet
    required: true
    default: World

outputs:
  time: # id of output
    description: The time we greeted you

runs:
  using: node20
  main: dist/index.js
```

此文件定义了 `who-to-greet` 输入和 `time` 输出。它还告诉操作运行器如何开始运行此 JavaScript 操作。

## 添加操作工具包包

操作工具包是一个 Node.js 包的集合，允许您快速构建更一致的 JavaScript 操作。

工具包 [`@actions/core`](https://github.com/actions/toolkit/tree/main/packages/core) 包提供了与工作流命令、输入和输出变量、退出状态以及调试消息的接口。

该工具包还提供了一个 [`@actions/github`](https://github.com/actions/toolkit/tree/main/packages/github) 包，该包返回一个经过身份验证的 Octokit REST 客户端，并访问 GitHub Actions 上下文。

该工具包提供的不仅仅是 `core` 和 `github` 包。有关更多信息，请参见 [actions/toolkit](https://github.com/actions/toolkit) 代码仓库。

在您的终端中，安装 actions 工具包 `core` 和 `github` 包。

```shell
npm install @actions/core @actions/github
```

您现在应该看到一个 `node_modules` 目录和一个 `package-lock.json` 文件，这些文件跟踪任何已安装的依赖项及其版本。您不应该将 `node_modules` 目录提交到您的代码仓库中。

## 编写操作代码

此操作使用工具包获取操作元数据文件中所需的 `who-to-greet` 输入变量，并在日志中的调试消息中打印“Hello \[who-to-greet\]”。接下来，脚本获取当前时间并将其设置为输出变量，以便在作业中后续运行的操作可以使用。

GitHub Actions 提供有关 Webhook 事件、Git 引用、工作流、操作和触发工作流的人的上下文信息。要访问上下文信息，可以使用 `github` 包。您将编写的操作将把 Webhook 事件有效负载打印到日志中。

添加一个名为 `src/index.js` 的新文件，内容如下。

```javascript
import * as core from "@actions/core";
import * as github from "@actions/github";

try {
  // \`who-to-greet\` input defined in action metadata file
  const nameToGreet = core.getInput("who-to-greet");
  core.info(\`Hello ${nameToGreet}!\`);

  // Get the current time and set it as an output variable
  const time = new Date().toTimeString();
  core.setOutput("time", time);

  // Get the JSON webhook payload for the event that triggered the workflow
  const payload = JSON.stringify(github.context.payload, undefined, 2);
  core.info(\`The event payload: ${payload}\`);
} catch (error) {
  core.setFailed(error.message);
}
```

如果在上述 `index.js` 示例中抛出错误， `core.setFailed(error.message);` 使用 actions toolkit [`@actions/core`](https://github.com/actions/toolkit/tree/main/packages/core) 包来记录消息并设置失败的退出代码。有关更多信息，请参见 [为操作设置退出代码](https://docs.github.com/en/actions/creating-actions/setting-exit-codes-for-actions) 。

## 创建 README

为了让人们知道如何使用你的操作，你可以创建一个 README 文件。当你计划公开分享你的操作时，README 最为有用，但它也是提醒你或你的团队如何使用该操作的好方法。

在你的 `hello-world-javascript-action` 目录中，创建一个 `README.md` 文件，指定以下信息：

- 该操作的详细描述。
- 必需的输入和输出参数。
- 可选的输入和输出参数。
- 操作使用的机密。
- 操作使用的环境变量。
- 在工作流中使用您的操作的示例。

```markdown
# Hello world JavaScript action

This action prints "Hello World" or "Hello" + the name of a person to greet to the log.

## Inputs

### \`who-to-greet\`

**Required** The name of the person to greet. Default \`"World"\`.

## Outputs

### \`time\`

The time we greeted you.

## Example usage

\`\`\`yaml
uses: actions/hello-world-javascript-action@e76147da8e5c81eaf017dede5645551d4b94427b
with:
  who-to-greet: Mona the Octocat
\`\`\`
```

## 提交、标记并推送你的操作

GitHub 在运行时下载工作流中的每个操作运行，并将其作为完整的代码包执行，然后您才能使用工作流命令，如 `run` ，与运行器机器进行交互。这意味着您必须包含运行 JavaScript 代码所需的任何包依赖项。例如，此操作使用 `@actions/core` 和 `@actions/github` 包。

检查您的 `node_modules` 目录可能会导致问题。作为替代方案，您可以使用 [`rollup.js`](https://github.com/rollup/rollup) 或 [`@vercel/ncc`](https://github.com/vercel/ncc) 等工具将您的代码和依赖项合并为一个文件以进行分发。

1. 通过在终端中运行此命令来安装 `rollup` 及其插件。
	`npm install --save-dev rollup @rollup/plugin-commonjs @rollup/plugin-node-resolve`
2. 在您的代码仓库根目录中创建一个名为 `rollup.config.js` 的新文件，并添加以下代码。
	```javascript
	import commonjs from "@rollup/plugin-commonjs";
	import { nodeResolve } from "@rollup/plugin-node-resolve";
	const config = {
	  input: "src/index.js",
	  output: {
	    esModule: true,
	    file: "dist/index.js",
	    format: "es",
	    sourcemap: true,
	  },
	  plugins: [commonjs(), nodeResolve({ preferBuiltins: true })],
	};
	export default config;
	```
3. 编译你的 `dist/index.js` 文件。
	`rollup --config rollup.config.js`
	你会看到一个新的 `dist/index.js` 文件，其中包含你的代码和任何依赖项。
4. 在你的终端中，提交更新。
	```shell
	git add src/index.js dist/index.js rollup.config.js package.json package-lock.json README.md action.yml
	git commit -m "Initial commit of my first action"
	git tag -a -m "My first action release" v1.1
	git push --follow-tags
	```

当你提交并推送你的代码时，你的更新后的代码仓库应该如下所示：

```shell
hello-world-javascript-action/
├── action.yml
├── dist/
│   └── index.js
├── package.json
├── package-lock.json
├── README.md
├── rollup.config.js
└── src/
    └── index.js
```

## 在工作流中测试你的操作

现在你准备在工作流中测试你的操作了。

公共操作可以被任何代码仓库中的工作流使用。当操作位于私有代码仓库时，代码仓库设置决定该操作是否仅在同一代码仓库内可用，或也可用于同一用户或组织拥有的其他代码仓库。有关更多信息，请参见 [管理代码仓库的 GitHub Actions 设置](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/enabling-features-for-your-repository/managing-github-actions-settings-for-a-repository) 。

### 使用公共操作的示例

此示例演示了如何从外部代码仓库运行您的新公共操作。

将以下 YAML 复制到 `.github/workflows/main.yml` 中的新文件，并用您的用户名和您上面创建的公共代码仓库的名称更新 `uses: octocat/hello-world-javascript-action@1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b` 行。您还可以将 `who-to-greet` 输入替换为您的名字。

```yaml
on:
  push:
    branches:
      - main

jobs:
  hello_world_job:
    name: A job to say hello
    runs-on: ubuntu-latest

    steps:
      - name: Hello world action step
        id: hello
        uses: octocat/hello-world-javascript-action@1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b
        with:
          who-to-greet: Mona the Octocat

      # Use the output from the \`hello\` step
      - name: Get the output time
        run: echo "The time was ${{ steps.hello.outputs.time }}"
```

当此工作流被触发时，运行器将从您的公共代码仓库下载 `hello-world-javascript-action` 动作并执行它。

### 使用私有操作的示例

将工作流代码复制到您操作的代码仓库中的 `.github/workflows/main.yml` 文件中。您还可以将 `who-to-greet` 输入替换为您的名字。

```yaml
on:
  push:
    branches:
      - main

jobs:
  hello_world_job:
    name: A job to say hello
    runs-on: ubuntu-latest

    steps:
      # To use this repository's private action,
      # you must check out the repository
      - name: Checkout
        uses: actions/checkout@v5

      - name: Hello world action step
        uses: ./ # Uses an action in the root directory
        id: hello
        with:
          who-to-greet: Mona the Octocat

      # Use the output from the \`hello\` step
      - name: Get the output time
        run: echo "The time was ${{ steps.hello.outputs.time }}"
```

在您的代码仓库中，点击 **操作** 选项卡，然后选择最新的工作流运行。在 **作业** 下或在可视化图表中，点击 **一个问候的作业** 。

点击 **你好，世界操作步骤** ，您应该在日志中看到“Hello Mona the Octocat”或您用于 `who-to-greet` 输入的名称。要查看时间戳，请点击 **获取输出时间** 。

## 用于创建 JavaScript 操作的模板代码仓库

GitHub 提供了用于创建 JavaScript 和 TypeScript 操作的模板代码仓库。您可以使用这些模板快速开始创建一个包含测试、代码检查和其他推荐实践的新操作。

- [`javascript-action` 模板代码仓库](https://github.com/actions/javascript-action)
- [`typescript-action` 模板代码仓库](https://github.com/actions/typescript-action)

## GitHub.com 上的示例 JavaScript 操作

您可以在 GitHub.com 上找到许多 JavaScript 操作的示例。

- [DevExpress/testcafe-action](https://github.com/DevExpress/testcafe-action)
- [duckduckgo/privacy-configuration](https://github.com/duckduckgo/privacy-configuration)