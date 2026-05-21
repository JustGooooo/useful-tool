# Docker 镜像下载器实施计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 通过 GitHub Issue Forms 触发 Actions，自动拉取 Docker 镜像并导出为 .tar.gz，发布到 Release 或 Artifact。

**Architecture:** Issue Forms 收集镜像列表和架构 → workflow 逐个 docker pull + docker save → 判断大小选择 Release 或 Artifact → issue 回复结果和 docker load 命令。

**Tech Stack:** GitHub Actions, Bash (docker), actions/github-script, softprops/action-gh-release, actions/upload-artifact

---

### Task 1: 创建 Issue Form 模板

**Files:**
- Create: `.github/ISSUE_TEMPLATE/docker-downloader.yml`

- [ ] **Step 1: 创建模板文件**

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

- [ ] **Step 2: 提交**

```bash
git add .github/ISSUE_TEMPLATE/docker-downloader.yml
git commit -m "feat: 添加 docker-downloader Issue Form 模板"
```

---

### Task 2: 创建 docker-downloader Workflow

**Files:**
- Create: `.github/workflows/docker-downloader.yml`

- [ ] **Step 1: 创建完整 workflow**

```yaml
name: Docker 镜像下载

on:
  issues:
    types: [labeled]

jobs:
  download-docker:
    runs-on: ubuntu-latest
    if: contains(github.event.issue.labels.*.name, 'docker-downloader')
    permissions:
      issues: write
      contents: write
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: 解析 Issue 参数
        id: parse
        env:
          ISSUE_BODY: ${{ github.event.issue.body }}
        run: |
          extract_field() {
            echo "$ISSUE_BODY" | grep -A10 "$1" | grep -v "^###" | grep -v "^--$" | sed '/^$/d' | head -1 | xargs
          }

          # 镜像列表：从 textarea 提取（可能多行）
          IMAGES=$(echo "$ISSUE_BODY" | sed -n '/### 镜像列表/,/### CPU 架构/p' | grep -v "^###" | grep -v "^--$" | sed '/^$/d' | sed 's/^[[:space:]]*//' | tr '\n' ',' | sed 's/,$//')
          PLATFORM=$(extract_field "### CPU 架构")

          if [ -z "$IMAGES" ]; then
            echo "::error::未找到镜像列表"
            exit 1
          fi
          if [ -z "$PLATFORM" ]; then
            echo "::error::未找到 CPU 架构"
            exit 1
          fi

          # 提取短架构名用于文件名
          ARCH=$(echo "$PLATFORM" | tr '/' '-')

          echo "images=$IMAGES" >> $GITHUB_OUTPUT
          echo "platform=$PLATFORM" >> $GITHUB_OUTPUT
          echo "arch=$ARCH" >> $GITHUB_OUTPUT
          IMAGE_COUNT=$(echo "$IMAGES" | tr ',' '\n' | wc -l)
          echo "image_count=$IMAGE_COUNT" >> $GITHUB_OUTPUT

          echo "🐳 镜像: $IMAGES"
          echo "🏗️ 架构: $PLATFORM"
          echo "📦 数量: $IMAGE_COUNT"

      - name: 更新 Issue 标题
        uses: actions/github-script@v7
        with:
          script: |
            const count = '${{ steps.parse.outputs.image_count }}';
            const arch = '${{ steps.parse.outputs.platform }}';
            const images = '${{ steps.parse.outputs.images }}'.split(',');
            const firstImage = images[0].trim();
            let title = `[docker] ${firstImage}`;
            if (parseInt(count) > 1) {
              title += ` +${parseInt(count) - 1}`;
            }
            title += ` - ${arch}`;
            await github.rest.issues.update({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: context.issue.number,
              title: title
            });

      - name: 磁盘清理
        run: |
          docker system prune -a -f
          docker volume prune -f

      - name: 拉取并打包镜像
        id: pull
        env:
          IMAGES: ${{ steps.parse.outputs.images }}
          PLATFORM: ${{ steps.parse.outputs.platform }}
          ARCH: ${{ steps.parse.outputs.arch }}
        run: |
          IFS=',' read -r -a image_array <<< "$IMAGES"
          TOTAL_SIZE=0

          for image in "${image_array[@]}"; do
            image=$(echo "$image" | xargs)  # trim
            echo "正在拉取: $image ($PLATFORM)"
            docker pull "$image" --platform "$PLATFORM"

            # 文件名清理：/ 和 : 替换为 _
            image_name="${image//\//_}"
            image_name="${image_name//:/_}"
            filename="${image_name}-${ARCH}.tar.gz"

            echo "正在打包: $filename"
            docker save "$image" -o "${image_name}-${ARCH}.tar"
            gzip -c "${image_name}-${ARCH}.tar" > "$filename"
            rm "${image_name}-${ARCH}.tar"

            filesize=$(stat -c%s "$filename")
            TOTAL_SIZE=$((TOTAL_SIZE + filesize))
          done

          echo "total_size=$TOTAL_SIZE" >> $GITHUB_OUTPUT
          echo "total_size_mb=$((TOTAL_SIZE / 1024 / 1024))" >> $GITHUB_OUTPUT

          echo "📊 总大小: $((TOTAL_SIZE / 1024 / 1024)) MB"
          ls -lh *.tar.gz

      - name: 选择上传方式
        id: upload_method
        env:
          TOTAL_SIZE: ${{ steps.pull.outputs.total_size }}
        run: |
          # 2GB = 2147483648 bytes
          if [ "$TOTAL_SIZE" -lt 2147483648 ]; then
            echo "method=release" >> $GITHUB_OUTPUT
            echo "📁 文件 < 2GB，使用 Release 上传"
          else
            # 5GB = 5368709120 bytes
            if [ "$TOTAL_SIZE" -lt 5368709120 ]; then
              echo "method=artifact" >> $GITHUB_OUTPUT
              echo "📁 文件 ≥ 2GB，使用 Artifact 上传（90 天保留）"
            else
              echo "::error::文件超过 5GB，无法上传"
              exit 1
            fi
          fi

      - name: 创建 GitHub Release
        if: steps.upload_method.outputs.method == 'release'
        id: release
        uses: softprops/action-gh-release@v2
        with:
          tag_name: docker-${{ steps.parse.outputs.arch }}-$(TZ="Asia/Shanghai" date +'%Y%m%d-%H%M%S')
          name: "Docker Images (${{ steps.parse.outputs.platform }}) - $(TZ='Asia/Shanghai' date +'%Y-%m-%d %H:%M')"
          body: |
            ## Docker 镜像 (${{ steps.parse.outputs.platform }})

            - **架构**: ${{ steps.parse.outputs.platform }}
            - **镜像数**: ${{ steps.parse.outputs.image_count }}
            - **总大小**: ${{ steps.pull.outputs.total_size_mb }} MB

            ### 加载命令

            ```
            docker load -i <filename>.tar.gz
            ```
          draft: false
          prerelease: false
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: 上传 Release Assets
        if: steps.upload_method.outputs.method == 'release'
        uses: softprops/action-gh-release@v2
        with:
          tag_name: ${{ steps.release.outputs.tag_name }}
          files: ${{ github.workspace }}/*.tar.gz
          token: ${{ secrets.GITHUB_TOKEN }}

      - name: 上传 Artifact
        if: steps.upload_method.outputs.method == 'artifact'
        uses: actions/upload-artifact@v4
        with:
          name: docker-images-${{ steps.parse.outputs.arch }}
          path: ${{ github.workspace }}/*.tar.gz
          retention-days: 90

      - name: 成功反馈
        if: success()
        uses: actions/github-script@v7
        with:
          script: |
            const arch = '${{ steps.parse.outputs.platform }}';
            const count = '${{ steps.parse.outputs.image_count }}';
            const method = '${{ steps.upload_method.outputs.method }}';
            const images = '${{ steps.parse.outputs.images }}'.split(',');
            const archFile = '${{ steps.parse.outputs.arch }}';

            let body = `✅ Docker 镜像下载成功！\n\n`;
            body += `- **架构**：${arch}\n`;
            body += `- **镜像数**：${count}\n`;

            if (method === 'release') {
              const tag = '${{ steps.release.outputs.tag_name }}';
              const url = `https://github.com/${context.repo.owner}/${context.repo.repo}/releases/tag/${tag}`;
              body += `- **上传方式**：Release\n`;
              body += `- **Release 链接**：[点击下载](${url})\n`;
            } else {
              body += `- **上传方式**：Artifact（90 天保留）\n`;
              body += `\n⚠️ 文件超过 2GB，已上传到 Artifact，90 天后自动删除。\n`;
            }

            body += `\n### 加载命令\n\`\`\`\n`;
            for (const img of images) {
              const name = img.trim().replace(/\//g, '_').replace(/:/g, '_');
              body += `docker load -i ${name}-${archFile}.tar.gz\n`;
            }
            body += `\`\`\``;

            await github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: body
            });

            await github.rest.issues.addLabels({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              labels: ['success']
            });

      - name: 失败反馈
        if: failure()
        uses: actions/github-script@v7
        with:
          script: |
            const arch = '${{ steps.parse.outputs.platform }}';
            const run_url = `https://github.com/${context.repo.owner}/${context.repo.repo}/actions/runs/${context.runId}`;

            await github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: `❌ Docker 镜像下载失败\n\n- **架构**：${arch}\n- **构建日志**：[点击查看](${run_url})`
            });

            await github.rest.issues.addLabels({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              labels: ['failure']
            });
```

- [ ] **Step 2: 验证 YAML 语法**

```bash
python -c "import yaml; yaml.safe_load(open('.github/workflows/docker-downloader.yml'))"
```

- [ ] **Step 3: 提交**

```bash
git add .github/workflows/docker-downloader.yml
git commit -m "feat: 添加完整的 docker-downloader workflow"
```

---

### Task 3: 更新 README

**Files:**
- Modify: `README.md`

- [ ] **Step 1: 在工具列表表格中添加 docker-downloader**

在现有表格的 `deb-downloader` 行后添加新行：

```markdown
| docker-downloader | 下载 Docker 镜像并导出为 tar.gz | [提交请求](../../issues/new?template=docker-downloader.yml) |
```

- [ ] **Step 2: 在 README 末尾添加 docker-downloader 使用说明**

```markdown

## docker-downloader

下载指定 Docker 镜像，导出为 .tar.gz 发布到 GitHub Release，用于离线加载。

### 使用方式

1. 点击上方「提交请求」链接
2. 填写表单：
   - **镜像列表**：每行一个镜像（如 `nginx:latest`、`ghcr.io/owner/repo:tag`）
   - **CPU 架构**：选择 `linux/amd64`、`linux/arm64` 等
3. 提交 issue，等待 Actions 自动执行
4. 执行完成后，issue 中会回复下载链接和 `docker load` 命令

### 离线加载

```bash
# 解压并加载
docker load -i nginx_latest-linux-amd64.tar.gz
```

### 查找镜像信息

| 网站 | 说明 |
|------|------|
| [Docker Hub](https://hub.docker.com/) | 官方 Docker 镜像仓库 |
| [GitHub Container Registry](https://ghcr.io/) | GitHub 托管的容器镜像 |
| [阿里云容器镜像](https://cr.console.aliyun.com/) | 阿里云容器镜像服务 |
```

- [ ] **Step 3: 提交**

```bash
git add README.md
git commit -m "docs: README 添加 docker-downloader 使用说明"
```
