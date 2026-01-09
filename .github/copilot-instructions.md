<!-- Copilot / AI agent guidance for the Fuwari blog template -->
# 快速上手 - AI 代码助理指南

下面是让 AI 编码代理快速在此仓库内高效工作的关键信息：架构要点、主要工作流、项目约定与示例引用。

## 一眼看清架构（大局）
- 静态站点：基于 Astro 构建，渲染由 `src/` 中的页面与组件驱动。
- 内容来源：Markdown 博客文章存放在 `src/content/posts/`（草稿 `draft.md` 也在此）。
- 组件与布局：UI 组件在 `src/components/`，布局在 `src/layouts/`，页面模板在 `src/pages/`。
- 扩展与插件：自定义 remark/rehype 插件位于 `src/pages/plugins/`（比如 `expressive-code/` 相关扩展在 `src/pages/plugins/expressive-code/`）。

## 关键文件（优先阅读）
- 项目配置： [package.json](package.json)、[astro.config.mjs](astro.config.mjs)、[tailwind.config.cjs](tailwind.config.cjs)
- 站点配置： [src/config.ts](src/config.ts)
- 内容目录： [src/content/posts/](src/content/posts/)
- 新文章脚本： [scripts/new-post.js](scripts/new-post.js)
- Markdown / 插件入口： [src/pages/plugins/](src/pages/plugins/)
- 样式与变量： [src/styles/](src/styles/)

## 主要开发命令（必知）
- `pnpm install` — 安装依赖（仓库使用 `pnpm`; `preinstall` 已强制设置）。
- `pnpm dev` — 本地开发服务器（默认端口 4321）。
- `pnpm build` — 生产构建；构建后会运行 `pagefind --site dist` 生成搜索索引（注意：`build` 包含额外步）。
- `pnpm preview` — 本地预览构建。
- `pnpm new-post <filename>` — 使用脚本在 `src/content/posts/` 下创建新文章。
- `pnpm format` / `pnpm lint` — 使用 Biome 进行格式化与检查（目标是 `./src`）。

## 项目特有约定与模式
- 前端框架：Astro + Svelte 组件。Svelte 组件位于 `src/components/` 的 `.svelte` 文件。
- Markdown 扩展：仓库集成了多个 remark/rehype 插件（见 `src/pages/plugins/`），因此不要直接替换全局 Markdown 逻辑——优先复用现有插件。
- 图片与资源：文章中引用相对图片（示例 frontmatter `image: ./cover.jpg`），构建时会被处理；若操作图片请注意 `sharp` 已作为依赖。
- 翻译/多语言：i18n 资源在 `src/i18n/`，站点默认语言在 `src/config.ts` 中配置。

## 修改内容或添加功能时的具体提示
- 新增页面/布局：遵循现有 `src/layouts/Layout.astro` 与 `src/layouts/MainGridLayout.astro` 的结构；优先复用 `PostCard.astro`、`PostPage.astro` 等组件。
- 新增 Markdown 行为：若要添加自定义解析器或指令，请在 `src/pages/plugins/` 中添加并在 `astro.config.mjs` 或相关管道处引入。
- 新文章：使用 `pnpm new-post my-post`（该脚本位于 `scripts/new-post.js`）确保 frontmatter 字段完整（参见 README 的 frontmatter 示例）。

## 常见陷阱（从代码可观测到的）
- 构建包含 `pagefind`：`pnpm build` 在成功构建后还会运行搜索索引命令，缺少 `pagefind` 可导致构建后期失败。
- 强制 pnpm：`preinstall` 使用 `only-allow pnpm`，CI 或本地若使用 npm/yarn 会失败。
- CSS 与 Tailwind：样式受 `tailwind.config.cjs` 与 `postcss.config.mjs` 影响，直接修改样式前确认这些配置。

## 调试建议与快速检查点
- 若页面空白或组件未渲染：先运行 `pnpm dev` 并观察终端错误；Astro 的编译错误通常会指明文件与行。
- 路由与动态页面：页面路由在 `src/pages/`（例如 `src/pages/posts/[...slug].astro`），调试动态路由时模拟不同路径查看 `params`。
- 类型检查：使用 `pnpm type-check`（运行 `tsc --noEmit`）来发现 TypeScript 类型问题。

## 例子（可直接模仿）
- 新文章 frontmatter 示例见 README 的 “Frontmatter of Posts”。
- 表格索引与搜索：`pnpm build` 完成后，`dist/` 下会包含 Pagefind 输出用于站内搜索。

## 如果需要更改 CI / 部署
- 默认演示部署为 Vercel（参考 README）；构建命令应与 `pnpm build` 保持一致并保证 `pagefind` 可用。

---
请检查本指南是否覆盖了你需要的细节，或指出任何不准确或不完整之处，我会据此迭代更新。
