[中文](README.zh-CN.md)

# 🗂 obsidian-organize

> Guided automation for Obsidian notes — classify, structure, and update indexes without losing control

![Knowledge Graph](assets/graph.png)

▲ Organized Obsidian knowledge graph — PARA classification + auto-generated bidirectional links

## Why obsidian-organize?

As your notes grow, things get messy:

- 📁 Hundreds of files dumped in one folder, impossible to find anything
- 🏷 You want tags but never bother to categorize manually
- 🔗 You know your notes are related but never create links
- 📑 You want MOC indexes but always forget to update them

**obsidian-organize automates the repetitive parts.**
Tell Claude to organize a note, and it will:

1. Read and understand content, auto-classify into PARA folders
2. Generate complete frontmatter (title, category, tags, summary)
3. Maintain managed graph content such as related-note sections and MOCs
4. Report ambiguous notes instead of silently guessing

On first `Organize` or `Generate` use, it proposes a short vault config wizard and writes `.obsidian-organize.yml` only after your approval.

If the vault is using git, single-note requests use one git commit. Batch requests use up to two commits: note-local changes first, shared graph updates second. If you decline `git init`, the skill can continue without commit-based rollback.

## ✨ Three Modes

### Rewrite — "simplify this note"

Strip redundancy, keep core logic and code examples. Original auto-backed up to Archives. Rewrite does not touch metadata, category, or links.

Say "rewrite note, format only" to just clean up formatting (extra spaces, inconsistent punctuation, heading levels) without changing content.

### Organize — "organize note"

Auto-classify → generate metadata → move to folder → maintain managed blocks (`frontmatter`, `## Related Notes`, MOCs). Organize does not rewrite the main note body unless you explicitly run Rewrite first.

### Generate — "generate a note about XXX"

Four input sources:

- 💬 Give a topic, generate from scratch
- 🗣 Turn the current conversation into a note
- 🔗 Fetch content from a URL and generate
- 📋 Paste content and organize into a note

Generate creates a draft from one of these sources, then hands that draft into the standard organize pipeline.

## 📸 Before & After

### Before

```
vault/
├── random-thoughts.md
├── study-notes.md
├── interview-prep.md
├── some-ideas.md
├── proxy-explained.md
└── ...(200+ notes scattered in root)
```

### After

```
vault/
├── Areas/
│   ├── networking/
│   ├── study-habits/
│   └── ...
├── Projects/
├── Resources/
│   └── images/
├── Archives/
├── MOCs/
│   ├── Networking Knowledge Map.md
│   └── ...
└── Inbox/
```

![PARA folder structure](assets/sidebar.png)

## 🚀 Quick Start

### Install

```bash
npx skills add https://github.com/HuangMingwang/obsidian-organize
```

### Usage

In Claude Code, `cd` to your Obsidian vault directory, then say:

| You say | Effect |
|---------|--------|
| "organize note xxx.md" | Classify, fill metadata, move note, update related notes + MOC |
| "simplify note xxx.md" | Rewrite content only, backup original |
| "rewrite note, format only" | Clean up formatting without changing content |
| "generate a note about Docker" | Generate a draft + run the standard organize flow |
| "rewrite note Inbox/" | Batch-rewrite a folder with bounded batches and progress reporting |
| "organize this folder" | Batch organize a folder, skip ambiguous items, finalize links + MOCs at the end |

On first `Organize` or `Generate` run, it proposes a bounded config wizard for your vault and writes `.obsidian-organize.yml` after approval.

## Large Batch Requests

When you point obsidian-organize at a folder or a very large set of notes, it first counts the candidates, tells you the scope, then processes them in small batches with visible progress.

Ambiguous items are skipped and summarized at the end instead of blocking the whole batch. Shared graph updates such as related-note sections, reverse links, and MOCs are written in one final pass. If the runtime supports concurrency, only note-local work is parallelized. Batch requests use up to two commits rather than one commit per note.

## 📖 Methodology

Built on three proven note-taking methods:

- **PARA** — Organize by Projects / Areas / Resources / Archives, giving every note a home
- **MOC (Map of Contents)** — Maintain index pages per domain, replacing deep folder hierarchies
- **Zettelkasten** — Build semantic connections between notes through bidirectional links

obsidian-organize automates the repetitive maintenance around all three — you keep control over the structure.

## 🔒 Safety

- If git is enabled, single-note requests use one git commit; batch requests use up to two commits
- Originals auto-backed up to Archives before rewriting
- Asks you when a single note is ambiguous, and skips ambiguous batch items instead of guessing
- Preserves `![[image.png]]` wiki-image embeds in place
- Never deletes any files

## 📄 License

MIT
