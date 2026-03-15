# Content Restructuring Rules

Use this document for content shaping only.

It applies to `Rewrite` and the content-writing step inside `Generate`.

It does NOT define:

- frontmatter
- file moves
- related-note linking
- MOC updates
- git flow
- diff presentation
- split-file workflow

## Layering Model

Content rewrite uses two layers:

- `note_type`: determines what kind of note is being rewritten
- `rewrite_profile`: determines the default output shape for that type

The supported `note_type` values are:

- `technical_note`
- `tutorial_step`
- `meeting_note`
- `reading_excerpt`
- `draft_note`

Each `note_type` has one default `rewrite_profile` for now.

## Common Goals

- **中度压缩**：删除明显冗余、重复、情绪化、装饰性内容
- **以好理解为第一优先级**：保证逻辑连续、概念清楚，不因压缩导致断层
- **突出核心价值**：原则、方法、结论、可执行要点
- **结构化呈现**：层级清晰，方便快速回看与长期复习

## Allowed Additions

允许补充少量解释性内容，但只能用于：

- 澄清隐含逻辑
- 补全必要前因后果
- 将零散结论串成可理解的链路

所有补充都必须服务于理解原文，而不是扩展原文。

## Hard Constraints

1. 不改变原意：以“忠实 + 更清晰”为准则。
2. 不解释整理过程，不写元描述文字。
3. 不使用口语化表达，不写废话，不使用表情。
4. 输出必须可直接复制使用。
5. 数学公式只用 LaTeX 语法。
6. 图示只用 PlantUML、Mermaid 或 SVG。

## What to Remove

- 重复内容
- 情绪化或主观化表达
- 大段原文照抄且未被提炼的段落
- 已明显过时的版本号、失效链接、旧环境配置
- 对理解核心内容无帮助的背景铺垫
- 零散碎片、失效 TODO、未整理草稿
- 无信息量的过渡句，例如：
  - “接下来我们来看看……”
  - “众所周知……”
  - “下面我们详细介绍一下……”
  - “首先我们需要了解……”
  - “话不多说，直接上代码……”

## What to Preserve

- 关键术语与定义
- 因果关系和必要上下文
- 条件限制与适用边界
- 方法和流程的先后顺序
- 关键代码示例
- 对比表格
- 流程图或架构图
- 对理解关键概念有帮助的例子
- 面试相关问答中的有效结论

## Output Style

- 使用清晰层级标题和项目符号
- 表述偏结论化、工程化，但保留必要上下文
- 能用短句说清的地方，不写成长段
- 标题结构应服务内容，而不是机械套模板

## Note-Type Detection Signals

Use title, the first 30-50 lines, heading structure, code blocks, quote blocks, and semantic keywords together.

Do NOT rely on a single keyword. Use weighted signals and choose the strongest pattern.

If the top candidates are too close and would produce materially different outputs, ask the user.

If the note is obviously mixed, loose, or weakly structured and no type wins clearly, fall back to `draft_note`.

### `technical_note`

Strong signals:

- 概念定义、原理解释、机制分析
- 对比表、架构图、公式、代码块
- 标题常见词：原理、机制、实现、区别、设计、模型、优化

Typical information shape:

- 概念
- 背后逻辑
- 机制或流程
- 边界与注意事项
- 示例

### `tutorial_step`

Strong signals:

- 明显步骤顺序
- 命令清单或配置片段
- 安装、配置、部署、排障、校验
- 标题常见词：如何、安装、配置、部署、排查、实操、步骤

Typical information shape:

- 目标
- 前提条件
- 操作步骤
- 验证方式
- 常见问题

### `meeting_note`

Strong signals:

- 日期时间、参会人、议题
- 决策、行动项、负责人、截止时间
- 讨论纪要、状态同步、待确认问题

Typical information shape:

- 主题
- 结论
- 决策
- 行动项
- 风险或待确认事项

### `reading_excerpt`

Strong signals:

- 引用块、书名、文章名、作者、章节
- 原文摘录比例高
- 观点总结、论据整理、启发
- 标题常见词：摘录、书摘、读书、课程、文章、章节

Typical information shape:

- 主题
- 核心观点
- 支撑论据或例子
- 启发或结论

### `draft_note`

Strong signals:

- 碎片短句或未完成提纲
- 主题跳跃
- TODO、想法、待补充、疑问较多
- 结构不稳定或多次换话题

Typical information shape:

- 主线归并
- 去重后的几点核心内容
- 未解决问题

## Default Rewrite Profiles

The current system uses one default `rewrite_profile` per `note_type`.

### `technical_note` → concept / mechanism / boundary / example

Required blocks:

- 这是什么
- 核心机制或原理
- 关键步骤或推导
- 边界、限制、注意事项

Optional blocks:

- 示例
- 代码
- 对比

Avoid:

- 把技术笔记压缩成只剩结论的速记卡

### `tutorial_step` → goal / prerequisite / steps / verification

Required blocks:

- 要做什么
- 前提条件
- 操作步骤
- 结果校验

Optional blocks:

- 常见错误
- 回滚方式
- 命令示例

Avoid:

- 把可执行步骤改写成大段原理说明

### `meeting_note` → topic / decisions / action items / open issues

Required blocks:

- 讨论主题
- 结论或决策
- 行动项
- 待确认问题

Optional blocks:

- 背景
- 争议点
- 时间安排

Avoid:

- 保留大段流水账发言顺序

### `reading_excerpt` → topic / key ideas / supporting points / takeaways

Required blocks:

- 主题
- 核心观点
- 支撑论据或例子
- 启发或结论

Optional blocks:

- 引用摘录
- 章节映射
- 延伸问题

Avoid:

- 把摘录整理成技术教程式结构

### `draft_note` → main threads / merged points / open questions

Required blocks:

- 这篇草稿主要在说什么
- 归并后的几条主线
- 未解决问题

Optional blocks:

- 后续待补充
- 可能拆分的话题

Avoid:

- 为了整齐硬造完整结论

## Formatting Guidance

The exact headings are flexible.

Keep section names natural for the note's language and content.

The profile tells you what information must survive the rewrite, not the exact heading strings you must use.
