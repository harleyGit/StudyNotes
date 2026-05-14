---
name: study-note-cleanup
description: Convert verbose Chinese Markdown explanations into concise StudyNotes-style learning notes. Canonical rules are maintained in .ai_skill/study-note-cleanup/SKILL.md.
---

# Study Note Cleanup Skill

Canonical rule file:

```text
.ai_skill/study-note-cleanup/SKILL.md
```

When this skill is triggered, read and follow `.ai_skill/study-note-cleanup/SKILL.md`. Do not duplicate or modify the rules here unless the canonical file path changes.

Usage prompt:

```text
请使用 study-note-cleanup 规则，把 <input.md> 整理成 StudyNotes 风格笔记，输出到 <output.md>。要求保留技术准确性，压缩重复解释，添加目录、核心代码块、分段说明和必要示例。
```
