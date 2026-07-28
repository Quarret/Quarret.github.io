# AstroPaper 博客 — Subagent 编排提示词

**日期：** 2026-07-28  
**用途：** 控制器（当前会话父 agent）按本文件调度 subagent，执行实现计划  
**工作区：** `/Users/aoko/Documents/code/MyBlog`  
**Spec（产品意图）：** `docs/superpowers/specs/2026-07-28-astropaper-blog-design.md`  
**Plan（实现细节，权威）：** `docs/superpowers/plans/2026-07-28-astropaper-blog.md`

---

## 0. 关于上下文是否干净（给用户 / 控制器）

**是的：每个新 dispatch 的 subagent 默认是干净上下文。**

- Subagent **不会**自动继承本会话的聊天历史、中间讨论或你未写入 prompt 的假设。
- 它只看到你在 **本次 dispatch 的 `prompt` 里提供的内容**，以及它自己随后用工具读到的文件。
- 因此控制器必须在每个 Task 的 dispatch 中显式给出：任务 brief、Global Constraints、依赖接口、报告路径；**不要假设** subagent「还记得」上一轮对话。
- 副作用（磁盘上的代码、`git` 提交、`docs/`）会保留；干净的是 **对话上下文**，不是工作区。

**控制器自己的上下文不干净**——父 agent 仍保有整段编排历史。进度请写入 ledger，避免压缩后重复执行已完成 Task。

---

## 1. 控制器必须遵守的流程

使用 skill：`superpowers:subagent-driven-development`。

对每个 Task（顺序执行，**禁止并行多个 implementer**）：

1. 记录 `BASE=$(git rev-parse HEAD)`（Task 开始前；空仓库则记 `BASE=EMPTY`）
2. 派发 **implementer**（只做本 Task；结束必须本地 commit，Task 6 除外的特殊规则见下）
3. 派发 **task reviewer**（spec 合规 + 代码质量，同一审查；给 diff/review package）
4. 若有 Critical/Important：派发 **fix** subagent → 再审查，直到通过
5. 在 ledger 标记完成：`.superpowers/sdd/progress.md`（该目录已被 gitignore）
6. 全部 Task 完成后：派发 **final whole-branch code reviewer**

**不要**在 Task 之间向用户询问「要不要继续」；仅在 `BLOCKED`、计划矛盾、或全部完成时停下。

---

## 2. Global Constraints（每个 subagent prompt 必须粘贴）

```
- 工作区：/Users/aoko/Documents/code/MyBlog
- 远程仓库名：Quarret.github.io；公网 https://Quarret.github.io
- site.url 配置值：https://Quarret.github.io/（带末尾 /）
- 禁止设置 Astro base 子路径
- site.lang / Astro i18n defaultLocale / 文件 src/i18n/lang/zh-CN.ts → 字面量均为 zh-CN
- 品牌：title "Aoko's Blog"；author "Aoko"；description "记录技术与生活"
- socials 仅一条：{ name: "github", url: "https://github.com/Quarret" }
- 包管理器：pnpm
- 不做：评论、统计、自定义域名、主题视觉重写、多语言路由
- 保留 docs/superpowers/ 与 .superpowers/（rsync 时 exclude）
- 允许在 main 分支开发与提交
- Task 1–5：每个 Task 结束必须本地 git commit（用 plan 中的 message）
- 禁止提交：.superpowers/、node_modules/、dist/、.astro/、public/pagefind
- 冲突时：实现细节以 plan 为准；产品意图以 spec 为准；仍冲突则 BLOCKED 问用户
- 验收：pnpm build + dist/curl；不要把人工浏览器当成唯一门禁
```

---

## 3. 本地 Git Commit 策略（维护友好）

| Task | Commit 时机 | 建议 message（与 plan 一致） |
|------|-------------|------------------------------|
| 1 脚手架 | `pnpm build` 通过后 | `chore: scaffold AstroPaper v6 as blog baseline` |
| 2 配置 | 配置改完且 build 通过后 | `chore: configure site identity for Quarret.github.io` |
| 3 中文 UI | `zh-CN.ts` 就绪且 build 抽查通过后 | `feat: add zh-CN UI strings for AstroPaper` |
| 4 内容 | 示例已删、欢迎帖/关于页就位、build 验收后 | `content: replace demos with Chinese welcome post and about page` |
| 5 CI | workflow + gitignore 就绪后 | `ci: add GitHub Pages deploy workflow` |
| 6 发布 | **不要求**新功能 commit；以 push 已有 commits 为主 | 仅当修复部署问题才追加 commit |

规则：

- Commit 前 `git status`：确认不包含 `.superpowers/`
- 使用 HEREDOC 写 message（见 plan）
- **不要** `--no-verify` 跳过 hooks（除非 hooks 损坏且已向用户报告）
- **本阶段默认 push 只在 Task 6**；Task 1–5 只本地 commit
- Task 1 首次 commit **之前**必须确保 `.gitignore` 含 `.superpowers/`

---

## 4. Code Review 要求（强制）

### 4.1 每 Task：Task Reviewer

审查目标（两份 verdict 都要有）：

1. **Spec / Plan 合规**
   - 本 Task Files 列表是否做完、有无越权改动
   - Global Constraints 有无违反（尤其 `base`、`zh-CN`、社交字段、禁止提交物）
2. **代码质量**
   - 是否按 plan 粘贴/修改，无无关重构
   - 配置与上游 API 一致（`socials[].url` 不是 `href`）
   - 危险操作（误删 `docs/superpowers`）一票否决

输出至少包含：`Spec: ✅/❌`、`Quality: Approved / Changes required`、Findings（Critical / Important / Minor）。

### 4.2 全部完成后：Final Code Review

审查范围：从第一个 scaffold commit 到 `HEAD` 的整段 diff，对照 spec §7.3 验收项与 plan Spec Coverage Checklist。

重点清单：

- [ ] 仅一篇欢迎帖 `welcome.md`，无英文示例帖
- [ ] 中文导航/UI（`zh-CN`）
- [ ] `astro-paper.config.ts` 品牌与 GitHub 社交正确
- [ ] 无 `base`；`site.url` 正确
- [ ] `.github/workflows/deploy.yml` 存在且合理
- [ ] `.gitignore` 含 `.superpowers/`
- [ ] 本地 git 历史按 Task 可分段回滚（至少 Task 1–5 各有 commit）

---

## 5. Implementer Dispatch 模板

```
你是 Implementer。只实现 Task N，不要做后续 Task。

先读（按顺序）：
1) 任务 brief 文件：<BRIEF_PATH>（控制器从 plan 抽出的 Task N 全文）
2) 如需核对产品意图：docs/superpowers/specs/2026-07-28-astropaper-blog-design.md 相关章节
3) 工作区现状（ls / git status / 相关文件）

## Global Constraints
<粘贴第 2 节约束块>

## 场景
本项目是空目录起步的 AstroPaper 中文个人博客，部署到 Quarret.github.io。
你当前负责任务链中的 Task N：<一句话>。
上游依赖：<Task N-1 已产生的接口/文件，若 N=1 写「无」>。

## 开工前
若 brief 有歧义、缺权限、或与磁盘现状冲突：先提问或报告 NEEDS_CONTEXT / BLOCKED，不要猜测。

## 你的工作
1. 严格按 brief 步骤执行（命令与文件内容以 brief/plan 为准）
2. 跑 brief 中的验证命令；失败则修复后再继续
3. Task 1–5：按 brief 做本地 git commit；Task 6：按 brief push/验收
4. 自检：对照 Global Constraints 与「有没有改到本 Task 以外的文件」
5. 把完整报告写入：<REPORT_PATH>
6. 回复父 agent 时只返回：status（DONE | DONE_WITH_CONCERNS | NEEDS_CONTEXT | BLOCKED）、
   commit SHA(s)、一句验证摘要、concerns（如有）

## 禁止
- 读取并执行整个 plan 里的其他 Task
- 引入评论/统计/自定义域名/主题大改
- 提交 .superpowers/ 或构建产物
- 跳过验证与 commit（Task 1–5）
```

---

## 6. Task Reviewer Dispatch 模板

```
你是 Task Reviewer（spec 合规 + 代码质量）。不要改代码。

输入文件：
- Brief：<BRIEF_PATH>
- Implementer 报告：<REPORT_PATH>
- Diff / review package：<REVIEW_PACKAGE_PATH>
- Global Constraints：<粘贴第 2 节>

审查方法：
1. 对照 brief 的 Files / Steps / Expected，逐项勾选
2. 对照 Global Constraints，查找违规
3. 质量：无关改动、错误 API、可能破坏 docs/superpowers 的操作
4. 不要要求重新跑 implementer 已报告且可信的同一测试；关注 diff 本身

输出：
- Spec: ✅ 或 ❌（列出 Missing / Extra）
- Quality: Approved 或 Changes required
- Findings：Critical / Important / Minor
- 对「diff 看不出、跨 Task」的项标 ⚠️ Cannot verify from diff（交给控制器裁决）
```

---

## 7. Task 专属注意（消歧）

| Task | 易错点 | 正确做法 |
|------|--------|----------|
| 1 | `rsync` 覆盖 `docs/`；把 `.superpowers` 提交进 git | exclude `docs` 与 `.superpowers`；commit 前确认 gitignore |
| 2 | `socials` 写成 `href` / `GitHub`；设置 `base` | 严格用 plan 中完整 config；无 `base` |
| 3 | 文件名 `zh.ts` 或 `zh_CN.ts`；只改 en.ts | 必须 `zh-CN.ts`，保留 `en.ts` 作 fallback |
| 4 | 欢迎帖文件名随意；删光 posts 后忘记新建 | 固定 `welcome.md`；URL `/posts/welcome/` |
| 5 | 漏 ignore `.superpowers/` | 确认 gitignore 条目存在 |
| 6 | `gh` 未登录仍报成功；RSS 路径写死错 | 失败则 BLOCKED；先 `find dist -iname '*rss*'` 再 curl |

**Task 3 中文不生效时的排查顺序：**  
`astro-paper.config.ts` 的 `lang` → `astro.config.ts` 的 `locales/defaultLocale` → `src/i18n/lang/zh-CN.ts` 是否存在 → 组件是否把 `site.lang` 传给 `useTranslations`。

---

## 8. 进度 Ledger

路径：`.superpowers/sdd/progress.md`（gitignore 内，不提交）

启动时先读；已标记 complete 的 Task **禁止重做**。

完成一行示例：

```
Task 1: complete (commits EMPTY..<sha7>, review clean)
Task 2: complete (commits <base7>..<head7>, review clean)
```

---

## 9. 完成定义（控制器对用户宣布 Done 的条件）

1. Plan Task 1–6 全部 complete（或 Task 6 因权限 BLOCKED 且已明确告知用户手动步骤）
2. 本地 Task 1–5 均有对应 commit
3. Final code review 无未处理 Critical/Important
4. 若 Task 6 成功：公网验收命令通过（欢迎帖标题可见）

---

## 10. 控制器启动清单（复制即用）

```
采用 Subagent-Driven Development 执行：
- Plan: docs/superpowers/plans/2026-07-28-astropaper-blog.md
- Spec: docs/superpowers/specs/2026-07-28-astropaper-blog-design.md
- Orchestrator: docs/superpowers/specs/2026-07-28-astropaper-blog-subagent-prompt.md

要求：
1. 每 Task 独立 implementer（干净上下文）+ task reviewer（spec+quality）
2. Task 1–5 必须本地 git commit
3. 全部完成后 final code review
4. 用 .superpowers/sdd/progress.md 记进度
5. 不要并行 implementer；不要跳过审查
```
