---
name: wolai-archive-figure
description: "Archives anime/game figures into Wolai after search and user confirmation; page title is Chinese product name; fields include original name, manufacturer, release date/price, Xianyu market price, scale, character, series, product page, images (download from official site first, upload via create_upload_session—never external image.link), and reviews. No fixed parent page. Page icon via update_page using only built-in emoji or font_awesome. URLs as rich-text inline link, never bookmark blocks. Triggers: 手办归档、figure、Wolai 手办、模型归档、GK、景品、比例 figure."
---

# wolai-archive-figure

## 常量

- **无固定 parent_id**：不预置父页面、不另建「手办库」汇总页。
- **风格**：SKILL 正文压缩；写入 Wolai 的正文可正常中文，避免废话段。

## 前置

1. Wolai MCP 已启用（常见 server 名 `user-wolai`；以 IDE MCP 列表为准）。
2. **先读工具 JSON**（schema）再 `call_mcp_tool`：`mcps/<wolai-server>/tools/*.json` 或等价路径。

## 流程

**搜索确认 → 调研（含官网拉图）→ 下载图片 → MCP。** 定款 → 定 `parent_id` → `create_page` → **`update_page` 设 icon** → `create_block`（文本）→ **逐张上传图片**（见「图片」）。

### 第一步：搜索并确认手办（必做）

用户给出名称（中文/日文/英文/角色+厂商均可）后，**先搜再归档**，禁止未定位商品就建页。

1. **多源搜索**（并行尽量多开）：
   - 中文：`{关键词} 手办`、`萌款`、`Hpoi`、`MyFigureCollection 中文`
   - 日文/官方：`{关键词} フィギュア`、`AmiAmi`、`Good Smile`、`Max Factory`、`MegaHouse` 等厂商站
   - 聚合：MyFigureCollection (MFC)、figma wiki、百度/谷歌图搜辅助辨款
2. **整理候选列表**，每条至少含：中文译名、原名、制造商、比例/系列、发售年月（若可知）、**商品页 URL**、缩略图。
3. **用户确认**：
   - **唯一明确匹配** → 简述依据（厂商+原名+比例），再进入调研。
   - **多个候选 / 不确定** → 用编号列表展示（附链接），**等用户选定后再继续**；勿擅自猜款。
   - **零结果** → 回报已搜渠道，请用户补充 JAN/厂商/商品页链接/盒照。
4. 定款后 `search_pages(query=中文译名)`：已有同名页则更新，勿重复建。

### 父节点（无固定父页）

`create_page` 仍须 `parent_id`，本 skill **不设常量**：

1. 用户已指定父节点 → 直接用其 `page_id` / 空间 ID。
2. 未指定 → `list_pages` 取工作空间根，或问用户放哪。

`title` = **手办中文版译名**（萌款/Hpoi/MFC 常用中文名；无中文则用社区通译并注明）。

### 页面 icon（仅我来内置）

建页后立刻 `update_page(page_id, icon={ type, icon })`。

| 允许 | 说明 |
|------|------|
| `type: "emoji"` | 单个 Unicode emoji（例：🎎 🗿 🧸）。 |
| `type: "font_awesome"` | 我来内置 Font Awesome 图标名。 |

**禁止**：`icon.type: "link"` 作页面图标。

## 归档前：信息收集（必做）

目标：**以官方商品页为准**填字段，禁止只抄二手标题。

1. **信息源**：制造商官网商品页、AmiAmi/HobbySearch 商品页、MFC/Hpoi/萌款条目、包装盒/JAN 数据库、B 站/YouTube 评测。
2. **产出（内化）**：

| 字段 | 要求 |
|------|------|
| 原名 | 日文/英文官方商品全名 |
| 制造商 | 出荷厂商（与 sculpt/paint 分工时在行内注明） |
| 发售日期 | 官方预定发售日；再版注明「再版 {日期}」 |
| 发售价格 | **原始货币 + 约合人民币**（按发售日附近汇率或标注汇率来源） |
| 市场价格 | **闲鱼**在售/近期成交均价区间；写调研日期；样本不足则写「样本少，约 ¥…」并说明 |
| 比例 | 如 1/7、1/8、无比例/Nendoroid 系列名 |
| 角色 | 角色中文名（可链已有 Wolai 角色页 `bi_link`） |
| 作品 | 作品中文名 |
| 商品页 | 官方或 AmiAmi 等权威经销商页 URL |
| 图片 | 商品页**全部**官图；**仅从官网（及下述备选源）抓取 URL → 本地下载 → 上传 Wolai**（见「图片」） |
| 评测 | 至少 1 条图文或视频评测链接 |

### 闲鱼市价调研

1. 闲鱼搜 `{中文译名}` / `{原名}` / `{角色名} {比例} {厂商}`。
2. 看**近期**标价与「想要」热度，取常见区间（如 ¥800–950）；剔除明显散件/祖国版/标题党。
3. 写进「市场价格：」并附调研日期，例：`¥820–900（2026-05，闲鱼在售约 15 条）`。

## 外链（仅普通链接）

- **允许**：`text` / `heading` 富文本行内 `{ title, link }`；评测视频可用 B 站/YouTube 链。
- **禁止**：`type: "bookmark"` 书签块。

## 页面模板 → blocks

正文须按此顺序与格式（示例）：

```markdown
**原名：**

**制造商：**

**发售日期：**

**发售价格：** （同时写原始货币和人民币）

**市场价格：** （调研一下闲鱼上该手办的平均价格）

**比例：**

**角色：**

**作品：**

**商品页：**

（官网官图全部下载后上传到 Wolai，商品页有多少贴多少张）

# 评测
（找一些评测文章或视频贴在这）
```

### 骨架映射

| 顺序 | 块类型 | 内容 |
|------|--------|------|
| 1 | `text` | `**原名：**{官方原名}` |
| 2 | `text` | `**制造商：**{厂商}` |
| 3 | `text` | `**发售日期：**{YYYY-MM-DD 或 YYYY-MM}` |
| 4 | `text` | `**发售价格：**{原价}（约合 ¥{人民币}）` |
| 5 | `text` | `**市场价格：**{闲鱼区间}（{调研日期}，{简要说明}）` |
| 6 | `text` | `**比例：**{比例或系列}` |
| 7 | `text` | `**角色：**{中文角色名}` |
| 8 | `text` | `**作品：**{中文作品名}` |
| 9 | `text` | `**商品页：**` + 富文本 `{ title, link }` |
| 10+ | `image`×N | 官图**上传块**（无 `link`），**有几张贴几张**（见下「图片」） |
| N+1 | `heading` L1 | `评测` |
| N+2+ | `text` | 每条评测一行富文本链接；视频写 `[{标题} - Bilibili](url)` 等 |

标签行用富文本 `bold: true` 亦可，须与上表字段一一对应。

**禁止**在模板外加节（如 JAN、涂装师），除非用户明确要求扩展。

### 图片

**硬性规则**：所有图片必须**下载到本地**后，经 `create_upload_session` **上传到 Wolai**。**禁止**在 `create_block` / `update_block` 中对 `image` 使用外部 `link`（含 MFC/Hpoi CDN、官站直链、图床）。

#### 图源优先级（只用于**发现 URL**，写入 Wolai 前一律下载）

1. **制造商官网商品页**（必查）：如 `alter-web.jp/products/…`、`goodsmile.info/…`；从页面 HTML/图集取**官图原图 URL**（必要时用浏览器或 `curl` 解析，勿用聚合站缩略图）。
2. **作品/品牌官网商品介绍**（同厂商域名或作品公式站上的该款商品页）。
3. **授权经销商官页**（仅当 1–2 无图或图不全）：AmiAmi、HobbySearch 等**该 SKU 商品页**官图，补足缺失角度。
4. **MFC / Hpoi / 萌款**：仅作辨款与补全 URL；若与官网重复，**仍以官网文件为准**下载。

**忌**：只贴一张；用玩家返图当主图（官图实在没有时正文注明，且最多补 1 张并标注「玩家摄影」）。

#### 数量与顺序

商品页展示图**尽量全贴**（通常 3–8 张）；顺序与官页一致：盒图 → 本体正面 → 侧面/背面 → 特典/替换件。

#### 下载

1. 从**优先级最高且图最全**的页面收集全部官图 URL（去重、要原图不要列表缩略图）。
2. 下载到工作区临时目录，例如 `.wolai-upload/{页面 slug}/01-box.jpg`（按官页顺序编号）。
3. 记录每张图的**来源页**（写入 `caption` 可选，如 `ALTER 官图`）。

#### 上传到 Wolai（每张图重复）

1. `create_block`：`{ "type": "image" }`（**不传 `link`**，占位块）。
2. `create_upload_session(block_id, filename, mime, file_size)` → 按返回的直传信息上传**本地文件**二进制（Shell `curl` 等；字段名以 MCP 返回为准）。
3. 上传成功后 `update_block(block_id, file_id)`。
4. 全部图片块创建并消费 `file_id` 后，再 `get_page_outline` 核对数量。

批量时：先 `create_block` 建好全部空 `image` 占位，再按块逐个 session + 上传 + `update_block`；或一张一轮，避免漏调 `update_block`。

归档结束可删除 `.wolai-upload/` 下该次临时文件。

#### 直传失败（`internal.aliyuncs.com`）

`create_upload_session` 返回的 `upload.url` 若含 **`oss-cn-shanghai-internal`**，签名仅对内网 OSS 有效；Cursor 等本地环境 PUT 常 **超时 / 502**，`update_block(file_id)` 会报「文件尚未上传」。

**处理顺序**：

1. 仍须先**下载官网原图**到 `.wolai-upload/`（不得跳过）。
2. 在可访问内网 OSS 的环境完成 PUT（如 Wolai 官方 WorkBuddy / 同区域云主机），再 `update_block(file_id)`。
3. 若短期无法直传：可用**制造商官网原图 URL** 写入 `image.link`（禁止 MFC/聚合站），`caption` 注明 `ALTER 官图，待直传`；公网 presigned 可用后改回上传块。
4. 向 Wolai 反馈：请 MCP 返回**公网** presigned URL 或提供服务端代传工具。

### 评测节

- 图文：模玩堂、知乎专栏、博客等 → 行内 `link`。
- 视频：B 站 / YouTube 开箱或测评 → 行内 `link`，标题含 UP/频道名。
- 至少 1 条；有多条 quality 评测则都写，优先中文。

## 后续编辑

`get_page_outline` 验结构 → `rewrite_section` / `insert_under_heading` 补图、补评测或更新市价。

## 校验 / 忌

`get_page_outline` 验结构；`get_page` 看 icon；图片块应已显示为我来托管图（非外链预览失败）。忌：盲调 MCP；**未搜索确认就建页**；**多候选未让用户选**；**市价未查闲鱼**；发售价只写单币种；官图只贴一张；**image 块用外部 link**；**未下官网官图就用 MFC 直链写页**；**bookmark 挂外链**；**页 icon 用 link**。
