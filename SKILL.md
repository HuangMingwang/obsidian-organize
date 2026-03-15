---
name: obsidian-organize
description: "Organize Obsidian notes with auto-categorization into PARA folders, frontmatter generation, content simplification, bidirectional linking, and MOC index updates. Triggers on: organize notes, clean up vault, generate note, update MOC, add links, simplify note, categorize note, 整理笔记, 生成笔记, 精简笔记"
---

# Obsidian Note Auto-Organizer

Automatically organize Obsidian notes using PARA + MOC + lightweight Zettelkasten methodology.

## Two Modes

- **Mode A: Organize existing notes** — User points to a note (or batch), you categorize, simplify, link, and index it.
- **Mode B: Generate new notes** — User gives a topic, you create a well-structured note with full metadata, links, and MOC entry.

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
7. **Then proceed** with the user's original request (organize/generate note).

This ensures fully automatic onboarding — new users just run the skill and everything is set up.
</HARD-GATE>

---

## Mode A: Organize Existing Notes

Accept a single file path, glob pattern (e.g., `**/Java*.md`), or topic keyword (e.g., "organize all Java notes") for batch processing.

For each note, execute these steps in order:

### Step 1: Read Config & Analyze

1. Read `.obsidian-organize.yml` from vault root to load categorization rules.
2. Read the target note. Determine its primary topic and category by matching content against the `areas` keywords in config.
3. Read `references/folder-structure.md` for the generic PARA structure and decision logic.

### Step 2: Generate/Complete Frontmatter

Read `references/frontmatter-spec.md` for the schema.
- If no frontmatter exists: generate all fields from content analysis.
- If frontmatter exists: preserve existing fields, fill missing ones, update `updated` date.

### Step 3: Ask Whether to Simplify Content

<HARD-GATE>
Before simplifying any content, you MUST ask the user:
"1) 保留原文，只补充标识、关联和分类  2) 重构内容 + 补充标识、关联和分类"

If the user chooses **1**:
- Skip Step 3a and Step 4 entirely.
- Proceed directly to Step 5 (Backup) → Step 6 (Move) → Step 7 (Links) → Step 8 (MOC).
- Do NOT modify the note's body content at all.

If the user chooses **2**:
- Proceed with content simplification below.
</HARD-GATE>

### Step 3a: Simplify Content (only if user opted in)

Read `references/content-restructuring.md` for rules.

<HARD-GATE>
You MUST show the user a clear before/after diff of content changes. Do NOT write any simplified content until the user explicitly approves. This applies to EVERY note, no exceptions.
</HARD-GATE>

- Remove redundant content, filler phrases, copy-pasted raw text.
- Preserve core concepts, minimal code examples, personal insights, structured content.
- Reformat to the standard note template.

### Step 4: Evaluate Split (only if user opted in to simplification)

If the note exceeds 300 lines after simplification:
- Identify logical split points (distinct sub-topics at H2 level).
- Present numbered split suggestions with proposed new titles.
- Only split after user confirmation.
- Each sub-note gets its own frontmatter and is processed through Steps 5-8.

### Step 5: Backup Original

Copy the original file to `Archives/原始版本/`. If a file with the same name already exists, append a timestamp suffix (e.g., `Java_20260314.md`).

### Step 6: Move to Correct Folder

Move the note to the PARA folder matching its `category` frontmatter field. Create the target folder if it doesn't exist.
If a file with the same name exists at the destination, ask the user before overwriting.

### Step 7: Insert Bidirectional Links

Read `references/moc-and-linking.md` for rules.
- Use `Grep` to efficiently find related notes by searching for title/keyword matches.
- Add a `## 相关笔记` section at the end of the note.
- Update related notes to link back (bidirectional).

### Step 8: Update MOC

Read `references/moc-and-linking.md` for MOC format.
- Find or create the corresponding MOC in `MOCs/`.
- Add the note to the appropriate sub-group.

---

## Mode B: Generate New Note

### Step 1: Understand the Topic

Ask clarifying questions if the topic is ambiguous. Determine the target category.

### Step 2: Generate Content

Create the note following the simplified format standard from `references/content-restructuring.md`.
- Write concise, clear content in Chinese.
- Follow the standard template: one-line summary, bullet points, details, code examples (if relevant).

### Step 3: Complete Frontmatter

Auto-fill all frontmatter fields per `references/frontmatter-spec.md`.
Set `status: active` (since it's already in clean format).

### Step 4: Save to Correct Folder

Save directly to the PARA folder (skip Inbox). Read `references/folder-structure.md` for categorization.

### Step 5: Insert Bidirectional Links

Same as Mode A Step 7.

### Step 6: Update MOC

Same as Mode A Step 8.

---

## Batch Processing

When processing multiple notes:
- Process one note at a time, showing progress (e.g., "Processing 3/15: Java线程池.md").
- For content simplification, show diff for each note and wait for confirmation.
- Offer an "approve all remaining" shortcut if the user is satisfied with the quality after reviewing a few.

## Safety Rules

- ALWAYS backup before modifying any note.
- ALWAYS show diff before simplifying content — never auto-write.
- NEVER delete notes without explicit user confirmation.
- NEVER overwrite an existing note at the destination without asking.
- Create target folders if they don't exist.
- If categorization is ambiguous, ask the user rather than guessing.

## Language

The vault is primarily in Chinese. Generate all content (frontmatter summary, tags, note content) in Chinese. Use Chinese folder names as defined in the taxonomy.
