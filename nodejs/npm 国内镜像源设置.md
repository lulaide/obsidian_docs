---
source: "https://www.cnblogs.com/hopeblaze/articles/18605774"
tags:
  - "npm"
---
## 国内源

淘宝： `https://registry.npmmirror.com/`  
腾讯云： `https://mirrors.cloud.tencent.com/npm/`  
CNPM： `https://r.cnpmjs.org/`

## 设置

### 查询当前使用的镜像源

```bash
npm get registry
```

### 设置为淘宝镜像源

```bash
npm config set registry https://registry.npmmirror.com/
```

### 还原为官方源

```bash
npm config set registry https://registry.npmjs.org/
```