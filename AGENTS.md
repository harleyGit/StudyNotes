# AGENTS.md

This repository is a personal StudyNotes knowledge base. Codex CLI and OpenCode CLI should follow these instructions when working in this project.

## Collaboration Rules

- Prefer small, accurate edits over large rewrites.
- Preserve the existing Chinese StudyNotes writing style unless the user asks for a different style.
- Do not delete, rename, or revert existing notes unless the user explicitly asks.
- If unrelated files are already changed in the working tree, leave them alone.
- Keep Markdown readable in GitHub-flavored Markdown.
- Use concise Chinese explanations for user-facing summaries by default.

## Skill Rules

- All shared AI skills for this repository live under `.ai_skill/`.
- Keep each skill in its own folder, for example `.ai_skill/study-note-cleanup/SKILL.md`.
- Treat `.ai_skill/**/SKILL.md` as the canonical source of skill rules.
- Codex-specific files under `.codex/skills/` should only be lightweight entry files that reference `.ai_skill/`.
- OpenCode-specific files under `.opencode/skills/` should only be lightweight entry files that reference `.ai_skill/`.
- Do not maintain duplicate full skill rules in `.codex/skills/` and `.opencode/skills/`.

## Available Skills

- `study-note-cleanup`: use `.ai_skill/study-note-cleanup/SKILL.md` to整理 verbose Chinese Markdown into StudyNotes-style notes with a TOC, HTML heading anchors, core code first, compressed explanations, examples, and technical accuracy.
- Natural language trigger: when the user says `整理 xxx.md`, `整理下 xxx.md`, `整理笔记`, `优化 md`, or `整理成 StudyNotes 风格`, default to `study-note-cleanup`.

## StudyNotes Markdown Style

- Existing notes often use a manual table of contents at the top.
- Major sections may use HTML headings inside blockquotes, such as `> <h1 id="工具">工具</h1>`.
- Visual spacing commonly uses `<br/>`, `<br/><br/>`, and `<br/><br/><br/>`.
- Major separators commonly use `***`; normal section separators use `---`.
- Preserve useful tables, code blocks, and concrete examples.
- Preserve all ASCII diagrams and text diagrams. Do not delete diagrams made from characters such as `┌`, `└`, `│`, `─`, `→`, `←`, arrows, indentation, boxes, or flow lines.
- Preserve code comments inside code blocks, including `//`, `/* ... */`, `#`, SQL comments, HTML comments, and inline explanatory comments.
- Preserve command, script, and API request output blocks, including `$ curl ...`, server output, client output, logs, JSON responses, error messages, and before/after comparison results.
- Compress repetitive AI-generated explanations into concise, accurate Chinese paragraphs.
