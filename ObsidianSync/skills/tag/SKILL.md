---
name: tag
description: Inventory and manage tags in the Obsidian vault — list tags with counts, and rename, merge, or delete tags. Each mutating step asks for confirmation before editing notes. Use when the user says "标签管理", "@tag", "列出标签", "合并标签", "重命名标签", "清理标签", or wants to tidy up tags.
allowed-tools: Read, Grep, Glob, Bash, Edit
---

# tag — 标签管理

先读 `${CLAUDE_PLUGIN_ROOT}/references/config.md`。

## 流程
1. **盘点**：`rg "(^tags:|#)[^\s#]+" -o KB_ROOT` 统计每个标签出现次数，按频次降序输出（标签来自 frontmatter `tags` 与正文 `#标签`）。
2. **操作（✅每一步必须停止让用户确认）**：
   - **重命名** A→B：用 Edit 替换 frontmatter `tags` 数组项与正文 `#A`。
   - **合并** A→B：同重命名，先确认 A、B 语义相同。
   - **删除** A：移除所有 `tags` 中的 A 与正文 `#A`。
3. **输出**：整理后的标签树（按 PARA 或主题分组），便于回顾。

> 合并前先在盘点结果里核对 A、B 是否真为同一概念，避免误并。
