---
name: recall
description: Retrieve relevant notes from the Obsidian knowledge base to ground your current answer or task. SELF-INVOKE autonomously — before responding to a question, while planning a step, or producing content — whenever the user's existing notes might hold relevant prior knowledge, even if the user did not ask to search. Searches the vault by keyword/concept/tag and returns distilled relevant context to fold into your response. Read-only and cheap, so err on the side of calling it. Use when you think "let me check if I have a note on this." Distinct from query, which serves explicit user search requests.
allowed-tools: Read, Grep, Glob, Bash
---

# recall — 自发检索知识库做 grounding

先读 `${CLAUDE_PLUGIN_ROOT}/references/config.md` 取 KB_ROOT 与工具约定。

本 Skill 是给**你自己**用的：任务进行中，觉得库里可能有相关旧笔记时，主动调它检索，把结果折进当前回答。**不要等用户说"查一下"**——那是 `query` 的活。

## 流程
1. 从当前任务提炼检索词（关键词 / 概念 / 可能的标签）；一次检索可组合多个同义/相关词。
2. 检索：
   - 全文：`rg -l -i "词" KB_ROOT`
   - 标签：`rg "(^tags:|#)标签" KB_ROOT`
3. 命中的高相关笔记用 `Read` 读相关段（不必读全文，够用即止）。
4. 返回**精炼上下文**（非原始 dump）：最多 3–5 条最相关，每条 = 一句要点 + 来源 `[[路径]]`。
5. 把上下文折进当前回答，并标注"来自你的笔记：…"。

## 不臆造
库里没有就如实说"未在知识库找到相关笔记"，不要编内容，也不要把模型自带知识冒充成库里的笔记。

## 防过载（软上限，设宽）
- 同一任务/会话内**自主调用 ≤5 次**（上限刻意放宽，正常使用基本触不到）。
- 优先**少量宽检索**，而非多次窄检索：一次多带几个同义/相关词。
- **去重**：本回合已 recall 过的主题不要重复 recall。
- 想调第 6 次时，先把它合并成一次更宽的检索，而不是继续逐个调。
- 这是只读操作、成本很低，所以**漏调比误调更应避免**——只是别做无意义的重复检索。
