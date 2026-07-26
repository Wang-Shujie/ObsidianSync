---
name: save
description: Save a piece of knowledge into the Obsidian vault — analyze content, classify into PARA, pick a note template, generate tags, detect bidirectional links, then WRITE only after the user confirms the plan. Use when the user says "保存知识/笔记", "@save", "记一下", "存一下", "把这个记下来", "归档这条", or provides knowledge text to store. For URL content use the import Skill instead.
---

# save — 智能存储知识

先读 `${CLAUDE_PLUGIN_ROOT}/references/config.md`（路径 + 约定 + 确认卡片）与 `${CLAUDE_PLUGIN_ROOT}/references/classification.md`（PARA / 标签 / 链接 / 写入规范）。

若用户输入是 URL，提示改用 `obsidiansync:import`。

## 流程
1. **理解内容**：通读，提炼标题候选 + 核心要点。
2. **分类**：按 `classification.md` 的 PARA 判定 + 笔记类型→模板，给出：
   - 目标 PARA 目录（00/10/20/30/40）+ 一句理由
   - 笔记类型 → 套用哪个模板（概念 / 项目 / 资源 / 日常）
   - 不确定时一律落 `00_Inbox`，不强行猜测。
3. **生成标签**：按标签规则，优先复用库内已有标签，≤5 个。
4. **检测链接**：扫描库内现有笔记标题与 frontmatter `aliases`，找出本篇内容中可链接概念 → `[[候选]]` 列表（≤5 条）。
5. **展示方案（✅必须停止让用户确认）**：用 `config.md` 的确认卡片格式列出 路径 / 模板 / frontmatter / 标签 / 链接 / 摘要，请用户确认或修改。
6. **写入**：用户同意后，读对应模板 → 填充 frontmatter（`created`/`updated`=今天）与正文 → Write 落盘。
7. **报告**：返回写入路径与已建立的链接，提示可在 Obsidian 中打开。

> 双向链接零维护：在新笔记写 `[[X]]`，Obsidian 自动生成反向链接；无需手改被链接笔记。仅当用户希望对方显式列出关联时，才在其 `## 🔗 关联` 段补一条。
