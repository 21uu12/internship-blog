# 大数据实习日志

个人实习博客，记录大数据学习过程中的笔记与思考。

## 本地预览

```bash
# 安装依赖（第一次克隆后运行）
npm install

# 启动本地预览服务
hexo server
# 打开浏览器访问 http://localhost:4000
```

## 写新文章

```bash
# 新建文章（标题替换成你要写的内容）
hexo new "今天学了什么"

# 文章文件在：source/_posts/今天学了什么.md
# 用任意编辑器打开该文件，修改 frontmatter 中的 categories 和 tags，然后写正文
```

frontmatter 示例：

```markdown
---
title: 今天学了什么
date: 2026-06-21 09:00:00
categories: 技术笔记
tags:
  - Hive
  - SQL
---
```

## 发布文章

```bash
git add .
git commit -m "新文章：今天学了什么"
git push
# Vercel 自动构建，几十秒后博客更新
```

## 分类说明

| 分类 | 用途 |
|---|---|
| 每日记录 | 当天实习内容、学到了什么 |
| 技术笔记 | 知识点整理（Hive / Spark / StarRocks 等） |
| 问题排查 | 踩坑记录与解决方法 |
| 周总结 | 每周复盘 |
