# 博客写作与内容维护规范

> 适用仓库：`WirepullerDX/blog`
> 主题：Firefly 6.16.8 / Astro 7
> 站点：`https://blog.rxcdjg.dpdns.org`
> 最后更新：2026-10-29


本文档面向两个读者：

1. **博客作者**：按本文创建文章、动态、项目、相册和维护特殊页面。
2. **AI Agent**：把本文当作内容维护的执行规范，不要凭模板记忆猜测字段或路径。

## 快速开始

```text
新建文章：src/content/posts/<slug>/index.md + cover.webp
新建动态：pnpm new-d "内容"
新建项目：src/content/projects/<slug>/index.md + cover.webp
新建相册：src/config/galleryConfig.ts + public/gallery/<album-id>/
发布前：pnpm check → pnpm build → pnpm preview
发布：commit → push master → Cloudflare Pages 自动部署
```

## 目录

1. [核心原则](#1-核心原则)
2. [项目结构速查](#2-项目结构速查)
3. [本地环境与命令](#3-本地环境与命令)
4. [博客文章](#4-博客文章)
5. [正文写作规范](#5-正文写作规范)
6. [加密文章](#6-加密文章)
7. [动态](#7-动态)
8. [项目](#8-项目)
9. [相册与 R2](#9-相册与-r2)
10. [特殊页面](#10-特殊页面)
11. [SEO 与可访问性](#11-seo-与可访问性)
12. [发布前流程](#12-发布前流程)
13. [Git 与部署流程](#13-git-与部署流程)
14. [AI Agent 执行规范](#14-ai-agent-执行规范)
15. [内容模板索引](#15-内容模板索引)
16. [发布检查清单](#16-发布检查清单)
17. [常见问题](#17-常见问题)
18. [维护记录](#18-维护记录)

---

## 1. 核心原则

- 只用 **pnpm**，不要使用 `npm install` 或 `yarn`。
- 内容源文件在 `src/content/`，页面和构建产物不是内容源。
- 不要直接修改 `dist/`、`src/constants/lqips.json` 或 `src/constants/github-card-data.json`，它们由构建脚本生成。
- 文章、项目、相册的目录名会进入 URL。发布后不要随意改名，否则旧链接会失效。
- 文件名、目录名使用小写英文、数字和连字符：`my-first-post`，不要使用空格或中文路径。
- 日期按站点时区 `Asia/Shanghai` 理解。
- 正文大图优先放 Cloudflare R2；封面放文章目录，交给 Astro 优化。
- 不提交密码、API Key、Cloudflare Token 或其他秘密。文章 `password` 字段会明文出现在 Git 仓库中，不是秘密存储方案。
- 单次提交只做一类事情：文章归文章，配置归配置，修复归修复。
- 不运行全局格式化，不制造无关换行或缩进改动。

---

## 2. 项目结构速查

| 路径 | 用途 |
|---|---|
| `src/content/posts/` | 博客文章 |
| `src/content/projects/` | 项目展示 |
| `src/content/dynamic/` | 动态/短内容 |
| `src/content/spec/about.md` | 关于页面正文 |
| `src/content/spec/friends.mdx` | 友链页本站信息及自定义内容 |
| `src/content/spec/guestbook.md` | 留言板说明 |
| `public/gallery/<album-id>/` | 相册本地图片或 `urls.txt` |
| `src/assets/images/` | 需要 Astro 优化的站点图片 |
| `public/` | 原样复制到站点的静态文件 |
| `src/config/siteConfig.ts` | 站点核心配置 |
| `src/config/profileConfig.ts` | 左侧资料卡、头像、社交链接 |
| `src/config/navBarConfig.ts` | 顶部导航 |
| `src/config/galleryConfig.ts` | 相册元数据 |
| `src/config/friendsConfig.ts` | 友链列表 |
| `src/config/commentConfig.ts` | Twikoo 评论配置 |
| `src/layouts/Layout.astro` | 全站 `<head>`、统计分析脚本等 |
| `public/_headers` | Cloudflare Pages 安全响应头 |
| `docs/` | 项目文档 |
| `dist/` | 构建产物，自动生成，不要手改 |

---

## 3. 本地环境与命令

### 3.1 环境要求

- Node.js `>=22.23.0`，Cloudflare Pages 固定使用 `22.23.0`
- pnpm `11.22.0`
- Git

### 3.2 常用命令

```bash
# 安装依赖
pnpm install

# 本地开发，默认 http://localhost:4321
pnpm dev

# Astro 内容/组件检查
pnpm check

# TypeScript 检查
pnpm type-check

# 生产构建（会更新 LQIP、GitHub 卡片数据、Pagefind 索引）
pnpm build

# 预览生产构建
pnpm preview

# 新建文章（脚本生成单文件 src/content/posts/my-new-post.md）
# 推荐再整理为 src/content/posts/my-new-post/index.md，方便同目录放封面
pnpm new-post my-new-post

# 新建动态
pnpm new-d "今天把 NAS 的备份链路重新整理了一遍。"
```

### 3.3 构建链

`pnpm build` 会依次执行：

1. 生成 GitHub 卡片缓存
2. 生成图片 LQIP
3. 生成 VNDB 封面（当前页面未启用，实际跳过）
4. Astro 构建
5. 清理未使用的 Live2D 资源
6. 字体子集化
7. 压缩内联脚本
8. Pagefind 搜索索引

因此：

- 不要单独手改生成文件；
- Pagefind 的搜索行为要以 `pnpm build && pnpm preview` 为准，开发模式不能完整代表线上；
- 构建成功不等于页面视觉一定正确，仍要打开页面检查。

---

## 4. 博客文章

### 4.1 目录规范

推荐使用“目录 + `index.md`”的形式：

```text
src/content/posts/my-first-post/
├── index.md
└── cover.webp
```

路由由内容路径决定，例如：

```text
src/content/posts/my-first-post/index.md
→ /posts/my-first-post/
```

`pnpm new-post` 默认只生成单个 `src/content/posts/my-new-post.md`。如果文章需要本地封面，推荐手动创建同名目录，把文件移动为 `src/content/posts/my-new-post/index.md`，再放入 `cover.webp`。

注意：

- 不要在 frontmatter 里依赖 `slug` 字段，当前项目的文章路由以内容 ID/文件路径为准。`new-post` 脚本生成的 `slug` 字段不会参与路由。
- 发布后不要修改目录名。
- 不要在已有文章目录中混放无关文件。
- 文章正文不需要手写 H1，页面会自动使用 frontmatter 的 `title` 作为 H1。

### 4.2 完整 Frontmatter 模板

```yaml
---
title: "文章标题"
published: 2026-10-29
updated: 2026-10-29
description: "用于文章卡片、搜索结果和分享卡片的简介。"
image: "./cover.webp"
tags: ["NAS", "Docker", "自托管"]
category: "技术实践"
draft: false
pinned: false
author: "阿杰"
comment: true
series: "博客搭建"
seriesOrder: 1
licenseName: "CC BY-NC-SA 4.0"
licenseUrl: "https://creativecommons.org/licenses/by-nc-sa/4.0/"
---
```

### 4.3 字段规范

| 字段 | 必填 | 类型 | 说明 |
|---|---:|---|---|
| `title` | 是 | string | 文章标题，页面 H1 |
| `published` | 是 | date | 发布日期，建议 `YYYY-MM-DD` 或带时区的 ISO 时间 |
| `updated` | 否 | date | 最后更新时间，用于“上次编辑”提示 |
| `description` | 推荐 | string | 卡片、搜索和分享摘要，建议 60–120 个中文字符 |
| `image` | 推荐 | string | 封面路径；本地封面写 `./cover.webp` |
| `tags` | 否 | string[] | 标签数组，避免同义标签混用 |
| `category` | 否 | string/null | 单分类，不要塞多个分类 |
| `draft` | 否 | boolean | `true` 时生产构建隐藏，开发环境仍可见 |
| `pinned` | 否 | boolean | 置顶；首页排序优先于普通文章 |
| `author` | 否 | string | 留空显示资料卡名称 |
| `comment` | 否 | boolean | 默认 `true`；密码文章会强制隐藏评论 |
| `series` | 否 | string | 系列名称，必须和其他文章完全一致 |
| `seriesOrder` | 否 | number | 系列内升序，数字越小越靠前 |
| `password` | 否 | string | 前端解密密码；不是安全存储 |
| `passwordHint` | 否 | string | 密码提示 |
| `licenseName` | 否 | string | 文章许可证名称 |
| `licenseUrl` | 否 | string | 许可证链接 |
| `sourceLink` | 否 | string | 原文/来源链接，可选 |

### 4.4 Frontmatter 编写规则

- 日期不要加引号，直接写 `published: 2026-10-29`，让 YAML 解析为日期。
- 标题里有冒号、井号或引号时，整个标题加双引号。
- Windows 路径不要写反斜杠，统一使用 `/`。
- 封面建议放在文章同目录，写 `image: "./cover.webp"`。
- 没有封面就不写 `image` 或留空字符串。
- 随机封面 `image: "api"` 当前未启用，不要使用。
- `seriesOrder` 只在同一 `series` 内有意义。
- `pinned` 不要滥用，置顶文章会长期占据首页靠前位置。
- `draft: true` 只适合未完成稿件，不要靠草稿隐藏敏感信息。

### 4.5 封面图规范

- 格式：WebP 优先
- 建议比例：16:9 或 3:2
- 建议尺寸：1600 × 900 左右
- 建议体积：300 KB 以内
- 文件名固定为 `cover.webp`
- 路径写相对路径：`"./cover.webp"`

不要把封面放到 `public/` 再手写绝对路径，这样会绕过 Astro 的响应式图片优化。

---

## 5. 正文写作规范

### 5.1 标题层级

- H1：由系统根据 `title` 自动生成，正文不要重复写 H1。
- H2：文章的主要章节。
- H3：章节下的小节。
- 不要从 H2 直接跳到 H4。

推荐结构：

```markdown
## 背景

## 方案

### 关键步骤

## 结果

## 总结
```

### 5.2 基础排版

支持标准 Markdown：

- 粗体、斜体、删除线
- 行内代码
- 有序/无序列表
- 表格
- 引用
- 任务列表
- 外部链接

外部链接会自动处理为安全的新窗口链接，正常写标准 Markdown 即可。

### 5.3 提醒框

使用 GitHub Callout 语法：

```markdown
> [!NOTE]
> 这是一条提示。

> [!TIP]
> 这是一条建议。

> [!WARNING]
> 这是需要小心的事项。
```

不要使用当前主题未启用的 Python-Markdown `!!! note` 语法。

### 5.4 数学公式

行内公式：

```markdown
质能方程是 $E = mc^2$。
```

块级公式：

```markdown
$$
\int_{0}^{1} x^2 \, dx = \frac{1}{3}
$$
```

### 5.5 Mermaid 图表

````markdown
```mermaid
flowchart LR
  A[Markdown] --> B[Astro]
  B --> C[Cloudflare Pages]
```
````

Mermaid 在构建阶段处理。语法错误可能直接导致构建失败，提交前必须运行 `pnpm check` 和 `pnpm build`。

### 5.6 PlantUML

PlantUML 通过外部服务渲染，网络不稳定时可能失败。除非确实需要 UML，否则优先使用 Mermaid。不要把它用在关键正文中。

### 5.7 代码块与代码组

普通代码块：

````markdown
```ts
const answer = 42;
```
````

代码组：

````markdown
::: code-group labels=[npm, pnpm]

```bash
npm install
```

```bash
pnpm install
```

:::
````

代码块必须写语言标识，例如 `bash`、`ts`、`js`、`json`、`yaml`。

### 5.8 图片网格

````markdown
[grid]
![图片一](https://img.rxcdjg.dpdns.org/gallery/demo/01.webp)
![图片二](https://img.rxcdjg.dpdns.org/gallery/demo/02.webp)
![图片三](https://img.rxcdjg.dpdns.org/gallery/demo/03.webp)
[/grid]
````

网格适用于 2–4 张同组图片，图片必须有可读的 `alt`。

### 5.9 GitHub 卡片

```markdown
::github{repo="owner/repository"}
```

示例：

```markdown
::github{repo="CuteLeaf/Firefly"}
```

构建脚本会尝试刷新卡片数据。私有仓库的 GitHub 卡片在未配置 `GITHUB_TOKEN` 时可能无法获取数据，公开仓库不受影响。

### 5.10 图片写入规则

- **封面**：本地 `./cover.webp`，交给 Astro 优化。
- **正文大图**：上传 R2，正文写完整 HTTPS URL。
- **小图标/装饰图**：可以放 `public/` 或 `src/assets/`。
- 图片必须有有意义的 `alt`，纯装饰图片才使用空 alt。
- 不要用外部图片作为唯一封面，远程图片无法享受 Astro 的本地优化。

---

## 6. 加密文章

在 frontmatter 加：

```yaml
password: "测试密码"
passwordHint: "提示：站点副标题 + 年份"
```

行为：

- 构建时把正文 HTML 加密
- 页面只显示密码框
- 正确密码在浏览器本地解密
- 错误密码显示“密码错误，请重试”
- 密码在当前 `sessionStorage` 会话中缓存
- 加密文章不显示评论区

重要限制：

- 密码会明文出现在 Git 仓库中
- 公开仓库里的密码没有真正的保密性
- 不要用文章密码保存身份证号、密钥、账号密码等敏感信息
- 密码文章的内容仍会出现在页面标题、描述和 RSS 标题中，摘要会被替换为加密提示

测试时必须验证：

1. 错误密码有错误提示
2. 正确密码能解锁
3. 刷新页面后当前会话仍能解锁
4. 解密后的图片和交互组件正常加载
5. 评论区不显示

---

## 7. 动态

### 7.1 创建

优先使用脚本：

```bash
pnpm new-d "今天把 NAS 的备份链路重新整理了一遍。"
```

脚本会自动创建：

```text
src/content/dynamic/2026-10-29-111213.md
```

### 7.2 格式

```markdown
---
published: 2026-10-29 11:12:13
pinned: false
location: "杭州"
---

今天把 NAS 的备份链路重新整理了一遍。
```

规则：

- `published` 是必填日期时间
- 动态以短内容为主，建议 1–3 段
- `location` 不写也可以
- 支持 Markdown 和图片
- 动态页有评论开关，评论系统由全局配置决定
- 不要手动伪造日期；使用 `pnpm new-d` 或按站点时区写真实时间

---

## 8. 项目

### 8.1 目录

```text
src/content/projects/my-project/
├── index.md
└── cover.webp
```

访问地址由目录名决定：

```text
/projects/my-project/
```

### 8.2 模板

```yaml
---
title: "项目名称"
published: 2026-10-29
description: "一句话说明这个项目解决什么问题。"
image: "./cover.webp"
status: "developing"
tags: ["Astro", "工具", "开源"]
order: 10
link:
  - label: "GitHub"
    icon: "fa7-brands:github"
    value: "https://github.com/owner/repository"
  - label: "在线演示"
    icon: "material-symbols:open-in-new"
    value: "https://example.com"
---

## 项目简介

## 功能

## 使用方式

## 相关链接
```

### 8.3 字段规则

- `title`、`published` 必填
- `draft: true` 在开发环境可见、生产隐藏
- `order` 数字越大越靠前；未设置的项目排在后面
- `status` 推荐使用：`planning`、`developing`、`published`、`archived`
- `tags` 用于筛选
- `image` 使用本地 `./cover.webp`
- `link.icon` 支持 Iconify 图标名，也支持公开图片 URL

## 9. 相册与 R2

### 9.1 相册配置

在 `src/config/galleryConfig.ts` 的 `albums` 中添加：

```ts
{
	id: "2026-travel",
	name: "2026 旅行",
	description: "一些路上的照片",
	location: "云南",
	date: "2026-10-01",
	tags: ["旅行", "风景"],
	cover: "https://img.rxcdjg.dpdns.org/gallery/2026-travel/cover.webp",
}
```

`id` 同时是 URL 和 `public/gallery/<id>/` 目录名。

### 9.2 本地图片

```text
public/gallery/2026-travel/
├── cover.webp
├── 01.webp
└── 02.webp
```

封面优先级：

1. 配置中的 `cover`
2. 目录中的 `cover.*`
3. 第一张图片

本地相册图片不会经过 Astro 图片优化，并且会占用 Cloudflare Pages 的文件数量和单文件大小限制。

### 9.3 R2 远程图片

在相册目录创建 `urls.txt`：

```text
https://img.rxcdjg.dpdns.org/gallery/2026-travel/01.webp
https://img.rxcdjg.dpdns.org/gallery/2026-travel/02.webp
https://img.rxcdjg.dpdns.org/gallery/2026-travel/03.webp
```

- 每行一个 URL
- 空行和 `#` 开头的行会被忽略
- 可以同时存在本地图片和 `urls.txt`

### 9.4 R2 上传规范

当前使用：

- Bucket：`blog-image`
- 域名：`img.rxcdjg.dpdns.org`
- `r2.dev`：关闭

上传示例：

```powershell
pnpm exec wrangler r2 object put "blog-image/gallery/2026-travel/01.webp" --file ".\01.webp" --content-type "image/webp" --cache-control "public, max-age=31536000, immutable" --remote
```

推荐目录：

```text
posts/2026/10/<slug>-01.webp
covers/<slug>.webp
gallery/<album-id>/01.webp
```

验证：

```powershell
curl.exe -sS -D - -o NUL "https://img.rxcdjg.dpdns.org/gallery/2026-travel/01.webp"
```

必须看到：

```text
HTTP/1.1 200 OK
Content-Type: image/webp
Cache-Control: public, max-age=31536000, immutable
cf-cache-status: HIT
```

检查缓存要用普通 GET，不要用 `curl -I`；HEAD 可能返回 `DYNAMIC`，不能代表 GET 未缓存。

---

## 10. 特殊页面

### 10.1 关于页

文件：`src/content/spec/about.md`

- 使用普通 Markdown
- 不需要 frontmatter 标题
- 保持个人介绍与站点定位一致
- GitHub 卡片可以使用 `::github{repo="owner/repo"}`

### 10.2 友链页

文件：`src/content/spec/friends.mdx`

包含两部分：

1. `site` 对象：本站名称、描述、URL、头像
2. JSX/Markdown 自定义内容：友链说明、申请流程

`site` 中的信息必须与 `siteConfig.ts` 同步，尤其是：

- `name`
- `desc`
- `url`
- `avatar`

修改站点描述后，记得同步这里的 `desc`。

### 10.3 留言板

文件：`src/content/spec/guestbook.md`

仅写留言规则和说明；实际评论区由全局 Twikoo 配置提供。

---

## 11. SEO 与可访问性

### 11.1 标题

- 文章标题要具体，不要只写“记录”“总结”
- 中文标题不必强行追求英文的 30–60 字符标准
- 同一篇文章不要重复使用多个主题标题

### 11.2 描述

- `description` 用于卡片、搜索结果和分享预览
- 建议 60–120 个中文字符
- 说清楚“这篇文章解决什么问题”，不要堆关键词
- 不要复制文章第一段作为所有文章的固定描述

### 11.3 图片

- 每张内容图片写明 `alt`
- 封面不要用纯装饰图
- 大图压成 WebP 后再上传 R2
- 不要把十几 MB 的原图直接放进仓库或 R2

### 11.4 链接

- 内链使用站内相对路径：`/posts/another-post/`
- 外链使用完整 URL
- 链接文字要描述目标，不要只写“点击这里”

### 11.5 结构化数据

文章页会自动生成 BlogPosting JSON-LD。不要手工在正文中重复插入 JSON-LD。

### 11.6 分享卡片

当前 `siteConfig.post.share: true`，文章分享按钮开启。`siteConfig.post.generateOgImages: false`，不自动生成 1200×630 OG 图，分享卡片使用文章封面。

如果文章没有封面，分享预览可能缺少图片；重要文章应提供封面。

---

## 12. 发布前流程

### 12.1 人工检查

1. 新建/修改内容文件
2. 确认 frontmatter 字段无拼写错误
3. 确认图片路径存在
4. 运行：

```bash
pnpm check
pnpm type-check
pnpm build
pnpm preview
```

5. 打开：

```text
http://localhost:4321/posts/<slug>/
http://localhost:4321/projects/<slug>/
http://localhost:4321/gallery/<album-id>/
http://localhost:4321/dynamic/
```

6. 检查：

- 桌面 1440px
- 手机 375px
- 深色/浅色模式
- 图片能否加载
- 目录能否跳转
- 代码块是否横向溢出
- 分享按钮是否存在
- 评论区是否正常
- 控制台无红色错误

### 12.2 AI Agent 验收

AI Agent 在交付前必须报告：

- 修改了哪些文件
- 新增内容的 URL
- 执行过哪些检查
- 是否有构建警告
- 是否还有未验证项
- 是否已提交/推送

---

## 13. Git 与部署流程

### 13.1 提交前

```bash
git status --short
git diff
```

确认只包含本次任务需要的文件。

### 13.2 提交信息

使用 Conventional Commits：

```text
feat(content): add NAS backup article
fix(post): correct code block
docs(guide): update writing workflow
chore(site): update navigation config
```

不要使用：

```text
update
fix
改了一下
```

### 13.3 推送与部署

- 当前生产分支：`master`
- 推送到 `master` 会触发 Cloudflare Pages 构建
- Cloudflare Pages 构建命令：`pnpm build`
- 输出目录：`dist`
- Node 环境：`22.23.0`
- 不要设置 `CF_WORKERS`
- 不要依赖 GitHub Pages workflow；当前部署在 Cloudflare Pages

### 13.4 回滚

优先使用 Git 回滚：

```bash
git revert <commit>
```

也可以使用 Cloudflare Pages 控制台回滚到上一次成功部署。

注意：Git 回滚不会删除已经上传到 R2 的图片。R2 对象需要单独处理。

### 13.5 Fork 同步

当前仓库是 `CuteLeaf/Firefly` 的公开 fork，使用 GitHub 的一键 Sync fork。

如果未来要把仓库改成私有，必须先执行 `Leave fork network`，之后会失去一键同步，只能手动添加 upstream 合并。不要在普通写文章任务中更改仓库可见性或 fork 状态。

---

## 14. AI Agent 执行规范

AI Agent 接手内容任务时，按以下顺序执行：

1. 阅读 `AGENTS.md`、`CLAUDE.md` 和本文档。
2. 执行 `git status --short`，确认当前工作区是否有用户未提交的改动。
3. 判断任务属于哪一类：
   - 文章
   - 动态
   - 项目
   - 相册
   - 特殊页面
   - 配置/主题代码
4. 内容任务优先只改 `src/content/`；不要顺手改主题代码。
5. 不要修改 `siteConfig.site_url`、导航、评论、统计等配置，除非用户明确要求。
6. 保持现有文章的 slug 和文件路径不变。
7. 日期使用 `Asia/Shanghai`。
8. 新建文章使用目录 + `index.md`，封面放同目录。
9. 正文大图用 R2 URL，不要加入仓库。
10. 不手动修改生成的 JSON 文件。
11. 运行 `pnpm check`；涉及构建、图片或组件时运行 `pnpm build`。
12. 用 `pnpm preview` 检查生产构建效果。
13. 如果任务涉及密码文章，必须实际测试错误密码、正确密码、刷新缓存和图片加载。
14. 如果任务涉及 R2，必须验证 GET 200 和 `cf-cache-status: HIT`。
15. 不要提交或推送，除非用户明确要求。
16. 如果用户要求提交，使用仓库现有作者身份或用户指定身份，不要自行编造。
17. 提交前检查 `git diff --check`。
18. 推送后报告 commit hash、部署状态和验证结果。

### AI Agent 禁止事项

- 禁止使用 `npm` 代替 `pnpm`
- 禁止直接编辑 `dist/`
- 禁止手动编辑 `src/constants/lqips.json`
- 禁止手动编辑 `src/constants/github-card-data.json`
- 禁止把密码、Token、Cookie 写进仓库
- 禁止强制推送、重写历史或删除分支
- 禁止为了“优化”而大面积重构主题样式
- 禁止在内容任务中修改仓库可见性
- 禁止把未验证的构建结果说成已通过

---

## 15. 内容模板索引

- 文章：见第 4.2 节
- 动态：见第 7.2 节
- 项目：见第 8.2 节
- 相册：见第 9.1 节
- 加密文章：见第 6 节
- 友链页本站信息：见第 10.2 节

---

## 16. 发布检查清单

### 文章

- [ ] 目录名是 ASCII kebab-case
- [ ] `title`、`published` 存在
- [ ] `description` 能概括文章
- [ ] `image: "./cover.webp"` 文件存在
- [ ] 正文从 H2 开始
- [ ] 图片有 alt
- [ ] 代码块写了语言
- [ ] 外部链接使用完整 URL
- [ ] 系列文章 `series`/`seriesOrder` 正确
- [ ] `pnpm check` 通过
- [ ] `pnpm build` 通过
- [ ] 375px 手机预览正常
- [ ] 控制台无错误

### 项目

- [ ] `title`、`published` 存在
- [ ] `status` 使用标准值
- [ ] `order` 符合展示顺序
- [ ] 封面路径正确
- [ ] GitHub 链接有效
- [ ] 项目详情页可访问

### 相册

- [ ] `galleryConfig.ts` 有对应 `id`
- [ ] `public/gallery/<id>/` 存在
- [ ] 本地图片或 `urls.txt` 有内容
- [ ] 封面能加载
- [ ] R2 图片 GET 返回 200
- [ ] `cf-cache-status: HIT`
- [ ] 手机瀑布流正常

### 动态

- [ ] 使用 `pnpm new-d` 或时间格式正确
- [ ] 动态页能显示
- [ ] 首页侧栏最新动态正常
- [ ] 图片和链接可点击

### 加密文章

- [ ] `password` 和 `passwordHint` 正确
- [ ] 错误密码有提示
- [ ] 正确密码可解锁
- [ ] 刷新后当前会话可解锁
- [ ] 解密后图片正常
- [ ] 评论区隐藏
- [ ] RSS 不泄露正文

---

## 17. 常见问题

### 为什么文章构建后是 404？

检查：

- 文件是否在 `src/content/posts/` 下
- 是否是 `index.md` 或 `.mdx`
- 目录名是否包含空格或非法字符
- 是否 `draft: true`

### 为什么封面不显示？

检查：

- `image` 是否写 `./cover.webp`
- 文件是否和 `index.md` 同目录
- 文件名大小写是否一致
- 是否误写成了 `/src/content/...`

### 为什么搜索搜不到？

Pagefind 是构建时索引：

```bash
pnpm build
pnpm preview
```

开发服务器下的搜索结果不完整。

### 为什么 R2 图片是 200 但 `cf-cache-status` 是 DYNAMIC？

- 用 GET 检查，不要用 `curl -I`
- 确认 Cache Rule 已生效
- 第二次 GET 应看到 `HIT`

### 为什么控制台有 Cloudflare Analytics 报错？

检查：

- 是否只保留手动 JS 安装的站点
- 是否还启用了 `rxcdjg.dpdns.org` 的自动设置
- 自动设置会向子域名注入第二段 beacon

### 为什么密码文章不能当秘密存储？

因为 `password` 明文写在 Markdown 源码里。私有仓库也不能改变“作者自己可见、构建时加密”的事实；公开仓库更等于公开密码。

---

## 18. 维护记录

| 日期 | 变更 |
|---|---|
| 2026-10-29 | 初版：文章、动态、项目、相册、特殊页面、SEO、流程和 AI Agent 规范 |
