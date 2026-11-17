# 文章维护指南

`articles/` 是博客文章的源目录。[按主题浏览全部文章](INDEX.md)。

## 选择栏目与主题

每篇文章属于一个栏目与一个主题。文件路径决定栏目，frontmatter 中的 `category` 必须与路径一致。`topic` 决定页面中的主题分组，与存放文件的子目录名称解耦。

| 栏目 `category` | 主题 `topic` | 显示名称 |
| --- | --- | --- |
| `learning-notes` | `rendering` | 渲染基础 |
| `learning-notes` | `pipeline` | 渲染管线 |
| `learning-notes` | `performance` | 性能分析与优化 |
| `learning-notes` | `nodes` | 材质节点 |
| `learning-notes` | `tools` | 工具开发 |
| `learning-notes` | `interview` | 面试复习 |
| `learning-notes` | `troubleshooting` | 问题排查 |
| `portfolio` | `showcase` | 作品展示 |
| `portfolio` | `shading` | 材质与角色渲染 |
| `portfolio` | `environment` | 天空与环境 |
| `portfolio` | `animation` | 动画与交互 |
| `portfolio` | `optimization` | 性能优化 |

主题定义的唯一配置为 [`src/data/articleTopics.json`](../src/data/articleTopics.json)。新增主题时更新该配置与本指南。

## 新建文章

在 `learning-notes/` 或 `portfolio/` 的合适子目录下创建 `.md` 文件。文件名作为 slug，必须全站唯一；例如 `sky-system.md` 对应 `/article/sky-system`。

现有文件的路径与名称保持稳定，可通过修改 `topic` 调整主题。移动文件会改变 OSS 正文地址，必须完整部署。

```md
---
title: UE 材质节点实践
category: learning-notes
topic: nodes
date: 2026-10-08
readTime: 5 min read
excerpt: 用一句话说明本文解决的问题与包含的内容。
tags: [UE, Shader]
cover: ""
pinned: false
pinnedOrder: 0
---

# 实践目标

说明场景与目标。

## 实现过程

记录关键步骤、参数与截图。

## 结果与复盘

总结结果、限制和下一步。
```

| 字段 | 约定 |
| --- | --- |
| `title` | 清晰的内容标题，不重复添加“学习记录”“渲染理论”等栏目或主题前缀 |
| `category` | `learning-notes` 或 `portfolio`，与顶层目录一致 |
| `topic` | 上表中该栏目支持的主题 ID |
| `date` | `YYYY-MM-DD`，用于列表排序 |
| `readTime` | 例如 `5 min read`，列表与阅读页显示为中文阅读时间 |
| `excerpt` | 简短摘要，说明内容范围，避免重复标题 |
| `tags` | 技术关键词，用英文逗号分隔 |
| `cover` | 图片地址；留空则显示文字封面 |
| `pinned` / `pinnedOrder` | 是否进入首页精选与排序（数字小的在前） |

## 正文与图片

正常使用 Markdown 标题；阅读器会将正文标题降低一级，页面自身的文章标题作为 H1。图片建议放到 `public/assets/article-images/`，正文引用：

```md
![图片说明](/assets/article-images/example.png)
```

也可使用已有的图床链接。正文中的相对资源链接以文章 Markdown 地址所在目录为基准。

## 生成与检查

```bash
npm run generate:articles
npm run build
```

生成器扫描两个栏目目录，更新 `src/data/generated/articles.ts` 和 `articles/INDEX.md`。缺失／无效主题、栏目与路径不一致、重复 slug 会使生成失败，并指出相关文件。

不要手改生成文件；元信息始终在 Markdown frontmatter 中维护。生成器不会修改文章正文。

## 发布

- 新增、重命名、移动文章，或修改任何元信息：完整发布网站与正文。
- 只改已有文章正文：可以仅同步 Markdown。
- 本地开发的阅读页仍从 OSS 加载正文；仅修改本地 `.md` 不会立即改变远程正文。

具体命令与环境配置见 [OSS 部署说明](../DEPLOY_OSS.md)。
