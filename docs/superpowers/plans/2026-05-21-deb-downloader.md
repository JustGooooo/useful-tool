# Deb 包离线下载器实施计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 通过 GitHub Issue Forms 触发 Actions，自动下载指定 Ubuntu 版本的 .deb 包及依赖，打包发布到 Release。

**Architecture:** Issue Forms 表单收集参数 → workflow 解析 issue body → debootstrap 创建目标环境 → apt 下载包及依赖 → 打包 .tar.gz → 创建 Release → issue 回复结果。

**Tech Stack:** GitHub Actions, YAML, Bash (debootstrap, apt-get, chroot), actions/github-script, softprops/action-gh-release

---

### Task 1: 初始化 Git 仓库和项目结构

**Files:**
- Create: `README.md`

- [ ] **Step 1: 初始化 git 仓库**

```bash
cd A:\PersonalData\Coding\Personal\ClaudeSpace\useful-tools
git init
```

- [ ] **Step 2: 创建基础 README.md**

```markdown
# useful-tools

通过 GitHub Actions 实现的自动化下载工具集。

## 工具列表

| 工具 | 说明 | Issue 模板 |
|------|------|-----------|
| deb-downloader | 下载 Ubuntu .deb 包及依赖 | [提交请求](../../issues/new?template=deb-downloader.yml) |

## 使用方式

1. 点击上方链接提交 Issue
2. 等待 Actions 自动执行
3. 在 Issue 中获取下载链接
```

- [ ] **Step 3: 提交**

```bash
git add README.md
git commit -m "init: 初始化项目，添加基础 README"
```

---

### Task 2: 创建 Issue Form 模板

**Files:**
- Create: `.github/ISSUE_TEMPLATE/deb-downloader.yml`

- [ ] **Step 1: 创建 Issue 模板目录**

```bash
mkdir -p .github/ISSUE_TEMPLATE
```

- [ ] **Step 2: 创建 deb-downloader.yml**

```yaml
---
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

- [ ] **Step 3: 提交**

```bash
git add .github/ISSUE_TEMPLATE/deb-downloader.yml
git commit -m "feat: 添加 deb-downloader Issue Form 模板"
```

---

### Task 3: 创建 deb-downloader Workflow — 触发与参数解析

**Files:**
- Create: `.github/workflows/deb-downloader.yml`

- [ ] **Step 1: 创建 workflow 文件，写入触发条件和参数解析逻辑**

```yaml
name: Deb 包下载

on:
  issues:
    types: [opened]

jobs:
  download-deb:
    runs-on: ubuntu-latest
    if: contains(github.event.issue.labels.*.name, 'deb-downloader')
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: 解析 Issue 参数
        id: parse
        env:
          ISSUE_BODY: ${{ github.event.issue.body }}
        run: |
          # Issue Forms 渲染格式为: ### Label\n\nvalue
          # 提取各字段值
          PACKAGE_NAME=$(echo "$ISSUE_BODY" | grep -A1 "### 软件包名" | tail -1 | xargs)
          UBUNTU_VERSION=$(echo "$ISSUE_BODY" | grep -A1 "### Ubuntu 版本" | tail -1 | xargs)
          PACKAGE_VERSION=$(echo "$ISSUE_BODY" | grep -A1 "### 包版本（可选）" | tail -1 | xargs)

          # 校验必填参数
          if [ -z "$PACKAGE_NAME" ]; then
            echo "::error::未找到软件包名"
            exit 1
          fi
          if [ -z "$UBUNTU_VERSION" ]; then
            echo "::error::未找到 Ubuntu 版本"
            exit 1
          fi

          echo "package_name=$PACKAGE_NAME" >> $GITHUB_OUTPUT
          echo "ubuntu_version=$UBUNTU_VERSION" >> $GITHUB_OUTPUT
          echo "package_version=$PACKAGE_VERSION" >> $GITHUB_OUTPUT

          echo "📦 包名: $PACKAGE_NAME"
          echo "🐧 Ubuntu: $UBUNTU_VERSION"
          echo "📋 版本: ${PACKAGE_VERSION:-最新版}"
```

- [ ] **Step 2: 验证 YAML 语法**

```bash
# 安装 yamllint 验证
pip install yamllint
yamllint .github/workflows/deb-downloader.yml
```

- [ ] **Step 3: 提交**

```bash
git add .github/workflows/deb-downloader.yml
git commit -m "feat: 添加 deb-downloader workflow 触发和参数解析"
```

---

### Task 4: 添加 debootstrap 环境创建步骤

**Files:**
- Modify: `.github/workflows/deb-downloader.yml`

- [ ] **Step 1: 在 workflow 中添加 debootstrap 步骤**

在 `解析 Issue 参数` step 之后添加：

```yaml
      - name: 安装 debootstrap
        run: sudo apt-get update && sudo apt-get install -y debootstrap

      - name: 创建目标 Ubuntu 环境
        env:
          UBUNTU_VERSION: ${{ steps.parse.outputs.ubuntu_version }}
        run: |
          echo "正在为 Ubuntu $UBUNTU_VERSION 创建最小根文件系统..."
          sudo debootstrap "$UBUNTU_VERSION" /target http://archive.ubuntu.com/ubuntu

          # 验证 chroot 环境可用
          sudo chroot /target cat /etc/os-release
```

- [ ] **Step 2: 提交**

```bash
git add .github/workflows/deb-downloader.yml
git commit -m "feat: 添加 debootstrap 环境创建步骤"
```

---

### Task 5: 添加包下载逻辑

**Files:**
- Modify: `.github/workflows/deb-downloader.yml`

- [ ] **Step 1: 在 workflow 中添加下载步骤**

在 `创建目标 Ubuntu 环境` step 之后添加：

```yaml
      - name: 下载包及依赖
        id: download
        env:
          PACKAGE_NAME: ${{ steps.parse.outputs.package_name }}
          PACKAGE_VERSION: ${{ steps.parse.outputs.package_version }}
        run: |
          # 在 chroot 中配置输出目录
          sudo mkdir -p /target/output

          # 如果指定了版本，附加版本号
          if [ -n "$PACKAGE_VERSION" ] && [ "$PACKAGE_VERSION" != "留空=最新版本" ]; then
            DOWNLOAD_TARGET="${PACKAGE_NAME}=${PACKAGE_VERSION}"
          else
            DOWNLOAD_TARGET="$PACKAGE_NAME"
          fi

          # 更新源并下载
          sudo chroot /target apt-get update
          sudo chroot /target apt-get install -y apt-utils

          # 获取所有递归依赖
          sudo chroot /target bash -c "
            cd /output && \
            apt-get download \$(apt-cache depends --recurse \
              --no-recommends --no-suggests \
              --no-conflicts --no-breaks \
              --no-replaces --no-enhances \
              ${DOWNLOAD_TARGET} 2>/dev/null \
              | grep '^\w' | sort -u) 2>&1 || true
          "

          # 统计下载结果
          DEB_COUNT=$(ls /target/output/*.deb 2>/dev/null | wc -l)
          echo "deb_count=$DEB_COUNT" >> $GITHUB_OUTPUT

          if [ "$DEB_COUNT" -eq 0 ]; then
            echo "::error::未下载到任何 .deb 文件，包名可能不存在"
            exit 1
          fi

          echo "✅ 共下载 $DEB_COUNT 个 .deb 文件"
          ls -lh /target/output/*.deb
```

- [ ] **Step 2: 提交**

```bash
git add .github/workflows/deb-downloader.yml
git commit -m "feat: 添加 deb 包及依赖下载逻辑"
```

---

### Task 6: 添加打包和 Release 发布步骤

**Files:**
- Modify: `.github/workflows/deb-downloader.yml`

- [ ] **Step 1: 在 workflow 中添加打包和发布步骤**

在 `下载包及依赖` step 之后添加：

```yaml
      - name: 打包为 tar.gz
        id: package
        env:
          PACKAGE_NAME: ${{ steps.parse.outputs.package_name }}
          UBUNTU_VERSION: ${{ steps.parse.outputs.ubuntu_version }}
        run: |
          TIMESTAMP=$(TZ="Asia/Shanghai" date +'%Y%m%d-%H%M%S')
          ARCHIVE_NAME="deb-${PACKAGE_NAME}-ubuntu${UBUNTU_VERSION}-${TIMESTAMP}.tar.gz"
          RELEASE_NAME="${PACKAGE_NAME} for Ubuntu ${UBUNTU_VERSION} - $(TZ='Asia/Shanghai' date +'%Y-%m-%d %H:%M')"

          tar -czf "$ARCHIVE_NAME" -C /target/output .

          echo "archive_name=$ARCHIVE_NAME" >> $GITHUB_OUTPUT
          echo "release_name=$RELEASE_NAME" >> $GITHUB_OUTPUT
          echo "tag_name=deb-${PACKAGE_NAME}-ubuntu${UBUNTU_VERSION}-${TIMESTAMP}" >> $GITHUB_OUTPUT

          echo "📦 已打包: $ARCHIVE_NAME ($(du -h "$ARCHIVE_NAME" | cut -f1))"

      - name: 创建 GitHub Release
        uses: softprops/action-gh-release@v2
        with:
          tag_name: ${{ steps.package.outputs.tag_name }}
          name: ${{ steps.package.outputs.release_name }}
          body: |
            ## ${{ steps.parse.outputs.package_name }} for Ubuntu ${{ steps.parse.outputs.ubuntu_version }}

            - **软件包**: ${{ steps.parse.outputs.package_name }}
            - **Ubuntu 版本**: ${{ steps.parse.outputs.ubuntu_version }}
            - **包含 .deb 文件数**: ${{ steps.download.outputs.deb_count }}

            ### 离线安装方法

            ```bash
            tar -xzf ${{ steps.package.outputs.archive_name }}
            sudo dpkg -i *.deb
            ```
          files: ${{ steps.package.outputs.archive_name }}
          draft: false
          prerelease: false
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

- [ ] **Step 2: 提交**

```bash
git add .github/workflows/deb-downloader.yml
git commit -m "feat: 添加打包和 Release 发布步骤"
```

---

### Task 7: 添加 Issue 反馈步骤（成功/失败）

**Files:**
- Modify: `.github/workflows/deb-downloader.yml`

- [ ] **Step 1: 添加成功反馈 step**

在 `创建 GitHub Release` step 之后添加：

```yaml
      - name: 成功反馈
        if: success()
        uses: actions/github-script@v7
        with:
          script: |
            const package_name = '${{ steps.parse.outputs.package_name }}';
            const ubuntu_version = '${{ steps.parse.outputs.ubuntu_version }}';
            const deb_count = '${{ steps.download.outputs.deb_count }}';
            const release_url = `https://github.com/${context.repo.owner}/${context.repo.repo}/releases/tag/${{ steps.package.outputs.tag_name }}`;

            await github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: `✅ Deb 包下载成功！\n\n- **软件包**：${package_name}\n- **Ubuntu 版本**：${ubuntu_version}\n- **包含 .deb 文件数**：${deb_count}\n- **Release 链接**：[点击下载](${release_url})`
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
            const package_name = '${{ steps.parse.outputs.package_name }}';
            const ubuntu_version = '${{ steps.parse.outputs.ubuntu_version }}';
            const run_url = `https://github.com/${context.repo.owner}/${context.repo.repo}/actions/runs/${context.runId}`;

            await github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: `❌ Deb 包下载失败\n\n- **软件包**：${package_name}\n- **Ubuntu 版本**：${ubuntu_version}\n- **构建日志**：[点击查看](${run_url})`
            });

            await github.rest.issues.addLabels({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              labels: ['failure']
            });
```

- [ ] **Step 2: 提交**

```bash
git add .github/workflows/deb-downloader.yml
git commit -m "feat: 添加 Issue 成功/失败反馈步骤"
```

---

### Task 8: 更新 README 并添加 .gitignore

**Files:**
- Modify: `README.md`
- Create: `.gitignore`

- [ ] **Step 1: 创建 .gitignore**

```
/target/
*.deb
*.tar.gz
```

- [ ] **Step 2: 更新 README.md，添加详细使用说明**

```markdown
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

### 离线安装

```bash
# 解压
tar -xzf deb-nginx-ubuntu24.04-*.tar.gz

# 安装所有 deb 包
sudo dpkg -i *.deb
```
```

- [ ] **Step 3: 提交**

```bash
git add .gitignore README.md
git commit -m "docs: 添加 .gitignore 和详细 README"
```

---

### Task 9: 推送到 GitHub 并测试

- [ ] **Step 1: 在 GitHub 创建仓库 useful-tools**

通过 GitHub 网页创建仓库，或使用 gh CLI：
```bash
gh repo create useful-tools --public --source=. --remote=origin --push
```

- [ ] **Step 2: 推送代码**

```bash
git push -u origin main
```

- [ ] **Step 3: 测试 Issue 模板**

1. 访问仓库的 Issues 页面
2. 点击「New Issue」，选择「Deb 包下载请求」模板
3. 填写测试参数：包名=`curl`，Ubuntu 版本=`24.04`
4. 提交 issue，观察 Actions 是否自动触发
5. 检查 issue 中是否收到成功/失败反馈
6. 如果成功，检查 Release 页面是否有对应文件
