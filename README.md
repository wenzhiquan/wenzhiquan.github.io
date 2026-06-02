# 温智全的博客

基于 [Hexo](https://hexo.io/) 框架 + [NexT](https://theme-next.js.org/) 主题搭建的个人博客。

线上地址：https://wenzhiquan.github.io

## 分支说明

| 分支 | 用途 |
|------|------|
| `gh-pages` | 博客源码（Hexo 工程、Markdown 文章、主题配置等） |
| `main` | 部署分支（由 `hexo deploy` 自动推送生成的静态文件） |

**日常写博客请在 `gh-pages` 分支操作。**

## 快速开始

### 1. 克隆仓库并切换到源码分支

```bash
git clone git@github.com:wenzhiquan/wenzhiquan.github.io.git
cd wenzhiquan.github.io
git checkout gh-pages
```

### 2. 安装依赖

```bash
npm install --legacy-peer-deps
```

## 常用命令

```bash
# 新建文章
npx hexo new "文章标题"

# 新建草稿（不会被发布）
npx hexo new draft "草稿标题"

# 将草稿发布为正式文章
npx hexo publish "草稿标题"

# 本地预览（默认 http://localhost:4000）
npx hexo server

# 生成静态文件
npx hexo generate

# 清除缓存和已生成的静态文件
npx hexo clean

# 部署到 GitHub Pages（推送到 main 分支）
npx hexo deploy

# 清除 + 生成 + 部署（一步到位）
npx hexo clean && npx hexo generate && npx hexo deploy
```

## 写文章

### 创建新文章

```bash
npx hexo new "我的新文章"
```

会在 `source/_posts/` 下生成 Markdown 文件，打开编辑即可。

### Front Matter

每篇文章开头需要包含 YAML 格式的元信息：

```yaml
---
title: 文章标题
date: 2026-06-02 12:00:00
categories:
  - 分类名
tags:
  - 标签1
  - 标签2
---
```

### 文章摘要

在正文中插入 `<!-- more -->` 标记，该标记之前的内容会作为首页摘要展示。

### 图片

文章图片统一放在 `source/uploads/in-post/` 目录下，建议按主题建子目录：

```
source/uploads/in-post/
├── zsh/
│   ├── oh-my-zsh-install.png
│   └── ...
├── security/
│   └── ...
```

在文章中引用：

```markdown
![图片描述](/uploads/in-post/zsh/oh-my-zsh-install.png)
```

## 发布流程

```bash
# 1. 确保在 gh-pages 分支
git checkout gh-pages

# 2. 写完文章后，提交源码
git add .
git commit -m "新增文章：xxx"
git push origin gh-pages

# 3. 部署到线上
npx hexo clean && npx hexo deploy
```

`hexo deploy` 会自动将生成的静态文件推送到 `main` 分支，GitHub Pages 会自动更新。

## 目录结构

```
.
├── _config.yml          # Hexo 主配置
├── _config.next.yml     # NexT 主题配置
├── source/
│   ├── _posts/          # 文章（Markdown）
│   ├── _drafts/         # 草稿
│   └── uploads/         # 图片等静态资源
├── themes/              # 主题（当前通过 npm 安装 NexT）
└── package.json
```
