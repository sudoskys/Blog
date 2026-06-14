# Blog Project — sudoskys/Blog

## 项目概览

Hexo 静态博客，部署在 Vercel。主题：Stellaris / Volantis（双主题配置）。

## 关键路径

| 用途 | 路径 |
|-----|-----|
| 博文 | `source/_posts/<slug>.md` |
| Front matter 模板 | `scaffolds/post.md` |
| Hexo 主配置 | `_config.yml` |
| 主题配置 | `_config.stellaris.yml`, `_config.volantis.yml` |
| 部署配置 | `vercel.json` |

## 常用命令

```bash
pnpm install          # 安装依赖
pnpm run server       # 本地预览（hexo server）
pnpm run build        # 构建（hexo generate）
pnpm run clean        # 清理缓存（hexo clean）
```

## 写作约定

- 新博文放在 `source/_posts/`，文件名用小写连字符命名（slug 风格）
- 中文文章：front matter `original: true`，标题和 tags 用中文
- 英文文章：标题英文，tags 英文
- 含 Mermaid 图表时在 front matter 加 `mermaid: true`
- 作者署名格式：`**sudoskys**\nGitHub: [github.com/sudoskys](https://github.com/sudoskys)`

## Skills

- `blog-post`：博文选题、起草、审查。写/改/审博文时调用。
