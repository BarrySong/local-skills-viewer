---
name: local-skills-viewer-builder
description: Use when a user wants to build a local HTML viewer for AI agent skills, commands, plugins, or tool instructions across Codex, Claude Code, ccswitch, OpenClaw, custom folders, or similar local skill systems. The skill guides the agent to generate a single-file Chinese-friendly HTML viewer plus a local update script, while adapting scan paths and entrypoints to the user's operating system instead of hardcoding one machine's directories.
---

# Local Skills Viewer Builder

Use this skill to recreate a local skills viewer project like the reference project: a single HTML file that scans local `SKILL.md`-style files, groups them by purpose and source, supports search and filtering, and opens skill details in a modal.

Do not treat this skill as a finished app package. Treat it as a build recipe. The user's machine, operating system, agent tool, and skill directories may differ.

## How To Use This Skill

| Invocation | What to do |
| --- | --- |
| default | Build a local skills viewer project in the user's chosen folder. Discover or ask for scan roots, then generate the viewer, update script, and minimal style assets. |
| audit | Review an existing local skills viewer project for missing scan roots, stale hardcoded paths, broken update scripts, or UI regressions. Do not rewrite unless the user asks. |
| extend | Add support for another skill system, folder layout, operating system, or metadata format while preserving the existing viewer behavior. |
| package | Turn an already working local viewer into a reusable skill/methodology document. Include instructions and acceptance criteria, not user-specific generated output. |

## Output Contract

Generate these files in the user's target project folder:

- `generate_skill_dashboard.py` or equivalent generator script.
- `本地Skills查看器.html` as the main single-file viewer.
- `tokens.css` if the generator uses shared design tokens.
- One or more update entrypoints appropriate for the user's OS:
  - macOS: `更新本地Skills查看器.command`
  - Windows: `更新本地Skills查看器.bat` or `.ps1`
  - Linux: `更新本地Skills查看器.sh`
- Optional `.hallmark/log.json` only when the user asks for Hallmark-style design traceability or the local project already uses it.

Do not generate legacy duplicate HTML files unless the user explicitly asks for backward compatibility.

## Environment Discovery

Before implementing, determine the target environment:

- Operating system: macOS, Windows, Linux, or mixed.
- Preferred runtime: Python is preferred when available; otherwise use Node.js or the user's existing local runtime.
- Skill systems to scan: Codex, Claude Code, ccswitch, OpenClaw, custom folders, plugin caches, team repositories, or other local instruction folders.
- File naming conventions: `SKILL.md`, `skill.md`, command markdown files, plugin manifest files, or custom names.
- Whether the viewer must be Chinese-facing, bilingual, or English-facing.

Use common defaults only as suggestions. Never permanently hardcode another person's absolute home path.

Recommended default roots when present:

- Codex: `~/.codex/skills`, `~/.codex/plugins/cache`
- Claude Code or agents-style folders: `~/.agents/skills`
- ccswitch: `~/.cc-switch/skills`
- OpenClaw or other tools: ask the user or inspect local config/docs before assuming.
- Custom roots: support a config file or environment variable such as `EXTRA_SKILL_ROOTS`.

## Generator Requirements

The generator should:

1. Read a configurable list of scan roots.
2. Recursively find skill files, usually `SKILL.md`; support extra filename patterns when the user needs them.
3. Parse YAML frontmatter when available.
4. Extract at least:
   - skill name
   - description
   - source path
   - source type
   - category
   - official usage section
5. Fall back gracefully when frontmatter is incomplete:
   - name from parent folder
   - description from first useful paragraph
6. Deduplicate by skill name while preserving all source paths.
7. Prefer the longest or most specific description when duplicated skills disagree.
8. Generate a fully self-contained HTML file with embedded data and client-side interactions.
9. Update the displayed generation time on every generator run.
10. Never modify the user's skill files.

## Official Usage Extraction

For each skill file, look for the first heading matching:

- `How to use`
- `Usage`
- `When to use`
- `Invocation`
- `用法`
- `使用`

Extract content until the next heading of the same or higher level.

Preferred parsing order:

1. Markdown table rows, especially `Invocation | What it does`.
2. Bullet or numbered lists.
3. First meaningful paragraph.
4. Fallback to the official description and trigger category.

Normalize Markdown lightly:

- remove inline code ticks but preserve command text
- unwrap links to link text
- remove bold/italic markers
- collapse repeated whitespace

## Classification Rules

Use simple transparent heuristics first. Let the user tune categories later.

Suggested Chinese categories:

- 产品设计与前端
- 数据分析与报表
- 文档 / 表格 / 演示
- 浏览器与网页控制
- Codex / Agent 技能管理
- Claude / OpenClaw / 其他 Agent
- 飞书 / Lark
- 创意生产
- iOS / Swift
- 投资研究
- 搜索与命令行工具
- 知识库 / 个人工具
- 邮箱 / 团队协作
- 其他

Source labels should come from detected root ownership, not from fixed text. Examples:

- `Codex 个人技能`
- `Codex 插件缓存`
- `Claude Code 技能`
- `ccswitch 技能`
- `OpenClaw 技能`
- `项目内自定义技能`
- `额外扫描目录`

## Viewer UI Requirements

The HTML viewer must work by directly opening the file in a browser.

The default goal is visual and functional parity with the reference viewer. Do not redesign it into a landing page, SaaS dashboard, docs site, table app, or generic template unless the user explicitly asks for a different style.

Required capabilities:

- Title: `本地 Skills 查看器` or a user-approved equivalent.
- Summary stats:
  - deduplicated skill count
  - source file count
  - category count
  - multi-source duplicate count
- Search by skill name, Chinese summary, English description, category, source, and path.
- Chinese category filter.
- Source filter.
- Card view with:
  - skill name
  - category chip
  - Chinese-facing summary only
  - source shown compactly as a pill or short source label
  - clear affordance to open details
- Detail modal with:
  - Chinese explanation
  - official usage
  - trigger or use-case explanation
  - English original description
  - all source paths
- Responsive layout for desktop and mobile.
- No separate table/card toggle unless the user asks for it.
- No sort control unless the user asks for it.

## Reference UI Contract

Build the generated HTML with this exact product shape.

### Page Structure

Use one single-page workbench:

```text
main.page
├── section.topbar
│   ├── eyebrow: 本地 AI 工具 Skills 速查
│   ├── h1: 本地 Skills 查看器
│   ├── subhead explaining Codex / Claude Code / ccswitch / plugins / custom skills
│   └── generation time + update instruction
├── section.stats
│   ├── 去重技能
│   ├── 技能文件来源
│   ├── 中文分类
│   └── 多来源重复技能
├── section.workbench
│   ├── aside.panel.sidebar
│   │   ├── search input
│   │   ├── 中文分类 button list
│   │   └── 来源分布 bars
│   └── section.panel.main
│       ├── result count
│       ├── source filter select
│       └── card grid
└── div#detailModal
    └── modal panel with title, meta chips, official usage, explanation, trigger, original description, source paths
```

Do not include:

- marketing hero sections
- table/card view toggle
- sort dropdown
- fake refresh buttons inside the static HTML
- nested cards or decorative background blobs

### Visual Design Tokens

Use this visual language:

- Warm paper background with subtle green transition: cream / parchment page, not pure white.
- Ink color: deep blue-green.
- Accent color: aqua / teal chips and active filters.
- Secondary accent: earthy brown for hints and section labels.
- Cards and panels: off-white surface, 1px warm beige border, 8px radius or a close equivalent.
- Shadows: very subtle; avoid glossy SaaS gradients.
- Typography:
  - Display heading: Chinese serif or system serif fallback.
  - Body: system sans with Chinese support.
  - Skill names and paths: monospace.
- Layout rhythm:
  - Generous page padding on desktop.
  - Four stat cards in one row on desktop.
  - Left sidebar and right content panel in the main workbench.
  - Two-column skill card grid on desktop.
  - Single-column stacked layout on mobile.

Recommended token names if generating CSS:

```css
--font-display
--font-body
--font-outlier
--color-ink
--color-muted
--color-page
--color-panel
--color-line
--color-accent
--color-accent-2
--color-accent-soft
--radius
--space-*
```

### Header Behavior

Header content should match the reference intent:

- Eyebrow: `本地 AI 工具 Skills 速查`
- Main title: `本地 Skills 查看器`
- Subhead should explain that the page summarizes visible Codex, Claude Code, ccswitch, plugin, and custom skills into Chinese purpose, category, source, and trigger/use scenarios.
- Right side should show:
  - `生成时间：YYYY-MM-DD HH:mm:ss`
  - update instruction such as `双击更新本地Skills查看器.command`
  - short note: after adding or editing skills, rerun the update script and the generation time will refresh.

If the user's OS is not macOS, change only the update entrypoint wording. Keep the header layout and meaning.

### Sidebar Behavior

Sidebar must include:

- Search label: `搜索技能、中文用途、路径`
- Search placeholder similar to `例如：飞书、PPT、dashboard、hallmark`
- Chinese category list with:
  - `全部`
  - category count on the right
  - active category highlighted with aqua background
- Source distribution bars:
  - source label on left
  - count on right
  - muted track with dark green fill

Sidebar should remain visually separate from the content area as a `panel`, not a floating marketing card.

### Main Content Behavior

Main content must include:

- Result count text: `当前显示 X / Y 个技能`
- Source filter select:
  - default option: `全部来源`
  - options from detected source labels
- Empty state: `没有找到匹配的技能。换个关键词或分类试试。`
- Skill cards in a two-column grid on desktop.

### Skill Card Contract

Each skill card should be compact and clickable.

Card content order:

1. Top row:
   - skill name in monospace-like bold text
   - category chip on the right
2. Chinese-facing summary only.
3. Source shown as one or more compact source pills.
4. Hint: `点击查看官方用法和完整路径`

Do not show the raw English description on the card unless no Chinese summary can be produced. Put long text in the modal.

Card summary generation:

- Generate the Chinese card summary from that specific skill's `name`, `description`, and official usage when available.
- Do not use the category-level fallback as the first choice.
- Skills in the same category must not all receive the same Chinese summary unless their source descriptions are genuinely identical.
- Category-level wording is only a last-resort fallback for missing or unusable descriptions.

Cards must support:

- mouse click to open modal
- keyboard focus
- Enter/Space to open modal
- visible hover/focus state

### Detail Modal Contract

The detail modal is the main long-form reading surface.

Modal requirements:

- Full-screen dim overlay.
- Centered panel with max width around 920px.
- Modal panel max height around 88vh with internal scrolling.
- Header:
  - skill name
  - category pill
  - source type pills only
  - close button `×`
- Do not show `来源数：1` or source count chips in the modal header.
- Body sections in this order:
  1. `中文说明`
  2. `你可以这样说`
  3. `官方用法`
  4. `触发说明`
  5. `英文原描述`
  6. `完整文件路径`

Official usage presentation:

- Use a pale green official-usage block.
- Show a title row with left label `官方用法` and right original heading such as `How to use this skill`.
- If usage is a table, render each row as command + meaning.
- Command column should use monospace.
- Meaning column should preserve enough text to understand the usage.

Modal interactions:

- Click close button to close.
- Click overlay outside the panel to close.
- Escape key closes.
- Opening modal should focus the close button.
- Closing modal should restore normal page state.

### Search And Filter Contract

Search should match against:

- skill name
- Chinese summary
- Chinese detail
- Chinese example
- English description
- category
- source labels
- source paths

Filtering behavior:

- Category filter and source filter combine with search.
- Category `全部` resets category only.
- Source `全部来源` resets source only.
- Result count updates immediately.
- Cards re-render immediately.

Sort behavior:

- Do not expose a sort UI.
- Internally sort by category first, then skill name.

### Responsive Contract

Desktop:

- Page max width around 1360px.
- Stats in four columns.
- Workbench as sidebar + main content.
- Cards in two columns.

Mobile:

- Header stacks vertically.
- Stats become two columns.
- Workbench becomes one column.
- Cards become one column.
- Modal uses smaller padding and full available width.
- Long paths and commands wrap without horizontal scrolling.

### Content Copy Contract

Use Chinese-facing UI text by default. Preserve these labels unless the user asks for another language:

- `本地 AI 工具 Skills 速查`
- `本地 Skills 查看器`
- `去重技能`
- `技能文件来源`
- `中文分类`
- `多来源重复技能`
- `搜索技能、中文用途、路径`
- `中文分类`
- `来源分布`
- `当前显示 X / Y 个技能`
- `全部来源`
- `点击查看官方用法和完整路径`
- `中文说明`
- `你可以这样说`
- `官方用法`
- `触发说明`
- `英文原描述`
- `完整文件路径`

## Design Guidance

- Keep cards compact; put long usage content in the modal.
- Use restrained colors and clear spacing.
- Avoid decorative landing-page layouts; this is a workbench tool.
- Avoid nested cards.
- Ensure long paths wrap cleanly on mobile.
- The generated page should look like a polished local utility, not a marketing website.

## Update Entry Point

The update script should rerun the generator in the project folder and leave the user with one fresh HTML file.

Implementation rules:

- Use the local project directory as the working directory.
- Print clear human-readable status.
- Keep the window open on macOS/Windows double-click entrypoints so users can see errors.
- Regenerate `本地Skills查看器.html`.
- Do not open or navigate to another HTML page during update.
- Do not require a web server.
- On every run, update the generation timestamp inside the HTML.

If the viewer includes a visible update hint, phrase it as an instruction, not a fake browser button, unless the agent has implemented a real local protocol or app integration. A static `file://` page cannot safely rescan arbitrary local files by itself in normal browsers.

## Configurability

Prefer one of these patterns:

- A small config file such as `skills-viewer.config.json`.
- Environment variables such as `EXTRA_SKILL_ROOTS`.
- A clearly marked list near the top of the generator script.

The config should allow:

- adding scan roots
- changing filename patterns
- overriding source labels
- adding category keywords
- choosing the output filename

Do not require users to edit large HTML blobs.

## Implementation Outline

Build in this order:

1. Create the generator script.
2. Add configurable scan roots and filename patterns.
3. Implement safe markdown/frontmatter parsing.
4. Implement deduplication, source labeling, categorization, and official usage extraction.
5. Render data into a single-file HTML with embedded JSON.
6. Add CSS tokens or inline CSS.
7. Add client-side search, category filter, source filter, and modal logic.
8. Add OS-appropriate update entrypoint.
9. Run the generator once.
10. Verify the generated HTML opens locally.

## Validation Checklist

Before finishing, verify:

- The target folder contains only the expected project files.
- Running the update script regenerates `本地Skills查看器.html`.
- Generation time changes after rerun.
- The generated page visually matches the reference workbench structure: topbar, four stats, left sidebar, right card grid, and modal.
- The color palette matches the reference: warm paper background, off-white panels, aqua chips, deep blue-green ink, warm beige borders.
- Desktop layout uses four stat cards, sidebar + main content, and two-column skill cards.
- Mobile layout stacks cleanly without overlapping text.
- Search works for skill name, Chinese text, English text, source, and path.
- Category filter works.
- Source filter works.
- Category and source filters combine correctly with search.
- Detail modal opens and closes.
- Detail modal closes via close button, overlay click, and Escape key.
- Skill cards open via mouse click and keyboard Enter/Space.
- Official usage appears when a skill has a matching section.
- Official usage appears in a pale green block with command + meaning rows when table data exists.
- Skills without frontmatter still appear with sensible fallback text.
- Duplicate skill names show multiple source paths instead of duplicated cards.
- Cards show Chinese summary only, source pills, and the hint `点击查看官方用法和完整路径`.
- Cards in the same category have distinct Chinese summaries when their source descriptions differ.
- Modal body uses the required section order: Chinese explanation, example prompt, official usage, trigger, English description, file paths.
- No absolute path from the original author machine is hardcoded.
- No obsolete duplicate HTML output is generated unless requested.
- There is no table/card toggle, no sort dropdown, and no fake in-browser refresh button.

## Common Mistakes To Avoid

- Do not hardcode one user's `~/.codex` paths as the only supported locations.
- Do not make a browser button that pretends to rescan local files from a static HTML page.
- Do not generate both old and new HTML names unless compatibility is required.
- Do not put every long description directly on the card.
- Do not remove the user's existing skills or modify source `SKILL.md` files.
- Do not create a heavy frontend build system when a single HTML file satisfies the request.
- Do not create extra documentation files unless the user asks; the generated project should stay small.
