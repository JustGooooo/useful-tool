# Docker 镜像下载器设计文档

## 概述

通过 GitHub Actions 自动拉取指定 Docker 镜像，导出为 .tar.gz 发布到 GitHub Release 或 Artifact，用于离线加载。

## 触发方式

使用 GitHub Issue Forms 作为触发入口，通过 `docker-downloader` label 触发 workflow。

## Issue Form 模板

文件：`.github/ISSUE_TEMPLATE/docker-downloader.yml`

```yaml
---
name: Docker 镜像下载请求
description: 下载指定 Docker 镜像并打包发布到 Release
title: "[docker] "
labels: ["docker-downloader"]
body:
  - type: textarea
    id: images
    attributes:
      label: 镜像列表
      description: "每行一个镜像，格式如 nginx:latest 或 ghcr.io/owner/repo:tag"
      placeholder: |
        nginx:latest
        redis:7-alpine
    validations:
      required: true

  - type: dropdown
    id: platform
    attributes:
      label: CPU 架构
      options:
        - "linux/amd64"
        - "linux/arm64"
        - "linux/arm/v7"
        - "windows/amd64"
    validations:
      required: true
```

## Workflow 设计

文件：`.github/workflows/docker-downloader.yml`

### 触发条件

```yaml
on:
  issues:
    types: [labeled]
```

通过 `if: contains(github.event.issue.labels.*.name, 'docker-downloader')` 过滤。

### 执行流程

1. **解析参数**：从 issue body 提取镜像列表和架构
2. **磁盘清理**：`docker system prune -a -f`（参考 DockerTarBuilder，释放 runner 空间）
3. **逐个拉取并打包**（参考 DockerTarBuilder 模式）：
   ```bash
   docker pull "$image" --platform "$PLATFORM"
   image_name="${image//\//_}"
   image_name="${image_name//:/_}"
   docker save "$image" -o "${image_name}-${ARCH}.tar"
   gzip -c "${image_name}-${ARCH}.tar" > "${image_name}-${ARCH}.tar.gz"
   rm "${image_name}-${ARCH}.tar"
   ```
4. **判断上传方式**：
   - 计算所有 .tar.gz 总大小
   - `< 2GB` → 上传到 Release
   - `≥ 2GB` → 上传到 Artifact（90 天保留）
5. **创建 Release**（分两步，参考 DockerTarBuilder）：
   - Step 1：创建 Release 元数据
   - Step 2：上传 .tar.gz assets
6. **Issue 反馈**：回复下载链接 + `docker load` 命令

### 权限

```yaml
permissions:
  issues: write
  contents: write
```

## Release 策略

- Tag 格式：`docker-<arch>-<timestamp>`（如 `docker-linux-amd64-20260521-160000`）
- Release 名称：`Docker Images (<arch>) - <YYYY-MM-DD HH:MM>`
- Release body 包含每个镜像的 `docker load` 命令

## Issue 反馈格式

### 成功（Release）

```
✅ Docker 镜像下载成功！

- 架构：linux/amd64
- 镜像数：2
- 上传方式：Release
- Release 链接：[点击下载](...)

### 加载命令
docker load -i nginx_latest-linux-amd64.tar.gz
docker load -i redis_7-alpine-linux-amd64.tar.gz
```

### 成功（Artifact）

```
✅ Docker 镜像下载成功！

- 架构：linux/amd64
- 镜像数：2
- 上传方式：Artifact（90 天保留）
- 下载链接：[点击下载](...)

⚠️ 文件超过 2GB，已上传到 Artifact，90 天后自动删除。
```

### 失败

```
❌ Docker 镜像下载失败

- 架构：linux/amd64
- 构建日志：[点击查看](...)
```

Label：添加 `success` 或 `failure`

## 项目结构变更

```
useful-tools/
├── .github/
│   ├── ISSUE_TEMPLATE/
│   │   ├── deb-downloader.yml
│   │   └── docker-downloader.yml     # 新增
│   └── workflows/
│       ├── deb-downloader.yml
│       └── docker-downloader.yml     # 新增
└── README.md                          # 更新
```

## 参考

- [DockerTarBuilder](https://github.com/modaiwang/DockerTarBuilder) 的 docker pull + save + gzip 模式
- Release 两步创建模式（先 metadata 再 upload assets）
- 磁盘清理：`docker system prune -a -f`
