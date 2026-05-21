# useful-tools

[![GitHub](https://img.shields.io/github/license/JustGooooo/useful-tool.svg?label=LICENSE&logo=github&logoColor=%20)](https://github.com/JustGooooo/useful-tool/blob/main/LICENSE)
![GitHub Stars](https://img.shields.io/github/stars/JustGooooo/useful-tool.svg?style=flat&logo=appveyor&label=Stars&logo=github)

通过 GitHub Actions 实现的自动化下载工具集，支持下载 Docker 镜像和 Ubuntu deb 包，用于离线部署。

## 工具列表

| 工具 | 说明 | 去提交 |
|------|------|--------|
| **docker-downloader** | 下载 Docker 镜像并打包为 tar.gz | [![Issue](https://img.shields.io/badge/提交请求-Docker-blue?style=flat&logo=docker)](https://github.com/JustGooooo/useful-tool/issues/new?template=docker-downloader.yml) |
| **deb-downloader** | 下载 Ubuntu deb 包及所有依赖 | [![Issue](https://img.shields.io/badge/提交请求-Deb-orange?style=flat&logo=ubuntu)](https://github.com/JustGooooo/useful-tool/issues/new?template=deb-downloader.yml) |

---

## docker-downloader

下载指定 Docker 镜像，导出为 `.tar.gz` 发布到 GitHub Release，用于离线 `docker load`。

### 使用步骤

#### 1、点击上方「提交请求」按钮，填写 Issue 表单

> **镜像列表**：每行一个镜像，格式如 `nginx:latest` 或 `ghcr.io/owner/repo:tag`<br>
> **CPU 架构**：下拉选择 `linux/amd64`、`linux/arm64`、`linux/arm/v7`、`windows/amd64`<br>
> 标题会自动生成，无需手动填写

#### 2、等待 Actions 自动执行

> 自动拉取镜像 → 打包为 tar.gz → 发布到 Release<br>
> 如果镜像总大小超过 2GB，会自动切换到 Artifact 上传（90 天保留）<br>
> 如果超过 5GB，抱歉，本项目无法处理

#### 3、加载离线 Docker 镜像

```bash
docker load -i nginx_latest-linux-amd64.tar.gz
```

### 查找镜像

| 网站 | 说明 |
|------|------|
| [Docker Hub](https://hub.docker.com/) | 官方 Docker 镜像仓库 |
| [GitHub Container Registry (ghcr.io)](https://ghcr.io/) | GitHub 托管的容器镜像 |
| [阿里云容器镜像](https://cr.console.aliyun.com/) | 国内加速 |

---

## deb-downloader

下载指定 Ubuntu 版本的 `.deb` 软件包及其所有依赖，打包为 `.tar.gz` 发布到 GitHub Release，用于离线 `dpkg -i` 安装。

### 使用步骤

#### 1、点击上方「提交请求」按钮，填写 Issue 表单

> **软件包名**：如 `nginx`、`vim`、`curl`<br>
> **Ubuntu 版本**：如 `22.04`、`24.04`、`26.04`<br>
> **包版本**（可选）：如 `1.24.0-1ubuntu1`，留空则下载最新版<br>
> 标题会自动生成，无需手动填写

#### 2、等待 Actions 自动执行

> 通过 debootstrap 创建目标 Ubuntu 环境 → 下载包及所有递归依赖 → 打包发布到 Release

#### 3、离线安装

```bash
tar -xzf deb-nginx-ubuntu24.04-*.tar.gz
sudo dpkg -i *.deb
```

### 查找软件包

| 网站 | 说明 |
|------|------|
| [packages.ubuntu.com](https://packages.ubuntu.com/) | Ubuntu 官方软件包查询 |
| [清华镜像 (TUNA)](https://mirrors.tuna.tsinghua.edu.cn/ubuntu/) | 国内加速 |
| [中科大镜像 (USTC)](https://mirrors.ustc.edu.cn/ubuntu/) | 国内加速 |
| [阿里云镜像](https://mirrors.aliyun.com/ubuntu/) | 国内加速 |
