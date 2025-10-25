---
source: "https://redis.io/docs/latest/operate/oss_and_stack/install/install-stack/docker/"
tags:
  - "redis"
---

## 在 Docker 上运行 Redis 开源版

要使用 `redis:<version>` 镜像启动 Redis 开源服务器，请在终端中运行以下命令：

```bash
docker run -d --name redis -p 6379:6379 redis:latest
```

## 使用 redis-cli 连接

然后，您可以使用 `redis-cli` 连接到服务器，就像您连接到任何 Redis 实例一样。

如果您本地没有安装 `redis-cli` ，可以从 Docker 容器中运行它：

```bash
$ docker exec -it redis redis-cli
```

如果您在本地安装了 `redis-cli` ，可以从终端运行它：

```bash
$ redis-cli -h 127.0.0.1 -p 6379
```

## 使用本地配置文件

默认情况下，Redis Docker 容器使用内部配置文件来配置 Redis。要使用本地配置文件启动 Redis，您可以执行以下操作之一：

您可以创建自己的 Dockerfile，将上下文中的 `redis.conf` 添加到 `/data/` ，如下所示。

```
FROM redis
COPY redis.conf /usr/local/etc/redis/redis.conf
CMD [ "redis-server", "/usr/local/etc/redis/redis.conf" ]
```

另外，您可以使用 docker run 选项指定类似的内容。

```bash
$ docker run -v /myredis/conf:/usr/local/etc/redis --name myredis redis redis-server /usr/local/etc/redis/redis.conf
```

其中 `/myredis/conf/` 是一个包含您的 `redis.conf` 文件的本地目录。使用这种方法意味着您不需要为您的 Redis 容器准备 Dockerfile。

映射的目录应该是可写的，因为根据配置和操作模式，Redis 可能需要创建额外的配置文件或重写现有的文件。