# Frontmatter Schema & Generation Rules

This document defines note metadata only.

Flow decisions such as when to organize, when to rewrite, and when to commit belong to `SKILL.md`.

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
source: https://example.com/article
---
```

`source` is optional. Use it when a note is generated from a URL or another external source that should be preserved.

## Field Generation Rules

### title

- Use existing H1 heading (`# xxx`) if present.
- Otherwise derive from filename (remove `.md` extension).

### category

- Set to the PARA folder path determined by the categorization rules.
- Format: `Areas/Java`, `Projects/面试准备`, `Resources/生活`, etc.

### tags

- Extract 2-5 fine-grained topic keywords from note content.
- Use the vault's primary language from `.obsidian-organize.yml` unless the user explicitly wants another language.
- Tags should enable cross-domain discovery.
- Do NOT duplicate the category name as a tag when that would add no value.

### created

- Use file creation date if available (run `stat -f %SB -t %Y-%m-%d <file>` on macOS).
- If unavailable, use today's date.

### updated

- Always set to today's date when modifying.

### status

- `draft`: note exists but has not entered the managed organize flow yet.
- `active`: note has entered the managed state through `Organize` or `Generate`.
- `archived`: note is outdated or no longer relevant.
- Default to `draft` for unprocessed existing notes.
- Set to `active` after successful organization.
- Set to `active` for newly generated notes.

`active` does NOT mean the note has been rewritten. Rewrite and metadata management are separate concerns.

### summary

- Generate a single sentence summarizing the note's core content.
- Use the vault's primary language unless the user explicitly asks for another language.
- Make it informative enough to understand the note without opening it.
- This summary is reused in MOC entries and other index views.

### source

- Preserve the original URL or upstream source when the note is generated from external material.
- Do not invent a `source` value for notes that do not have one.

## Handling Existing Frontmatter

- Preserve all existing fields. Do NOT overwrite user-set values.
- Only fill in fields that are missing.
- Always update the `updated` field to today's date.
- If existing frontmatter has non-standard fields, preserve them as-is.
