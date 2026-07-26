---
name: maintain
description: Run a read-only health check on the Obsidian vault — broken wikilinks, orphan notes, missing or invalid frontmatter, stale Inbox items, tag drift — and report findings by severity; fix only on per-item confirmation. Use when the user says "健康检查/维护", "@maintain", "检查知识库", "清理一下", "体检", or wants vault hygiene.
allowed-tools: Read, Grep, Glob, Bash, Edit
---

# maintain — 知识库健康检查

先读 `${CLAUDE_PLUGIN_ROOT}/references/config.md`；标签漂移判定参考 `${CLAUDE_PLUGIN_ROOT}/references/classification.md`。

## 扫描项（只读）
1. **断链**：`[[X]]` 中 `X.md` 在库内不存在 → 建议创建占位笔记或修正拼写。
2. **孤儿笔记**：无任何出入链 → 建议补链接或归档。
3. **frontmatter 缺失/非法**：缺 `tags`/`created`/`updated`，或日期格式错误。
4. **Inbox 积压**：`00_Inbox` 中超过 7 天（可调）未处理 → 提醒分类。
5. **标签漂移**：疑似拼写不一致的标签（如 `机器学习`/`ML`/`machine-learning` 并存）→ 建议合并。

## 输出
按严重度分级：🔴 需处理 / 🟡 建议处理 / 🟢 良好。

## 修复
逐项询问是否修复（✅必须停止让用户确认）；修复用 Edit（最小 diff），不批量重写。每步报告改动路径。
