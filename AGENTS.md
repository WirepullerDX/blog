# Repository Guidelines

## Project Structure & Module Organization

Firefly is an Astro 7 site with Svelte islands and TypeScript configuration. Main source code lives in `src/`: routes in `src/pages`, layouts in `src/layouts`, reusable UI in `src/components`, styles in `src/styles`, content in `src/content`, helpers in `src/utils`, and Markdown/HTML plugins in `src/plugins`. Site configuration is split across `src/config` with matching type definitions in `src/types`; prefer imports from `@/config` when available. Static files served directly belong in `public`, source-managed images in `src/assets`, docs in `docs`, and automation in `scripts`.

## Task Guides

Use the relevant task skill alongside these repository-wide instructions.

| Skill | Use when |
| --- | --- |
| [blog-writing](.agents/skills/blog-writing/SKILL.md) | Creating or editing articles, dynamics, projects, or special pages |
| [blog-media](.agents/skills/blog-media/SKILL.md) | Preparing images, article covers, gallery media, or Cloudflare R2 assets |
| [blog-release](.agents/skills/blog-release/SKILL.md) | Running publication checks, preparing a PR, or verifying an ESA deployment |

The [writing guide](docs/WRITING_GUIDE.md) is the detailed reference for content and media. The [content task prompt](docs/AGENT_CONTENT_PROMPT.md) is available as a copyable starting point.

## Deployment & Branching

- Production branch is `esa`.
- `master` is a frozen Cloudflare Pages archive. Never base content work on it or open PRs against it.
- Base topic branches on `origin/esa` and use `esa` as the PR base.
- Changes merged into `esa` trigger ESA Pages automatically. `esa.jsonc` defines the build and output directory.
- Match Git actions to the user's request. A request for a local draft or preview ends with local verification. A request to open a PR includes pushing its topic branch. Merge when the user requests a merge.
- Do not run `wrangler pages` commands; `wrangler` is only for authorized R2 uploads.
- Do not add `public/_headers`; security headers and CSP are configured in the ESA console.
- R2 and Twikoo remain external Cloudflare services and should not be migrated during content tasks.

## Build, Test, and Development Commands

Use the Node.js and pnpm versions specified in [package.json](package.json).

- `pnpm dev` or `pnpm start`: run the local Astro dev server.
- `pnpm check`: run Astro diagnostics.
- `pnpm type-check`: run TypeScript with `--noEmit`.
- `pnpm format`: format `src` and `scripts` with Biome.
- `pnpm lint`: run Biome checks and safe fixes on `src` and `scripts`.
- `pnpm build`: run the production build defined in [`package.json`](package.json) and write the site and Pagefind index to `dist`.
- `pnpm preview`: preview the production build locally.
- `pnpm new-post`: scaffold a new content post.
- `pnpm new-d`: scaffold a new dynamic entry.

## Coding Style & Naming Conventions

Biome is the formatter and linter. It uses tabs for indentation and double quotes for JavaScript/TypeScript strings. Keep Astro and Svelte components in `PascalCase` (`PostCard.astro`, `Search.svelte`), config modules in `camelCase` ending with `Config.ts`, and utilities in descriptive kebab case such as `date-utils.ts`. Keep `src/types` aligned with `src/config`. Avoid unrelated formatting churn.

## Testing Guidelines

There is no dedicated unit-test framework configured. Before submitting changes, run `pnpm check`, `pnpm type-check`, and `pnpm build` for rendering, content, or generated asset work. For content work, use local `pnpm build && pnpm preview`; do not assume an ESA Pages PR preview is available. For visual or interactive changes, verify with `pnpm dev` or `pnpm preview` and include screenshots in the PR. Name future tests near the feature they cover, using the local file name as the stem.

## Commit & Pull Request Guidelines

Use Conventional Commits, matching the current history: `feat: ...`, `fix: ...`, and `chore: ...`. Use the configured Git identity or the identity provided by the user. Keep commits and PRs focused on one concern. PRs should include a concise summary, linked issues when relevant, validation commands run, and screenshots for UI changes. Discuss major features or design changes in an issue or discussion before implementation.

## Security & Configuration Tips

Do not commit secrets, tokens, or service keys in config files. Keep deployment-specific settings in the target platform environment, and review generated data such as `src/constants/lqips.json` and `src/constants/github-card-data.json` before committing it.
