[Containerd 镜像配置文档](https://github.com/containerd/containerd/blob/main/docs/hosts.md#registry-host-namespace)

### 创建 docker.io 的文件夹

```
mkdir -p /etc/containerd/certs.d/docker.io/
```

### 配置 docker.m.daocloud.io`

```
cat <<EOF | sudo tee /etc/containerd/certs.d/docker.io/hosts.toml
server = "https://docker.io"

[host."https://docker.m.daocloud.io"] capabilities = ["pull", "resolve"]
EOF
```

### 重启 Containerd

```
systemctl restart containerd
```