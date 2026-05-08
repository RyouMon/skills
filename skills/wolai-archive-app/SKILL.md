---
name: wolai-archive-app
description: Archives an app into Wolai after deep exploration; summarizes install, basic usage, per-page advanced usage, and per-page FAQ; sets page icon via update_page using only built-in emoji or font_awesome (no link icons). Topics: comma-separated bi_link. Triggers: 归档应用、Wolai 归档、页面图标、我来内置图标.
---

# wolai-archive-app

## 常量

- **parent_id**（新页父节点）: `tnd54YWouEWd1Se5Q33mgN`
- **风格**: ultra caveman — SKILL 正文已压缩；写入 Wolai 的正文可正常中文，避免废话段。

## 前置

1. Wolai MCP 已启用（常见 server 名 `user-wolai`；以 IDE MCP 列表为准）。
2. **先读工具 JSON**（schema）再 `call_mcp_tool`：`mcps/<wolai-server>/tools/*.json` 或等价路径。

## 流程
 
**探索 + 写稿 → 再 MCP。** `create_page` → **`update_page` 设 icon** → `create_block` / `rewrite_section`。`parent_id` 常量；`title` = 应用名。

### 页面 icon（仅我来内置）

`create_page` 无 icon 字段；建页后立刻 `update_page(page_id, icon={ type, icon })`。

| 允许 | 说明 |
|------|------|
| `type: "emoji"` | `icon` = **单个** Unicode emoji，与我在编辑器里可选表情一致即可。 |
| `type: "font_awesome"` | `icon` = **我来内置** Font Awesome 风格图标名（与侧栏/页面图标库一致；无 MCP 枚举时按应用类型选常见名，拿不准则改用 emoji）。 |

**禁止**：`icon.type: "link"`（外链图片）作页面图标 — 不算「内置图标库」。

**选型**：看应用领域与品牌感（例：终端/CLI→⌨️或终端类 FA；DB→🗄️；AI→🤖；Web→🌐）。与标题气质一致、勿花哨抢正文。

## 归档前：深度探索（必做）

目标：**摸透应用再下笔**，禁止只扫 README 一句就归档。

1. **信息源**（尽量多开）：官网/文档站导航、GitHub README & Releases、包管理器安装页、应用内主要界面（路由/菜单）、已有 Wiki/博客教程。
2. **手段**：浏览器/文档 MCP、仓库内 `README`/`docs/`、官方搜索；能点进子页就点，记录**页面/文档标题与 URL**。
3. **产出（内化，不必贴进 Wolai）**：页面清单、安装入口、基础操作路径、各页高阶功能点、各页易错点。

## 四节正文规则（写入 Wolai 须满足）

| 节 | 要求 |
|----|------|
| **安装** | **官方**安装文档链接 + **官方**下载/包管理器入口（`bookmark` / 富文本 `link`）；步骤用短列表或引用官方小节标题；非官方教程仅作补充并标注来源。 |
| **用法**（= 基础用法） | 一条「最小可跑通」路径：从打开应用到完成最常见任务；命令/按钮名与官方一致；避免一上来堆高级项。 |
| **进阶** | **按页面（或按文档大章节）分类**：每类用 `heading` L2 或小标题 + 正文；写该页/该模块的高阶能力、配置、工作流，不重复「用法」已写的入门步。 |
| **FAQ** | **按页面（或按模块）分类**：同上结构；每条问题短问 + 短答 + 必要时链官方 issue/文档；优先真实踩坑与文档已写常见问题。 |

`quote`：全文一句话定位（谁用、解决啥）。元数据行仍：主题/主页/文档/指南/源码。

## 骨架 → blocks

`quote` → `text`×5「主题/主页/文档/指南/源码」→ `divider` → `heading` L1「安装」「用法」「进阶」「FAQ」→ 各节按上表填**实质内容**（非「待补」除非确实缺官方源并注明）。URL → `bookmark` / 富文本 `link`（见 `create_block` schema）。

### 主题（可多关联、逗号分隔）

- **语义**：一条里挂多个 Wolai 内页面/块（同空间知识网络），读作「主题：A, B, C」。
- **示例**：`主题：MCP, 人工智能`；`主题：数据库`（单主题也同格式）。
- **实现**：`text` 的 `content` 用**富文本数组**串联——前缀 `主题：` + 多个 `{ type: "bi_link", title: string, block_id: string }`，项与项之间插 `", "`（逗号+空格）字面片段。`block_id` 来自目标页/块（`get_page` / `search_pages` / `get_page_outline` 等拿到后再写）。
- **无 ID 时**：可暂纯文本 `主题：MCP, 人工智能`；有 ID 后把对应词换成 `bi_link` 即可。

## 后续编辑

outline → section content → `rewrite_section`；标题下补 → `insert_under_heading`；锚点前后 → `insert_blocks_relative`。

## 校验 / 忌

`get_page_outline` 验结构；`get_page` 可看 icon 是否生效。忌：盲调 MCP；**未做多源探索就写四节**；全文 README 无结构粘贴；大页 `include_blocks:true`；**页 icon 用 `link` 外链图**。
