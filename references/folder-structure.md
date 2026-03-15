# PARA Folder Structure & Categorization Rules

## Generic Folder Tree

```
📁 Projects/                 # 有截止日期的具体项目
📁 Areas/                    # 长期关注的领域（子目录由 .obsidian-organize.yml 定义）
📁 Resources/                # 参考资料（子目录由 .obsidian-organize.yml 定义）
📁 Archives/                 # 归档
  📁 原始版本/                # 精简前的备份
📁 MOCs/                     # 索引笔记
📁 Inbox/                    # 新笔记默认落地点
```

Areas and Resources sub-folders are NOT hardcoded here. They are defined in the vault's `.obsidian-organize.yml` config file.

## Categorization Decision Tree

1. **Read `.obsidian-organize.yml`** to load the `areas` and `resources` sections with their keywords.
2. **Has a project deadline or deliverable?** → `Projects/<项目名>/`
3. **Match note content against `areas` keywords** → Place in the first matching `Areas/<子目录>/`
4. **Match note content against `resources` keywords** → Place in `Resources/<子目录>/`
5. **File type match** (PDF → `Resources/PDFs/`, image → `Resources/图片/`)
6. **No match found?** → Leave in `Inbox/` and ask the user.

## Keyword Matching Logic

For each area defined in config, scan the note's:
- Title / H1 heading
- H2 headings
- First 50 lines of body content

If multiple areas match, pick the one with the **most keyword hits** as primary. Use tags for secondary areas.

## Cross-Domain Notes

When a note spans multiple areas:
- Place in the **primary** area (highest keyword match count).
- Use `tags` in frontmatter for cross-referencing secondary areas.
- Bidirectional linking will connect it to related notes in other areas.

## New Categories

If content clearly doesn't fit any area defined in config:
- Propose a new sub-folder under `Areas/` to the user.
- Only create after user confirmation.
- After confirmation, remind the user to add the new area to `.obsidian-organize.yml` for future use.
