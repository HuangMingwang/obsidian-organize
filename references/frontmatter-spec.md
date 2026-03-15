# Frontmatter Schema & Generation Rules

## Schema

```yaml
---
title: 笔记标题
category: Areas/Java
tags:
  - 并发
  - JVM
created: 2026-03-14
updated: 2026-03-14
status: draft | active | archived
summary: 一句话描述这篇笔记的核心内容
---
```

## Field Generation Rules

### title
- Use existing H1 heading (`# xxx`) if present.
- Otherwise derive from filename (remove `.md` extension).

### category
- Set to the PARA folder path determined by the categorization decision tree.
- Format: `Areas/Java`, `Projects/面试准备`, `Resources/生活`, etc.

### tags
- Extract 2-5 fine-grained topic keywords from note content.
- Use Chinese tags to match the vault's language.
- Tags should enable cross-domain discovery. Examples:
  - A Redis note: `[Redis, 缓存, NoSQL]`
  - A Java concurrency note: `[并发, 线程池, JUC]`
  - A blockchain note: `[以太坊, 智能合约, Solidity]`
- Do NOT duplicate the category as a tag (e.g., don't tag "Java" on a note in `Areas/Java/`).

### created
- Use file creation date if available (run `stat -f %SB -t %Y-%m-%d <file>` on macOS).
- If unavailable, use today's date.

### updated
- Always set to today's date when modifying.

### status
- `draft`: Note has not been simplified/organized yet.
- `active`: Note has been simplified and is in clean, usable form.
- `archived`: Note is outdated or no longer relevant.
- Default to `draft` for unprocessed existing notes.
- Set to `active` after successful simplification.
- Set to `active` for newly generated notes (Mode B).

### summary
- Generate a single sentence in Chinese summarizing the note's core content.
- Should be informative enough to understand the note without opening it.
- Used in MOC index entries and potential .base database views.
- Examples:
  - "Java 线程池的核心参数、工作原理和最佳实践"
  - "Raft 共识算法的选举、日志复制和安全性机制"
  - "英语定语从句的分类、关系词选择和易错点"

## Handling Existing Frontmatter

- Preserve all existing fields — do NOT overwrite user-set values.
- Only fill in fields that are missing.
- Always update the `updated` field to today's date.
- If existing frontmatter has non-standard fields, preserve them as-is.
