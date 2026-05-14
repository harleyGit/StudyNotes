# study-note-cleanup

This file makes the same note-cleanup skill easy to reference from OpenCode CLI.

Canonical rules live in:

```text
.ai_skill/study-note-cleanup/SKILL.md
```

When using OpenCode CLI, attach or reference that file and say:

```text
请按 .ai_skill/study-note-cleanup/SKILL.md 的规则，把输入 Markdown 整理成 StudyNotes 风格笔记。
```

Trigger phrases:

- `使用 study-note-cleanup 整理笔记`
- `把 old.md 整理成 new.md 这种格式`
- `按 StudyNotes 风格压缩这篇 Markdown`
- `整理 AI 生成的中文学习笔记`

The skill converts verbose Chinese Markdown explanations into concise StudyNotes-style learning notes with a TOC, HTML heading anchors, key code first, compressed explanations, examples, and accurate technical content.
