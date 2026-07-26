---
name: link
description: Detect and propose bidirectional links between Obsidian notes, and connect orphan notes that have no inbound/outbound links. Lists specific "insert [[X]] at paragraph N" suggestions and applies only the ones the user picks, after confirmation. Use when the user says "建立链接", "@link", "连一下笔记", "找孤立笔记", "把笔记串起来", or wants to wire notes together.
---

# link — 建立双向链接

先读 `${CLAUDE_PLUGIN_ROOT}/references/config.md` 与 `${CLAUDE_PLUGIN_ROOT}/references/classification.md`（链接检测算法）。

## 流程
1. **找孤立笔记**：列出无 `[[ ]]` 出链且无入链的笔记。
2. **建议链接**：对目标笔记正文做概念匹配（标题 / `aliases`），产出「在第 N 段插入 `[[X]]`」的具体建议，去重，上限 5 条 / 篇。
3. **展示方案（✅必须停止让用户确认）**：列出全部建议，请用户勾选要应用的。
4. **应用**：用 Edit 逐条精确插入（仅在正文首次出现处）。报告改动。

> 不要批量改写正文；只在指定位置插入 `[[ ]]`，保持最小 diff。
