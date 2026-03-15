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

<HARD-GATE>
On EVERY invocation, you MUST first check for the vault-level config file `.obsidian-organize.yml` in the vault root.

**If `.obsidian-organize.yml` EXISTS:**
- Read it and use its rules for all categorization, MOC creation, and folder decisions.

**If `.obsidian-organize.yml` does NOT exist — run Auto-Init:**

1. **Scan the vault**: Use `Glob` to find all `.md` files. Read the first 20 lines of each note (title, headings, key terms).
2. **Cluster by topic**: Analyze all notes and identify 5-15 natural topic clusters based on content similarity, shared terminology, and domain patterns.
3. **Generate config**: For each cluster, derive:
   - An Area name (concise, 2-5 characters in the vault's primary language)
   - 3-8 representative keywords for auto-categorization
   - 2-5 suggested MOC sub-groups
4. **Also detect**:
   - Resources sub-folders (PDFs, images, non-technical content)
   - The vault's primary language (for `language` field)
5. **Present the generated config to the user** for review:
   - Show the proposed Areas, keywords, and sub-groups in a readable format
   - Ask: "这个分类结构可以吗？有需要调整的吗？"
6. **After user confirms**: Write `.obsidian-organize.yml` to vault root.
7. **Then proceed** with the user's original request.

This ensures fully automatic onboarding — new users just run the skill and everything is set up.
</HARD-GATE>

---

## Git Integration

### Git Detection

- Before every execution, check if the vault is a git repository.
- **Not a git repo** → Ask: "需要 git 来支持 diff 审核和版本回滚，是否执行 git init？"
  - User agrees → run `git init`
  - User declines → skip all commit steps, execute remaining flow normally

### Working Tree Status

- Before execution, check for uncommitted changes (`git status`).
- **Uncommitted changes detected** → Ask: "检测到未提交的变更，建议先 commit 或 stash，是否继续？"
  - User continues → proceed, only commit files changed by this skill
  - User aborts → stop execution

### Commit Strategy

| Mode | Message Format | Commit Contents |
|---|---|---|
| Rewrite | `refactor(notes): 重写 {文件名}` | Backup file + rewritten note |
| Rewrite (format-only) | `style(notes): 格式化 {文件名}` | Formatted note only (no backup) |
| Organize | `chore(notes): 整理 {文件名}` | Note move, frontmatter, image migration, links, MOC update |
| Generate | `feat(notes): 生成 {文件名}` | New note + frontmatter + links + MOC update |

Each note gets its own commit for per-note rollback granularity.

### Rollback Guide

- **Rewrite rollback**: `git checkout HEAD~1 -- {file}` to restore previous version, or check `Archives/原始版本/` backup
- **Organize rollback**: `git revert <commit>` to undo the entire organize operation (multi-file: move, MOC, links)

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
3. **Subagents when available**: If the runtime supports subagents, process batches in parallel with bounded concurrency.
   - Run at most 3 batches at the same time.
   - Each subagent must process notes inside its assigned batch sequentially, following the normal per-note flow and safety rules.
   - NEVER spawn one subagent per note for very large note sets.
4. **Sequential fallback**: If subagents are not available, use the exact same batch plan in the main agent and process batches sequentially. Large-batch behavior must still work correctly without subagents.
5. **Progress reporting**: Report progress after each note or batch using a concrete format such as "Batch 2/6, note 3/5: Java线程池.md".
6. **Keep per-note boundaries**: Large-batch mode changes scheduling only. Each note still gets its own backup/commit/review behavior based on the active mode.
7. **Pause on ambiguity**: If a note needs user confirmation (classification mismatch, destination filename conflict, split decision, etc.), pause the affected note and ask. Do not silently guess in order to keep the batch moving.

### Design Principle

Large-batch execution is an optimization layer, not a separate workflow. Reuse the normal single-note rules for Rewrite, Organize, and Generate; only the scheduling and progress reporting change.

---

## Mode: Rewrite

Simplify and restructure note content. Does NOT touch metadata, classification, or links.

**Two templates:**
- **Default (content rewrite)**: Simplify and restructure content. Triggered by「重写笔记」「精简笔记」without format-only qualifier.
- **Format-only**: Only clean up formatting, do not change content. Triggered when user explicitly mentions format cleanup, e.g.「重写笔记，只清理格式」「重写笔记 格式清理」「rewrite note, format only」.

---

### Template: Content Rewrite (Default)

### Flow

```
Read note → Backup to Archives/原始版本/ → Simplify content → >300 line check → Write back in place → git commit
```

### Steps

1. **Read note**: Read the target note content.
2. **Backup**: Copy the original file to `Archives/原始版本/{filename}_{YYYYMMDD_HHMMSS}.md`.
3. **Simplify & Rewrite**: Apply rules from `references/content-restructuring.md`:
   - Remove redundant, repetitive content
   - Remove emotional/subjective expressions
   - Restructure: core concepts → logic → methods → caveats → conclusions
   - Preserve: key terms, code examples, comparison tables, architecture diagrams
   - **Do NOT add `## 相关笔记` section**
   - **Do NOT generate/modify frontmatter** (preserve existing frontmatter as-is)
4. **>300 line check**: If still over 300 lines after simplification, ask user whether to split. On confirmation, create sub-notes in the same directory (no frontmatter, no move). Add `## 相关笔记` cross-links between sibling sub-notes.
5. **Write back**: Overwrite the original file in place (no move, no rename).
6. **git commit**: `refactor(notes): 重写 {文件名}` — includes backup file + rewritten note.

### What Rewrite Does NOT Do

- Does NOT generate/modify frontmatter
- Does NOT move files
- Does NOT add bidirectional links or `## 相关笔记` section (except for split sub-notes)
- Does NOT update MOC

### Review

No inline diff confirmation. User reviews via `git diff HEAD~1` after commit. Unsatisfied → rollback with git.

### Edge Cases

- **Empty file / frontmatter only, no body**: Skip, inform user "无内容可重写"
- **Existing `## 相关笔记` section**: Preserve as-is, do not modify
- **Re-rewriting an already rewritten note**: Execute normally (backup again, simplify again)

### Batch Processing

Supports directory-level invocation (e.g., `重写笔记 Areas/Java/`). For small sets, process one note at a time with its own backup and commit. For large sets, follow the global `Large Batch Processing` rules above; each note still keeps independent backup/commit boundaries.

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
   - **Preserve as-is**: Code block contents, frontmatter, `## 相关笔记` section, image links, URLs
3. **Write back**: Overwrite the original file in place.
4. **git commit**: `style(notes): 格式化 {文件名}`

### What Format-Only Does NOT Do

- Does NOT rewrite or simplify content
- Does NOT backup to Archives (formatting changes are safely reversible via git)
- Does NOT generate/modify frontmatter
- Does NOT move files, add links, or update MOC

### Batch Processing

Same as content rewrite — supports directory-level invocation with per-note commits.

---

## Mode: Organize

Categorize, add metadata, link, and index notes. Does NOT modify note body content.

### Flow

```
Rewrite detection → [optional: chain Rewrite] → Read config → Generate/complete frontmatter → Classify → Migrate images → Move note → Add bidirectional links → Update MOC → git commit
```

### Steps

1. **Rewrite detection**: Check if `Archives/原始版本/` contains a backup for this note.
   - **Not rewritten** → Ask: "该笔记尚未重写，是否先重写再整理？"
     - User agrees → Execute full Rewrite flow (backup → simplify → commit) → Prompt user to review via `git diff HEAD~1` → User confirms → Continue with Organize steps below
     - User declines → Skip rewrite, proceed directly with Organize steps
   - **Already rewritten** → Proceed directly with Organize steps
2. **Read config**: Load `.obsidian-organize.yml`. If absent, run Auto-Init flow.
3. **Generate/complete frontmatter**: Read `references/frontmatter-spec.md` for schema.
   - If no frontmatter: generate all fields from content analysis.
   - If frontmatter exists: preserve existing fields, fill missing ones, update `updated` date.
   - Set `status` to `active` (note is now in managed form).
   - Auto-execute, no user confirmation needed.
4. **Classify**: Match note content against `areas` keywords in config to determine target Area. If no match, ask user.
5. **Image migration**: See Image Migration section below.
6. **Move note**: Move to the PARA folder matching its `category`. Read `references/folder-structure.md` for structure. Create target folder if needed. If same-name file exists at destination, ask user.
7. **Bidirectional links**: Read `references/moc-and-linking.md` for rules.
   - Use `Grep` to find related notes by title/keyword matches.
   - Add `## 相关笔记` section at the end of the note (5-8 links).
   - Update related notes to link back.
8. **Update MOC**: Read `references/moc-and-linking.md` for MOC format.
   - Find or create the corresponding MOC in `MOCs/`.
   - Add the note to the appropriate sub-group.
9. **git commit**: `chore(notes): 整理 {文件名}` — includes all changes from this note's organize operation.

### What Organize Does NOT Do

- Does NOT modify note body content (unless chained Rewrite via rewrite detection)

### Auto-Execute vs Confirmation

- **Auto-execute**: frontmatter generation, image migration, file move, bidirectional links, MOC update
- **Needs confirmation**: classification mismatch (no keyword match), same-name file at destination

### Backup Strategy

Organize mode does NOT backup to Archives. Reason: all Organize changes (frontmatter, move, links) are tracked via git commit and can be rolled back with `git revert <commit>`. Content-level backup is handled by Rewrite mode.

### Edge Cases

- **Note already in correct location**: Skip move, continue with frontmatter, links, MOC update
- **Empty file / frontmatter only**: Classify based on filename and existing tags; if no match, ask user
- **Existing `## 相关笔记` section**: Merge and deduplicate, do not add duplicate links

### Batch Processing

Supports directory-level invocation (e.g., `整理笔记 Inbox/`). For large sets, follow the global `Large Batch Processing` rules above. Each note still gets its own commit, and any note that needs confirmation (classification, name conflict) must pause and ask before continuing that note.

---

## Mode: Generate

Create new notes from multiple input sources. After generation, automatically runs the full Organize flow.

### Input Sources

#### 1. Topic Generation (existing logic)

Trigger: `生成笔记 {topic}`

- Claude generates a structured note based on its own knowledge.
- Write concise, clear content in Chinese.
- Follow the standard template from `references/content-restructuring.md`.

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

1. Simplify content per content-restructuring.md rules
2. Generate complete frontmatter (set `status: active`). Read `references/frontmatter-spec.md`.
3. Classify into correct PARA folder. Read `references/folder-structure.md`.
4. Add bidirectional links. Read `references/moc-and-linking.md`.
5. Update MOC
6. git commit: `feat(notes): 生成 {文件名}`

### Custom Style

No predefined prompt templates. If the user specifies a style in their request (e.g., "用面试速查的风格整理这个链接成笔记"), adapt the output style accordingly.

### Edge Cases

- **URL inaccessible**: Inform user "无法抓取该链接", suggest pasting content
- **No relevant content in conversation**: Inform user "当前对话中未找到关于 X 的讨论"
- **Very short pasted content (<50 chars)**: Generate normally, no restriction

### Batch Processing

If one request expands into many generated notes (for example, multiple URLs, multiple pasted source items, or a bulk import request), follow the global `Large Batch Processing` rules above. Each generated note must run the full output flow independently and receive its own commit.

---

## Image Migration

Applies during Organize and Generate modes when moving notes.

### Scan Rules

- Only process `![alt](path)` format local image references.
- Ignore `![[]]` wiki-link format (preserve compatibility with standard Markdown platforms like Yuque). If a note only contains `![[]]` format images, inform user: "检测到 wiki 链接格式图片，未迁移"
- Ignore web images (`http://`, `https://` prefix).

### Target Path

`Resources/图片/{Area名}/{原文件名}`

Example: Note classified to `Areas/Java/`, image `img1.png` → `Resources/图片/Java/img1.png`

### Filename Conflict

If same-name image exists at target: append timestamp suffix `img1_20260315_153000.png` (format: `YYYYMMDD_HHMMSS`).

### Path Update

After migration, replace the original path in the note with the new relative path.

### Skip Conditions

- Image file does not exist (broken path) → Skip, no error
- Image already under `Resources/图片/` → Skip, no repeat move

---

## Safety Rules

- NEVER delete notes without explicit user confirmation.
- NEVER overwrite an existing note at the destination without asking.
- Create target folders if they don't exist.
- If categorization is ambiguous, ask the user rather than guessing.
- Only Rewrite mode creates backups in `Archives/原始版本/`; Organize and Generate rely on git for rollback.

## Language

The vault is primarily in Chinese. Generate all content (frontmatter summary, tags, note content) in Chinese. Use Chinese folder names as defined in the taxonomy.
