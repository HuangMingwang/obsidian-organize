# MOC Format & Bidirectional Linking Rules

## MOC Template

Each Area gets one MOC file in `MOCs/`. Naming: `<Area名> 知识地图.md`.

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

## JVM
- [[JVM]] - JVM 内存模型与 GC
```

## MOC Generation

MOC files are dynamically generated based on the `areas` and `moc_naming` fields in `.obsidian-organize.yml`.

For each area in config, the MOC file name is determined by the `moc_naming` template (e.g., `"{area_name} 知识地图"` → `Java 知识地图.md`).

Do NOT hardcode MOC file names. Always derive them from the config.

## MOC Update Rules

When adding a note to a MOC:

1. Read the existing MOC file (if it exists).
2. Find the appropriate `##` sub-group based on the note's sub-topic.
3. If no suitable sub-group exists, create a new `##` heading.
4. Add the entry: `- [[Note Title]] - summary from frontmatter`
5. Check for duplicates — do NOT add if the link already exists.
6. Update the MOC's `updated` field in frontmatter.
7. If the MOC doesn't exist yet, create it from the template above.

## Sub-group Guidelines

Sub-groups within MOCs are defined by the `moc_subgroups` field in `.obsidian-organize.yml`.

- If the config has sub-groups for the current area, use them as the initial `##` headings.
- If no sub-groups are configured, infer logical groupings from the note's content.
- Always create new sub-groups as needed when existing ones don't fit.

## Bidirectional Linking Rules

### Finding Related Notes

Use `Grep` to search efficiently. For a note being processed:

1. Extract the note's key concepts (from title, tags, and H2 headings).
2. Search the vault for other notes that mention these concepts:
   - `Grep` for the note's title in other files.
   - `Grep` for the note's primary keywords in other files.
3. Filter results: only include notes that are **truly related** (same domain or conceptual dependency).

### What Counts as Related

- Notes that explain prerequisite concepts (e.g., JMM is prerequisite for Volatile).
- Notes that cover the same system/component from different angles.
- Notes that reference each other's core concepts in their content.

### What Does NOT Count

- Notes that merely share a tag but have no conceptual connection.
- Notes in the same folder but about unrelated sub-topics.
- MOC index notes (don't link to MOCs from individual notes).

### Inserting Links

Add a `## 相关笔记` section at the **end** of the note (before any existing trailing content like references or footnotes):

```markdown
## 相关笔记
- [[Java 并发编程核心]] - 并发编程总览
- [[Java 并发编程核心：CAS 原理与硬件底层机制]] - 无锁原子操作
```

Rules:
- Do NOT insert links inline in the note's body text.
- Each entry: `- [[Title]] - one-sentence description`
- Limit to 5-8 most relevant links. Don't over-link.
- Sort by relevance (most related first).

### Bidirectional Update

When adding A → B:
1. Open note B.
2. Check if B already has a `## 相关笔记` section.
3. If not, create it.
4. Check if A is already linked in B. If not, add it.
5. Save B.
