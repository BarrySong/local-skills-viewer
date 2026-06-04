# Local Skills Viewer Builder Skill

这是一个方法论 Skill，用来指导 Codex、Claude Code、OpenClaw、ccswitch 或其他 AI agent 在用户自己的电脑上生成“本地 Skills 查看器”。

它不是已经生成好的查看器项目，也不携带某台电脑的固定路径或成品 HTML。Agent 读取 `skill/SKILL.md` 后，应根据用户自己的系统、skill 目录和使用习惯，生成对应的本地 HTML 查看器和更新脚本。

This repository contains a methodology skill for building a local AI skills viewer. It is not the generated viewer app itself. After reading `skill/SKILL.md`, an agent should adapt to the user's operating system, local skill folders, and preferred workflow, then create the local HTML viewer and update entrypoint on that machine.

## Included

- `skill/SKILL.md` - the reusable skill instructions

## Use Cases

- Build a local HTML viewer for Codex / Claude Code / ccswitch / OpenClaw skills.
- Scan local `SKILL.md` or similar agent instruction files.
- Generate a compact searchable viewer with categories, source filters, and detail modals.
- Avoid re-debugging the same viewer-building logic for every user.

## 中文说明

把 `skill/SKILL.md` 发给你的 agent，让它按文档在你的电脑上生成查看器。它会先识别你的系统和 skill 路径，再生成所需文件；不要直接套用别人的绝对路径。
