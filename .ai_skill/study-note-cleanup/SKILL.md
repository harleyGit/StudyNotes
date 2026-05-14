---
name: study-note-cleanup
description: Convert verbose Chinese Markdown explanations into concise StudyNotes-style learning notes with a table of contents, HTML heading anchors, key code first, compressed explanations, examples, preserved ASCII diagrams, and preserved technical accuracy. Use when asked to整理笔记, 整理下 xxx.md, optimize Markdown notes, convert old.md to new.md style, or clean AI-generated study notes for this repository.
---

# Study Note Cleanup Skill

Use this skill to turn a long, repetitive Chinese Markdown explanation into the compact StudyNotes note style.

## Goal

Produce a readable learning note, not a literal rewrite. Preserve the technical meaning and important examples, but remove repetition, empty transitions, and over-explaining.

## Input And Output

- Input is usually a verbose Markdown file.
- Output is a cleaned Markdown file in the user's StudyNotes style.
- Keep the topic-specific facts, SQL, commands, tables, identifiers, and examples accurate.
- Preserve all ASCII diagrams and text diagrams. Do not delete diagrams made from characters such as `┌`, `└`, `│`, `─`, `→`, `←`, arrows, indentation, boxes, or flow lines.
- Preserve code comments inside code blocks, including `//`, `/* ... */`, `#`, SQL comments, HTML comments, and inline explanatory comments. Do not delete or rewrite them unless they are clearly wrong or the user explicitly asks.
- Preserve command, script, and API request output blocks, including prompts like `$ curl ...`, server output, client output, logs, JSON responses, error messages, and before/after results. These outputs are used for comparison and must not be deleted.
- Do not invent new technical behavior. If the source is ambiguous, keep the wording conservative.

## Overall Structure

1. Start with a manual table of contents.
2. Add a separator and spacing block:

```md
***
<br/><br/><br/>
```

3. Use the main title as an HTML heading blockquote:

```md
> <h2 id="">标题</h2>
```

4. If the source contains a complete SQL, shell command, config, or other core code sequence, move a consolidated version near the top under a short label such as `**xxx.sql文件**内容如下：`.
5. After the core code, explain the essence in one compact paragraph.
6. Continue with sections ordered from background -> target state -> reason -> steps -> examples -> final summary/flow chart.

## Table Of Contents Rules

- Include only the main title and the most important second-level sections.
- Do not include every small subsection.
- Use the same visible title text as the later heading anchor.
- Format with `- [标题](#标题)` and tab-indented child entries.
- Preserve existing child TOC indentation when matching StudyNotes style is useful, but avoid trailing whitespace in edited files.

Example:

```md
- [修改user_security表user_id类型和字段值](#修改user_security表user_id类型和字段值)
	- [修改步骤](#修改步骤)
	- [删除旧字段+重命名新字段](#删除旧字段+重命名新字段)
	- [整个迁移流程图](#整个迁移流程图)
```

## Heading Rules

- Convert numbered top-level headings like `# 一、...` into clean semantic headings.
- Use `##` or `###` for normal Markdown headings.
- Use blockquoted HTML headings for major anchored sections that appear in the TOC:

```md
> <h3 id="修改步骤">修改步骤</h3>
```

- Remove Chinese numbering prefixes when they do not add meaning, such as `一、`, `二、`, `十、`.
- Keep a short, direct heading. Prefer `两张表原来的关系` over `一、先理解两张表原来的关系`.
- Use `<br/>`, `<br/><br/>`, or `<br/><br/><br/>` for visual spacing between large blocks, following the surrounding note style.

## Compression Rules

- Merge short broken lines into complete sentences.
- Turn repeated `因为：` / `所以：` / `意思：` blocks into one concise sentence.
- Convert blockquotes with one-line explanations into bold inline emphasis where appropriate.
- Remove filler sentences that only announce the next obvious step.
- Preserve meaningful warnings, motivations, and causal relationships.
- Keep the note in Chinese unless the source is mainly another language.

Examples of compression:

```md
因为原来的 `user_id BIGINT`，而现在 `user_id VARCHAR(255)` 类型完全不同。

而且数据还得保留，所以必须**分步骤迁移**，否则数据会丢。
```

instead of many separate lines for `因为`, old type, new type, data retention, and conclusion.

## Code Block Rules

- Preserve code fences and language tags when useful, such as `sql`, `sh`, `text`, `go`, `js`.
- ASCII diagrams in code fences are important learning material and must be kept. You may move them to a better section, but do not remove or simplify them unless the user explicitly asks.
- Code comments are part of the learning note. Keep existing comments in code blocks, especially comments that explain parameters, execution order, business meaning, or why a branch returns early.
- Execution result blocks are part of the note. Keep complete command/request examples and their outputs together, including `# 服务器输出：`, `# 客户端收到：`, logs, separators, and response bodies.
- Keep existing code block IDs only if they are already present and do not hurt readability, for example `sql id="s1"`.
- If multiple source code blocks form one runnable script, add a consolidated code block near the top.
- Add short comments inside the consolidated code only when they clarify phases.
- Do not change SQL identifiers, constraint names, table names, column names, or command flags unless the source clearly contains a typo.

## List And Emphasis Rules

- Use bullets for categories, reasons, and examples.
- Bold key terms, table names, field names, and critical conclusions.
- Prefer inline code for identifiers: `` `user_security.user_id` ``.
- If the original note contains accidental invisible characters inside inline code, remove them unless they are required.
- Avoid deeply nested lists. Use separate sections if the structure becomes hard to read.

## Table Rules

- Preserve useful tables because they make data migration or state changes easier to understand.
- Remove duplicate tables if they repeat the same point without adding clarity.
- For an intentionally empty example table, include a placeholder row like `| | | |` so the table renders.
- Add `<br/>` before or after tables when the surrounding StudyNotes style uses visual spacing.

## Separator Rules

- Use `---` for normal section breaks.
- Use `***` for major breaks, especially after the TOC and before a new large chapter.
- Do not overuse separators between every small paragraph.

## Content Selection Rules

- Keep the central topic summary.
- Keep the original relationship/background needed to understand the change.
- Keep all ASCII diagrams, flow charts, box drawings, sequence diagrams, and visual call-chain diagrams.
- Keep command execution results, script outputs, API request outputs, logs, JSON responses, error outputs, and comparison outputs.
- Keep the target/new structure.
- Keep the reason direct modification is unsafe.
- Keep step-by-step operations.
- Keep one clear example for joins, mappings, cascades, or final flow when relevant.
- Drop long generic background that is not needed for the note's task.

## Accuracy Checklist

Before finishing, verify:

- The TOC anchors match the major heading IDs/text.
- The consolidated code is complete and in the correct order.
- All ASCII diagrams from the source are still present in the output.
- Code comments from source code blocks are still present in the output.
- Command/script/API output blocks from the source are still present in the output.
- Every renamed field, dropped constraint, recreated index, and foreign key still matches the source.
- The examples before and after migration are consistent.
- No source-critical warning was removed.
- Formatting renders as Markdown and matches StudyNotes style.

## Usage Prompt Template

Use this prompt in Codex CLI or OpenCode CLI:

```text
请使用 study-note-cleanup 规则，把 <input.md> 整理成 StudyNotes 风格笔记，输出到 <output.md>。要求保留技术准确性，压缩重复解释，添加目录、核心代码块、分段说明和必要示例。
```
