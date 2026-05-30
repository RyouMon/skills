---
name: wolai-archive-character
description: "Collects anime/game character or actor profiles into Wolai after research; page title is Chinese name; fields include original name, English name, full-body/setting image, official anime character page, and wiki links. No fixed parent page. Page icon via update_page using only built-in emoji or font_awesome. URLs as rich-text inline link, never bookmark blocks. Triggers: 角色归档、动漫角色、演员介绍、Wolai 角色、人物百科、官网角色页."
---

# wolai-archive-character

## 常量

- **无固定 parent_id**（与 wolai-archive-app 不同）：不预置父页面、不另建「角色库」汇总页。
- **风格**：SKILL 正文压缩；写入 Wolai 的正文可正常中文，避免废话段。

## 前置

1. Wolai MCP 已启用（常见 server 名 `user-wolai`；以 IDE MCP 列表为准）。
2. **先读工具 JSON**（schema）再 `call_mcp_tool`：`mcps/<wolai-server>/tools/*.json` 或等价路径。

## 流程

**调研 + 写稿 → 再 MCP。** 定 `parent_id` → `create_page` → **`update_page` 设 icon** → `create_block`。

### 父节点（无固定父页）

`create_page` 仍须 `parent_id`，但本 skill **不设常量**：

1. 用户已指定父节点 → 直接用其 `page_id` / 空间 ID。
2. 未指定 → `list_pages` 取工作空间根，或问用户放哪。
3. 建页前 `search_pages(query=中文名)`：已有同名页则更新，勿重复建。

`title` = **角色中文名**（演员同理，用常用中文译名或官方中文名）。

### 页面 icon（仅我来内置）

`create_page` 无 icon 字段；建页后立刻 `update_page(page_id, icon={ type, icon })`。

| 允许 | 说明 |
|------|------|
| `type: "emoji"` | `icon` = **单个** Unicode emoji。 |
| `type: "font_awesome"` | `icon` = **我来内置** Font Awesome 风格图标名。 |

**禁止**：`icon.type: "link"`（外链图片）作页面图标。

**选型**：角色气质或作品感（例：少女→🌸；战斗→⚔️；演员→🎬）。与标题一致、勿抢正文。

## 归档前：信息收集（必做）

目标：**查清再写**，禁止只抄一句简介就归档。

1. **信息源**（尽量多开）：**作品官网/番剧公式站角色介绍页**、Fandom/萌百/番剧 Wiki、维基百科、MyAnimeList/AniDB、声优/演员 DB、官方设定集扫图。
2. **手段**：浏览器/搜索 MCP、官网导航进「キャラクター/CHARACTER/角色」子页；记录**官网角色页、百科页标题与 URL**、图源出处。
3. **产出（内化）**：

| 字段 | 要求 |
|------|------|
| 中文名 | 页面标题；多译名时取最常用 |
| 原产地名 | 日文/韩文等原名 + 读音（括号内假名/罗马音） |
| 英文名 | 官方或 Wiki 常用英文名；无则留空行 |
| 设定图 | **全身像或官方设定图**一张；优先官网/设定集，其次 Wiki 高清图 |
| 官网角色页 | 动漫角色：**必查**作品官网（含 TV 动画/游戏公式站）该角色的介绍页 URL；有多地区站时优先日文公式站 |
| 百科链接 | 至少一条权威 Wiki（作品 Wiki > 泛百科） |

**动漫角色**：存在官网角色介绍页时，**必须**写入页面「官网」节；官网图可优先作设定图来源。

**演员**：「原产地名」写本名/原名；「英文名」写常用英文名；图片用官方肖像或剧照；无「官网」节要求；百科链 IMDB/维基/豆瓣等；若有事务所/官方 profile 可写在百科节首行。

## 外链（仅普通链接）

- **允许**：`text` / `heading` 等富文本 `content` 里行内 `{ title: string, link: string }`。
- **禁止**：`create_block` 的 **`type: "bookmark"`**。外链不得用书签块。

## 页面模板 → blocks

正文须按此顺序与格式（示例）：

```markdown
原产地名：和栗 薫子（わぐり かおるこ）

英文名：

（角色的全身像或设定图一张）

# 官网

[和栗 薫子 | 薫る花は凛と咲く 公式サイト](https://kaoruhana-anime.com/character/kaoruko/)

# 百科

[Kaoruko Waguri - Kaoru Hana Wa Rin To Saku](https://kaoruhana.wiki/wiki/Kaoruko_Waguri)
```

### 骨架映射

| 顺序 | 块类型 | 内容 |
|------|--------|------|
| 1 | `text` | `原产地名：{原名}（{读音}）` |
| 2 | `text` | `英文名：{英文名}`；无则仅 `英文名：` |
| 3 | `image` | 全身像/设定图（见下「图片」） |
| 4 | `heading` L1 | `官网`（**动漫角色且有公式角色页时必写**） |
| 5 | `text` | 富文本行内链接：`[{角色名 \| 作品名 公式サイト}]({url})`；多地区/多媒介官网各一行 |
| 6 | `heading` L1 | `百科` |
| 7 | `text` | 富文本行内链接：`[{Wiki 标题}]({url})`；多条 Wiki 各一行 |

**禁止**在模板外加节（如声优、出场回数），除非用户明确要求扩展。

### 官网节

- **适用**：动漫/游戏角色；须先搜 `{作品名} 公式サイト キャラクター` 或进官网「角色/CHARACTER」索引定位该角色页。
- **写法**：每条链接独立一行 `text`；标题建议 `{角色日文/中文名} | {作品名} 公式サイト`（或对应语言，如 Official Site）。
- **多条**：TV 动画站 + 原作/游戏站各有角色页时，**都写**，不要只留 Wiki。
- **缺失**：全网确认无官网角色页时，可省略「官网」节（`heading` + 链接均不写），勿写「暂无」占位。

### 图片

优先顺序：

1. **`image.link`**：已知稳定外链（Wiki 图床/CDN）→ `create_block` 直接 `{ type: "image", link: "https://...", caption?: "来源说明" }`。
2. **上传**：本地/需下载的文件 → 空 `image` 占位 → `create_upload_session` → 上传 → `update_block(file_id)`。

选图：**全身像 > 设定集立绘 > 官方 KV**；避免截图、低分辨率、带大量 UI 的游戏实机图。

### 百科节

- 每条链接独立一行 `text`，富文本 `{ title, link }`。
- 标题用 Wiki 页名或「{英文名} - {作品名}」格式。
- 优先作品专属 Wiki；可追加中文 Wiki（萌百、百度百科等）作为补充行。

## 后续编辑

`get_page_outline` 验结构 → `rewrite_section` / `insert_under_heading` / `insert_blocks_relative` 补图或链。

## 校验 / 忌

`get_page_outline` 验结构；`get_page` 看 icon。忌：盲调 MCP；**未查官网角色页就写百科**；**有官网角色页却漏写「官网」节**；缺图留空占位不说明；**页 icon 用 link 外链**；**用 bookmark 挂外链**；擅自加模板外大段传记。
