# 🗂 obsidian-organize

> 一句话让 Claude 帮你整理 Obsidian 笔记 — 自动分类、打标签、建链接、更新索引

![关系图谱](assets/graph.png)

▲ 整理后的 Obsidian 知识图谱 — PARA 分类 + 双向链接自动生成

## 为什么需要 obsidian-organize？

笔记越记越多，但——

- 📁 文件全堆在一个文件夹里，找不到
- 🏷 想打标签但懒得手动分类
- 🔗 知道笔记之间有关联，但从来不建链接
- 📑 MOC 索引想维护但总是忘记更新

**obsidian-organize 让这些全自动完成。**
只需对 Claude 说「整理笔记」，它会：

1. 读懂内容，自动归入 PARA 文件夹
2. 生成完整的 frontmatter（标题、分类、标签、摘要）
3. 找到相关笔记，添加双向链接
4. 更新对应 MOC 索引页

每条笔记一次 git commit，不满意随时回滚。

## ✨ 三种模式

### 重写 — 「帮我精简这篇笔记」

去掉冗余和废话，保留核心逻辑和代码示例。原文自动备份到 Archives。

### 整理 — 「整理笔记」

自动分类 → 生成元数据 → 移动到对应文件夹 → 添加双向链接 → 更新 MOC。

### 生成 — 「生成一篇关于 XXX 的笔记」

支持四种输入源：

- 💬 给个主题，从零生成
- 🗣 把当前对话整理成笔记
- 🔗 从 URL 抓取内容生成
- 📋 粘贴一段内容整理成笔记

生成后自动执行完整的整理流程（分类 + 链接 + MOC）。

## 📸 效果展示

### 整理前

```
vault/
├── 乱七八糟的笔记1.md
├── 学习笔记.md
├── 面试准备.md
├── 一些想法.md
├── proxy原理.md
└── ...（200+ 篇笔记散落在根目录）
```

### 整理后

```
vault/
├── Areas/
│   ├── 计算机网络/
│   ├── 学习习惯/
│   └── ...
├── Projects/
├── Resources/
│   └── 图片/
├── Archives/
├── MOCs/
│   ├── 计算机网络 MOC.md
│   └── ...
└── Inbox/
```

![PARA 文件夹结构](assets/sidebar.png)

## 🚀 快速开始

### 安装

```bash
npx skills add https://github.com/HuangMingwang/obsidian-organize
```

### 使用

在 Claude Code 中 `cd` 到你的 Obsidian vault 目录，然后说：

| 你说 | 效果 |
|------|------|
| 「整理笔记 xxx.md」 | 自动分类、打标签、建链接、更新 MOC |
| 「精简笔记 xxx.md」 | 精简内容，原文备份 |
| 「生成一篇关于 Docker 的笔记」 | 从零生成 + 自动整理 |
| 「整理这个文件夹」 | 批量整理目录下所有笔记 |

首次运行会自动扫描 vault，生成分类配置（`.obsidian-organize.yml`），确认后开始工作。

## 📖 背后的方法论

本项目基于三套经典笔记管理方法：

- **PARA** — 按 Projects / Areas / Resources / Archives 四层分类，让每条笔记都有归属
- **MOC（Map of Contents）** — 为每个领域维护索引页，替代传统文件夹层级导航
- **Zettelkasten（卡片笔记法）** — 通过双向链接建立笔记间的语义关联

obsidian-organize 把这三套方法论的日常操作全部自动化，你只需要专注写笔记。

## 🔒 安全机制

- 每条笔记单独 git commit，不满意可逐条回滚
- 重写前自动备份原文到 Archives
- 分类不确定时会询问你，不会乱猜
- 不会删除任何文件

## 📄 License

MIT
