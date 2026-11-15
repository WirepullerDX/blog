---
name: blog-writing
description: 在 WirepullerDX/blog 中起草、创建、编辑或验收文章、动态、项目和特殊页面。适用于 Markdown、MDX、frontmatter、分类标签、日期、slug、草稿和加密文章。
---

# 博客内容写作

1. 从 [写作指南](../../../docs/WRITING_GUIDE.md) 选择对应内容类型的章节，并打开一份相近的现有内容。需要核对字段或默认值时，读取 [内容 schema](../../../src/content.config.ts)。
2. 新文章使用 `src/content/posts/<slug>/index.md`，需要组件时使用 `index.mdx`。`pnpm new-post <slug>` 默认生成单文件，使用脚手架后按写作指南整理有封面的文章。动态的入口是 `pnpm new-d`，数据在 `src/content/dynamic/`。
3. 已发布内容沿用原有 slug、路径和发布日期，内容更新写入 `updated`。新内容按 `Asia/Shanghai` 填日期，分类和标签沿用站内现有用词。遇到加密内容时，按 schema 和写作指南核对 `password` 字段及预览行为。
4. 交付前核对改动路由、字段、封面和链接。需要准备封面、相册或 R2 素材时，读 [blog-media](../blog-media/SKILL.md)。本地验收或发布按 [blog-release](../blog-release/SKILL.md) 运行实际需要的步骤。
