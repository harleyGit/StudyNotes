---
name: study-note-cleanup
description: Format Chinese Markdown explanations into StudyNotes-style learning notes without changing, adding, deleting, compressing, or reordering source content. Use when asked to整理笔记, 整理下 xxx.md, optimize Markdown notes, convert old.md to new.md style, or clean AI-generated study notes for this repository.
---

# Study Note Cleanup Skill

Use this skill to format a Chinese Markdown explanation in the StudyNotes note style while preserving the source content exactly.

## Goal

Produce a formatted learning note, not a rewrite or summary. **Content preservation has the highest priority.** Do not change, add, delete, compress, summarize, correct, infer, or reorder any source content unless the user explicitly gives a separate instruction for that specific change.

When the user says an old document was整理优化成 a new document and asks to summarize the writing/style skill, treat the new document as the target visual style only. The default rule is: **content is unchanged, wording is unchanged, section logic is unchanged, and only presentation formatting may be adjusted.**

## Input And Output

- Input is usually a verbose Markdown file.
- Output is a formatted Markdown file in the user's StudyNotes style.
- Preserve the source facts, SQL, commands, tables, identifiers, and examples as written. Accuracy here means fidelity to the source, not independent technical correction.
- Preserve every concept name, API name, sentence, example, repetition, warning, and source scope. Do not shorten local/example names.
- Preserve the original content scope, wording, heading order, paragraph order, reasoning order, and repeated explanations. Formatting must not become a content rewrite.
- Do not merge synonymous or repeated wording, move code or paragraphs, remove blank lines, reduce line breaks, or consolidate sections. Only repair Markdown syntax or visual formatting when the source content and order remain unchanged.
- Preserve all ASCII diagrams and text diagrams. Do not delete diagrams made from characters such as `┌`, `└`, `│`, `─`, `→`, `←`, arrows, indentation, boxes, or flow lines.
- Preserve code comments inside code blocks, including `//`, `/* ... */`, `#`, SQL comments, HTML comments, and inline explanatory comments. Even apparently incorrect comments must remain unchanged unless the user explicitly requests correction.
- Preserve command, script, and API request output blocks, including prompts like `$ curl ...`, server output, client output, logs, JSON responses, error messages, and before/after results. These outputs are used for comparison and must not be deleted.
- Do not invent new technical behavior or correct technical claims. If the source is ambiguous or appears incorrect, preserve it exactly and do not add a correction unless explicitly requested.
- Keep unrelated-looking topics, Q&A fragments, duplicate sections, and leftover headings because they are source content. Do not remove or split them unless explicitly requested.
- For tool setup or operation notes, preserve install checks, environment variables, shell commands, directory trees, daily workflow commands, and role distinctions between tools.

## Overall Structure

1. Keep the original opening and all existing sections in order. Do not add a table of contents, title, separator, or spacing block by default.
2. Preserve existing separator and spacing blocks, for example:

```md
***
<br/><br/><br/>
```

3. Preserve existing HTML heading blockquotes, for example:

```md
> <h2 id="">标题</h2>
```

4. Do not move any SQL, shell command, config, code, explanation, example, or diagram. Keep every item in its original position relative to the other source items.
5. Do not add a summary, explanation, conclusion, core-code block, or inferred context that is not already present in the source.
6. Preserve the original section order, reasoning order, and visual rhythm. Only normalize Markdown syntax when doing so does not alter source content or order.
7. Do not apply a preferred `new.md` structure if it would move, merge, shorten, or rewrite source content.
8. Do not trim unrelated-looking Q&A fragments, headings, or topics. They remain part of the source unless the user explicitly asks to remove them.
9. Preserve any existing summary in place; do not add, duplicate, or move a summary before step-by-step instructions.
10. Do not move a method or code snippet to the top. Keep it where it appears in the source.

## Table Of Contents Rules

- Preserve the existing table of contents exactly when one exists.
- If the source has no table of contents, do not add one unless the user explicitly asks for one.
- Preserve existing TOC labels, links, hierarchy, indentation, and entry order. Do not rename headings to match an inferred topic.
- Only when a TOC is explicitly requested, derive its labels from existing headings without inventing titles or changing the source body. Keep the same heading order.

Example of an existing TOC to preserve, not a template to insert by default:

```md
- [修改user_security表user_id类型和字段值](#修改user_security表user_id类型和字段值)
	- [修改步骤](#修改步骤)
	- [删除旧字段+重命名新字段](#删除旧字段+重命名新字段)
	- [整个迁移流程图](#整个迁移流程图)
```

## Heading Rules

- Do not convert, rename, re-level, or remove headings.
- Preserve existing `#`, `##`, `###`, and HTML heading forms.
- Do not add blockquoted HTML headings or anchors that are absent from the source.

```md
> <h3 id="修改步骤">修改步骤</h3>
```

- Preserve heading wording, numbering, hierarchy, duplicate titles, and anchor values exactly.
- Preserve existing spacing tags. Do not add or remove `<br/>` elements solely to improve visual rhythm.
- Keep headings that appear oversized, duplicated, numbered, or pseudo-semantic; do not reinterpret them.

## Content Preservation Rules

- Do not merge, shorten, summarize, paraphrase, translate, or rewrite any sentence or phrase.
- Do not remove filler, repeated explanations, conversational transitions, duplicate examples, or empty-looking lines.
- Do not convert paragraphs, blockquotes, lists, or fenced output into another form if that changes the source representation or order.
- Preserve all concrete numbers, formulas, causal chains, examples, warnings, and wording exactly.
- Preserve the source language and terminology exactly; do not add translations or inferred definitions.
- Keep every source line. When uncertain whether a line is accidental or duplicated, keep it.

The following is forbidden unless explicitly requested by the user:

- creating a shorter equivalent paragraph;
- merging repeated lines or sections;
- moving code near the beginning;
- adding a conclusion or technical correction;
- deleting a topic because it appears unrelated;
- replacing a source statement with an interpretation.

## Code Block Rules

- Preserve code fences and existing language tags, such as `sql`, `sh`, `text`, `go`, `js`.
- Preserve every code block, diagram, output block, and its position. Do not move, consolidate, split, simplify, or deduplicate blocks.
- Code comments are part of the learning note. Keep existing comments in code blocks, especially comments that explain parameters, execution order, business meaning, or why a branch returns early.
- Execution result blocks are part of the note. Keep complete command/request examples and their outputs together, including `# 服务器输出：`, `# 客户端收到：`, logs, separators, and response bodies.
- Keep all existing code block IDs, for example `sql id="s1"`.
- Do not add code, comments, imports, guards, error handling, runnable consolidated scripts, or production-use warnings absent from the source.
- Do not change SQL identifiers, constraint names, table names, column names, or command flags, even when a typo seems obvious.
- Do not fix source code or technical explanations without explicit permission. Note cleanup is formatting only, not behavior correction or technical review.
- Preserve guard conditions, state assignments, callback nesting, and captured variables in async/concurrency examples. These details usually carry the bug explanation.
- Preserve the complete contents of code, comments, diagrams, directory trees, commands, outputs, logs, JSON responses, and errors byte-for-byte, including indentation and blank lines.

## Paragraph And Inline Style

- Preserve every word, punctuation mark, symbol, emoji, label, and paragraph boundary. Do not translate terms or remove conversational phrasing.
- Existing terms may be wrapped in inline code or bold without changing the enclosed text, order, or punctuation. Do not introduce new labels or conclusions.
- Do not change inline code contents or remove invisible/control characters without explicit permission; their meaning may be significant.
- Preserve the original blockquote and list structure. Do not turn output blocks into inline values.

## Original Logic And Order

- The source is the only authority for section order and reasoning flow. Do not impose a background -> code -> explanation -> summary template.
- For code-explanation, UI delegate, async/lifecycle, state-machine, and tool-setup notes, preserve the exact original sequence of explanation, code, examples, and output.
- Keep numbered headings, tiny sections, troubleshooting, incomplete fragments, repeated summaries, and all topics in their original positions.
- Do not add missing steps or complete incomplete examples. If a suspected problem matters, mention it separately in the completion message as unverified, not as new content in the note.
- A request such as `整理`, `优化 md`, or `整理成 StudyNotes 风格` is not permission to shorten, correct, supplement, or restructure content.

## List And Emphasis Rules

- Preserve all list items, their wording, numbering, order, and nesting. Do not flatten, split, merge, or convert lists to paragraphs.
- Optional inline emphasis must wrap existing text only. It must not alter list membership, sequence, or technical meaning.

## Table Rules

- Preserve all tables, including duplicate or empty tables. Keep every column, row, cell value, and their order.
- Only adjust Markdown table delimiter syntax or alignment spacing outside cell content when necessary for rendering.
- Do not insert placeholder data rows, delete duplicates, or add explanations.

## Separator Rules

- Preserve existing separators and spacing tags in place. Do not add new section boundaries or remove existing ones.
- Existing StudyNotes separators may look like:

```md
***
<br/>

## 大章节
```

and:

```md
---
<br/>

### 小章节
```

- Do not normalize blank-line counts or strip trailing spaces globally; they can affect Markdown hard breaks, code, and diagrams.

## Editing Workflow

1. Read the entire source and applicable instructions before editing. Identify the existing order, protected code/output/diagram blocks, and any explicitly authorized exceptions.
2. Work from the actual pre-edit source, not a reconstructed summary or preferred template. Do not overwrite unrelated user changes.
3. Make only local presentation edits allowed by this skill: wrapping existing terms in inline code/bold, repairing unambiguous Markdown delimiters, or aligning table syntax outside cell content. If a repair could change meaning or block boundaries, leave it unchanged and ask.
4. Do not treat format-only permission as permission to add a TOC, anchors, headings, commentary, or content. These need explicit authorization if absent from the source.
5. Compare the result against the pre-edit source in order. Review every change as formatting only; reverse any accidental content edit made during this task.
6. If no safe formatting improvement is available, leave the file unchanged. Do not manufacture changes or target a shorter line count.

## Accuracy Checklist

Before finishing, verify:

- Every sentence, phrase, punctuation mark, heading label, number, formula, identifier, link destination, and table cell remains unchanged, except for specifically authorized changes.
- Section, paragraph, list, example, code, and diagram order is identical to the source.
- All repeated text, duplicate code, conversational fragments, and unrelated-looking topics remain present with the same occurrence counts.
- Every protected code, comment, diagram, command, output, log, and response block remains byte-for-byte identical and in its original relative position.
- No technical correction, new example, explanation, conclusion, caveat, inferred architecture, or translation was inserted.
- No TOC, heading, anchor, code block, or placeholder row was added without explicit permission. Existing labels and links were preserved.
- Every difference is an allowed formatting edit or a specifically authorized exception. Do not use a broad whitespace-stripped comparison as proof; it can hide meaningful changes in code and diagrams.
- No length-reduction target was used. An unchanged file is a valid result.
- Report the actual checks performed and any limitations. Do not claim technical verification, compilation, or content equivalence that was not checked.

## Usage Prompt Template

Use this prompt in Codex CLI or OpenCode CLI:

```text
请使用 study-note-cleanup 规则，对 <input.md> 仅做格式整理，输出到 <output.md>。保持原文措辞、全部内容及原有逻辑顺序，不增加、删除、修改、压缩、去重或移动内容，不修正技术表述。保留全部代码、注释、图示和输出原文；目录、标题和锚点沿用原有内容，不自行新增。没有安全的格式改进时保持文件不变。
```
