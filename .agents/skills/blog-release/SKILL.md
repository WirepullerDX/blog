---
name: blog-release
description: 验收 WirepullerDX/blog 的改动，运行 Astro 检查和本地生产预览，按用户要求准备提交、推送、开 PR 或验证 ESA Pages 部署。适用于发布前检查、PR 和部署状态核对。
---

# 博客验收与发布

1. 读取 [仓库总纲](../../../AGENTS.md) 的分支、提交和验证规则。开始前检查 `git status --short` 和当前分支，保留用户已有变更。需要创建主题分支时以最新 `origin/esa` 为基准。
2. 对渲染、内容或生成资源的改动，按总纲运行 `pnpm check`、`pnpm type-check` 和 `pnpm build`。用 `pnpm preview` 检查受影响路由和功能，Pagefind 搜索使用生产构建产物验收。命令与构建流程见 [package.json](../../../package.json)，内容验收细节见 [写作指南](../../../docs/WRITING_GUIDE.md#12-发布前流程)。
3. Git 操作做到用户请求的阶段。准备 PR 时核对基线、只纳入任务改动，并检查 `git diff --check`。创建 PR 时推送主题分支，目标分支用 `esa`。验证和 PR 说明应记录具体改动、实际检查及未验证项。
4. 检查 ESA 时，以 [esa.jsonc](../../../esa.jsonc) 的构建配置和实际部署状态为准。核对部署提交与预期提交是否一致，再访问本次涉及的线上路由或资源。只有确实可访问的预览才附链接。
