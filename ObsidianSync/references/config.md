# obsidiansync 共享配置与约定

本插件所有 Skill 执行前先读本文件（路径、工具约定、铁律、确认卡片）。
涉及分类 / 标签 / 链接 / 写入规范的 Skill 另读 `classification.md`。

> 文件定位：本文件位于 `${CLAUDE_PLUGIN_ROOT}/references/config.md`。插件内引用其它文件一律用 `${CLAUDE_PLUGIN_ROOT}/...` 前缀。

## 路径
- **KB_ROOT**（知识库根）按以下优先级解析，取第一个非空值：
  1. 环境变量 `OBSIDIAN_KB_ROOT`（推荐：写入 shell profile 或 Claude Code 环境变量设置，重装插件不丢失）；
  2. 本文件下方的「KB_ROOT 默认值」；
  3. 都没有 → 在会话中询问用户知识库根路径；得到路径后追加询问「是否把该路径保存为默认值（写入 config.md，下次起自动使用）？」
     - 用户确认 → 用 Edit 将下方「KB_ROOT 默认值」行中的占位符替换为该路径（此次「是否保存」确认即写操作授权，无需另发确认卡片；下次起规则 2 自动命中）；
     - 用户拒绝 → 仅本会话沿用，下次会话再次询问。
- **KB_ROOT 默认值**：`<首次使用前改为你的知识库根，例如 /Users/you/Obsidian/MyVault>`
- **PARA 目录**（KB_ROOT 下）：`00_Inbox` `10_Projects` `20_Areas` `30_Resources` `40_Archives`
- **附件**：`90_Attachments`
- **模板**（KB_ROOT/80_Templates/）：`概念笔记模板.md` / `项目笔记模板.md` / `资源笔记模板.md` / `日常记录模板.md`

## 工具约定
- 搜索 / 列目录 / 统计：优先 `rg`（ripgrep，已安装），回退 `grep -r`。
- 读笔记：Read。写新笔记：Write。改已有笔记：Edit（精确匹配，最小 diff）。
- 抓取 URL：WebFetch，或 mcp `web_reader` / `4_5v_mcp__analyze_image`。
- 日期：今天由系统 `currentDate` 提供；frontmatter 用 `YYYY-MM-DD`（日常记录可含 `HH:mm`）。

## 三条铁律（所有写操作必须遵守）
1. **写前确认**：任何 创建 / 修改 / 删除 文件的操作，必须先把方案展示给用户并等待确认，绝不静默写盘。
2. **不破坏**：未经用户明确同意，不删文件、不覆盖已有内容；改已有笔记用 Edit 精确替换，不动其余部分。
3. **可回溯**：每步操作都用绝对路径，并报告改了什么、改在哪。

## 确认卡片（写操作统一格式）
向用户展示方案时使用下方结构，等用户回复「确认」或修改意见后再动手：

    📥 存储方案
    - 路径：<目录/文件名.md>
    - 模板：<模板名>
    - 标签：#a #b #c
    - 链接：[[X]] [[Y]]
    - 摘要：<一句话>
    是否写入？(可调整任意项)

只读操作（query、maintain 的扫描阶段、tag 的盘点）直接出结果，无需确认。

## Skill 之间的协作
- `import` 抓取 URL 后，复用 `save` 的分类 / 标签 / 链接 / 写入流程（落 `30_Resources`）。
- `link` 的链接检测、`maintain` 的标签漂移判定，规则见 `classification.md`。
- 各 Skill 互不依赖文件状态，可独立调用。
