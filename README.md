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
- `content/about/`、`content/authors/`、`content/resources/`：站点页面
- `.github/workflows/hugo.yml`：Hugo CI / GitHub Pages workflow
