---
name: query
description: Search and retrieve knowledge from the Obsidian vault by keyword, tag, date, or directory, returning ranked matches with the matching snippet and the dimension that matched (title/tag/body/date). Read-only. Use when the user says "查询/搜索知识", "@query", "找一下", "库里有没有", "搜一下笔记", or wants to look something up in the vault.
allowed-tools: Read, Grep, Glob, Bash
---

# query — 检索知识

先读 `${CLAUDE_PLUGIN_ROOT}/references/config.md` 取 KB_ROOT 与工具约定。

## 流程
1. 解析检索条件：关键词 / 标签 / 日期范围 / 目录，可组合。
2. 构造检索：
   - 全文：`rg -l -i "关键词" KB_ROOT`
   - 标签：`rg "(^tags:|#)标签" KB_ROOT`
   - 目录限定：在路径后追加
   - 日期：读 frontmatter `created`/`updated` 过滤
3. 汇总：按相关度排序，每条给出文件路径（可点击）+ 命中片段 + 命中维度（标题 / 标签 / 正文 / 日期）。
4. 无结果时给放宽建议（近义词、去掉某条件、扩大目录）。

只读，无需用户确认。
