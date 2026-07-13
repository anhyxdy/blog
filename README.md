# anh's blog

一个基于 [Astro](https://astro.build) 的个人博客模板，适合记录 Go 后端、技术笔记和日常随想。

## 项目特点

- 支持文章列表、归档、分类和标签
- 支持文章封面图和正文内图片
- 支持深色模式、搜索、RSS、站点地图
- 文章使用 Markdown 编写，维护简单

## 运行环境

- Node.js 20+
- pnpm 9+

## 本地启动

```bash
pnpm install
pnpm dev
```

启动后访问：

```text
http://127.0.0.1:4321/
```

## 常用命令

```bash
pnpm dev       # 启动开发环境
pnpm check     # 类型检查
pnpm build     # 构建生产版本
pnpm preview   # 预览构建结果
pnpm new-post  # 新建文章
```

## 写文章

最简单的方式是用脚手架新建一篇文章：

```bash
pnpm new-post my-first-post
```

会在 `src/content/posts/` 下生成对应的 Markdown 文件。

文章头部示例：

```md
---
title: 我的第一篇文章
published: 2026-07-13
updated: 2026-07-13
description: 这是一篇文章简介
image: ./cover.jpg
tags: [Go, 后端, 学习]
category: 技术笔记
draft: false
lang: zh_CN
---
```

### 字段说明

- `title`：文章标题
- `published`：发布时间，决定首页和归档页排序
- `updated`：更新时间，可选
- `description`：文章简介，会显示在首页卡片和搜索结果里
- `image`：文章封面图
- `tags`：标签，可多个
- `category`：分类，通常一个
- `draft`：是否草稿，`true` 的文章不会在生产环境展示
- `lang`：文章语言，可选

## 图片怎么放

### 文章封面图

推荐把文章和图片放在同一个目录，例如：

```text
src/content/posts/my-post/
  index.md
  cover.jpg
```

然后在 frontmatter 里写：

```md
image: ./cover.jpg
```

### 正文图片

正文里直接用 Markdown 图片语法：

```md
![架构图](./arch.png)
```

如果图片放在 `public` 目录，例如：

```text
public/images/arch.png
```

那就写：

```md
![架构图](/images/arch.png)
```

## 分类和标签

分类和标签都写在文章 frontmatter 里，不需要单独建文件。

```md
tags: [Go, Gin, 后端]
category: 技术笔记
```

查看方式：

- 归档页：`/archive/`
- 按标签筛选：`/archive/?tag=Go`
- 按分类筛选：`/archive/?category=技术笔记`
- 未分类文章：`/archive/?uncategorized=true`

## 个人信息修改

如果你要继续定制博客，主要改这些地方：

- `src/config.ts`：站点标题、副标题、头像、简介、社交链接、banner 开关
- `src/content/spec/about.md`：关于页内容
- `astro.config.mjs`：正式站点域名 `site`
- `public/favicon/`：站点图标
- `src/assets/images/`：头像、banner 等本地图片

## 删除文章

直接删除对应的 Markdown 文件即可。

如果文章有配套图片，也一起删除对应图片文件。

## 部署

构建前建议先执行：

```bash
pnpm check
pnpm build
```

然后把生成的 `dist/` 部署到 Vercel、Netlify、GitHub Pages 或你自己的服务器。

## 备注

这个项目已经不再使用模板示例内容，后续你只需要维护自己的文章和配置就行。
