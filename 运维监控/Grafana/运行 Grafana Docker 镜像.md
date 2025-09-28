---
source: "https://grafana.com/docs/grafana/latest/setup-grafana/installation/docker/"
tags:
  - "Grafana"
---
> 注意
> 
> 从 Grafana 发布的 `12.4.0` 开始， `grafana/grafana-oss` Docker Hub 代码仓库将不再更新。相反，我们鼓励您使用 `grafana/grafana` Docker Hub 代码仓库。这两个代码仓库具有相同的 Grafana OSS Docker 镜像。

本主题指导您通过官方 Docker 镜像安装 Grafana。具体而言，它涵盖了通过 Docker 命令行界面 (CLI) 和 docker-compose 运行 Grafana。

![YouTube Video](https://img.youtube.com/vi/FlDfcMbSLXs/maxresdefault.jpg)

Grafana Docker 镜像有两个版本：

- **Grafana 企业版** : `grafana/grafana-enterprise`
- **Grafana 开源版** : `grafana/grafana`

> **注意：** 推荐和默认的 Grafana 版本是 Grafana 企业版。它是免费的，并包含 OSS 版本的所有功能。此外，您可以选择升级到 [完整的企业功能集](https://grafana.com/products/enterprise/?utm_source=grafana-install-page) ，其中包括对 [企业插件](https://grafana.com/grafana/plugins/?enterprise=1&utcm_source=grafana-install-page) 的支持。

Grafana 的默认镜像是使用 Alpine Linux 项目创建的，可以在 Alpine 官方镜像中找到。有关配置 Grafana Docker 镜像的说明，请参阅 [配置 Grafana Docker 镜像](https://grafana.com/docs/grafana/latest/setup-grafana/configure-docker/) 。

## 通过 Docker CLI 运行 Grafana

本节将向您展示如何使用 Docker CLI 运行 Grafana。

> **注意：** 如果您使用的是 Linux 系统（例如 Debian 或 Ubuntu），您可能需要在命令前添加 `sudo` ，或者将您的用户添加到 `docker` 组中。有关更多信息，请参阅 [Docker Engine 的 Linux 后安装步骤](https://docs.docker.com/engine/install/linux-postinstall/) 。

要运行最新的稳定版本的 Grafana，请运行以下命令：

```bash
docker run -d -p 3000:3000 --name=grafana grafana/grafana-enterprise
```

其中：

- [`docker run`](https://docs.docker.com/engine/reference/commandline/run/) 是一个 Docker CLI 命令，用于从镜像运行一个新容器
- `-d` (`--detach`) 在后台运行容器
- `-p <host-port>:<container-port>` (`--publish`) 将容器的端口发布到主机，允许您通过主机端口访问容器的端口。在这种情况下，我们可以通过主机的端口 `3000` 访问容器的端口 `3000` 。
- `--name` 为容器分配一个逻辑名称（例如 `grafana` ）。这允许您通过名称而不是 ID 来引用容器。
- `grafana/grafana-enterprise` 是要运行的镜像。

### 停止 Grafana 容器。

要停止 Grafana 容器，请运行以下命令：

```bash
# The `docker ps` command shows the processes running in Docker
docker ps

# This will display a list of containers that looks like the following:
CONTAINER ID   IMAGE  COMMAND   CREATED  STATUS   PORTS    NAMES
cd48d3994968   grafana/grafana-enterprise   "/run.sh"   8 seconds ago   Up 7 seconds   0.0.0.0:3000->3000/tcp   grafana

# To stop the grafana container run the command
# docker stop CONTAINER-ID or use
# docker stop NAME, which is `grafana` as previously defined
docker stop grafana
```

### 保存您的 Grafana 数据

默认情况下，Grafana 使用嵌入式 SQLite 版本 3 数据库来存储配置、用户、仪表板和其他数据。当您将 Docker 镜像作为容器运行时，对这些 Grafana 数据的更改会写入容器内的文件系统，这些更改只会在容器存在期间保留。如果您停止并删除容器，任何文件系统更改（即 Grafana 数据）将被丢弃。为了避免丢失数据，您可以使用 [Docker 卷](https://docs.docker.com/storage/volumes/) 或 [绑定挂载](https://docs.docker.com/storage/bind-mounts/) 为您的容器设置持久存储。

> **注意：** 虽然这两种方法相似，但有一些细微的区别。如果您希望存储完全由 Docker 管理，并且仅通过 Docker 容器和 Docker CLI 访问，您应该选择使用持久存储。然而，如果您需要对存储进行完全控制，并希望允许 Docker 以外的其他进程访问或修改存储层，那么绑定挂载是您环境的正确选择。

当您希望 Docker 引擎管理存储卷时，请使用 Docker 卷。

要使用 Docker 卷进行持久存储，请完成以下步骤：

1. 创建一个 Docker 卷供 Grafana 容器使用，并为其赋予一个描述性名称（例如 `grafana-storage` ）。运行以下命令：
	```bash
	# create a persistent volume for your data
	docker volume create grafana-storage
	# verify that the volume was created correctly
	# you should see some JSON output
	docker volume inspect grafana-storage
	```
2. 通过运行以下命令启动 Grafana 容器：
	```bash
	# start grafana
	docker run -d -p 3000:3000 --name=grafana \
	  --volume grafana-storage:/var/lib/grafana \
	  grafana/grafana-enterprise
	```

#### 使用绑定挂载

如果您计划在主机上使用目录作为数据库或配置，在 Docker 中运行 Grafana 时，您必须以具有访问和写入您映射的目录权限的用户启动容器。

要使用绑定挂载，请运行以下命令：

```bash
# create a directory for your data
mkdir data

# start grafana with your user id and using the data directory
docker run -d -p 3000:3000 --name=grafana \
  --user "$(id -u)" \
  --volume "$PWD/data:/var/lib/grafana" \
  grafana/grafana-enterprise
```

### 使用环境变量配置 Grafana

Grafana 支持使用 [环境变量](https://grafana.com/docs/grafana/latest/setup-grafana/configure-grafana/#override-configuration-with-environment-variables) 指定自定义配置设置。

```bash
# enable debug logs

docker run -d -p 3000:3000 --name=grafana \
  -e "GF_LOG_LEVEL=debug" \
  grafana/grafana-enterprise
```

## 在 Docker 容器中安装插件

您可以从官方和社区的 [插件页面](https://grafana.com/grafana/plugins) 安装 Grafana 插件，或者使用自定义 URL 安装私有插件。这些插件允许您添加新的可视化类型、数据源和应用程序，以帮助您更好地可视化数据。

Grafana 目前支持三种类型的插件：面板、数据源和应用程序。有关管理插件的更多信息，请参阅 [插件管理](https://grafana.com/docs/grafana/latest/administration/plugin-management/) 。

要在 Docker 容器中安装插件，请完成以下步骤：

1. 通过 `GF_PLUGINS_PREINSTALL` 环境变量将您希望安装的插件作为逗号分隔的列表传递给 Docker。
	这会启动一个后台进程，在 Grafana 服务器启动时安装插件列表。
	例如：
	```bash
	docker run -d -p 3000:3000 --name=grafana \
	  -e "GF_PLUGINS_PREINSTALL=grafana-clock-panel, grafana-simple-json-datasource" \
	  grafana/grafana-enterprise
	```
2. 要指定插件的版本，请将版本号添加到 `GF_PLUGINS_PREINSTALL` 环境变量中。
	例如：
	```bash
	docker run -d -p 3000:3000 --name=grafana \
	  -e "GF_PLUGINS_PREINSTALL=grafana-clock-panel@1.0.1" \
	  grafana/grafana-enterprise
	```
	> **注意：** 如果您不指定版本号，将使用最新版本。
3. 要从自定义 URL 安装插件，请使用以下约定来指定 URL： `<plugin ID>@[<plugin version>]@<url to plugin zip>` 。
	例如：
	```bash
	docker run -d -p 3000:3000 --name=grafana \
	  -e "GF_PLUGINS_PREINSTALL=custom-plugin@@https://github.com/VolkovLabs/custom-plugin.zip" \
	  grafana/grafana-enterprise
	```

## 示例

以下示例运行最新的稳定版本的 Grafana，监听 3000 端口，容器名称为 `grafana` ，在 `grafana-storage` docker 卷中进行持久存储，设置服务器根 URL，并安装官方的 [时钟面板](https://grafana.com/grafana/plugins/grafana-clock-panel) 插件。

```bash
# create a persistent volume for your data
docker volume create grafana-storage

# start grafana by using the above persistent storage
# and defining environment variables

docker run -d -p 3000:3000 --name=grafana \
  --volume grafana-storage:/var/lib/grafana \
  -e "GF_SERVER_ROOT_URL=http://my.grafana.server/" \
  -e "GF_PLUGINS_PREINSTALL=grafana-clock-panel" \
  grafana/grafana-enterprise
```

## 通过 Docker Compose 运行 Grafana

Docker Compose 是一个软件工具，使得定义和共享由多个容器组成的应用程序变得简单。它通过使用一个 YAML 文件，通常称为 `docker-compose.yaml` ，列出构成应用程序的所有服务。您可以通过一个命令以正确的顺序启动容器，并通过另一个命令将其关闭。有关使用 Docker Compose 的好处和如何使用它的更多信息，请参阅 [使用 Docker Compose](https://docs.docker.com/get-started/08_using_compose/) 。

### 开始之前

要通过 Docker Compose 运行 Grafana，请在您的机器上安装 compose 工具。要确定 compose 工具是否可用，请运行以下命令：

```bash
docker compose version
```

如果 compose 工具不可用，请参考 [安装 Docker Compose](https://docs.docker.com/compose/install/) 。

### 运行最新稳定版本的 Grafana

本节将向您展示如何使用 Docker Compose 运行 Grafana。本节中的示例使用 Compose 版本 3。有关兼容性的更多信息，请参阅 [Compose 和 Docker 兼容性矩阵](https://docs.docker.com/compose/compose-file/compose-file-v3/) 。

> **注意：** 如果您使用的是 Linux 系统（例如，Debian 或 Ubuntu），您可能需要在命令前添加 `sudo` ，或将您的用户添加到 `docker` 组中。有关更多信息，请参阅 [Docker Engine 的 Linux 后安装步骤](https://docs.docker.com/engine/install/linux-postinstall/) 。

要使用 Docker Compose 运行最新稳定版本的 Grafana，请完成以下步骤：

1. 创建一个 `docker-compose.yaml` 文件。
	```bash
	# first go into the directory where you have created this docker-compose.yaml file
	cd /path/to/docker-compose-directory
	# now create the docker-compose.yaml file
	touch docker-compose.yaml
	```
2. 现在，将以下代码添加到 `docker-compose.yaml` 文件中。
	例如：
	```yaml
	services:
	  grafana:
	    image: grafana/grafana-enterprise
	    container_name: grafana
	    restart: unless-stopped
	    ports:
	      - '3000:3000'
	```
3. 要运行 `docker-compose.yaml` ，请运行以下命令：
	```bash
	# start the grafana container
	docker compose up -d
	```
	其中：
	d = 分离模式
	up = 启动容器并使其运行

要确定 Grafana 是否正在运行，请打开一个浏览器窗口并输入 `IP_ADDRESS:3000` 。登录屏幕应该会出现。

### 停止 Grafana 容器

要停止 Grafana 容器，请运行以下命令：

```bash
docker compose down
```

> **注意：** 有关使用 Docker Compose 命令的更多信息，请参阅 [docker compose](https://docs.docker.com/engine/reference/commandline/compose/) 。

### 保存您的 Grafana 数据

默认情况下，Grafana 使用嵌入式 SQLite 版本 3 数据库来存储配置、用户、仪表板和其他数据。当您将 Docker 镜像作为容器运行时，对这些 Grafana 数据的更改会写入容器内的文件系统，这些更改只会在容器存在期间保持。如果您停止并删除容器，任何文件系统的更改（即 Grafana 数据）将被丢弃。为了避免丢失数据，您可以为您的容器设置持久存储，使用 [Docker 卷](https://docs.docker.com/storage/volumes/) 或 [绑定挂载](https://docs.docker.com/storage/bind-mounts/) 。

当您希望 Docker 引擎管理存储卷时，请使用 Docker 卷。

要使用 Docker 卷进行持久存储，请完成以下步骤：

1. 创建一个 `docker-compose.yaml` 文件
	Bash
	```bash
	# first go into the directory where you have created this docker-compose.yaml file
	cd /path/to/docker-compose-directory
	# now create the docker-compose.yaml file
	touch docker-compose.yaml
	```
2. 将以下代码添加到 `docker-compose.yaml` 文件中。
	```yaml
	services:
	  grafana:
	    image: grafana/grafana-enterprise
	    container_name: grafana
	    restart: unless-stopped
	    ports:
	      - '3000:3000'
	    volumes:
	      - grafana-storage:/var/lib/grafana
	volumes:
	  grafana-storage: {}
	```
3. 保存文件并运行以下命令：
	```bash
	docker compose up -d
	```

#### 使用绑定挂载

如果您计划在主机上使用目录作为数据库或配置，在 Docker 中运行 Grafana 时，您必须以具有访问和写入您映射的目录权限的用户启动容器。

要使用绑定挂载，请完成以下步骤：

1. 创建一个 `docker-compose.yaml` 文件
	```bash
	# first go into the directory where you have created this docker-compose.yaml file
	cd /path/to/docker-compose-directory
	# now create the docker-compose.yaml file
	touch docker-compose.yaml
	```
2. 创建您将挂载数据的目录，在本例中是 `/data` ，例如在您当前的工作目录中：
	```bash
	mkdir $PWD/data
	```
3. 现在，将以下代码添加到 `docker-compose.yaml` 文件中。
	```yaml
	services:
	  grafana:
	    image: grafana/grafana-enterprise
	    container_name: grafana
	    restart: unless-stopped
	    # if you are running as root then set it to 0
	    # else find the right id with the id -u command
	    user: '0'
	    ports:
	      - '3000:3000'
	    # adding the mount volume point which we create earlier
	    volumes:
	      - '$PWD/data:/var/lib/grafana'
	```
4. 保存文件并运行以下命令：
	```bash
	docker compose up -d
	```

### 示例

以下示例运行最新的稳定版本的 Grafana，监听 3000 端口，容器名称为 `grafana` ，在 `grafana-storage` docker 卷中进行持久存储，设置服务器根 URL，并安装官方的 [时钟面板](https://grafana.com/grafana/plugins/grafana-clock-panel/) 插件。

```yaml
services:
  grafana:
    image: grafana/grafana-enterprise
    container_name: grafana
    restart: unless-stopped
    environment:
      - GF_SERVER_ROOT_URL=http://my.grafana.server/
      - GF_PLUGINS_PREINSTALL=grafana-clock-panel
    ports:
      - '3000:3000'
    volumes:
      - 'grafana_storage:/var/lib/grafana'
volumes:
  grafana_storage: {}
```

> 注意
> 
> 如果您想指定插件的版本，请将版本号添加到 `GF_PLUGINS_PREINSTALL` 环境变量中。例如： `-e "GF_PLUGINS_PREINSTALL=grafana-clock-panel@1.0.1,grafana-simple-json-datasource@1.3.5"` 。如果您不指定版本号，则使用最新版本。

请参阅 [入门指南](https://grafana.com/docs/grafana/latest/getting-started/build-first-dashboard/) 以获取有关登录、设置数据源等信息。

## 配置 Docker 镜像

请参阅 [配置 Grafana Docker 镜像](https://grafana.com/docs/grafana/latest/setup-grafana/configure-docker/) 页面以获取有关自定义环境、日志、数据库等选项的详细信息。

## 配置 Grafana

有关自定义您的环境、日志、数据库等选项的详细信息，请参阅 [配置](https://grafana.com/docs/grafana/latest/setup-grafana/configure-grafana/) 页面。