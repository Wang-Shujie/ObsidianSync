# ObsidianSync

[English](./README.md) | [简体中文](./README.zh-CN.md)

![license](https://img.shields.io/badge/license-MIT-blue)
![version](https://img.shields.io/badge/version-1.0.0-green)
![claude-code](https://img.shields.io/badge/Claude%20Code-plugin-purple)

> 一个 Claude Code 插件，把 Claude 变成你本地 Obsidian 知识库的贴心管理员——PARA 自动分类、自动标签、双向链接、健康检查、摘要、URL 导入，拆成 **7 个模型自动调用的 Skill**。所有写操作都先给你方案，**经你确认**后才落盘。

ObsidianSync **不**运行后台同步守护进程，也**不**推送到远端。这里的 "Sync" 指 *让知识库与自身保持一致*：当你保存或导入一条笔记时，Claude 帮你分类、打标签、链接相关笔记，并写入正确的 PARA 目录——永远先展示方案、等你点头，再动磁盘。

---

## 目录

- [功能特性](#功能特性)
- [工作原理](#工作原理)
- [环境要求](#环境要求)
- [安装](#安装)
- [配置](#配置)
- [使用方法](#使用方法)
- [目录结构](#目录结构)
- [开发指南](#开发指南)
- [从单 Skill 版本迁移](#从单-skill-版本迁移)
- [贡献方式](#贡献方式)
- [许可证](#许可证)
- [致谢](#致谢)

---

## 功能特性

- **7 个聚焦的 Skill**——`save` / `query` / `link` / `maintain` / `tag` / `summary` / `import`。Claude 根据你的自然语言自动判断用哪个，无需记忆命令。
- **PARA 组织**——每条笔记按透明、有序的规则归入 `00_Inbox` / `10_Projects` / `20_Areas` / `30_Resources` / `40_Archives`。不确定一律落 Inbox，绝不误归档。
- **尊重已有标签的自动打标**——新标签先与库内已有标签比对，避免 `机器学习` / `ML` / `machine-learning` 并存。
- **双向链接**——用笔记标题 + frontmatter `aliases` 作锚点，在新笔记正文里匹配并建议 `[[双链]]`，按相关性排序。
- **写前确认**——所有写操作（创建 / 修改 / 删除）先展示确认卡片，等你同意。只读操作直接出结果。
- **知识库健康检查**——断链 / 孤儿笔记 / frontmatter 异常 / Inbox 积压 / 标签漂移，分级报告。
- **摘要与 MOC**——刷新单篇摘要，或为整库生成索引笔记（Map of Content）。
- **URL 导入**——抓取 URL 并走完整 `save` 流程，落入 `30_Resources`。
- **零外部依赖**——仅用 Claude Code 自带工具 + 系统 `ripgrep`（`rg`）。

## 工作原理

每个 Skill 短小自含：触发条件与流程写在各自的 `skills/<名字>/SKILL.md`。共享规则——路径、三条铁律、确认卡片、PARA/标签/链接逻辑——只放一份在 `references/`，通过 `${CLAUDE_PLUGIN_ROOT}/references/...` 引入，七个文件间零重复。

**三条铁律**（所有写操作必须遵守）：

1. **写前确认**——绝不静默创建、修改、删除文件。
2. **不破坏**——未经明确同意：不删文件、不覆盖内容。改已有笔记用精确最小 diff。
3. **可回溯**——每步用绝对路径，报告改了什么、改在哪。

当 Skill 要写盘时，你会看到这样的卡片，只有你回复 **确认** 才会真正写入：

```
📥 存储方案
- 路径：   20_Areas/自注意力机制.md
- 模板：   概念笔记模板.md
- 标签：   #概念 #深度学习 #注意力
- 链接：   [[Transformer]]
- 摘要：   自注意力用 Q/K/V 矩阵计算序列内相关性。
是否写入？(可调整任意项)
```

## 环境要求

- [Claude Code](https://docs.claude.com/en/docs/claude-code)（需支持插件）。
- 一个本地 [Obsidian](https://obsidian.md) 知识库（插件读写的是磁盘上的 `.md` 文件；Obsidian 客户端可选，但推荐用于浏览）。
- 系统 `PATH` 中有 `ripgrep`（`rg`，多数系统已预装；否则 `brew install ripgrep`）。

## 安装

ObsidianSync 通过 Claude Code 的 **marketplace** 安装。发布后指向仓库即可；本地开发也可指向目录。

### 方式 A：从已发布 marketplace 安装（推荐）

```
/plugin marketplace add https://github.com/[TODO-owner]/obsidiansync
/plugin install obsidiansync@obsidian-sync
```

随后重启 Claude Code（或新开会话）。

### 方式 B：从本地克隆安装（开发 / 自用）

克隆仓库后，把它的目录加为 marketplace——`marketplace.json` 在工作区根，其 `source` 指向 `./ObsidianSync`：

```
git clone https://github.com/[TODO-owner]/obsidiansync [TODO-path]
/plugin marketplace add [TODO-克隆后的工作区根绝对路径]
/plugin install obsidiansync@obsidian-sync
```

### 方式 C：写入 settings.json（团队 / 持久化）

加入 `~/.claude/settings.json`（用户级）或 `.claude/settings.json`（项目级）：

```json
{
  "extraKnownMarketplaces": {
    "obsidian-sync": {
      "source": { "source": "file", "path": "[TODO-工作区根绝对路径]" }
    }
  },
  "enabledPlugins": {
    "obsidiansync@obsidian-sync": true
  }
}
```

> 本地 marketplace 的 `source` 写法在不同 Claude Code 版本中可能不同（`"file"` + `path`，或直接给路径）。方式 C 报错时改用方式 A/B。

### 验证

重启后问 Claude「列出可用 Skill」，应能看到 `obsidiansync:save`、`obsidiansync:query` … `obsidiansync:import` 七个。或直接说「用 obsidiansync 存一条笔记」测试。

## 配置

知识库路径由 [`references/config.md`](./references/config.md) 顶部的 **`KB_ROOT`** 规则解析，按以下优先级取第一个非空值：

1. 环境变量 **`OBSIDIAN_KB_ROOT`**（推荐持久化方式：写入 shell profile 或 Claude Code 环境变量设置，重装插件不丢失）。
2. `references/config.md` 里的 **`KB_ROOT` 默认值**（首次使用前改成你的知识库根）。
3. 两者都未设 → Claude 在会话中**询问你**，并沿用至本会话结束。

```md
# references/config.md（节选）
- KB_ROOT（按优先级解析）：环境变量 OBSIDIAN_KB_ROOT → 下方默认值 → 会话中询问
- KB_ROOT 默认值：<你的知识库根>            # 例如 ~/Obsidian/MyVault
- PARA 目录（KB_ROOT 下）：00_Inbox  10_Projects  20_Areas  30_Resources  40_Archives
- 附件：90_Attachments
- 模板（KB_ROOT/80_Templates/）：概念 / 项目 / 资源 / 日常
```

| 设置项 | 位置 | 默认 / 说明 |
|---|---|---|
| 知识库根（`KB_ROOT`） | `references/config.md`（或 `OBSIDIAN_KB_ROOT` 环境变量） | 优先级：环境变量 → 配置默认值 → 会话中询问 |
| PARA 目录名 | `references/config.md` | `00_Inbox` `10_Projects` `20_Areas` `30_Resources` `40_Archives` |
| 分类 / 标签 / 链接规则 | `references/classification.md` | 直接编辑即可，无需改 Skill |
| 笔记模板 | `<KB_ROOT>/80_Templates/*.md` | 新模板需在 `classification.md` 登记 |

## 使用方法

Skill 是**模型自动调用**的——Claude 按你的话判断用哪个。下面带 `@` 前缀只是为了便于指明，不必手敲。

| Skill | 触发示例 | 作用 |
|---|---|---|
| `save` | "把这段存一下：……" | 分析→PARA 分类→选模板→标签→链接→**确认**→写入 |
| `query` | "查一下库里关于 X 的笔记" | 关键词/标签/日期/目录多维检索（只读） |
| `link` | "把孤立笔记连起来" | 找孤立笔记，建议插入 `[[X]]`，勾选后应用 |
| `maintain` | "检查知识库健康度" | 断链/孤儿/frontmatter/Inbox 积压/标签漂移，分级报告 |
| `tag` | "列出标签，把 #ML 和 #机器学习 合并" | 标签盘点、重命名/合并/删除（每步确认） |
| `summary` | "给这篇生成摘要" / "给整库做 MOC" | 单篇刷新摘要；整库生成索引笔记 |
| `import` | "导入这篇文章 https://…" | 抓取 URL→走 save 流程→存入 30_Resources |

### 示例

```
# 保存
"把这段存一下：自注意力用 Q/K/V 矩阵衡量序列内 token 的相关性。"

# 检索（只读，即时）
"库里上个月打了 #深度学习 标签的笔记有哪些？"

# 健康检查
"跑一次知识库健康报告，重点看断链和 Inbox 积压。"

# 导入
"把 https://example.com/article 导入知识库。"
```

只读 Skill（`query`、`maintain` 的扫描阶段、`tag` 的盘点）直接出结果。所有写 Skill 先展示确认卡片。

## 目录结构

```
ObsidianSync/
├── .claude-plugin/
│   └── plugin.json              # 插件清单（name/version/description）
├── skills/                      # 7 个 Skill，每个一个目录
│   ├── save/SKILL.md            # @save    智能存储
│   ├── query/SKILL.md           # @query   多维检索
│   ├── link/SKILL.md            # @link    建立双向链接
│   ├── maintain/SKILL.md        # @maintain 健康检查
│   ├── tag/SKILL.md             # @tag     标签管理
│   ├── summary/SKILL.md         # @summary 生成摘要/MOC
│   └── import/SKILL.md          # @import  导入 URL
├── references/                  # 共享，用 ${CLAUDE_PLUGIN_ROOT}/references/... 引用
│   ├── config.md                # 路径 / 工具约定 / 三条铁律 / 确认卡片
│   └── classification.md        # PARA / 标签 / 链接 / 写入规范
└── README.md
```

marketplace 清单在上一层——**工作区根**：

```
<工作区根>/
└── .claude-plugin/
    └── marketplace.json         # 本地 marketplace，source → ./ObsidianSync
```

## 开发指南

### 改知识库路径
编辑 `references/config.md` 顶部的 `KB_ROOT`，所有 Skill 自动生效（或在对话中临时指定，本会话沿用）。

### 加新 Skill
1. 在 `skills/` 下建 `<名字>/SKILL.md`。frontmatter 必填 `name`（小写/数字/连字符，≤64 字符）和 `description`（≤1024 字符，写清「做什么 + 何时用」，中英触发词都列）。
2. 可选 `allowed-tools` 限定工具（如只读 Skill 加 `Read, Grep, Glob, Bash`）。
3. 共享规则写到 `references/`，用 `${CLAUDE_PLUGIN_ROOT}/references/xxx.md` 引用。
4. 重装 / 重启后生效。

### 改分类 / 标签 / 链接规则
直接编辑 `references/classification.md`，无需动各 Skill。

### 加模板
把模板 `.md` 放进知识库 `80_Templates/`（含标准 frontmatter 与 `📝 摘要 / 📖 正文 / 🔗 关联 / 📚 参考资料 / 🏷️ 标签` 结构），并在 `classification.md` 的「笔记类型→模板」表登记。

### 依赖
仅依赖 Claude Code 自带工具（`Bash`/`Read`/`Write`/`Edit`/`WebFetch`/`Grep`/`Glob`）与系统 `rg`。无任何外部包。

### 本地开发循环
```
/plugin marketplace add <工作区根>     # 一次
# 编辑 SKILL.md / references/*.md
# 在 Claude Code 中重装或重启以重新加载
```

## 从单 Skill 版本迁移

本插件由原来的单文件 `ObsidianSync/SKILL.md`（已删除）重构而来：那个文件的 7 个命令被拆成 `skills/` 下 7 个独立 Skill，共享逻辑沉到 `references/`。旧的符号链接安装方式（`ln -s … ~/.claude/skills/obsidiansync`）已不再适用——请改用上面的 [marketplace 安装](#安装)。

## 贡献方式

欢迎贡献。`[TODO: 在你的仓库里指向 CONTRIBUTING.md / issue 模板]`

1. Fork 仓库并建特性分支（`git checkout -b feat/<名字>`）。
2. 保持 Skill 短小自含；复用内容放进 `references/`。
3. 遵守三条铁律——任何 Skill 不得在没有确认卡片的情况下写盘。
4. 新增 Skill 或改动共享规则时，同步更新本 README 与 `references/`。
5. 提交 Pull Request，说明改动并附一个使用示例。

**适合上手的好议题：** 新模板、为触发词补充更多语言、`query` 的新检索维度、`maintain` 的新健康检查项。

提交前请先检索已有 issue/PR，避免重复。`[TODO: 如需要可加行为准则链接]`

## 许可证

基于 **MIT License** 发布，见 [`LICENSE`](./LICENSE)。

> 仓库中引用了 `LICENSE` 文件但尚未包含。首次公开发布前，请加入标准 MIT 文本（含你的姓名/年份）。`[TODO]`

## 致谢

- [Obsidian](https://obsidian.md)——本插件所管理的知识库。
- [Claude Code](https://docs.claude.com/en/docs/claude-code)——agent 平台与插件模型。
- **PARA 方法**（Tiago Forte）——组织模型来源。

`[TODO: 作者署名、联系方式、仓库地址、赞助链接（按需）]`
