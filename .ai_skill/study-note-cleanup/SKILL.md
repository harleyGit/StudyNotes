---
name: study-note-cleanup
description: Convert verbose Chinese Markdown explanations into concise StudyNotes-style learning notes with a table of contents, HTML heading anchors, key code first, compressed explanations, examples, preserved ASCII diagrams, and preserved technical accuracy. Use when asked to整理笔记, 整理下 xxx.md, optimize Markdown notes, convert old.md to new.md style, or clean AI-generated study notes for this repository.
---

# Study Note Cleanup Skill

Use this skill to turn a long, repetitive Chinese Markdown explanation into the compact StudyNotes note style.

## Goal

Produce a readable learning note, not a literal rewrite. Preserve the technical meaning and important examples, but remove repetition, empty transitions, and over-explaining.

When the user says an old document was整理优化成 a new document and asks to summarize the writing/style skill, treat the new document as the target style, but do not infer permission to change technical meaning. The default rule is: **content is basically unchanged, section logic is unchanged, only formatting and unnecessary redundant description are optimized.**

## Input And Output

- Input is usually a verbose Markdown file.
- Output is a cleaned Markdown file in the user's StudyNotes style.
- Keep the topic-specific facts, SQL, commands, tables, identifiers, and examples accurate.
- Preserve the user's intended concept names and public API names. Shorten local/example class names only when the optimized note already does so or when it clearly improves readability without changing technical meaning.
- Preserve the original content scope and reasoning order unless the user explicitly asks for a rewrite. Cleanup means formatting, compression, heading cleanup, and redundant description removal, not changing the note's argument.
- Preserve all ASCII diagrams and text diagrams. Do not delete diagrams made from characters such as `┌`, `└`, `│`, `─`, `→`, `←`, arrows, indentation, boxes, or flow lines.
- Preserve code comments inside code blocks, including `//`, `/* ... */`, `#`, SQL comments, HTML comments, and inline explanatory comments. Do not delete or rewrite them unless they are clearly wrong or the user explicitly asks.
- Preserve command, script, and API request output blocks, including prompts like `$ curl ...`, server output, client output, logs, JSON responses, error messages, and before/after results. These outputs are used for comparison and must not be deleted.
- Do not invent new technical behavior. If the source is ambiguous, keep the wording conservative.
- If the source contains unrelated leftover topics before or inside the target note, remove them when they are not referenced by the TOC or final title. Do not mix two unrelated notes in one cleanup output unless the user asks.
- For tool setup or operation notes, preserve install checks, environment variables, shell commands, directory trees, daily workflow commands, and role distinctions between tools.

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
7. Prefer the `new.md` pattern when converting verbose notes: title and core concept first, minimal setup code, one-sentence purpose, short context, then focused breakdowns.
8. Preserve the user's visual rhythm: large sections use `***` plus `<br/>`; normal section transitions use `---` plus `<br/>`; small internal spacing can use a single `<br/>`.
9. When a note is already logically ordered, do not over-restructure it. Clean headings, spacing, and prose while keeping the original reasoning sequence.
10. If the source starts with unrelated Q&A fragments or another note's heading, trim them so the final output focuses on the TOC topic and main heading.
11. For practical setup guides, start with a concise core-logic summary before the step-by-step instructions, especially when multiple tools coordinate together.
12. For method-explanation notes, prefer showing the complete method or complete core snippet near the top, then explain each step in the same order as the code executes.

## Table Of Contents Rules

- Include only the main title and the most important second-level sections.
- Do not include every small subsection.
- Use the same visible title text as the later heading anchor.
- Format with `- [标题](#标题)` and tab-indented child entries.
- Preserve existing child TOC indentation when matching StudyNotes style is useful, but avoid trailing whitespace in edited files.
- If the title text is cleaned for readability, keep the TOC visible text aligned with the final heading while preserving the existing anchor ID when it is already stable and meaningful.
- For short operation guides, a single top-level TOC entry is acceptable when listing every step would make the table of contents noisy.

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
- Prefer unnumbered semantic headings for StudyNotes output, for example `## 为什么还需要 UUID session` instead of `## 二、为什么还需要 UUID session`.
- Use `##` for major reasoning questions and `###` only when a subsection is genuinely nested. Avoid creating tiny heading fragments for every example line.
- Convert oversized pseudo-headings used only for emphasis, such as `# 强状态流转系统` or `# Swift enum 状态机核心写法`, into bold inline conclusions instead of real headings.
- Do not keep broken numbering when earlier sections were removed. Re-level headings so the note reads continuously.
- For step-by-step operation guides, keep numbered step headings like `## 一、前置检查` and `## 二、步骤1：...` when they help the user execute in order.
- Remove duplicate titles when the source has both an HTML heading and a Markdown `#` title. Keep the blockquoted HTML heading as the main title.

## Compression Rules

- Merge short broken lines into complete sentences.
- Turn repeated `因为：` / `所以：` / `意思：` blocks into one concise sentence.
- Convert blockquotes with one-line explanations into bold inline emphasis where appropriate.
- Remove filler sentences that only announce the next obvious step.
- Preserve meaningful warnings, motivations, and causal relationships.
- Keep the note in Chinese unless the source is mainly another language.
- Compress AI-style step announcements such as `我按...给你完整拆解`, `这个才是灵魂`, `很多人这里不懂`, and similar conversational transitions unless they carry technical meaning.
- Convert repeated isolated `text` blocks that contain only a phrase into inline code or one compact sentence, for example `键盘避让`, `确认密码输入框`, `页面需要上移 82pt`.
- Keep one clear explanation for each concept. If the same point appears in background, example, and summary, merge it into the most useful location.
- Prefer `说明：...` or `意思：...` as inline prose only when helpful; do not create separate paragraphs for every `意思`.
- Preserve important concrete numbers and formulas, but collapse their explanation into a compact example paragraph when possible.
- Keep important cause-and-effect chains explicit. Do not compress away the sequence when the point depends on timing, lifecycle, state transitions, or async callback ordering.
- Merge micro-sections that only show one state/value check into the preceding explanation. Use `###` plus inline bold phrases rather than creating a new `##` for every branch.
- Convert short output-only code fences to inline code when the output is a single value and not a meaningful log block.
- Convert opening role descriptions into a compact bold bullet list when multiple tools/modules are compared.
- Keep executable instructions direct. Remove conversational lead-ins, but keep enough context for the reader to follow the steps without the original chat.
- Do not delete meaningful examples merely to shorten the note. If an example demonstrates a different behavior branch such as insert/delete/replace, keep it and compress only the wording.
- Convert tiny explanation blocks like `输入：`, `得到：`, `结果：`, `返回：` into inline sentences when they are part of the same example.
- Keep the source's before/after reasoning chain intact. For example, when explaining `shouldChangeCharactersIn`, preserve the flow `currentText -> Range conversion -> updatedText -> digit extraction -> count limit -> return Bool`.

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
- For programming notes, keep the main source code near the top before long explanation, even if the original explanation starts with background.
- Do not silently fix apparent source typos inside code unless the user asks for code correction. Note cleanup is primarily formatting and explanation cleanup, not behavior correction.
- Preserve guard conditions, state assignments, callback nesting, and captured variables in async/concurrency examples. These details usually carry the bug explanation.
- If a source code block is illustrative rather than full production code, it may be shortened only when the omitted parts are not relevant to the explanation.

## Paragraph And Inline Style

- Prefer full Chinese sentences over one phrase per line.
- Use inline code for UI labels, API names, method names, variables, constants, formulas, and short quoted states, for example `activeTextField`, `keyboardWillChangeFrameNotification`, `view.transform = .identity`.
- Use bold for conclusions and key judgments, for example `**必须和系统键盘动画一致**` or `**这是核心公式**`.
- Combine short example fragments into one paragraph: condition -> value -> result -> meaning.
- Avoid excessive blockquotes. Keep blockquoted HTML headings for anchored major sections; turn explanatory blockquotes into normal prose or bold inline emphasis.
- Remove accidental invisible/control characters in normal prose and inline code when they appear during copy/paste cleanup.
- Keep English technical terms when they are canonical, and pair them with Chinese meaning when useful, for example `Keyboard Avoidance / 键盘避让`.
- When explaining syntax, keep the canonical syntax in code blocks and put equivalent forms, meanings, and conclusions in compact surrounding prose.
- Use bold labels for important setup requirements and role definitions, for example `**核心逻辑：**`, `**绑定项目路径**`, and `**AGENTS.md = 项目专属规则**`.
- Prefer inline code for small values and short states, such as `86`, `1`, `{2,0}`, `.unknown`, and `true`; reserve fenced code blocks for real code, multi-line flows, tables, or output that benefits from alignment.

## Section Breakdown Pattern

For code-explanation notes, use this order when it fits the content:

1. TOC.
2. Major title anchor.
3. Core code or minimal core declaration/change, such as a complete method block or `private var scanSessionID = UUID()`.
4. Explain what it is not and what it is, especially when the concept is easy to misunderstand.
5. Explain when it is refreshed and what problem that invalidates.
6. Focused subsections for why the simple fix is not enough, why extra state/token is needed, code-level walkthrough, lifecycle/timing example, and final role summary.
7. End with a concise conclusion, not a long repeated summary.

For UI delegate / input-validation notes, preserve this reasoning pattern when present:

1. Show the full delegate method or full core snippet first.
2. Explain the current text, replacement string, and `NSRange` parameters with one concrete example.
3. Explain why `NSRange` must be converted to `Range<String.Index>`.
4. Explain how `replacingCharacters(in:with:)` predicts the text after the user's action before the text field really changes.
5. Keep separate examples for insert, delete, and replace when present.
6. Explain any custom sanitizer/helper, such as extracting digits.
7. Explain the final limit check and return value.
8. End with equivalent pseudocode and common use cases if present.

When the source has many numbered headings like `一、二、三、...`, convert them into semantic `##` / `###` headings and merge tiny adjacent headings into one section.

For async/lifecycle bug notes, preserve this reasoning pattern when present:

1. Show the tempting simple fix and its limitation.
2. Explain the uncancellable async work or delayed callback.
3. Distinguish similarly named operations, such as manager-level `stop()` versus lower-level `scanner?.stopScan()`.
4. Walk through a concrete old-session/new-session timeline.
5. Show the exact guard/token check that prevents stale callbacks.
6. Summarize the division of responsibilities, such as state machine for current state and session token for callback ownership.

For syntax/state-machine explanation notes, preserve this reasoning pattern when present:

1. Start with the core enum/class code and the exact syntax being explained.
2. Explain the syntax meaning and its simple equivalent form.
3. Show success and failure branches with one concrete state example each.
4. Explain why the domain fits the pattern, using a flow diagram if present.
5. Provide one or two realistic usage scenarios.
6. Show the advanced or recommended version only after the basic syntax is clear.
7. End with the core principle in one bold sentence.

For tool setup / CLI workflow guides, preserve this reasoning pattern when present:

1. Main title and short core-logic summary.
2. Role split between tools, preferably as bold bullets.
3. Prerequisite checklist and required logins.
4. Standard project directory tree.
5. Environment/profile setup with exact variables.
6. Automation scripts or post-switch commands.
7. Terminal integration commands.
8. Daily workflow commands.
9. Concept positioning, such as `AGENTS.md` versus `Skill`.

Troubleshooting sections may be kept when they contain concrete commands or likely failure modes. If they are generic, incomplete, or not part of the optimized target style, compress or omit them.

## List And Emphasis Rules

- Use bullets for categories, reasons, and examples.
- Bold key terms, table names, field names, and critical conclusions.
- Prefer inline code for identifiers: `` `user_security.user_id` ``.
- If the original note contains accidental invisible characters inside inline code, remove them unless they are required.
- Avoid deeply nested lists. Use separate sections if the structure becomes hard to read.
- Use tab-indented child bullets only where the surrounding StudyNotes style already uses them; otherwise keep bullets shallow.
- Convert repeated single-line examples into bullets only when they are parallel categories, such as trigger conditions or advantages.
- If a nested list becomes three levels deep, flatten it into short paragraphs or separate headings.
- Keep numbered lists for ordered timelines, reproduction steps, lifecycle sequences, and async execution order. Use bullets for unordered characteristics, advantages, or category lists.
- In setup guides, keep ordered steps for execution order, but convert checklist items and tool responsibilities into bold bullets for faster scanning.
- Avoid mixing Markdown ordered list numbers with bold pseudo-numbers too heavily. Use `- **1.xxx**` only when matching the surrounding StudyNotes style; otherwise prefer normal `1.` lists.

## Table Rules

- Preserve useful tables because they make data migration or state changes easier to understand.
- Remove duplicate tables if they repeat the same point without adding clarity.
- For an intentionally empty example table, include a placeholder row like `| | | |` so the table renders.
- Add `<br/>` before or after tables when the surrounding StudyNotes style uses visual spacing.

## Separator Rules

- Use `---` for normal section breaks.
- Use `***` for major breaks, especially after the TOC and before a new large chapter.
- Do not overuse separators between every small paragraph.
- In StudyNotes-style optimized files, major `##` sections commonly use:

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

- Keep extra blank lines modest. Do not leave long runs of empty lines except where the existing note intentionally uses `<br/><br/><br/>` for top spacing.
- Normalize accidental separator whitespace such as `*** ` to `***`.
- Prefer `***` between major top-level explanation sections in compact notes when the surrounding note uses that style, not just after the TOC.

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
- Keep concrete runtime flow examples because they make code behavior easier to understand.
- Keep exact formulas and calculations, especially when the note explains UI layout, algorithms, database migration, networking, or concurrency.
- Keep exact state values and token/version examples, such as `scanSessionID = A/B/C`, `.cancelled`, `.scanning(round: 1)`, and `A == C ? false`, because they make stale-callback bugs understandable.
- Keep practical pros/cons only if they add decision value; merge generic advantage lists into one concise conclusion.
- Remove repeated final summaries when the same points were already explained in the body.
- Preserve distinctions between similarly named methods or layers when that distinction is central to the note.
- Preserve domain flow diagrams such as BLE lifecycle arrows because they explain why a state machine is appropriate.
- Remove unrelated introductory snippets that belong to a different concept, even if they are technically valid.
- Preserve directory tree blocks, environment variable blocks, shell commands, and CLI command examples exactly enough that the reader can execute them.
- Preserve tool responsibility boundaries, especially when one tool manages environment switching, another manages repository operations, and another manages AI agent/session behavior.
- Preserve behavior-branch examples such as input, deletion, replacement, paste, and validation failure when they demonstrate that the code handles more than one path.

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
- Important variables, method names, notification names, constants, formulas, numeric examples, and output values still match the source.
- For async/lifecycle notes, stale callback timelines, state transitions, token comparisons, and guard checks still match the source.
- For syntax/state-machine notes, the enum cases, associated values, guard-case examples, switch examples, and domain flow diagrams still match the source.
- The output no longer contains unrelated leftover sections from a different note topic.
- For tool setup guides, prerequisites, environment variable names, paths, command names, branch names, and daily workflow commands still match the source.
- For UI delegate/input-validation notes, parameters, example values, conversion steps, updated text examples, helper function behavior, max length, and final Bool results still match the source.
- The cleanup did not change the content scope or the order of the explanation logic unless the user explicitly requested restructuring.
- The final note is significantly shorter than the verbose source unless the source is already compact.
- The note reads like a StudyNotes reference, not a chat transcript.

## Usage Prompt Template

Use this prompt in Codex CLI or OpenCode CLI:

```text
请使用 study-note-cleanup 规则，把 <input.md> 整理成 StudyNotes 风格笔记，输出到 <output.md>。要求保留技术准确性，压缩重复解释，添加目录、核心代码块、分段说明和必要示例。
```
