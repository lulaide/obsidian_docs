
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


## Docker Desktop

**Settings → Docker Engine**

原本的内容可能是

```json
{
  "builder": {
    "gc": {
      "defaultKeepStorage": "20GB",
      "enabled": true
    }
  },
  "experimental": false
}
```

将 registry-mirrors 字段添加进去

```json
{
    "registry-mirrors": [
        "https://docker.1ms.run",
        "https://docker.m.daocloud.io",
        "https://docker.xuanyuan.me"
    ]
}
```