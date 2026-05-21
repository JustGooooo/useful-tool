---
name: readme-style
description: Use when writing or updating README.md files for GitHub projects, especially tool collections or Action-based projects that need clear usage instructions and visual appeal
---

# README 风格指南

## Overview

为 GitHub 项目编写清晰、实用、有视觉层次的 README。风格参考 [DockerTarBuilder](https://github.com/modaiwang/DockerTarBuilder)，强调步骤引导和实用性。

## 核心原则

- **实用优先**：用户关心的是「怎么用」，不是项目背景故事
- **视觉层次**：用 badge、引用块、分隔线创造清晰的阅读节奏
- **步骤明确**：每个工具的使用方式必须是可操作的步骤，不是泛泛描述

## 结构模板

### 1. 顶部 Badge

```markdown
# 项目名

[![GitHub](https://img.shields.io/github/license/用户名/仓库名.svg?label=LICENSE&logo=github)](链接)
![GitHub Stars](https://img.shields.io/github/stars/用户名/仓库名.svg?style=flat&logo=github)
```

只放 LICENSE 和 Stars，不要堆砌太多 badge。

### 2. 一句话介绍

一句话说明项目做什么、面向谁。不要写长段背景介绍。

### 3. 工具列表表格

```markdown
| 工具 | 说明 | 去提交 |
|------|------|--------|
| **tool-name** | 一句话说明 | [![Issue](https://img.shields.io/badge/提交请求-名称-blue?style=flat&logo=图标)](链接) |
```

- 工具名加粗
- 用 badge 作为提交按钮，比普通链接更醒目
- 颜色区分不同工具类型（blue=docker, orange=deb 等）

### 4. 每个工具独立 Section

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

#### 3、最终操作（命令）

\```bash
具体可复制的命令
\```

### 查找资源

| 网站 | 说明 |
|------|------|
| [名称](URL) | 一句话说明 |
```

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
