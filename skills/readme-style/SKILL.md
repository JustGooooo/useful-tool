---
name: readme-style
description: Use when writing or updating README.md files for GitHub projects, especially tool collections or Action-based projects that need clear usage instructions and visual appeal
---

# README 风格指南

## Overview

为 GitHub 项目编写清晰、实用、有视觉层次的 README。融合两种风格：
- [DockerTarBuilder](https://github.com/modaiwang/DockerTarBuilder)：步骤引导、实用性
- [claw-code](https://github.com/ultraworkers/claw-code)：导航栏、callout、文档地图

## 核心原则

- **实用优先**：用户关心的是「怎么用」，不是项目背景故事
- **视觉层次**：用导航栏、badge、callout、分隔线创造清晰的阅读节奏
- **步骤明确**：每个工具的使用方式必须是可操作的步骤，不是泛泛描述

## 结构模板

### 1. 顶部导航栏

用 `<p align="center">` 居中的锚点链接，让用户快速跳转：

```markdown
# 项目名

<p align="center">
  <a href="#section-1">Section 1</a>
  ·
  <a href="#section-2">Section 2</a>
  ·
  <a href="https://github.com/.../issues">Issues</a>
</p>
```

- 用 `·` 分隔链接
- 只放主要章节，不要放太多
- 最后一个可以放 Issues 或 Discussions 入口

### 2. Badge 行

用 `<p align="center">` 居中排列 badge：

```markdown
<p align="center">
  <a href="LICENSE 链接">
    <img src="https://img.shields.io/github/license/用户名/仓库名.svg?label=LICENSE&logo=github" alt="LICENSE">
  </a>
  <img src="https://img.shields.io/github/stars/用户名/仓库名.svg?style=flat&logo=github&label=Stars" alt="Stars">
  <img src="https://img.shields.io/github/last-commit/用户名/仓库名.svg?style=flat&logo=github" alt="Last Commit">
</p>
```

推荐 badge（按顺序）：
1. LICENSE
2. Stars
3. Last Commit（可选，展示项目活跃度）

### 3. 一句话介绍 + TIP callout

```markdown
一句话介绍项目做什么。

> [!TIP]
> 简短的使用提示或亮点说明。
```

### 4. 目录（TOC）

GitHub 支持自动目录，手动写锚点链接更可控：

```markdown
## 目录

- [工具列表](#工具列表)
- [首次使用](#首次使用)
- [tool-name](#tool-name)
  - [使用步骤](#使用步骤)
  - [查找资源](#查找资源)
```

锚点规则：标题转小写，空格变 `-`，中文保持原样，去掉特殊字符。

### 5. 工具列表表格

```markdown
| 工具 | 说明 | 去提交 |
|------|------|--------|
| **tool-name** | 一句话说明 | [![Issue](https://img.shields.io/badge/提交请求-名称-blue?style=flat&logo=图标)](链接) |
```

- 工具名加粗
- 用 badge 作为提交按钮，比普通链接更醒目
- 颜色区分不同工具类型（blue=docker, orange=deb 等）

### 6. 每个工具独立 Section

用 `---` 分隔线隔开每个工具。每个工具结构：

```markdown
## tool-name

一句话说明工具功能。

### 使用步骤

#### 1、步骤标题

> 补充说明用引用块，细节用 `<br>` 换行<br>
> 引用块放的是「需要知道但不影响主流程」的信息

#### 2、下一步骤

> ...

> [!NOTE]
> 需要特别注意的事项用 callout，比普通引用块更醒目。

#### 3、最终操作（命令）

\```bash
具体可复制的命令
\```

### 查找资源

| 网站 | 说明 |
|------|------|
| [名称](URL) | 一句话说明 |
```

### 7. 文档地图（如有多个文档）

```markdown
## 文档地图

- [`USAGE.md`](./USAGE.md) — 使用指南
- [`CONTRIBUTING.md`](./CONTRIBUTING.md) — 贡献指南
- [`LICENSE`](./LICENSE) — 开源协议
```

适合项目有多个文档文件时使用，集中索引。

## GitHub Callout 语法

GitHub 支持 5 种 callout，用于替代普通引用块中需要强调的内容：

| 语法 | 用途 | 视觉效果 |
|------|------|---------|
| `> [!NOTE]` | 补充信息 | 蓝色 |
| `> [!TIP]` | 使用技巧 | 绿色 |
| `> [!IMPORTANT]` | 必须注意 | 紫色 |
| `> [!WARNING]` | 警告 | 黄色 |
| `> [!CAUTION]` | 危险操作 | 红色 |

使用原则：
- **`> [!NOTE]`**：补充说明、运行时间提示
- **`> [!TIP]`**：首页介绍亮点、使用技巧
- **`> [!IMPORTANT]`**：首次使用必须做的事（如创建 label）
- **`> [!WARNING]`**：限制条件、已知问题
- 普通 `>` 引用块仍用于参数说明等常规补充

## 格式规范

### 引用块（`>`）的使用

引用块用于**补充说明**，不是主要内容：
- 参数说明的细节
- 限制条件（大小限制、时间限制）
- 注意事项和提示

主要内容（步骤、命令）用普通段落或代码块。

### 命令块

- 必须是**可直接复制执行**的命令
- 不要写「例如」或占位符，写真实命令
- 文件名用通配符时注明（如 `*.tar.gz`）

### 链接表格

查找资源类信息用表格，每行：网站名 + 一句话说明。不要放超过 5 个链接。

## 反模式

| 做法 | 为什么不好 |
|------|-----------|
| 开头写大段项目背景 | 用户只想知道怎么用 |
| 用普通文字链接做「提交请求」按钮 | 没有视觉吸引力，容易被忽略 |
| 命令用占位符 `xxx.tar.gz` | 用户需要额外思考替换什么 |
| 把所有信息都用引用块 | 引用块是补充，不是主体 |
| 堆砌 10+ 个 badge | 视觉噪音，反而降低可读性 |
| 每个工具重复完整的安装说明 | 太冗余，用步骤引导即可 |
| 手动写目录但不更新 | 目录与实际标题不匹配会误导读者 |
| 所有内容都用 callout | callout 是强调工具，滥用等于没强调 |
