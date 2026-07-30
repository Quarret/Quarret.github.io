# Aoko's Blog

基于 [AstroPaper](https://github.com/satnaing/astro-paper) 的个人博客，部署于 GitHub Pages。

## 开发

```bash
pnpm install
pnpm dev
```

## 写作

在 `src/content/posts/` 新建 Markdown / MDX 文件，填写 frontmatter 后本地预览即可。

站点配置见 `astro-paper.config.ts`。

## 构建与预览

```bash
pnpm build
pnpm preview
```

推送到 `main` 后，GitHub Actions 会自动部署。
