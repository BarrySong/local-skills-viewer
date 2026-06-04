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

Design guidance:

- Keep cards compact; put long usage content in the modal.
- Use restrained colors and clear spacing.
- Avoid decorative landing-page layouts; this is a workbench tool.
- Avoid nested cards.
- Ensure long paths wrap cleanly on mobile.

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
- Search works for skill name, Chinese text, English text, source, and path.
- Category filter works.
- Source filter works.
- Detail modal opens and closes.
- Official usage appears when a skill has a matching section.
- Skills without frontmatter still appear with sensible fallback text.
- Duplicate skill names show multiple source paths instead of duplicated cards.
- No absolute path from the original author machine is hardcoded.
- No obsolete duplicate HTML output is generated unless requested.

## Common Mistakes To Avoid

- Do not hardcode one user's `~/.codex` paths as the only supported locations.
- Do not make a browser button that pretends to rescan local files from a static HTML page.
- Do not generate both old and new HTML names unless compatibility is required.
- Do not put every long description directly on the card.
- Do not remove the user's existing skills or modify source `SKILL.md` files.
- Do not create a heavy frontend build system when a single HTML file satisfies the request.
- Do not create extra documentation files unless the user asks; the generated project should stay small.
