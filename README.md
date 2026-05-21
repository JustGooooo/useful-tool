# useful-tools

<p align="center">
  <a href="#docker-downloader">Docker 镜像下载</a>
  ·
  <a href="#deb-downloader">Deb 包下载</a>
  ·
  <a href="#pip-downloader">Pip 包下载</a>
  ·
  <a href="#首次使用创建-label">首次使用</a>
  ·
  <a href="https://github.com/JustGooooo/useful-tool/issues">Issues</a>
</p>

<p align="center">
  <a href="https://github.com/JustGooooo/useful-tool/blob/main/LICENSE">
    <img src="https://img.shields.io/github/license/JustGooooo/useful-tool.svg?label=LICENSE&logo=github" alt="LICENSE">
  </a>
  <img src="https://img.shields.io/github/stars/JustGooooo/useful-tool.svg?style=flat&logo=github&label=Stars" alt="Stars">
  <img src="https://img.shields.io/github/last-commit/JustGooooo/useful-tool.svg?style=flat&logo=github" alt="Last Commit">
</p>

<p align="center">
  <a href="https://star-history.com/#JustGooooo/useful-tool&Date">
    <picture>
      <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/svg?repos=JustGooooo/useful-tool&type=Date&theme=dark" />
      <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/svg?repos=JustGooooo/useful-tool&type=Date" />
      <img alt="Star history" src="https://api.star-history.com/svg?repos=JustGooooo/useful-tool&type=Date" width="600" />
    </picture>
  </a>
</p>

通过 GitHub Actions 实现的自动化下载工具集，支持下载 Docker 镜像、Ubuntu deb 包和 Python pip 包，用于离线部署。

> [!TIP]
> 提交 Issue 即可触发自动下载，无需本地环境。下载完成后会自动创建 Release 并在 Issue 中回复下载链接。

## 目录

- [工具列表](#工具列表)
- [首次使用：创建 Label](#首次使用创建-label)
- [docker-downloader](#docker-downloader)
  - [使用步骤](#使用步骤)
  - [查找镜像](#查找镜像)
- [deb-downloader](#deb-downloader)
  - [使用步骤](#使用步骤-1)
  - [查找软件包](#查找软件包)
- [pip-downloader](#pip-downloader)
  - [使用步骤](#使用步骤-2)
  - [查找 pip 包](#查找-pip-包)

## 工具列表

| 工具 | 说明 | 去提交 |
|------|------|--------|
| **docker-downloader** | 下载 Docker 镜像并打包为 tar.gz | [![Issue](https://img.shields.io/badge/提交请求-Docker-blue?style=flat&logo=docker)](https://github.com/JustGooooo/useful-tool/issues/new?template=docker-downloader.yml) |
| **deb-downloader** | 下载 Ubuntu deb 包及所有依赖 | [![Issue](https://img.shields.io/badge/提交请求-Deb-orange?style=flat&logo=ubuntu)](https://github.com/JustGooooo/useful-tool/issues/new?template=deb-downloader.yml) |
| **pip-downloader** | 下载 pip wheel 包（跨平台/跨版本） | [![Issue](https://img.shields.io/badge/提交请求-Pip-green?style=flat&logo=python)](https://github.com/JustGooooo/useful-tool/issues/new?template=pip-downloader.yml) |

---

## 首次使用：创建 Label

> [!IMPORTANT]
> Issue 模板需要对应的 Label 才能自动触发 Actions。首次使用前，请在仓库中创建以下 Label。

| Label 名称 | 颜色建议 | 用于 |
|------------|---------|------|
| `docker-downloader` | 蓝色 `#0075ca` | Docker 镜像下载 |
| `deb-downloader` | 橙色 `#d93f0b` | Deb 包下载 |
| `pip-downloader` | 绿色 `#0e8a16` | Pip 包下载 |

创建路径：仓库 → Issues → Labels → New label

只需创建一次，之后提交 Issue 时会自动打上对应 Label 并触发 Actions。

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

> [!NOTE]
> 大镜像可能需要较长时间，请耐心等待。可在 Actions 页面查看实时进度。

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

> [!NOTE]
> debootstrap 创建环境需要一定时间，首次运行可能较慢。

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

---

## pip-downloader

下载指定 Python 版本和平台的 pip wheel 包及其所有依赖，打包为 `.tar.gz` 发布到 GitHub Release，用于离线 `pip install`。

> [!NOTE]
> pip 可以跨平台下载 wheel，不需要对应平台的运行环境。支持 Linux、Windows、macOS 等主流平台。

### 使用步骤

#### 1、点击上方「提交请求」按钮，填写 Issue 表单

> **包名**：如 `numpy`、`torch`、`requests`<br>
> **Python 版本**：如 `3.10`、`3.11`、`3.12`、`3.13`<br>
> **目标平台**：下拉选择 `linux_x86_64`、`win_amd64`、`macosx_arm64` 等<br>
> **包版本**（可选）：如 `1.26.0`，留空则下载最新版<br>
> 标题会自动生成，无需手动填写

#### 2、等待 Actions 自动执行

> 自动下载 wheel 包及所有依赖 → 打包为 tar.gz → 发布到 Release

#### 3、离线安装

```bash
tar -xzf pip-numpy-py3.12-linux_x86_64-*.tar.gz
pip install --no-index --find-links=. numpy
```

### 查找 pip 包

| 网站 | 说明 |
|------|------|
| [PyPI](https://pypi.org/) | Python 官方包索引 |
| [清华镜像 (TUNA)](https://mirrors.tuna.tsinghua.edu.cn/pypi/) | 国内加速 |
| [阿里云镜像](https://mirrors.aliyun.com/pypi/) | 国内加速 |
