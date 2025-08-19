[Containerd 镜像配置文档](https://github.com/containerd/containerd/blob/main/docs/hosts.md#registry-host-namespace)

### 配置 docker.io

```
mkdir -p /etc/containerd/certs.d/docker.io/
cat <<EOF | sudo tee /etc/containerd/certs.d/docker.io/hosts.toml
server = "https://docker.io"

[host."https://docker.m.daocloud.io"]
  capabilities = ["pull", "resolve"]
[host."https://docker.xuanyuan.me"]
  capabilities = ["pull", "resolve"]
EOF
```

## 配置 registry.k8s.io

```
mkdir -p /etc/containerd/certs.d/registry.k8s.io
cat <<EOF | sudo tee /etc/containerd/certs.d/registry.k8s.io/hosts.toml
server = "https://registry.k8s.io"

[host."https://k8s.m.daocloud.io"]
  capabilities = ["pull", "resolve"]
[host."https://registry.k8s.io"]
  capabilities = ["pull", "resolve", "push"]
EOF
```

### 重启 Containerd

```
systemctl restart containerd
```