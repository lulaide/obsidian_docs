
## 修改配置文件

```bash
cat <<EOF | sudo tee /etc/docker/daemon.json
{
    "registry-mirrors": [
        "https://docker.1ms.run",
        "https://docker.m.daocloud.io",
        "https://docker.xuanyuan.me"
    ]
}
EOF
```

## 重启 docker 服务

```
sudo systemctl restart docker
```