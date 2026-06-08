---
name: local-skills-viewer-builder
description: Use when a user wants to build a local HTML viewer for AI agent skills, commands, plugins, or tool instructions across Codex, Claude Code, ccswitch, OpenClaw, custom folders, or similar local skill systems. Generate a single-file Chinese-friendly HTML viewer plus a local update script, while adapting scan paths and entrypoints to the user's operating system instead of hardcoding one machine's directories.
---

# Local Skills Viewer Builder

Use this skill to recreate the reference local skills viewer: a single HTML file that scans local skill files, groups them by purpose and source, supports search and filtering, and opens full details in a modal.

This is a build recipe, not a finished app package. The target machine, OS, skill roots, and agent tool may differ.

## How To Use

| Invocation | What to do |
| --- | --- |
| default | Build the local skills viewer in the user's chosen folder. Detect scan roots, then generate the viewer and update script. |
| audit | Review an existing viewer for wrong scan roots, bad categorization, weak summaries, or UI regressions. |
| extend | Add support for another skill system, folder layout, or metadata format without breaking the current viewer behavior. |
| package | Turn a working viewer into a reusable skill or methodology package. |

## Output Contract

Generate these files in the target folder:

- `generate_skill_dashboard.py` or equivalent generator script
- `本地Skills查看器.html`
- `tokens.css` if shared design tokens are used
- one update entrypoint for the user's OS:
  - macOS: `更新本地Skills查看器.command`
  - Windows: `更新本地Skills查看器.bat` or `.ps1`
  - Linux: `更新本地Skills查看器.sh`
- optional `.hallmark/log.json` only when the project already uses it or the user explicitly asks

Do not generate legacy duplicate HTML files unless the user explicitly asks for compatibility.

## Environment Discovery

Before implementation, determine:

- OS: macOS, Windows, Linux, or mixed
- runtime: prefer Python when available; otherwise use the user's existing local runtime
- skill systems to scan: Codex, Claude Code, ccswitch, OpenClaw, custom folders, plugin caches, project folders
- file naming conventions: `SKILL.md`, `skill.md`, or equivalent instruction files
- output language: Chinese, bilingual, or English

Suggested defaults when present:

- `~/.codex/skills`
- `~/.codex/plugins/cache`
- `~/.agents/skills`
- `~/.cc-switch/skills`

Never permanently hardcode another person's absolute path. Support extra roots through config or environment variables such as `EXTRA_SKILL_ROOTS`.

## Generator Requirements

The generator must:

1. Read configurable scan roots.
2. Recursively find skill files.
3. Parse frontmatter when available.
4. Extract at least:
   - skill name
   - description
   - source path
   - source label
   - category
   - official usage section
5. Fall back safely:
   - name from parent folder
   - description from first useful paragraph
6. Deduplicate by skill name while preserving all sources.
7. Prefer the longest or most specific description when duplicates disagree.
8. Generate one self-contained HTML file with embedded data.
9. Update generation time on every run.
10. Never modify source skill files.

## Usage Extraction

For each skill, look for the first heading matching:

- `How to use`
- `Usage`
- `When to use`
- `Invocation`
- `用法`
- `使用`

Extract until the next heading of the same or higher level.

Preferred parsing order:

1. Markdown table rows such as `Invocation | What it does`
2. Bullet or numbered lists
3. First useful paragraph
4. Fallback to the official description

Normalize Markdown lightly:

- remove inline-code ticks
- unwrap links to link text
- remove bold/italic markers
- collapse repeated whitespace

## Classification And Summary Rules

Use transparent heuristic rules. Do not rely only on first-match ordering for broad keywords like `skill`, `plugin`, or `codex`.

Recommended Chinese categories:

- 产品设计与前端
- 数据分析与报表
- 文档 / 表格 / 演示
- 浏览器与网页控制
- Codex 技能管理
- 飞书 / Lark
- 创意生产
- iOS / Swift
- 投资研究
- OpenCLI 与搜索
- 知识库 / 个人工具
- 邮箱 / 团队协作
- 其他

Source labels should come from detected root ownership, for example:

- `Codex 个人技能`
- `Codex 插件缓存`
- `Claude Code 技能`
- `ccswitch 技能`
- `OpenClaw 技能`
- `项目内自定义技能`
- `额外扫描目录`

Chinese card summaries must be generated from that specific skill's `name`, `description`, and usage when possible.

Required behavior:

- Do not use category-level template text as the first choice.
- Skills in the same category should have different Chinese summaries when their source descriptions differ.
- Category fallback is only for missing or unusable source descriptions.

## UI Contract

The generated HTML should match the reference viewer in structure and interaction. Do not redesign it into a landing page, docs site, SaaS dashboard, or generic template unless the user explicitly asks.

### Page Shape

Use one single-page workbench:

```text
main.page
├── section.topbar
├── section.stats
├── section.workbench
│   ├── aside.panel.sidebar
│   └── section.panel.main
└── div#detailModal
```

Topbar must include:

- eyebrow: `本地 AI 工具 Skills 速查`
- title: `本地 Skills 查看器`
- subhead explaining Codex / Claude Code / ccswitch / plugins / custom skills
- generation time
- update instruction

Stats must show:

- `去重技能`
- `技能文件来源`
- `中文分类`
- `多来源重复技能`

Sidebar must include:

- search label `搜索技能、中文用途、路径`
- search placeholder similar to `例如：飞书、PPT、dashboard、hallmark`
- category list with `全部` and per-category counts
- source distribution bars

Main content must include:

- result count `当前显示 X / Y 个技能`
- source filter select with default `全部来源`
- two-column card grid on desktop
- empty state `没有找到匹配的技能。换个关键词或分类试试。`

### Card Contract

Each skill card must show:

1. skill name
2. category chip
3. Chinese summary only
4. one or more source pills
5. hint `点击查看官方用法和完整路径`

Do not show long English descriptions on the card.

Cards must support:

- click to open
- keyboard focus
- Enter / Space to open
- visible hover / focus state

### Modal Contract

Use a centered, scrollable modal with dim overlay.

Modal header must include:

- skill name
- category pill
- source type pills only
- close button `×`

Do not show source-count chips like `来源数：1`.

Modal body order:

1. `中文说明`
2. `你可以这样说`
3. `官方用法`
4. `触发说明`
5. `英文原描述`
6. `完整文件路径`

Official usage block should use a pale-green surface and show command + meaning rows when table data exists.

Modal close actions:

- close button
- overlay click
- Escape key

### Search / Filter Contract

Search must match:

- skill name
- Chinese summary
- Chinese detail
- Chinese example
- English description
- category
- source labels
- source paths

Category and source filters must combine with search. Internally sort by category first, then skill name. Do not expose a sort UI.

### Visual Direction

Match the reference visual language:

- warm paper background
- off-white cards and panels
- warm beige borders
- deep blue-green text
- aqua / teal chips and active filters
- serif display title, sans body, monospace skill names and paths
- subtle shadows only

Layout:

- generous desktop padding
- four stat cards in one row on desktop
- sidebar + main panel on desktop
- one-column stack on mobile

Do not add:

- table/card toggle
- sort dropdown
- fake refresh button inside static HTML
- nested cards
- decorative gradient blobs

## Update Entry Point

The update script should:

- run from the local project directory
- print clear status
- keep the terminal window open on macOS / Windows double-click entrypoints
- regenerate `本地Skills查看器.html`
- avoid opening another page
- avoid requiring a web server
- refresh the displayed generation time on every run

If the page mentions updating, phrase it as an instruction, not a fake in-browser action.

## Configurability

Prefer one of:

- small config file such as `skills-viewer.config.json`
- environment variables such as `EXTRA_SKILL_ROOTS`
- clearly marked configuration block near the top of the generator

Allow overrides for:

- scan roots
- filename patterns
- source labels
- category keywords
- output filename

## Validation Checklist

Before finishing, verify:

- running the update script regenerates `本地Skills查看器.html`
- generation time changes after rerun
- visual structure matches the reference workbench
- desktop layout uses four stats, sidebar + main panel, two-column cards
- mobile layout stacks cleanly without overlap
- search works for name, Chinese text, English text, source, and path
- category and source filters work independently and together
- detail modal opens and closes via click, keyboard, overlay, and Escape
- official usage appears when present
- cards show Chinese summary only, source pills, and the open hint
- cards in the same category have distinct summaries when descriptions differ
- modal uses the required section order
- no absolute path from the original author's machine is hardcoded
- no duplicate legacy HTML output is generated unless requested
- there is no table/card toggle, sort dropdown, or fake in-browser refresh button

## Common Mistakes

- hardcoding one user's skill directories
- classifying skills by broad keywords only
- generating identical Chinese summaries for all skills in one category
- putting long descriptions directly on cards
- using category fallback too early
- exposing fake UI for local rescans inside a static `file://` page
- creating a heavy frontend build system when a single HTML file is enough
- adding extra documentation files the user did not ask for
