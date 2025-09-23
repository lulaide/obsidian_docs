---
source: "https://docs.github.com/en/actions/tutorials/create-actions/create-a-composite-action?platform=linux"
tags:
  - "CICD"
---
# 构建一个 Composite Action
## 介绍

在本指南中，您将了解创建和使用打包复合操作所需的基本组件。为了将本指南集中在打包操作所需的组件上，操作代码的功能是最小的。该操作打印“Hello World”，然后打印“Goodbye”，或者如果您提供自定义名称，它会打印“Hello \[who-to-greet\]”，然后打印“Goodbye”。该操作还将一个随机数映射到 `random-number` 输出变量，并运行名为 `goodbye.sh` 的脚本。

完成此项目后，您应该了解如何构建自己的复合操作并在工作流中测试它。

### 复合操作和可重用工作流

复合操作允许您将一系列工作流作业步骤收集到一个单一的操作中，然后可以在多个工作流中作为单个作业步骤运行。可重用工作流提供了另一种避免重复的方法，允许您从其他工作流中运行完整的工作流。有关更多信息，请参见 [重用工作流配置](https://docs.github.com/en/actions/using-workflows/avoiding-duplication) 。

## 先决条件

在开始之前，您需要在 GitHub 上创建一个代码仓库。

1. 在 GitHub 上创建一个新的公共代码仓库。您可以选择任何代码仓库名称，或使用以下 `hello-world-composite-action` 示例。您可以在项目推送到 GitHub 后添加这些文件。有关更多信息，请参见 [创建新代码仓库](https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-new-repository) 。
2. 将您的代码仓库克隆到您的计算机上。有关更多信息，请参见 [克隆代码仓库](https://docs.github.com/en/repositories/creating-and-managing-repositories/cloning-a-repository) 。
3. 在终端中，切换到您的新代码仓库目录。
	```shell
	cd hello-world-composite-action
	```
4. 在 `hello-world-composite-action` 代码仓库中，创建一个名为 `goodbye.sh` 的新文件，并添加示例代码：
	```shell
	echo "echo Goodbye" > goodbye.sh
	```
5. 在终端中，使 `goodbye.sh` 文件可执行。
	```shell
	chmod +x goodbye.sh
	```
6. 在终端中，检查你的 `goodbye.sh` 文件。
	```shell
	git add goodbye.sh
	git commit -m "Add goodbye script"
	git push
	```
1. 在 `hello-world-composite-action` 代码仓库中，创建一个名为 `action.yml` 的新文件，并添加以下示例代码。有关此语法的更多信息，请参见 [元数据语法参考](https://docs.github.com/en/actions/creating-actions/metadata-syntax-for-github-actions#runs-for-composite-actions) 。
	```yaml
	name: 'Hello World'
	description: 'Greet someone'
	inputs:
	  who-to-greet:  # id of input
	    description: 'Who to greet'
	    required: true
	    default: 'World'
	outputs:
	  random-number:
	    description: "Random number"
	    value: ${{ steps.random-number-generator.outputs.random-number }}
	runs:
	  using: "composite"
	  steps:
	    - name: Set Greeting
	      run: echo "Hello $INPUT_WHO_TO_GREET."
	      shell: bash
	      env:
	        INPUT_WHO_TO_GREET: ${{ inputs.who-to-greet }}
	    - name: Random Number Generator
	      id: random-number-generator
	      run: echo "random-number=$(echo $RANDOM)" >> $GITHUB_OUTPUT
	      shell: bash
	    - name: Set GitHub Path
	      run: echo "$GITHUB_ACTION_PATH" >> $GITHUB_PATH
	      shell: bash
	      env:
	        GITHUB_ACTION_PATH: ${{ github.action_path }}
	    - name: Run goodbye.sh
	      run: goodbye.sh
	      shell: bash
	```
	此文件定义了 `who-to-greet` 输入，将随机生成的数字映射到 `random-number` 输出变量，将操作的路径添加到运行器系统路径（以便在执行期间找到 `goodbye.sh` 脚本），并运行 `goodbye.sh` 脚本。
	有关管理输出的更多信息，请参见 [元数据语法参考](https://docs.github.com/en/actions/creating-actions/metadata-syntax-for-github-actions#outputs-for-composite-actions) 。
	有关如何使用 `github.action_path` 的更多信息，请参见 [上下文参考](https://docs.github.com/en/actions/learn-github-actions/contexts#github-context) 。
2. 在您的终端中，检查您的 `action.yml` 文件。
	```shell
	git add action.yml
	git commit -m "Add action"
	git push
	```
3. 在您的终端中，添加一个标签。此示例使用名为 `v1` 的标签。有关更多信息，请参见 [关于自定义操作](https://docs.github.com/en/actions/creating-actions/about-custom-actions#using-release-management-for-actions) 。
	```shell
	git tag -a -m "Description of this release" v1
	git push --follow-tags
	```

## 在工作流中测试你的操作

以下工作流代码使用了您在 [创建复合操作](https://docs.github.com/en/actions/creating-actions/creating-a-composite-action#creating-an-action-metadata-file) 中完成的 hello world 操作。

将工作流代码复制到另一个代码仓库中的 `.github/workflows/main.yml` 文件中，分别用代码仓库的所有者和您想要使用的提交的 SHA 替换 `OWNER` 和 `SHA` 。您还可以用您的名字替换 `who-to-greet` 输入。

```yaml
on: [push]

jobs:
  hello_world_job:
    runs-on: ubuntu-latest
    name: A job to say hello
    steps:
      - uses: actions/checkout@v5
      - id: foo
        uses: OWNER/hello-world-composite-action@SHA
        with:
          who-to-greet: 'Mona the Octocat'
      - run: echo random-number "$RANDOM_NUMBER"
        shell: bash
        env:
          RANDOM_NUMBER: ${{ steps.foo.outputs.random-number }}
```

从你的代码仓库中，点击 **操作** 标签，然后选择最新的工作流运行。输出应包括：“Hello Mona the Octocat”、“再见”脚本的结果，以及一个随机数。

## 在同一代码仓库中创建复合操作

1. 创建一个名为 `hello-world-composite-action` 的新子文件夹，可以放在代码仓库中的任何子文件夹内。然而，建议将其放在 `.github/actions` 子文件夹中，以便于组织。
2. 在 `hello-world-composite-action` 文件夹中，执行相同的步骤以创建 `goodbye.sh` 脚本。
	```shell
	echo "echo Goodbye" > goodbye.sh
	```
	```shell
	chmod +x goodbye.sh
	```
	```shell
	git add goodbye.sh
	git commit -m "Add goodbye script"
	git push
	```
3. 在 `hello-world-composite-action` 文件夹中，根据 [创建复合操作](https://docs.github.com/en/actions/creating-actions/creating-a-composite-action#creating-an-action-metadata-file) 中的步骤创建 `action.yml` 文件。
4. 使用该操作时，在 `uses` 键中使用复合操作的 `action.yml` 文件所在文件夹的相对路径。以下示例假设它位于 `.github/actions/hello-world-composite-action` 文件夹中。

```yaml
on: [push]

jobs:
  hello_world_job:
    runs-on: ubuntu-latest
    name: A job to say hello
    steps:
      - uses: actions/checkout@v5
      - id: foo
        uses: ./.github/actions/hello-world-composite-action
        with:
          who-to-greet: 'Mona the Octocat'
      - run: echo random-number "$RANDOM_NUMBER"
        shell: bash
        env:
          RANDOM_NUMBER: ${{ steps.foo.outputs.random-number }}
```

## GitHub 上的示例复合操作

您可以在 GitHub 上找到许多复合操作的示例。

- [microsoft/action-python](https://github.com/microsoft/action-python)
- [microsoft/gpt-review](https://github.com/microsoft/gpt-review)
- [tailscale/github-action](https://github.com/tailscale/github-action)