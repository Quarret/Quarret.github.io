# AstroPaper 个人博客 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 基于 AstroPaper v6 模板，在本地 `MyBlog` 定制出中文个人博客，并通过 GitHub Actions 部署到 `https://Quarret.github.io`。

**Architecture:** 克隆 AstroPaper v6 为独立仓库骨架；用 `astro-paper.config.ts` 配置站点元数据与功能开关；新增 `zh-CN` UI 文案；替换示例内容为一篇中文欢迎帖与中文关于页；用 GitHub Actions 将 `pnpm build` 产物发布到 GitHub Pages 用户主页站（无 `base` 子路径）。

**Tech Stack:** Astro 6、Tailwind CSS 4、TypeScript、Pagefind、pnpm、GitHub Pages + Actions

**Spec:** `docs/superpowers/specs/2026-07-28-astropaper-blog-design.md`

## Global Constraints

- 远程仓库名必须为 `Quarret.github.io`；公网 URL 为 `https://Quarret.github.io`（配置里 `site.url` 写 `https://Quarret.github.io/`）
- **禁止**设置 Astro `base` 子路径（用户主页站挂在根路径）
- 站点语言：`site.lang = "zh-CN"`；时区：`Asia/Shanghai`；i18n 文件必须为 `src/i18n/lang/zh-CN.ts`（三者字符串完全一致）
- 品牌占位：标题 `Aoko's Blog`；作者 `Aoko`；简介 `记录技术与生活`
- 社交链接仅保留 GitHub：`{ name: "github", url: "https://github.com/Quarret" }`（`name` 小写；字段名是 `url`）
- 包管理器使用 **pnpm**（与上游一致）
- 不引入评论、统计、自定义域名；不改主题视觉体系
- 保留已有 `docs/superpowers/` 与 `.superpowers/`，脚手架时不得覆盖丢失
- **允许在 `main` 分支开发**（用户主页站仓库惯例）；本阶段以本地仓库为主
- **每个 Task 1–5 结束后必须本地 git commit**（见各 Task Commit 步骤）；消息使用计划中的 HEREDOC 文案
- **禁止**提交 `.superpowers/`、`node_modules/`、`dist/`、`.astro/`、`public/pagefind`
- Task 6 需要 `gh` 已登录 Quarret 账户；失败则 `BLOCKED`，不得伪造部署成功
- Agent 验收优先用 `pnpm build` + 对 `dist/` / `curl` 的检查；不依赖人工打开浏览器（浏览器检查可作为补充，非门禁）
- 冲突裁决：**实现细节以本 plan 为准**；产品意图以 spec 为准；二者冲突时停下来问用户
- Subagent 编排：遵循 `docs/superpowers/specs/2026-07-28-astropaper-blog-subagent-prompt.md`

---

## File Structure

| 路径 | 职责 |
|---|---|
| `astro-paper.config.ts` | 站点 URL/标题/作者/语言/时区、功能开关、社交与分享链接 |
| `astro.config.ts` | Astro `site`、i18n locales、Markdown 插件（含 TOC collapse 标题） |
| `src/i18n/lang/zh-CN.ts` | 中文 UI 字符串（导航、分页、无障碍等） |
| `src/content/posts/welcome.md` | 唯一初始博文（中文欢迎帖） |
| `src/content/pages/about.md` | 中文关于页 |
| `.github/workflows/deploy.yml` | GitHub Pages 构建与部署 |
| `.gitignore` | 忽略 `node_modules/`、`dist/`、`.superpowers/` 等 |

删除：`src/content/posts/` 下全部上游示例帖（含 `_color-schemes/`、`_releases/`、`examples/` 及各 `.md`/`.mdx` 指南帖）。

---

### Task 1: 脚手架 — 导入 AstroPaper v6 并保留现有 docs

**Files:**
- Create: 工作区内 AstroPaper 全套源码（除下方保留项）
- Preserve: `docs/superpowers/**`、`.superpowers/**`
- Modify: 无（本任务只导入骨架）

**Interfaces:**
- Consumes: 无
- Produces: 可运行的 AstroPaper 默认站点；`package.json` 含 `pnpm` scripts：`dev` / `build` / `preview`

- [ ] **Step 1: 确认工作区现状**

Run:

```bash
cd /Users/aoko/Documents/code/MyBlog
ls -la
find docs .superpowers -type f 2>/dev/null | head -50
```

Expected: 已有 `docs/superpowers/specs/...` 与 `.superpowers/brainstorm/...`；尚无 `package.json` / `src/`。

- [ ] **Step 2: 将 AstroPaper 解压到临时目录（不污染当前 git 状态）**

Run:

```bash
cd /tmp
rm -rf astro-paper-scaffold
git clone --depth 1 https://github.com/satnaing/astro-paper.git astro-paper-scaffold
rm -rf astro-paper-scaffold/.git
```

Expected: `/tmp/astro-paper-scaffold` 含 `package.json`、`astro-paper.config.ts`、`src/`。

- [ ] **Step 3: 同步文件到 MyBlog，保留 docs 与 brainstorm**

Run:

```bash
cd /Users/aoko/Documents/code/MyBlog
rsync -a \
  --exclude '.git' \
  --exclude 'docs' \
  --exclude '.superpowers' \
  /tmp/astro-paper-scaffold/ ./
```

Expected: 根目录出现 `package.json`、`astro-paper.config.ts`、`src/`；`docs/superpowers/` 与 `.superpowers/` 仍在。

- [ ] **Step 4: 初始化独立 git 仓库，并先写入 `.gitignore` 忽略 `.superpowers/`**

Run:

```bash
cd /Users/aoko/Documents/code/MyBlog
git status 2>/dev/null || git init -b main
```

若 `.gitignore` 尚无 `.superpowers/`，在文件末尾追加：

```gitignore

# local brainstorm / agent artifacts
.superpowers/
```

（Task 5 会再次确认；此处提前追加是为了避免首次 `git add` 把 brainstorm 会话提交进去。）

Expected: 当前分支为 `main`；`grep -q '.superpowers' .gitignore` 成功。

- [ ] **Step 5: 安装依赖并验证默认模板可构建**

Run:

```bash
cd /Users/aoko/Documents/code/MyBlog
pnpm install
pnpm build
```

Expected: 安装成功；`pnpm build` 退出码 0；生成 `dist/`。

- [ ] **Step 6: Commit（本 Task 必做）**

```bash
cd /Users/aoko/Documents/code/MyBlog
git add -A
git status
# 确认：docs/superpowers 在暂存区；.superpowers / node_modules / dist 不在暂存区
git commit -m "$(cat <<'EOF'
chore: scaffold AstroPaper v6 as blog baseline

EOF
)"
```

Expected: 产生一次本地 commit；`git status` 干净（或仅剩未跟踪的可忽略项）。
---

### Task 2: 站点配置 — URL、品牌、功能、社交

**Files:**
- Modify: `astro-paper.config.ts`
- Modify: `astro.config.ts`（`i18n.locales` / `defaultLocale`；确认无 `base`）

**Interfaces:**
- Consumes: Task 1 的 AstroPaper 骨架
- Produces: `site.url = "https://Quarret.github.io/"`；`site.lang = "zh-CN"`；`socials` 仅 GitHub

- [ ] **Step 1: 用以下完整内容替换 `astro-paper.config.ts`**

```ts
import { defineAstroPaperConfig } from "./src/types/config";

export default defineAstroPaperConfig({
  site: {
    url: "https://Quarret.github.io/",
    title: "Aoko's Blog",
    description: "记录技术与生活",
    author: "Aoko",
    profile: "https://github.com/Quarret",
    ogImage: "default-og.jpg",
    lang: "zh-CN",
    timezone: "Asia/Shanghai",
    dir: "ltr",
  },
  posts: {
    perPage: 4,
    perIndex: 4,
    scheduledPostMargin: 15 * 60 * 1000,
  },
  features: {
    lightAndDarkMode: true,
    dynamicOgImage: true,
    showArchives: true,
    showBackButton: true,
    editPost: {
      enabled: true,
      url: "https://github.com/Quarret/Quarret.github.io/edit/main/",
    },
    search: "pagefind",
  },
  socials: [
    { name: "github", url: "https://github.com/Quarret" },
  ],
  shareLinks: [
    { name: "x", url: "https://x.com/intent/post?url=" },
    { name: "telegram", url: "https://t.me/share/url?url=" },
    { name: "mail", url: "mailto:?subject=See%20this%20post&body=" },
  ],
});
```

- [ ] **Step 2: 更新 `astro.config.ts` 的 i18n，并确认无 `base`**

将 `astro.config.ts` 中：

```ts
  i18n: {
    locales: ["en"],
    defaultLocale: "en",
    routing: {
      prefixDefaultLocale: false,
    },
  },
```

改为：

```ts
  i18n: {
    locales: ["zh-CN"],
    defaultLocale: "zh-CN",
    routing: {
      prefixDefaultLocale: false,
    },
  },
```

同时把 Markdown collapse 标题同时支持中英文 TOC，将：

```ts
[remarkCollapse, { test: "Table of contents" }],
```

改为：

```ts
[remarkCollapse, { test: /Table of contents|目录/ }],
```

确认 `defineConfig({...})` **没有** `base: "/something"` 字段。`site` 应继续为 `config.site.url`（即 `https://Quarret.github.io/`）。

- [ ] **Step 3: 验证配置可构建**

Run:

```bash
cd /Users/aoko/Documents/code/MyBlog
pnpm build
```

Expected: 退出码 0。（此时 UI 文案可能仍部分英文，直到 Task 3 完成。）

- [ ] **Step 4: Commit（本 Task 必做）**

```bash
git add astro-paper.config.ts astro.config.ts
git commit -m "$(cat <<'EOF'
chore: configure site identity for Quarret.github.io

EOF
)"
```

---

### Task 3: 中文 UI — 新增 `zh-CN` 文案

**Files:**
- Create: `src/i18n/lang/zh-CN.ts`
- Keep: `src/i18n/lang/en.ts`（作为 fallback，不删除）

**Interfaces:**
- Consumes: `site.lang = "zh-CN"`（Task 2）；`useTranslations(locale)` 按文件名 `zh-CN` 查找
- Produces: 导航/页脚/分页/搜索等 UI 显示中文

- [ ] **Step 1: 创建 `src/i18n/lang/zh-CN.ts`（完整内容）**

```ts
import type { UIStrings } from "../types";

export default {
  nav: {
    home: "首页",
    posts: "文章",
    tags: "标签",
    about: "关于",
    archives: "归档",
    search: "搜索",
  },
  post: {
    publishedAt: "发布于",
    updatedAt: "更新于",
    sharePostIntro: "分享本文：",
    sharePostOn: "分享到 {{platform}}",
    sharePostViaEmail: "通过邮件分享本文",
    tagLabel: "标签",
    backToTop: "回到顶部",
    goBack: "返回",
    editPage: "编辑本页",
    previousPost: "上一篇",
    nextPost: "下一篇",
  },
  pagination: {
    prev: "上一页",
    next: "下一页",
    page: "页",
  },
  home: {
    socialLinks: "社交链接",
    featured: "精选",
    recentPosts: "最近文章",
    allPosts: "全部文章",
  },
  footer: {
    copyright: "版权所有",
    allRightsReserved: "保留所有权利。",
  },
  pages: {
    tagTitle: "标签",
    tagDesc: "带有该标签的全部文章",
    tagsTitle: "标签",
    tagsDesc: "文章中使用过的全部标签。",
    postsTitle: "文章",
    postsDesc: "我发布过的全部文章。",
    archivesTitle: "归档",
    archivesDesc: "全部文章归档。",
    searchTitle: "搜索",
    searchDesc: "搜索任意文章…",
  },
  a11y: {
    skipToContent: "跳到正文",
    openMenu: "打开菜单",
    closeMenu: "关闭菜单",
    toggleTheme: "切换主题",
    searchPlaceholder: "搜索文章…",
    noResults: "没有找到结果",
    goToPreviousPage: "上一页",
    goToNextPage: "下一页",
  },
  notFound: {
    title: "404 未找到",
    message: "页面不存在",
    goHome: "返回首页",
  },
} satisfies UIStrings;
```

- [ ] **Step 2: 构建并核对中文 UI 字符串已进入产物（agent 门禁，不依赖浏览器）**

Run:

```bash
cd /Users/aoko/Documents/code/MyBlog
pnpm build
# 在产物中抽查关键中文 UI（路径随 Astro 输出可能略有差异，至少一个命中即可）
grep -R --include='*.html' -l '最近文章\|全部文章\|搜索文章' dist | head
test -f src/i18n/lang/zh-CN.ts
```

Expected: `pnpm build` 成功；`zh-CN.ts` 存在；`dist` 内 HTML 出现上述中文之一。

若 UI 仍全英文：检查 `site.lang`、`astro.config.ts` i18n、`zh-CN.ts` 文件名三者是否均为字面量 `zh-CN`，以及组件是否把 `config.site.lang` 传给 `useTranslations`。

- [ ] **Step 3: Commit（本 Task 必做）**

```bash
git add src/i18n/lang/zh-CN.ts
git commit -m "$(cat <<'EOF'
feat: add zh-CN UI strings for AstroPaper

EOF
)"
```

---

### Task 4: 内容 — 清空示例帖，写入欢迎帖与关于页

**Files:**
- Delete: `src/content/posts/` 下除新欢迎帖外的全部上游示例（文件与子目录）
- Create: `src/content/posts/welcome.md`
- Modify: `src/content/pages/about.md`

**Interfaces:**
- Consumes: frontmatter 必填 `title`、`description`、`pubDatetime`
- Produces: 线上仅 1 篇博文；关于页为中文短文

- [ ] **Step 1: 删除上游示例内容**

Run:

```bash
cd /Users/aoko/Documents/code/MyBlog
# 清空 posts 目录内全部内容（保留目录本身）
find src/content/posts -mindepth 1 -maxdepth 1 -exec rm -rf {} +
ls src/content/posts
```

Expected: `src/content/posts` 为空目录。

- [ ] **Step 2: 创建欢迎帖 `src/content/posts/welcome.md`**

```md
---
title: 欢迎来到 Aoko's Blog
author: Aoko
pubDatetime: 2026-07-28T02:00:00Z
featured: true
draft: false
tags:
  - 博客
description: 这是本站的第一篇文章，介绍如何开始写作。
---

你好，这里是 **Aoko's Blog**。

本站基于 [AstroPaper](https://github.com/satnaing/astro-paper) 搭建，通过 GitHub Pages 发布。

## 如何写新文章

1. 在 `src/content/posts/` 新建 Markdown 或 MDX 文件
2. 填写 frontmatter（至少包含 `title`、`description`、`pubDatetime`）
3. 本地运行 `pnpm dev` 预览
4. 推送到 `main` 后，GitHub Actions 会自动部署

祝写作愉快。
```

- [ ] **Step 3: 用以下内容替换 `src/content/pages/about.md`**

```md
---
title: "关于"
description: "关于 Aoko 与这个博客。"
---

我是 **Aoko**，这里记录技术与生活。

- GitHub： [Quarret](https://github.com/Quarret)
- 本站： [Aoko's Blog](https://Quarret.github.io)
```

- [ ] **Step 4: 构建并验收内容（agent 门禁）**

Run:

```bash
cd /Users/aoko/Documents/code/MyBlog
pnpm build
# 欢迎帖与关于页
grep -R --include='*.html' -l "欢迎来到 Aoko's Blog" dist | head
grep -R --include='*.html' -l '记录技术与生活\|关于 Aoko' dist | head
# 确认示例英文指南帖标题不应出现
! grep -R --include='*.html' -E 'Adding new posts in AstroPaper|How to configure AstroPaper' dist
# 欢迎帖路由文件应存在（Astro 静态输出）
find dist -type f -path '*welcome*' | head
# Pagefind 索引应生成（搜索功能）
test -d dist/pagefind -o -d public/pagefind && echo pagefind_ok
```

Expected:

1. 产物含欢迎帖标题
2. 关于页相关中文出现
3. 无上游英文示例帖标题
4. 存在 welcome 相关输出路径
5. Pagefind 目录存在（具体在 `dist/pagefind` 或构建脚本复制位置以实际为准，但必须有索引）

可选补充：`pnpm preview` 后 `curl -s http://127.0.0.1:4321/posts/welcome/ | grep 欢迎`（若端口被占可改用 preview 打印的端口）。

- [ ] **Step 5: Commit（本 Task 必做）**

```bash
git add src/content/posts src/content/pages/about.md
git commit -m "$(cat <<'EOF'
content: replace demos with Chinese welcome post and about page

EOF
)"
```

---

### Task 5: 部署流水线 — GitHub Actions + gitignore

**Files:**
- Create: `.github/workflows/deploy.yml`
- Modify: `.gitignore`

**Interfaces:**
- Consumes: `pnpm build` 产出 `dist/`
- Produces: push `main` 后自动部署到 GitHub Pages

- [ ] **Step 1: 创建 `.github/workflows/deploy.yml`**

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup pnpm
        uses: pnpm/action-setup@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: pnpm

      - name: Install dependencies
        run: pnpm install

      - name: Build
        run: pnpm build

      - name: Upload Pages artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: dist

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

- [ ] **Step 2: 更新 `.gitignore`（确认 `.superpowers/` 已忽略）**

若 Task 1 已追加则跳过追加，但仍须确认以下条目存在：

```gitignore
dist/
.astro/
node_modules/
public/pagefind

# local brainstorm / agent artifacts
.superpowers/
```

- [ ] **Step 3: 本地再构建确认 workflow 不依赖本机私货**

Run:

```bash
cd /Users/aoko/Documents/code/MyBlog
pnpm build
test -d dist && echo "dist ok"
test -f .github/workflows/deploy.yml && echo "workflow ok"
```

Expected: `dist ok` 与 `workflow ok`。

- [ ] **Step 4: Commit（本 Task 必做）**

```bash
git add .github/workflows/deploy.yml .gitignore
git commit -m "$(cat <<'EOF'
ci: add GitHub Pages deploy workflow

EOF
)"
```

---

### Task 6: 发布 — 创建远程仓库并验收公网站点

**Files:**
- 无本地新文件（远程仓库与 GitHub Settings）

**Interfaces:**
- Consumes: 本地 `main` 已包含 Task 1–5 成果
- Produces: `https://Quarret.github.io` 可访问

- [ ] **Step 1: 在 GitHub 创建空仓库 `Quarret.github.io`**

Run（需已登录 `gh`，账号为 Quarret）：

```bash
gh repo create Quarret/Quarret.github.io --public --description "Aoko's Blog" --source=/Users/aoko/Documents/code/MyBlog --remote=origin --push
```

若仓库已存在或 `--push` 失败，改用：

```bash
cd /Users/aoko/Documents/code/MyBlog
git remote remove origin 2>/dev/null || true
git remote add origin git@github.com:Quarret/Quarret.github.io.git
git push -u origin main
```

Expected: GitHub 上可见 `Quarret/Quarret.github.io`，默认分支 `main`。

- [ ] **Step 2: 启用 GitHub Pages（Actions 源）**

在仓库网页：Settings → Pages → Build and deployment → Source → **GitHub Actions**。

或尝试：

```bash
gh api -X PUT repos/Quarret/Quarret.github.io/pages -f build_type=workflow || true
```

Expected: Pages 源为 GitHub Actions。

- [ ] **Step 3: 等待 Actions 成功**

Run:

```bash
gh run list --repo Quarret/Quarret.github.io --limit 5
gh run watch --repo Quarret/Quarret.github.io
```

Expected: 最近一次 `Deploy to GitHub Pages` 为 success。

- [ ] **Step 4: 公网验收（对照 spec §7.3）**

先确认 RSS 实际路径（AstroPaper 可能是 `/rss.xml`）：

```bash
find dist -iname '*rss*' | head
```

再对公网执行（若 RSS 路径不同，用实际路径替换）：

```bash
curl -sI https://Quarret.github.io/ | head -n 5
curl -sI https://Quarret.github.io/posts/welcome/ | head -n 5
curl -sI https://Quarret.github.io/about/ | head -n 5
curl -sI https://Quarret.github.io/archives/ | head -n 5
curl -sI https://Quarret.github.io/rss.xml | head -n 5
curl -s https://Quarret.github.io/ | grep -o "欢迎来到 Aoko's Blog" | head -n 1
```

Expected: 主要 URL 返回 200（或 301/302 后 200）；首页 HTML 含欢迎帖标题。

若 Actions 失败或 `gh` 未登录：报告 `BLOCKED`，附上错误原文，不要宣称部署成功。

---

## Spec Coverage Checklist

| Spec 要求 | 对应 Task |
|---|---|
| AstroPaper 模板定制 | Task 1 |
| GitHub Pages 用户主页站、无 `base` | Task 2、Task 5、Task 6 |
| 中文为主（配置 + UI） | Task 2、Task 3 |
| 标准功能：搜索/RSS/SEO/归档/暗色 | Task 2（features 开关）+ 上游默认能力 |
| 品牌占位与仅 GitHub 社交 | Task 2 |
| 清空示例 + 中文欢迎帖 + 关于页 | Task 4 |
| Actions 部署 + `.gitignore` 含 `.superpowers/` | Task 5 |
| 公网验收 | Task 6 |
| 不做评论/统计/自定义域名 | 全计划均未引入 |

---

## Self-Review Notes

- 无 TBD/TODO 占位步骤；配置与文案均为完整可粘贴内容
- i18n 文件名 `zh-CN.ts` 与 `site.lang` / Astro `defaultLocale` 三者一致
- `socials[].name` 使用上游图标名 `github`（小写），字段为 `url`
- 脚手架用 `rsync --exclude docs --exclude .superpowers`，避免冲掉已有 spec/plan
- Task 1 在首次 commit 前即忽略 `.superpowers/`，避免把 brainstorm 会话提交进库
- Agent 验收以 `pnpm build` + `dist`/`curl` 为准，不把门禁绑在人工浏览器操作上
- 本地 commit 为 Task 1–5 硬性要求；远程 push 仅在 Task 6
