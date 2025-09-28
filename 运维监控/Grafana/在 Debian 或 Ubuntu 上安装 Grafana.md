---
source: https://grafana.com/docs/grafana/latest/setup-grafana/installation/debian/
tags:
  - Grafana
---
本主题解释了如何安装 Grafana 依赖项，在 13. 将 `/usr/local/grafana` 的所有者更改为 Grafana 用户，以将所有权应用到新创建的 `/usr/local/grafana/data` 目录：
	```bash
	sudo chown -R grafana:users /usr/local/grafana
	``` Debian 或 Ubuntu 上安装 Grafana，以及在您的 Debian 或 Ubuntu 系统上启动 Grafana 服务器。

安装 Grafana 有多种方法：使用 Grafana Labs APT 代码仓库、下载 `.deb` 包或下载二进制 `.tar.gz` 文件。请仅选择以下最适合您需求的一种方法。

> 注意
> 
> 如果您通过 `.deb` 包或 `.tar.gz` 文件安装，则必须手动更新 Grafana 以获取每个新版本。

以下视频演示了如何按照本文件中的说明在 Debian 和 Ubuntu 上安装 Grafana：

![YouTube Video](https://img.youtube.com/vi/_Zk_XQSjF_Q/maxresdefault.jpg)

## 从 APT 代码仓库安装

如果您从 APT 代码仓库安装，Grafana 会在您运行 `apt-get update` 时自动更新。

| Grafana 版本 | 包 | 代码仓库 |
| --- | --- | --- |
| Grafana 企业版 | grafana-enterprise | `https://apt.grafana.com stable main` |
| Grafana Enterprise（测试版） | grafana-enterprise | `https://apt.grafana.com beta main` |
| Grafana OSS | grafana | `https://apt.grafana.com stable main` |
| Grafana OSS（测试版） | grafana | `https://apt.grafana.com beta main` |

> 注意
> 
> Grafana Enterprise 是推荐的默认版本。它是免费的，并包含 OSS 版本的所有功能。您还可以升级到 [完整的企业功能集](https://grafana.com/products/enterprise/?utm_source=grafana-install-page) ，该功能集支持 [企业插件](https://grafana.com/grafana/plugins/?enterprise=1&utcm_source=grafana-install-page) 。

完成以下步骤以从 APT 代码仓库安装 Grafana：

1. 安装所需的包：
	```bash
	sudo apt-get install -y apt-transport-https software-properties-common wget
	```
2. 导入 GPG 密钥：
	```bash
	sudo mkdir -p /etc/apt/keyrings/
	wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg > /dev/null
	```
3. 要添加一个稳定版本的代码仓库，请运行以下命令：
	```bash
	echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
	```
4. 要添加 beta 版本的代码仓库，请运行以下命令：
	```bash
	echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com beta main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
	```
5. 运行以下命令以更新可用软件包的列表：
	```bash
	# Updates the list of available packages
	sudo apt-get update
	```
6. 要安装 Grafana OSS，请运行以下命令：
	```bash
	# Installs the latest OSS release:
	sudo apt-get install grafana
	```
7. 要安装 Grafana Enterprise，请运行以下命令：
	```bash
	# Installs the latest Enterprise release:
	sudo apt-get install grafana-enterprise
	```

## 使用 deb 包安装 Grafana

如果您使用 deb 包手动安装 Grafana，则必须手动更新 Grafana 以获取每个新版本。

完成以下步骤以使用 deb 包安装 Grafana：

1. 导航到 [Grafana 下载页面](https://grafana.com/grafana/download) 。
2. 选择您想要安装的 Grafana 版本。
	- 默认情况下选择最新的 Grafana 版本。
	- **版本** 字段仅显示标记的发布版本。如果您想安装夜间构建，请点击 **夜间构建** ，然后选择一个版本。
3. 选择一个 **版本** 。
	- **企业版：** 这是推荐的版本。它在功能上与开源版本完全相同，但包括您可以通过许可证解锁的功能（如果您选择的话）。
	- **开源版：** 此版本在功能上与企业版完全相同，但如果您想要企业功能，则需要下载企业版。
4. 根据您运行的系统，点击 [下载页面](https://grafana.com/grafana/download) 上的 **Linux** 或 **ARM** 选项卡。
5. 将代码从 [下载页面](https://grafana.com/grafana/download) 复制并粘贴到命令行中并运行。

## 将 Grafana 安装为独立二进制文件

完成以下步骤以使用独立二进制文件安装 Grafana：

1. 导航到 [Grafana 下载页面](https://grafana.com/grafana/download) 。
2. 选择您要安装的 Grafana 版本。
	- 默认情况下选择最新的 Grafana 版本。
	- **版本** 字段仅显示标记的发布版本。如果您想安装夜间构建，请点击 **夜间构建** ，然后选择一个版本。
3. 选择一个 **版本** 。
	- **企业版：** 这是推荐的版本。它在功能上与开源版本完全相同，但包含您可以选择通过许可证解锁的功能。
	- **开源版：** 此版本在功能上与企业版完全相同，但如果您想要企业功能，则需要下载企业版。
4. 根据您运行的系统，点击 [下载页面](https://grafana.com/grafana/download) 上的 **Linux** 或 **ARM** 选项卡。
5. 将 [下载页面](https://grafana.com/grafana/download) 上的代码复制并粘贴到您的命令行中并运行。
6. 在您的系统上为 Grafana 创建一个用户帐户：
	```bash
	sudo useradd -r -s /bin/false grafana
	```
7. 将解压后的二进制文件移动到 `/usr/local/grafana` ：
	```bash
	sudo mv <DOWNLOAD PATH> /usr/local/grafana
	```
8. 将 `/usr/local/grafana` 的所有者更改为 Grafana 用户：
	```bash
	sudo chown -R grafana:users /usr/local/grafana
	```
9. 创建一个 Grafana 服务器 systemd 单元文件：
	```bash
	sudo touch /etc/systemd/system/grafana-server.service
	```
10. 在您选择的文本编辑器中将以下内容添加到单元文件中：
	```ini
	[Unit]
	Description=Grafana Server
	After=network.target
	[Service]
	Type=simple
	User=grafana
	Group=users
	ExecStart=/usr/local/grafana/bin/grafana server --config=/usr/local/grafana/conf/grafana.ini --homepath=/usr/local/grafana
	Restart=on-failure
	[Install]
	WantedBy=multi-user.target
	```
11. 使用二进制文件手动启动 Grafana 服务器：
	```bash
	/usr/local/grafana/bin/grafana server --homepath /usr/local/grafana
	```
	> 注意
	> 
	> 在此步骤中手动调用二进制文件会自动创建 `/usr/local/grafana/data` 目录，该目录需要在安装完成之前创建和配置。
12. 按 `CTRL+C` 停止 Grafana 服务器。
13. 将 `/usr/local/grafana` 的所有者更改为 Grafana 用户，以将所有权应用于新创建的 `/usr/local/grafana/data` 目录：
	shell
	```shell
	sudo chown -R grafana:users /usr/local/grafana
	```
14. [配置 Grafana 服务器以在启动时使用 systemd 启动](https://grafana.com/docs/grafana/latest/setup-grafana/start-restart-grafana/#configure-the-grafana-server-to-start-at-boot-using-systemd) 。

## 在 Debian 或 Ubuntu 上卸载

完成以下任一步骤以卸载 Grafana。

要卸载 Grafana，请在终端窗口中运行以下命令：

1. 如果您配置了 Grafana 以使用 systemd 运行，请停止 Grafana 服务器的 systemd 服务：
	```bash
	sudo systemctl stop grafana-server
	```
2. 如果您配置了 Grafana 以使用 init.d 运行，请停止 Grafana 服务器的 init.d 服务：
	```bash
	sudo service grafana-server stop
	```
3. 要卸载 Grafana OSS：
	```bash
	sudo apt-get remove grafana
	```
4. 要卸载 Grafana Enterprise：
	```bash
	sudo apt-get remove grafana-enterprise
	```
5. 可选：要移除 Grafana 代码仓库：
	```bash
	sudo rm -i /etc/apt/sources.list.d/grafana.list
	```
- [启动 Grafana 服务器](https://grafana.com/docs/grafana/latest/setup-grafana/start-restart-grafana/)