# Deb 包离线下载器设计文档

## 概述

通过 GitHub Actions 自动下载指定 Ubuntu 版本的 .deb 软件包及其所有依赖，打包为 .tar.gz 发布到 GitHub Release，用于离线安装。

本项目（useful-tools）定位为通用"下载工具集"，deb-downloader 是第一个工具，后续可扩展 Docker 镜像下载等。

## 触发方式

使用 GitHub Issue Forms（YAML 表单）作为触发入口：

- 用户通过 Issue 模板填写参数并提交 issue
- issue 创建时自动触发 workflow（`issues: [opened]`）
- workflow 通过 `deb-downloader` label 过滤

## Issue Form 模板

文件：`.github/ISSUE_TEMPLATE/deb-downloader.yml`

```yaml
name: Deb 包下载请求
description: 下载指定 Ubuntu 版本的 .deb 包及其依赖，打包发布到 Release
labels: ["deb-downloader"]
body:
  - type: input
    id: package_name
    attributes:
      label: 软件包名
      description: "要下载的 apt 包名（如 nginx、vim、curl）"
      placeholder: "nginx"
    validations:
      required: true

  - type: input
    id: ubuntu_version
    attributes:
      label: Ubuntu 版本
      description: "填写 Ubuntu 版本号（如 22.04、24.04、26.04）"
      placeholder: "24.04"
    validations:
      required: true

  - type: input
    id: package_version
    attributes:
      label: 包版本（可选）
      description: "留空则下载最新版本，填写则指定版本（如 1.24.0-1ubuntu1）"
      placeholder: "留空=最新版本"
    validations:
      required: false
```

## Workflow 设计

文件：`.github/workflows/deb-downloader.yml`

### 触发条件

```yaml
on:
  issues:
    types: [opened]
```

### 执行流程

1. **过滤**：检查 issue 是否包含 `deb-downloader` label
2. **解析参数**：Issue Forms 渲染的 body 为结构化 markdown，格式为 `### Label\n\nvalue`，使用 `grep` + `sed` 提取各字段值
3. **创建目标环境**：使用 `debootstrap` 创建目标 Ubuntu 版本的最小根文件系统
   ```bash
   debootstrap <version> /target http://archive.ubuntu.com/ubuntu
   ```
4. **下载包及依赖**：在 chroot 环境中执行
   ```bash
   chroot /target apt-get update
   # 获取所有递归依赖
   chroot /target apt-cache depends --recurse --no-recommends --no-suggests \
     --no-conflicts --no-breaks --no-replaces --no-enhances <package> \
     | grep "^\w" | sort -u > /tmp/deps.txt
   # 下载主包和所有依赖
   chroot /target bash -c "cd /output && apt-get download <package> \$(cat /tmp/deps.txt)"
   ```
5. **打包**：将所有 .deb 文件打包为 `.tar.gz`
6. **创建 Release**：
   - Tag 格式：`deb-<package>-<ubuntu_version>-<timestamp>`
   - Release 名称：`<package> for Ubuntu <version> - <date>`
   - 上传 .tar.gz 作为 release asset
7. **反馈**：在 issue 中回复结果并打上 success/failure label

### 核心逻辑要点

- 使用 `debootstrap` 而非修改 sources.list，确保目标版本环境完全隔离
- 使用 `apt-cache depends --recurse` 递归获取所有依赖
- GitHub runner 使用 `ubuntu-latest`，通过 Docker 或直接安装 debootstrap

## Release 策略

- 每次触发创建新 release
- Tag 格式：`deb-<package_name>-<ubuntu_version>-<YYYYMMDD-HHMMSS>`
- Release 名称：`<package_name> for Ubuntu <version> - <YYYY-MM-DD HH:MM>`
- Release body 包含：包名、Ubuntu 版本、.deb 文件数量、下载说明

## Issue 反馈格式

### 成功

```
✅ Deb 包下载成功！

- 软件包：nginx
- Ubuntu 版本：24.04
- 包含 .deb 文件数：12
- Release 链接：[点击下载](https://github.com/...)
```

Label：添加 `success`

### 失败

```
❌ Deb 包下载失败

- 软件包：nginx
- Ubuntu 版本：24.04
- 失败原因：包名不存在或依赖无法解析
- 构建日志：[点击查看](https://github.com/.../actions/runs/...)
```

Label：添加 `failure`

## 项目结构

```
useful-tools/
├── .github/
│   ├── ISSUE_TEMPLATE/
│   │   └── deb-downloader.yml       # Issue Forms 表单模板
│   └── workflows/
│       └── deb-downloader.yml       # deb 包下载 workflow
└── README.md
```

## 扩展性

项目按下载类型拆分独立 workflow，后续扩展只需新增对应文件：
- `.github/ISSUE_TEMPLATE/docker-downloader.yml` + `.github/workflows/docker-downloader.yml`
- `.github/ISSUE_TEMPLATE/xxx-downloader.yml` + `.github/workflows/xxx-downloader.yml`

## 约束与注意事项

- GitHub Actions 单个 job 最长 6 小时，大型包下载需注意超时
- GitHub Release 单文件最大 2GB，超大包需考虑分卷
- debootstrap 需要网络访问 Ubuntu 仓库，确保 runner 网络通畅
- issue body 解析需处理特殊字符和格式问题
