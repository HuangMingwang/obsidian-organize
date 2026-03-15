---
name: obsidian-organize
description: "Organize Obsidian notes: Rewrite (simplify content), Organize (categorize/link/index), and Generate (create from topic/conversation/URL/paste). Triggers on: 重写笔记, 精简笔记, 整理笔记, 分类笔记, 生成笔记, 整理对话成笔记, 整理链接成笔记, 整理内容成笔记, 更新MOC, 添加链接, rewrite note, simplify note, organize note, categorize note, generate note"
---

# Obsidian Note Auto-Organizer

Automatically organize Obsidian notes using PARA + MOC + lightweight Zettelkasten methodology.

## Three Modes

| 模式 | 用途 | 独立性 |
|---|---|---|
| **Rewrite** | 精简笔记内容（不动结构） | 完全独立 |
| **Organize** | 分类、元数据、链接、MOC（不改内容） | 完全独立（可串联 Rewrite） |
| **Generate** | 从多种输入源生成新笔记 | 完全独立 |

## Trigger Routing

根据触发词路由到对应模式：

| 触发词 | 路由到 |
|---|---|
| 重写笔记、精简笔记、rewrite note、simplify note | **Rewrite** |
| 整理笔记、分类笔记、organize note、categorize note | **Organize** |
| 更新MOC、添加链接 | **Organize**（子操作） |
| 生成笔记、generate note | **Generate**（主题生成） |
| 整理对话成笔记 | **Generate**（对话提取） |
| 整理链接成笔记 | **Generate**（URL 抓取） |
| 整理内容成笔记 | **Generate**（粘贴内容） |

---

## Vault Path & Configuration

The vault root is the current working directory.

### Mode-to-Config Dependency

`.obsidian-organize.yml` is the canonical config for vault-specific behavior.

- `Rewrite` MAY run without `.obsidian-organize.yml`
- `Organize` and `Generate` REQUIRE `.obsidian-organize.yml`
- If config is missing for `Organize` or `Generate`, pause the normal flow, complete onboarding, write the config, then resume the original request

### Config Contract

This skill currently relies on the following fields:

- `language`: primary language for generated summaries, tags, and note content when the user does not override it
- `areas`: topic areas, each with a stable `name` and `keywords`
- `resources`: resource sub-folders, each with a stable `name` and `keywords`
- `moc_naming`: template for deriving MOC file names
- `moc_subgroups`: suggested sub-groups for each area's MOC
- `defaults.unmatched_category`: fallback destination for notes that match no configured rule
- `defaults.batch_mode`: batch ambiguity policy; current default is `skip_ambiguous`

Only fields that directly drive execution belong in `.obsidian-organize.yml`.

Example:

```yaml
language: zh-CN
areas:
  - name: 计算机网络
    keywords: [TCP, HTTP, 代理, CDN]
resources:
  - name: 文章
    keywords: [摘录, 书摘, 课程]
moc_naming: "{area_name} 知识地图"
moc_subgroups:
  计算机网络: [基础, 传输层, 应用层]
defaults:
  unmatched_category: Inbox
  batch_mode: skip_ambiguous
```

`references/folder-structure.md` and `references/moc-and-linking.md` assume these fields exist and must stay aligned with this contract.

### First-Run Onboarding Wizard

When `Organize` or `Generate` requires config and `.obsidian-organize.yml` is missing, run a bounded onboarding wizard before normal execution.

#### Discovery

The wizard MUST:

1. inventory Markdown files by path and filename
2. inspect headings and early content only
3. cap direct content inspection when the vault is large
4. infer a best-effort initial config instead of trying to solve taxonomy perfectly

Recommended discovery strategy:

- read file path and filename for all candidate Markdown notes
- read the first 20 lines for notes until enough signal is collected
- when the vault is large, sample representative notes across folders instead of deeply reading every note

Do NOT require full-body parsing of the entire vault before proposing a config.

#### Review

Present the proposed config to the user in bounded review steps:

1. primary language
2. proposed `Areas`
3. proposed `Resources`
4. `moc_naming`
5. fallback behavior for unmatched notes

Use human-readable summaries first, not raw YAML as the primary interface.

#### Completion

After the user confirms the proposal:

1. write `.obsidian-organize.yml`
2. re-enter the normal execution pipeline
3. resume the original `Organize` or `Generate` request

## Execution Status

Every targeted note, generated note, or batch item MUST end in exactly one status:

- `done`: completed successfully
- `skipped`: intentionally not processed because of safe, expected ambiguity or incompatibility
- `blocked`: cannot proceed without a user decision or a vault-level prerequisite
- `failed`: execution attempted but an actual error occurred

Batch summaries MUST report these statuses explicitly rather than collapsing all non-success results into a generic failure bucket.

## Execution Pipeline

All modes use the same four execution stages:

1. `Preflight`
2. `Plan`
3. `Apply`
4. `Finalize`

### Stage 1: Preflight

Before changing files:

- route the request to `Rewrite`, `Organize`, or `Generate`
- resolve the request scope: single note, directory, or multi-item generation request
- check whether the resolved mode requires config
- check whether the vault is a git repository
- check for a dirty worktree
- determine whether the request is a batch

`Preflight` makes no content changes.

### Stage 2: Plan

Before writing files, build a concrete execution plan.

For each candidate note, determine:

- whether it is `ready`
- whether it should be `skipped`
- whether it is `blocked`
- what note-local changes belong in `Apply`
- what shared graph updates belong in `Finalize`

`Plan` is where large requests are bounded and ambiguous notes are separated from safe work. Shared outputs are NOT written here.

### Stage 3: Apply

`Apply` performs note-local work only.

Allowed in `Apply`:

- rewrite content or formatting
- generate or complete frontmatter
- create a new generated note
- classify and move a note
- migrate local images tied to the current note

Disallowed in `Apply`:

- editing MOCs
- editing reverse related-note sections on other notes
- any other shared graph mutation

### Stage 4: Finalize

`Finalize` is the ONLY stage allowed to update shared graph outputs.

Use it to:

- compute and write the managed related-note section
- update or create MOCs
- deduplicate shared entries
- produce the final execution report
- record batch-level commits when shared outputs changed

---

## Git Integration

### Git Detection

- Before every execution, check if the vault is a git repository.
- **Not a git repo** → Ask: "需要 git 来支持 diff 审核和版本回滚，是否执行 git init？"
  - User agrees → run `git init`
  - User declines → skip all commit steps, execute remaining flow normally, and report that git rollback is unavailable

### Working Tree Status

- Before execution, check for uncommitted changes (`git status`).
- **Uncommitted changes detected** → Ask: "检测到未提交的变更，建议先 commit 或 stash，是否继续？"
  - User continues → proceed, only commit files changed by this skill
  - User aborts → stop execution

### Commit Strategy

#### Single-Note Requests

| Mode | Message Format | Commit Contents |
|---|---|---|
| Rewrite | `refactor(notes): 重写 {文件名}` | Backup file + rewritten note |
| Rewrite (format-only) | `style(notes): 格式化 {文件名}` | Formatted note only (no backup) |
| Organize | `chore(notes): 整理 {文件名}` | Note-local changes + any shared graph updates triggered by this one note |
| Generate | `feat(notes): 生成 {文件名}` | New note + note-local changes + any shared graph updates triggered by this one note |

#### Batch Requests

Batch requests use AT MOST two commits:

1. an `apply` commit for note-local changes
2. a `finalize` commit for shared graph updates

Do NOT promise one commit per note in batch mode. Shared outputs make that boundary misleading.

### Rollback Guide

- **Rewrite rollback**: `git checkout HEAD~1 -- {file}` to restore previous version, or check `Archives/原始版本/` backup
- **Single-note Organize / Generate rollback**: `git revert <commit>` to undo the request
- **Batch rollback**: revert the `finalize` commit first, then revert the `apply` commit if needed

---

## Large Batch Processing

Applies when the user targets a directory, says "all notes" / "整个文件夹", or the resolved note set is large enough that processing everything as one flat loop would be risky.

### Batch Detection

- Treat the request as a large batch when it resolves to more than 20 notes, or when the user explicitly asks for folder-level / vault-level cleanup.
- Before making changes, enumerate candidate `.md` files under the requested scope.
- Skip `Archives/原始版本/` backups and generated MOCs unless the user explicitly asks to include them.

### Execution Rules

1. **Scope confirmation**: Tell the user how many notes will be processed and the planned batch size before starting large-batch execution. Example: "将处理 84 篇笔记，按每批 5 篇执行，是否继续？"
2. **Small batches only**: Split work into bounded batches. Default to 5 notes per batch; reduce to 3 if notes are unusually long or complex. Do not process an unbounded list as a single job.
3. **Per-note results first**: During `Plan` and `Apply`, work per note and record whether each item is `done`, `skipped`, `blocked`, or `failed`.
4. **Ambiguity handling**: If one note is ambiguous but the rest of the batch is safe, mark that note as `skipped` and continue. Reserve `blocked` for conditions that stop the whole request.
5. **Shared outputs finalize later**: related-note sections, reverse links, and MOC updates are shared outputs and MUST wait until `Finalize`.
6. **Concurrency boundary**: Parallelism is allowed only for isolated work in `Plan` or note-local work in `Apply`. `Finalize` MUST run serially.
7. **Progress reporting**: Report progress after each note or batch using a concrete format such as "Batch 2/6, note 3/5: Java线程池.md".

### Design Principle

Large-batch execution is an optimization layer, not a separate workflow. Reuse the normal single-note rules for Rewrite, Organize, and Generate, but keep shared graph writes inside `Finalize`.

---

## Managed Blocks

This skill distinguishes managed structural content from the main note body.

Managed blocks are:

- frontmatter
- the managed related-note section
- MOC files and their entries

Unmanaged content is the main note body that users primarily read and edit.

Mode boundary rule:

- `Rewrite` may change unmanaged content only
- `Organize` may maintain managed blocks but must not rewrite unmanaged content
- `Generate` creates the initial unmanaged draft, then hands off managed-block work to `Organize`

---

## Mode: Rewrite

Simplify and restructure note content. Does NOT touch metadata, classification, or links.

**Two Rewrite Paths:**
- **Content rewrite**: Simplify and restructure content using a two-layer template system: `note_type` for routing and `rewrite_profile` for output shape. Triggered by「重写笔记」「精简笔记」without format-only qualifier.
- **Format-only**: Only clean up formatting, do not change content. Triggered when user explicitly mentions format cleanup, e.g.「重写笔记，只清理格式」「重写笔记 格式清理」「rewrite note, format only」.

---

### Template: Content Rewrite

### Flow

```
Read note → Backup to Archives/原始版本/ → Simplify content → >300 line check → Write back in place → git commit
```

### Steps

1. **Read note**: Read the target note content.
2. **Backup**: Copy the original file to `Archives/原始版本/{filename}_{YYYYMMDD_HHMMSS}.md`.
3. **Select template layers**:
   - If the user explicitly names a style, note type, or target shape, treat that as an override.
   - Otherwise detect the note's `note_type` from signals in `references/content-restructuring.md`.
   - Supported `note_type` values: `technical_note`, `tutorial_step`, `meeting_note`, `reading_excerpt`, `draft_note`.
   - Each `note_type` maps to a default `rewrite_profile` that controls output structure and information priority.
   - If detection is ambiguous and the top candidates would materially change the output, ask the user. If the note is obviously loose or mixed and no candidate wins clearly, fall back to `draft_note`.
4. **Simplify & Rewrite**: Apply the common rules and the selected `rewrite_profile` from `references/content-restructuring.md`:
   - Remove redundant, repetitive content
   - Remove emotional/subjective expressions
   - Preserve the information shape that matters for the selected `note_type`
   - **Do NOT generate/modify frontmatter** (preserve existing frontmatter as-is)
5. **>300 line check**: If still over 300 lines after simplification, ask user whether to split. On confirmation, create sub-notes in the same directory (no frontmatter, no move). Do NOT add auto-links between sibling sub-notes; linking stays in Organize.
6. **Write back**: Overwrite the original file in place (no move, no rename).
7. **git commit**: `refactor(notes): 重写 {文件名}` — includes backup file + rewritten note.

### What Rewrite Does NOT Do

- Does NOT modify managed blocks such as frontmatter or the managed related-note section
- Does NOT generate/modify frontmatter
- Does NOT move files
- Does NOT add bidirectional links or the managed related-note section
- Does NOT update MOC

### Template Layering

- `note_type` answers: "What kind of note is this?"
- `rewrite_profile` answers: "What information shape should this type default to?"
- The current default mapping is:
  - `technical_note` → concept / mechanism / boundary / example
  - `tutorial_step` → goal / prerequisite / steps / verification
  - `meeting_note` → topic / decisions / action items / open issues
  - `reading_excerpt` → topic / key ideas / supporting points / takeaways
  - `draft_note` → main threads / merged points / open questions

### Review

No inline diff confirmation. User reviews via `git diff HEAD~1` after commit. Unsatisfied → rollback with git.

### Edge Cases

- **Empty file / frontmatter only, no body**: Skip, inform user "无内容可重写"
- **Existing managed related-note section**: Preserve as-is, do not modify
- **Re-rewriting an already rewritten note**: Execute normally (backup again, simplify again)

### Batch Processing

Supports directory-level invocation (e.g., `重写笔记 Areas/Java/`). For large sets, follow the global `Large Batch Processing` rules above. Per-note rewrites still happen in `Apply`, and commit behavior follows the global git rules.

---

### Template: Format-Only

Only clean up formatting. Does NOT change content, meaning, or structure of the text.

### Flow

```
Read note → Clean formatting → Write back in place → git commit
```

### Steps

1. **Read note**: Read the target note content.
2. **Clean formatting**: Apply the following rules:
   - **Whitespace**: Remove trailing spaces, collapse multiple consecutive blank lines into one, remove extra spaces within lines (preserve intentional indentation and code block formatting)
   - **Punctuation**: Unify punctuation usage — use Chinese punctuation in Chinese context, English punctuation in English/code context. Fix mixed punctuation (e.g., `，` followed by `)` → `，` followed by `）`)
   - **Markdown structure**: Ensure heading levels are sequential (no skipping from `##` to `####`), unify list markers (consistent `-` or `*`), add language tags to fenced code blocks where detectable
   - **Preserve as-is**: Code block contents, frontmatter, the managed related-note section, image links, URLs
3. **Write back**: Overwrite the original file in place.
4. **git commit**: `style(notes): 格式化 {文件名}`

### What Format-Only Does NOT Do

- Does NOT rewrite or simplify content
- Does NOT backup to Archives (formatting changes are safely reversible via git)
- Does NOT generate/modify frontmatter
- Does NOT move files, add links, or update MOC

### Batch Processing

Same as content rewrite — supports directory-level invocation and follows the global batch + git rules.

---

## Mode: Organize

Categorize, add metadata, maintain managed graph content, and update indexes. Does NOT rewrite the main note body content.

### Flow

```
Preflight → Plan note-local changes → Optional Rewrite handoff → Apply frontmatter/classify/move/images → Finalize related notes + MOC → git record
```

### Steps

1. **Optional Rewrite handoff**: Organize can run independently. If the user explicitly wants both content cleanup and organization, or agrees after you offer Rewrite first, execute the full Rewrite flow before continuing. Do NOT infer "already rewritten" from backup files or other filesystem traces.
2. **Read config**: Load `.obsidian-organize.yml`. If absent, enter onboarding, write the config, then resume the organize request.
3. **Generate/complete frontmatter**: Read `references/frontmatter-spec.md` for schema.
   - If no frontmatter: generate all fields from content analysis.
   - If frontmatter exists: preserve existing fields, fill missing ones, update `updated` date.
   - Set `status` to `active` (note is now in managed form).
   - Auto-execute, no user confirmation needed.
4. **Classify**: Follow the decision tree in `references/folder-structure.md`: `Projects -> Areas -> Resources -> defaults.unmatched_category`.
   - If no configured rule matches, place the note in `defaults.unmatched_category`.
   - Treat only multiple plausible configured destinations or a proposal to invent a new category as ambiguity.
5. **Image migration**: See Image Migration section below.
6. **Move note**: Move to the PARA folder matching its `category`. Read `references/folder-structure.md` for structure. Create target folder if needed. If same-name file exists at destination, ask user.
7. **Finalize graph updates**: During `Finalize`, read `references/moc-and-linking.md` and update shared outputs.
   - Find related notes from the successful organize results.
   - Add or merge the managed related-note section.
   - Find or create the corresponding MOC in `MOCs/`.
   - Add the note to the appropriate sub-group.
8. **git record**: Single-note requests use one commit. Batch requests follow the global `apply` + `finalize` commit policy.

### What Organize Does NOT Do

- Does NOT rewrite unmanaged note body content (unless the user explicitly runs or approves Rewrite first)

### Auto-Execute vs Confirmation

- **Auto-execute**: frontmatter generation, image migration, file move, and shared graph updates during `Finalize`
- **Needs confirmation**: multiple plausible configured destinations in a single-note request, proposal to invent a new category, same-name file at destination

### Backup Strategy

Organize mode does NOT backup to Archives. Reason: organize changes are structural and rely on git-based rollback. Content-level backup is handled by Rewrite mode.

### Edge Cases

- **Note already in correct location**: Skip move, continue with frontmatter and any `Finalize` graph updates
- **Empty file / frontmatter only**: Classify based on filename and existing tags; if nothing matches, place it in `defaults.unmatched_category`
- **Existing managed related-note section**: Merge and deduplicate, do not add duplicate links

### Batch Processing

Supports directory-level invocation (e.g., `整理笔记 Inbox/`). For large sets, follow the global `Large Batch Processing` rules above. Ambiguous notes may be marked `skipped`, note-local work stays in `Apply`, and graph updates wait for `Finalize`.

---

## Mode: Generate

Create new note drafts from multiple input sources. After draft creation, automatically runs the standard Organize flow.

### Input Sources

#### 1. Topic Generation (existing logic)

Trigger: `生成笔记 {topic}`

- Claude generates a structured note based on its own knowledge.
- Write concise, clear content in the vault's primary language unless the user asks for another language.
- Follow the content rules from `references/content-restructuring.md`.

#### 2. Conversation Extraction

Trigger: `整理对话成笔记`（followed by topic description, e.g., "关于线程池的讨论"）

- Extract relevant content from the current conversation based on the user-specified topic.
- Apply content-restructuring.md rules to organize into a structured note.
- Prioritize: code examples, conclusions, key decisions from the conversation.

#### 3. URL Fetch

Trigger: `整理链接成笔记 {URL}`

- Fetch the target page content (supports HTML pages, PDFs, etc.).
- Extract main body, strip navigation/ads/irrelevant content.
- Apply content-restructuring.md rules to simplify.
- Record `source` URL in frontmatter.
- If URL is inaccessible, inform user and suggest pasting content instead.

#### 4. Paste Content

Trigger: `整理内容成笔记`（user then pastes text）

- Accept arbitrary pasted text from user.
- Apply content-restructuring.md rules to organize.

### Unified Output Flow

Regardless of input source, every generated note automatically goes through:

1. Create or simplify the unmanaged draft body per `references/content-restructuring.md`
2. Hand the draft into the standard `Organize` pipeline
3. Generate complete frontmatter (set `status: active`). Read `references/frontmatter-spec.md`.
4. Classify into correct PARA folder. Read `references/folder-structure.md`.
5. Queue shared graph updates for `Finalize`. Read `references/moc-and-linking.md`.
6. Record the request using the global git policy

### Custom Style

No predefined prompt templates. If the user specifies a style in their request (e.g., "用面试速查的风格整理这个链接成笔记"), adapt the output style accordingly.

### Edge Cases

- **URL inaccessible**: Inform user "无法抓取该链接", suggest pasting content
- **No relevant content in conversation**: Inform user "当前对话中未找到关于 X 的讨论"
- **Very short pasted content (<50 chars)**: Generate normally, no restriction

### Batch Processing

If one request expands into many generated notes (for example, multiple URLs, multiple pasted source items, or a bulk import request), follow the global `Large Batch Processing` rules above. Generate note-local drafts in `Apply`, then aggregate shared graph updates in `Finalize`.

---

## Image Migration

Applies during Organize and Generate modes when moving notes.

### Scan Rules

- Process `![alt](path)` format local image references when a move would otherwise break the path.
- For `![[image.png]]` wiki-image embeds: preserve the embed syntax, keep the image in place, and report that the wiki image was preserved in place. Do NOT migrate wiki images in this redesign.
- Ignore web images (`http://`, `https://` prefix).

### Target Path

`Resources/图片/{Area名}/{原文件名}`

Example: Note classified to `Areas/Java/`, image `img1.png` → `Resources/图片/Java/img1.png`

### Filename Conflict

If same-name image exists at target: append timestamp suffix `img1_20260315_153000.png` (format: `YYYYMMDD_HHMMSS`).

### Path Update

After migrating a standard Markdown image, replace the original path in the note with the new relative path.

### Skip Conditions

- Image file does not exist (broken path) → Skip, no error
- Image already under `Resources/图片/` → Skip, no repeat move

---

## Safety Rules

- NEVER delete notes without explicit user confirmation.
- NEVER overwrite an existing note at the destination without asking.
- Create target folders if they don't exist.
- If no configured category matches, place the note in `defaults.unmatched_category` rather than inventing a new category.
- If multiple configured categories are plausible in a single-note request, ask the user rather than guessing.
- If multiple configured categories are plausible in a batch request and the rest of the work is safe, mark that note as `skipped` and report it.
- Only Rewrite mode creates backups in `Archives/原始版本/`; Organize and Generate rely on git for rollback.

## Language

Follow the vault's primary language from `.obsidian-organize.yml` when generating summaries, tags, note content, and user-facing examples inside notes. If the current mode requires config and the config is missing, complete onboarding before continuing. If `Rewrite` runs without config, preserve the dominant language already present in the note unless the user asks for another language.
