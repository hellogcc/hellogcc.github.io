---
title: "README 历史：feat: migrate site to Hugo"
date: "2026-09-14"
type: "readme-history"
categories:
  - "站点更新"
  - "README历史"
tags:
  - "README"
  - "迁移"
  - "时间线"
summary: "feat: migrate site to Hugo"
---

这篇文章整理自仓库中一次 README 更新提交，用于把原本堆叠在 README 中的站点变化拆分为独立博客记录。

- 提交时间：2026-09-14
- 提交主题：`feat: migrate site to Hugo`
- 提交哈希：`7cbb96e`

## 新增内容

- # OSDT / HelloGCC Hugo Blog
- 基于 **Hugo** 重构的 OSDT / HelloGCC 社区博客，采用科技感深色博客风格，收录历史文章、社区活动以及 README 迁移时间线。
- ## 本地运行
- ```bash
- hugo server
- ```
- ## 构建
- ```bash
- hugo --gc --minify
- ```
- ## 内容结构
- `blog/`、`2021/`、`2023/`、`2024/`、`2025/`、`2026/`：作为 Hugo 挂载内容源保留
- ……共 15 条变更
## 移除内容

- ## Welcome to OSDT Blog
- (This is a blog written in Chinese language.)
- 寻找校对和格式志愿者！欢迎提交PR修复在迁移过程中的错误和缺失！
- 官网地址： [https://hellogcc.org](https://hellogcc.org)
- 博客目录请点击 [index.md](index.md)
## 当时的 README 摘要

    # OSDT / HelloGCC Hugo Blog
    
    基于 **Hugo** 重构的 OSDT / HelloGCC 社区博客，采用科技感深色博客风格，收录历史文章、社区活动以及 README 迁移时间线。
    
    ## 本地运行
    
    ```bash
    hugo server
    ```
    
    ## 构建
    
    ```bash
    hugo --gc --minify
    ```
    
    ## 内容结构
    
    - `blog/`、`2021/`、`2023/`、`2024/`、`2025/`、`2026/`：作为 Hugo 挂载内容源保留
    - `content/posts/readme-history-*.md`：根据 README Git 提交历史拆分的独立 post
