# useful-tools

通过 GitHub Actions 实现的自动化下载工具集。

## 工具列表

| 工具 | 说明 | Issue 模板 |
|------|------|-----------|
| deb-downloader | 下载 Ubuntu .deb 包及依赖 | [提交请求](../../issues/new?template=deb-downloader.yml) |

## deb-downloader

下载指定 Ubuntu 版本的 .deb 软件包及其所有依赖，打包为 .tar.gz 发布到 GitHub Release，用于离线安装。

### 使用方式

1. 点击上方「提交请求」链接
2. 填写表单：
   - **软件包名**：如 `nginx`、`vim`、`curl`
   - **Ubuntu 版本**：如 `22.04`、`24.04`
   - **包版本**（可选）：如 `1.24.0-1ubuntu1`
3. 提交 issue，等待 Actions 自动执行
4. 执行完成后，issue 中会回复 Release 下载链接

### 查找软件包信息

在下载前，你可以通过以下网站查询软件包名称、版本和依赖信息：

| 网站 | 说明 |
|------|------|
| [packages.ubuntu.com](https://packages.ubuntu.com/) | Ubuntu 官方软件包查询，可按版本和架构筛选 |
| [清华镜像 (TUNA)](https://mirrors.tuna.tsinghua.edu.cn/ubuntu/) | 清华大学 Ubuntu 镜像源 |
| [中科大镜像 (USTC)](https://mirrors.ustc.edu.cn/ubuntu/) | 中科大学 Ubuntu 镜像源 |
| [阿里云镜像](https://mirrors.aliyun.com/ubuntu/) | 阿里云 Ubuntu 镜像源 |

### 离线安装

```bash
# 解压
tar -xzf deb-nginx-ubuntu24.04-*.tar.gz

# 安装所有 deb 包
sudo dpkg -i *.deb
```
