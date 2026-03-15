# MOC Format & Bidirectional Linking Rules

This document covers two things only:

- how MOC files are named and updated
- how related-note sections are generated

Flow control still belongs to `SKILL.md`.
Shared-file writes described here happen during `Finalize`, not during note-local `Apply`.

## MOC Naming

Each area gets one MOC file in `MOCs/`.

The file name must be derived from `.obsidian-organize.yml` via `moc_naming`.

Example:

- template: `"{area_name} 知识地图"`
- area: `Java`
- result: `MOCs/Java 知识地图.md`

Do NOT hardcode MOC names in the workflow.

## Example MOC Template

This is an example only. The actual file name and headings should follow the config.

```markdown
---
title: Java 知识地图
category: MOCs
tags:
  - MOC
  - Java
created: 2026-03-14
updated: 2026-03-14
status: active
summary: Java 相关所有笔记的索引
---

## 基础
- [[Java]] - Java 基础知识
- [[Java 集合与容器]] - List/Map/Set 等集合框架

## 并发
- [[Java 并发编程核心]] - 并发编程总览
- [[Java线程池]] - 线程池原理与配置
```

## MOC Update Rules

During `Finalize`, when adding a successful note to a MOC:

1. Read the existing MOC if it exists.
2. Find the best `##` sub-group for the note.
3. If no suitable sub-group exists, create one.
4. Add the entry as `- [[Note Title]] - summary from frontmatter`.
5. Do not add duplicates.
6. Update the MOC's `updated` field.
7. If the MOC does not exist yet, create it using the naming template and example structure above.
8. Batch updates should aggregate candidate entries before editing the file so the MOC is written once per finalize pass.

## Sub-group Guidelines

Sub-groups are primarily defined by `moc_subgroups` in `.obsidian-organize.yml`.

- If config provides sub-groups for the current area, use them first.
- If no configured group fits, infer a small, clear `##` heading from the note's subject.
- Prefer stable, reusable group names over one-off headings.

## Bidirectional Linking Rules

The related-note section heading should follow the vault's primary language from `.obsidian-organize.yml`.

- Chinese vaults: `## 相关笔记`
- English vaults: `## Related Notes`
- If a managed related-note section already exists with the correct meaning, reuse that heading rather than creating a second section

### Finding Related Notes

For a note that completed note-local processing successfully:

1. Extract key concepts from the title, tags, and H2 headings.
2. Search the vault for notes that meaningfully overlap with those concepts.
3. Keep only notes that are genuinely related, not merely in the same folder or sharing a broad tag.

### What Counts as Related

- prerequisite concepts
- the same system viewed from another angle
- direct conceptual dependencies
- notes that clearly refer to each other's core ideas

### What Does Not Count

- notes that only share a broad tag
- notes in the same folder but on unrelated sub-topics
- MOC index notes

### Inserting Links

During `Finalize`, add the managed related-note section near the end of the note, before trailing references or appendices if they exist.

```markdown
## 相关笔记
- [[Java 并发编程核心]] - 并发编程总览
- [[Java 并发编程核心：CAS 原理与硬件底层机制]] - 无锁原子操作
```

Rules:

- Do NOT insert links inline in the main body.
- Each entry should be `- [[Title]] - one-sentence description`.
- Limit to the 5-8 most relevant links.
- Sort by relevance.

### Bidirectional Update

During `Finalize`, when adding A → B:

1. Open note B.
2. Reuse its existing managed related-note section if present.
3. Add A only if it is not already listed.
4. Keep the section deduplicated.

Do NOT update reverse links during note-local `Apply`.
