# AGENTS.md

## Cursor Cloud specific instructions

这是一个**纯文档仓库**（"每天了解一个AI项目"），包含 AI 项目分析文章，无源代码。

### 目录结构

- `docs/项目名/` — 每个项目一个子目录，包含三种分析文档：
  - `项目名-介绍.md` — Why-What-How 介绍
  - `项目名-架构.md` — Mermaid 架构图分析
  - `项目名-提示词.md` — Prompt 提示词分析
- `docs/compare/` — 项目对比文章
- `docs/index.md` — MkDocs 站点首页
- `README.md` — 项目总览及目录索引

### 开发工具

- **Lint:** `markdownlint 'docs/**/*.md'`
- **预览站点:** `mkdocs serve --dev-addr 0.0.0.0:8000` — http://localhost:8000，Mermaid 图自动渲染
- **构建静态站点:** `mkdocs build` — 输出到 `_site/`

### 添加新项目

1. 创建 `docs/项目名/` 目录
2. 添加 `项目名-介绍.md`、`项目名-架构.md`、`项目名-提示词.md`
3. 在 `mkdocs.yml` 的 `nav` 部分添加对应条目
4. 在 `README.md` 的项目表格中添加一行
