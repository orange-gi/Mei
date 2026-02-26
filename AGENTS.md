# AGENTS.md

## Cursor Cloud specific instructions

This is a **documentation-only repository** — a curated collection of AI project analyses written in Markdown. There is no source code, no backend/frontend services, no databases, and no automated test suites.

### Project structure

- Root-level `*.md` files: analyses of AI projects (Browser-Use, ClawdBot, DeepDiagram, FlashMLA, etc.)
- `docs/`: additional project analyses organized in subdirectories (FossFLOW, GitNexus, graphiti, nanobot, pentagi, whatsapp-web.js, etc.)
- `compare/`: comparison articles (e.g., Pathway vs Kafka, PostgreSQL vs Redis)
- Each project typically has three analysis files: 介绍 (introduction), 架构 (architecture with Mermaid diagrams), 提示词 (prompt analysis)

### Development tools

- **Linting:** `markdownlint '*.md' 'docs/**/*.md' 'compare/**/*.md'` — reports Markdown style issues (the existing docs have many lint warnings; this is pre-existing).
- **Preview site:** `mkdocs serve --dev-addr 0.0.0.0:8000` — serves the docs at http://localhost:8000 with Mermaid diagram rendering via MkDocs Material theme. Config is in `mkdocs.yml`.
- **Build static site:** `mkdocs build` — outputs to `_site/`.

### Gotchas

- The MkDocs setup uses a `site_content/` directory containing symlinks to the root-level and `docs/`/`compare/` Markdown files, because MkDocs requires `docs_dir` to be a child of the config file location.
- Markdown files contain Chinese content; make sure editors/tools support UTF-8.
- Many Markdown files contain Mermaid diagram blocks (` ```mermaid `) — the MkDocs Material theme renders these natively.
