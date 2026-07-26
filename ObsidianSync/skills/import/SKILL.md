---
name: import
description: Import external content from a URL into the Obsidian vault as a resource note — fetch the page, extract title/author/source/key points, then run the save flow into 30_Resources with source_url filled, writing only after the user confirms. Use when the user says "导入", "@import", "收藏这篇文章", "存一下这个链接", "抓一下这个网页", or provides a URL to capture into the vault.
---

# import — 导入外部内容（URL）

先读 `${CLAUDE_PLUGIN_ROOT}/references/config.md` 与 `${CLAUDE_PLUGIN_ROOT}/references/classification.md`。

## 流程
1. **抓取**：用 WebFetch（或 mcp `web_reader`）取正文，提取 标题 / 作者 / 来源 / 核心要点。
   - 抓取失败：报告原因，**不臆造**内容，停止。
2. **走 save 流程**：作为「资源」类型：
   - 目标目录 `30_Resources`
   - 模板 `资源笔记模板.md`
   - frontmatter 填 `source_url` / `author` / `type`（文章/书籍/工具/视频/论文）
   - 按 classification.md 生成标签 + 检测链接
3. **展示方案（✅必须停止让用户确认）**：用确认卡片列出 路径 / frontmatter / 摘要 / 标签 / 链接。
4. **写入**：用户确认后 Write，报告路径。

> 仅抓取公开可访问的 URL；遇到需要登录 / 付费的内容，告知用户并停止。
