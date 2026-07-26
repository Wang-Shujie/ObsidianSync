---
name: summary
description: Generate or refresh a summary for a single Obsidian note (its 📝 摘要 section), or build a Map-of-Content index note for the whole vault or a folder. Writes only after the user confirms. Use when the user says "生成摘要", "@summary", "总结一下", "给整库做索引", "做个 MOC", "梳理一下", or wants a summary or index.
---

# summary — 生成摘要 / 索引

先读 `${CLAUDE_PLUGIN_ROOT}/references/config.md`。

## 单篇
1. Read 指定笔记。
2. 生成 ≤2 句摘要，刷新其 `## 📝 摘要` 段。
3. 展示（✅必须停止让用户确认） → 用 Edit 替换该段（不动其它内容）。

## 整库 / 目录
1. 扫描目标范围内的笔记（标题 + 一行说明）。
2. 生成 MOC（Map of Content）索引笔记：按 PARA 分组，每条 `[[笔记]] — 一行说明`。
3. 展示结构（✅必须停止让用户确认） → Write 到 `20_Areas/知识库索引.md`（或用户指定路径）。
4. 报告路径。

> 索引笔记只放标题与一行说明，不抄正文；保持可导航、低维护。
