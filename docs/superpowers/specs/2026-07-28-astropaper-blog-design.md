# AstroPaper 个人博客设计规格

**日期：** 2026-07-28  
**状态：** 已确认（可执行）  
**本地目录：** `/Users/aoko/Documents/code/MyBlog`  
**远程仓库（计划）：** `Quarret/Quarret.github.io`  
**实现计划：** `docs/superpowers/plans/2026-07-28-astropaper-blog.md`  
**Subagent 编排提示词：** `docs/superpowers/specs/2026-07-28-astropaper-blog-subagent-prompt.md`

---

## 1. 目标

搭建一个可通过 GitHub 仓库自动部署的个人博客网站：

- 直接基于 **AstroPaper** 官方模板定制（非从零仿写）
- 部署到 **GitHub Pages** 用户主页站：`https://Quarret.github.io`
- 站点内容与界面文案以 **中文** 为主
- 第一版功能达到「标准版」：列表/详情、标签、关于、暗色模式、全文搜索、RSS、SEO/OG、归档

---

## 2. 已确认决策

| 项 | 选择 |
|---|---|
| 模板策略 | A — 基于 AstroPaper 模板 fork/克隆后定制 |
| 托管 | A — GitHub Pages |
| 主语言 | A — 中文为主 |
| 功能范围 | B — 标准版（含搜索、RSS、SEO、归档） |
| 站点品牌（占位） | 标题 `Aoko's Blog`；作者 `Aoko`；简介 `记录技术与生活` |
| Pages 形态 | A — 用户主页站 `https://Quarret.github.io` |
| GitHub 用户名 | `Quarret` |
| 初始内容 | A — 清空英文示例文，仅留 1 篇中文欢迎帖 |
| 社交链接 | A — 仅 GitHub：`https://github.com/Quarret` |
| 落地方式 | A — 本地克隆 AstroPaper → 定制 → 推送到 `Quarret.github.io` |

---

## 3. 架构

基于 **AstroPaper v6** 的静态站点：Markdown/MDX 内容经 Astro 构建为静态资源，由 GitHub Actions 发布到 GitHub Pages。

```
作者编写 Markdown/MDX
        ↓
pnpm build
  · Astro 生成 HTML/CSS/JS
  · Pagefind 生成搜索索引
        ↓
GitHub Actions（push 到 main 触发）
        ↓
GitHub Pages
        ↓
https://Quarret.github.io
```

### 3.1 技术栈（跟随上游 AstroPaper v6）

- Astro 6
- Tailwind CSS 4
- TypeScript
- Pagefind（全文搜索，构建时索引）
- 包管理器：与上游一致，优先 **pnpm**

### 3.2 部署形态

- 仓库名：`Quarret.github.io`
- `site.url`：`https://Quarret.github.io/`（末尾斜杠与 AstroPaper 上游示例一致）
- **不设置 `base`**（用户主页站挂在根路径，无需子路径前缀）
- Pages 源：GitHub Actions（非 `docs/` 或 branch 直接托管）
- 开发分支：允许并应在本地 **`main`** 上工作（用户主页站惯例；非 feature branch）

### 3.3 本地与远程命名

- 本地工作区可继续使用目录名 `MyBlog`
- 远程仓库名为 `Quarret.github.io`（GitHub 用户主页站硬性要求）
- 二者不一致可接受，不影响构建与部署

---

## 4. 站点结构与定制范围

### 4.1 保留的 AstroPaper 能力（第一版启用）

- 首页（近期文章）
- 文章列表（分页）与文章详情
- 标签列表与标签归档
- 归档页
- 关于页
- 暗色 / 亮色模式
- 全文搜索（Pagefind）
- RSS
- Sitemap、canonical、OG / SEO meta（含动态 OG 若上游默认开启则保持）

### 4.2 需要定制的部分

| 区域 | 动作 |
|---|---|
| `astro-paper.config.ts` | 写入站点 URL、标题、描述、作者、`lang: "zh-CN"`、时区、社交链接 |
| 导航 / UI 文案 | 中文化（首页、文章、标签、关于、搜索、归档等） |
| `src/content/posts/` | 删除上游英文示例帖；新增唯一帖 `src/content/posts/welcome.md`（URL 预期 `/posts/welcome/`） |
| `src/content/pages/about.md` | 改为中文关于页（简短自我介绍即可） |
| `.github/workflows/deploy.yml` | 按 AstroPaper / Astro 官方 GitHub Pages 流程配置 |
| `public/` 静态资源 | 可暂留默认 favicon / OG；后续可替换，不阻塞第一版 |
| `.gitignore` | 确保包含 `node_modules/`、`dist/`、`.superpowers/` 等 |

### 4.3 明确不改动（YAGNI）

- 不重写主题视觉体系（保持 AstroPaper 默认外观）
- 不引入评论（Giscus 等）
- 不引入访问统计（Umami / GA 等）
- 不配置自定义域名（第一版）
- 不做多语言路由 / i18n 切换（中文为主即可；上游若有英文 UI key，替换为中文字符串）
- 不从其他博客平台迁移历史文章

---

## 5. 配置要点

主配置文件：项目根目录 `astro-paper.config.ts`（AstroPaper v6 统一配置）。

**权威取值以实现计划 Task 2 的完整 `astro-paper.config.ts` 为准。** 关键字段约定：

```ts
site: {
  url: "https://Quarret.github.io/", // 注意末尾 /
  title: "Aoko's Blog",
  description: "记录技术与生活",
  author: "Aoko",
  profile: "https://github.com/Quarret",
  lang: "zh-CN",                      // 必须与 src/i18n/lang/zh-CN.ts 文件名一致
  timezone: "Asia/Shanghai",
}
socials: [
  { name: "github", url: "https://github.com/Quarret" }, // name 小写 = icons 文件名；字段是 url 不是 href
]
```

`astro.config.ts` 中：

- `site` 取自 `config.site.url`（即 `https://Quarret.github.io/`）
- `i18n.locales` / `defaultLocale` 均为 `"zh-CN"`
- **禁止**设置 `base` 子路径

---

## 6. 内容策略

### 6.1 欢迎帖

- **固定路径：** `src/content/posts/welcome.md`（不要另起文件名，避免验收 URL 漂移）
- **预期 URL：** `/posts/welcome/`
- 语言：中文
- 内容：简短说明本站用途、如何开始写新文章（指向 `src/content/posts/`）
- frontmatter：标题、描述、发布日期、作者、标签至少含 `博客`（正文见实现计划 Task 4）

### 6.2 关于页

- 中文短文：作者名、一句话简介、GitHub 链接
- 不需要完整简历

### 6.3 写作约定（给后续自己用）

- 新文章放入 `src/content/posts/`
- 使用上游规定的 frontmatter 字段
- 本地：`pnpm dev` 预览；`pnpm build` 验证含搜索索引的生产构建

---

## 7. 部署与运维

### 7.1 CI/CD

- 工作流文件：`.github/workflows/deploy.yml`
- 触发：`push` 到 `main`，以及 `workflow_dispatch`
- 步骤概要：checkout → 启用 pnpm → Node 安装依赖 → `pnpm build` → 上传 Pages artifact → `deploy-pages`
- 仓库 Settings → Pages → Source 设为 **GitHub Actions**

### 7.2 发布流程（日常）

1. 本地写文章或改配置
2. 提交并 push 到 `main`
3. Actions 构建成功后，站点自动更新

### 7.3 验收标准（第一版完成定义）

- [ ] 本地 `pnpm install` / `pnpm dev` / `pnpm build` 成功
- [ ] 打开本地预览：中文导航、欢迎帖、关于页、搜索、暗色模式可用
- [ ] 远程仓库为 `Quarret/Quarret.github.io`，Actions 部署成功
- [ ] 公网可访问 `https://Quarret.github.io`，RSS 与主要页面 200
- [ ] 无上游英文示例博客帖残留

---

## 8. 实现与质量门禁

详细步骤见实现计划；编排与审查见 Subagent 提示词。摘要：

1. 脚手架导入 AstroPaper v6（保留 `docs/superpowers/`）→ **本地 commit**
2. 站点配置 + i18n locales → **本地 commit**
3. 新增 `zh-CN` UI 文案 → **本地 commit**
4. 清空示例帖，写入欢迎帖/关于页 → **本地 commit**
5. GitHub Actions + `.gitignore` → **本地 commit**
6. 创建远程 `Quarret.github.io`、启用 Pages、push、按 §7.3 验收

**质量要求：**

- Task 1–5 每个 Task 结束后必须有一次本地 git commit（便于回滚与审查）
- 每个 Task 完成后做 **spec 合规审查 + 代码质量审查**；全部 Task 结束后做一次整分支 code review
- **不要**把 `.superpowers/` 提交进仓库
- Task 6 若缺 `gh` 登录或 Pages 权限，标记 BLOCKED 交还用户，禁止猜测绕过

---


## 9. 风险与注意点

| 风险 | 处理 |
|---|---|
| 用户主页站仓库名必须精确为 `Quarret.github.io` | 远程按此命名；本地目录名可不改 |
| 若误设 `base` 为子路径，资源 404 | 主页站明确不设 `base` |
| Pagefind 仅在 `pnpm build` 后完整可用 | 验收以 build/preview 与线上为准，不只看 dev |
| 上游 AstroPaper 大版本变更 | 第一版钉在克隆时的 v6 版本；不主动追新 |
| GitHub Pages 需公开仓库（免费档）或符合账户 Pages 权限 | 实现前确认 Quarret 账户可启用 Pages |

---

## 10. 范围外（第二期可选）

- Giscus 评论
- 访问统计
- 自定义域名
- 替换 favicon / 默认 OG 图为品牌素材
- 从旧博客批量迁文
- 多语言切换

---

## 11. 决策摘要

用 **AstroPaper v6 模板 + GitHub Actions → GitHub Pages 用户主页站** 交付一个中文个人博客；配置与文案本地化，内容从一篇中文欢迎帖干净起步；社交仅挂 GitHub；视觉保持模板默认，不做评论/统计/自定义域名。
