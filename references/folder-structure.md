# PARA Folder Structure & Categorization Rules

This document defines where notes should live after categorization.

It assumes the config contract documented in `SKILL.md`, especially the fields:

- `areas`
- `resources`
- `language`

## Generic Folder Tree

```text
Projects/                 # 有截止日期的具体项目
Areas/                    # 长期关注的领域（子目录由 .obsidian-organize.yml 定义）
Resources/                # 参考资料（子目录由 .obsidian-organize.yml 定义）
Archives/                 # 归档
  原始版本/                # Rewrite 备份
MOCs/                     # 索引笔记
Inbox/                    # 新笔记默认落地点
```

Areas and Resources sub-folders are not hardcoded here. They are derived from the vault config.

## Categorization Decision Tree

1. Read `.obsidian-organize.yml` and load `areas` and `resources`.
2. If the note has a concrete deadline or deliverable, place it in `Projects/<项目名>/`.
3. Otherwise match note content against `areas` keywords and place it in the best matching `Areas/<子目录>/`.
4. If no area matches, match against `resources` keywords and place it in `Resources/<子目录>/`.
5. If no keyword rule matches:
   - keep the note in `Inbox/`
   - ask the user before inventing a new category

## Keyword Matching Logic

For each candidate category, scan:

- title or H1 heading
- H2 headings
- the first 50 lines of body content

If multiple categories match, choose the one with the highest keyword hit count as the primary destination.

## Cross-Domain Notes

When a note spans multiple areas:

- place it in the primary area
- use tags for secondary concepts
- rely on bidirectional linking for cross-area navigation

## New Categories

If content clearly does not fit any configured area or resource:

- propose a new sub-folder to the user
- create it only after user confirmation
- remind the user to add the new category to `.obsidian-organize.yml`

## File-Type Defaults

If the config does not provide a more specific resource destination, these defaults are acceptable:

- PDFs → `Resources/PDFs/`
- local images handled by image migration → `Resources/图片/`
