---
title: Tech Stack
---

> 2026-09 Astro 7.3.1 最佳实践迁移记录

## 当前架构

- **框架与部署**：Astro 7.3.1、`output: 'server'`、`@astrojs/vercel`。依赖请求 locale 或远程数据的页面按请求渲染；本地 Markdown/MDX 由 Content Layer 在构建阶段解析。
- **多语言**：使用 Astro 官方 i18n，配置 `locales: ['zh', 'en']`、默认中文和 `routing: 'manual'`。Middleware 负责 `/` 与 `/en/...` 的 URL 保持、locale 解析和内部路径重写，因此现有链接不变。
- **链接与 SEO**：页面链接、canonical、hreflang 和语言切换统一使用 `astro:i18n` 的 `getRelativeLocaleUrl()` 封装；内部预取使用 `data-astro-prefetch`。
- **Markdown**：Astro 7 默认使用 Sätteri。本项目依赖多个 remark 插件，因此显式安装并配置 `@astrojs/markdown-remark` 的 Unified processor，避免升级后插件链行为变化。
- **远程数据**：`src/services/api.ts` 统一处理 URL、Bearer 鉴权、HTTP 状态、JSON、超时和错误；`src/live.config.ts` 使用 `defineLiveCollection()` 管理 Moments、Reviews、Places。分页元数据保留在类型化的 `PageResult<T>` 中，不伪装成 Live Entry。
- **写入接口**：Studio 使用 Astro Actions，输入由 Zod 校验，Action 层复用 Studio 鉴权，并把上游非 2xx 响应转换为明确的 `ActionError`。
- **环境变量**：服务端使用 `astro:env/server` 读取 `API_URL`、`API_SECRET`、`STUDIO_SECRET`、`AMAP_KEY`；浏览器端仅使用 `astro:env/client` 的公开 `MAPBOX_TOKEN`。变量示例见 `.env.example`。

## URL 与请求流程

Middleware 的顺序固定为：

1. Studio 鉴权：保护 `/studio` 和 `/en/studio` 及其 Action 请求。
2. locale 解析/重写：保存原始路径和 URL 到 `App.Locals`，再将英文路径重写为内部无 locale 前缀的路径。
3. Markdown 内容协商：对带有 `Accept: text/markdown` 的普通页面返回 Markdown；raw、Studio、RSS 等端点保持原协议。

## 为什么使用 Obsidian

我选择 Obsidian 的原因可能只有这一个：可能是笔记软件最好用的 VIM 键位绑定，相关插件也丰富。

- `im-control` 在 `Normal` 和 `Insert` 模式下切换 IME。
- `vimrc-support` 支持自定义 VIM 快捷键，例如在 wikilink 上使用 `gd` 直接打开。

## wikilink 与 graph view

项目保留基于 remark 的 wikilink 扩展，并针对 [issue#1059](https://github.com/datopian/portaljs/issues/1059) 做了兼容处理。Graph view 使用 [d3-force](https://d3js.org/d3-force) 表达页面之间的 backlinks 关系，支持节点拖拽、缩放和关联高亮。

## 编辑与发布

文章内容位于 `src/content` 子模块，通过 Obsidian 脚本提交后触发自动构建。页面中的编辑入口使用 Obsidian URI Schema，在本地调试时可以直接打开对应笔记：

```text
obsidian://open?vault=my%20vault&file=my%20note
```

