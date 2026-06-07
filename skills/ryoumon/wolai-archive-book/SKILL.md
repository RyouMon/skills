---
name: wolai-archive-book
description: "Archives books into Wolai with metadata, cover, TOC-based subpages, and part summaries. Accepts local files (EPUB/PDF/MOBI) or title-only lookup. Creates pages at workspace root (我的页面). Page icon via update_doc using only built-in emoji or font_awesome. URLs as rich-text inline link, never bookmark blocks. Triggers: 《xxx》归档到wolai、图书归档、Wolai 书籍、书籍归档、book archive."
---

# wolai-archive-book

## 常量

- **parent_id**（新页父节点 = 「我的页面」工作空间根）：`list_docs()` 取任一顶级文档 `id` → `get_doc(doc_id)` 读 `document.parent_id`（`parent_type === "workspace"`）。用户指定父节点时用其 ID。
- **风格**：SKILL 正文压缩；写入 Wolai 的正文可正常中文，避免废话段。

## 前置

1. Wolai MCP 已启用（server 名常见 `user-wolai`；以 IDE MCP 列表为准）。
2. **先读工具 JSON** 再 `CallMcpTool`：`mcps/user-wolai/tools/*.json`。
3. MCP 工具名以当前 schema 为准：`create_doc` / `update_doc` / `get_doc` / `get_page_outline` / `create_block` 等（旧 skill 里的 `create_page` = `create_doc`）。

## 触发与输入

| 输入 | 处理 |
|------|------|
| `《书名》归档到wolai` / `把 xxx 书归档` | 从书名号或上下文取书名 |
| 本地文件路径（`.epub` / `.pdf` / `.mobi` / `.azw3`） | **先读对应 skill** 抽元数据、封面、目录；无 skill 再兜底解析 |
| 仅书名 | 联网查元数据 + 目录（见「仅书名」） |

定款后 `search_docs(query=书名)`：已有同名根页则更新，勿重复建。

## 流程

**调研 + 写稿 → 再 MCP。**

1. 收集字段与目录（见下）。
2. `create_doc(parent_id=见常量, title=书名)`。
3. **`update_doc(doc_id, icon={ type, icon })`** 设页 icon（📕📗📘 等；非 link）。
4. `create_block` 写主页面板 + 按篇/章建 **`type: "page"`** 子页面块。
5. **章子页内小节标题**（见「章子页内小节」）：各章 `page` 建完后，逐章 `create_block` 写入去编号的 L1 `heading`。
6. 封面：优先上传/嵌入；必要时 `image.link`（见「封面」）。
7. `get_page_outline` 验主页面 + 抽样章子页结构。

## 归档前：信息收集（必做）

目标：**元数据可信、目录与实物一致**，禁止只写一句简介就建页。

### 字段

| 字段 | 要求 |
|------|------|
| 书名 | 页面 `title`；用用户指定版本的中文名（有括号副标题可保留） |
| 简短总结 | 2–4 句：`quote` 块；概括全书主旨、地位或读法，非目录复述 |
| 主题 | 学科/分类标签，如 `古典资产阶级政治经济学`；有 Wolai 主题页 ID 时用 `bi_link`（见 wolai-archive-app「主题」） |
| CIP | 图书在版编目一行；格式见示例：`CIP：国富论／（英）亚当·斯密（Smith, A. ）著；孙善春，李春长译．—北京：中国华侨出版社，2011．3` |
| ISBN | `ISBN：978-7-5113-1217-4`；无则留 `ISBN：` 并注明未查到 |
| 封面 | 与用户手头版本一致；见「封面」 |
| 目录 | **篇（可选）→ 章 → 节**；章对应子页面，节写入章子页内 |

### 本地文件（skill 优先）

**铁律**：有本地图书文件时，**先查并 Read 匹配格式的 Agent Skill**，按该 skill 的流程提取元数据、目录、封面；**无对应 skill 或 skill 未覆盖所需字段时**，再用「兜底」自行解析。**禁止**跳过 skill 直接写解析脚本。

| 格式 | 优先 skill | 路径 |
|------|-----------|------|
| `.pdf` | `pdf` | `skills/anthropics/pdf/SKILL.md`（本仓库有 `openai/pdf` 时用其读/审 PDF 流程亦可） |
| `.epub` / `.mobi` / `.azw3` | 无专用 skill | 走「兜底」 |

**PDF（经 `pdf` skill）**：元数据用 skill 内 `pypdf` `PdfReader.metadata`；目录优先读 PDF embedded outline（书签）。skill 未写 outline 时，兜底 `pymupdf` `doc.get_toc()`；仍无则请用户确认或联网补。

**兜底（仅无 skill 或 skill 不够时）**

1. **EPUB**：`ebooklib` + `lxml` — `OPF` 元数据（`dc:title` / `dc:creator` / `dc:identifier` / `dc:subject`）、`cover`、`nav.xhtml` / `toc.ncx` 目录。
2. **MOBI/AZW3**：能转 EPUB 则走 EPUB 兜底；否则读内嵌 metadata + 用户补目录。

从文件得到 CIP/ISBN 时以文件内为准；缺项再联网补。

### 仅书名

多源并行（尽量交叉验证）：

- 豆瓣读书、Open Library、Google Books、国家图书馆 / CIP 查询
- 出版社或电商商品页（封面 + ISBN）

**多个版本 / 译本**：列出候选（书名、译者、出版社、ISBN），**等用户选定**再建页。

### 目录与「篇」

1. 解析 TOC 为 `{ parts?: [{ title, chapters: [{ title, sections?: [string] }] }], chapters?: [...] }`；**节**即 `1.1` / `第1节` 等子标题。
2. 书有「第一篇 / Part I / 卷一」等**大分卷** → 用 `heading` L1（或示例中的 L2，与全书第一篇层级一致即可）+ 篇下 `quote` 总结 + 章级 `page` 子块。
3. **无分卷** → 跳过篇标题，章直接 `page` 子块挂在主页面下（或单篇 `heading`「正文」+ 章列表，按原书习惯）。
4. 每**篇**下加 `quote`：**该部分讲什么**（3–5 句，非逐章罗列）。
5. 章子页块：`create_block` 的 **`type: "page"`**，`content` 保留**章级**标题（含「第X章」）；**节标题另写入章内**（见下），不写章内笔记正文（除非用户另要求）。

## 页面模板 → blocks

主页面正文顺序（与示例一致）：

```markdown
| 简短总结图书内容

主题：古典资产阶级政治经济学
CIP：国富论／（英）亚当·斯密（Smith, A. ）著；孙善春，李春长译．—北京：中国华侨出版社，2011．3
ISBN：978-7-5113-1217-4

（封面图）

---

# 第一篇 论劳动生产力进步的原因，……
| （总结该部分讲了什么）

（子页面：第一章 …）
（子页面：第二章 …）

# 第二篇 …
| …
…
```

### 骨架映射（主页面）

| 顺序 | 块类型 | 内容 |
|------|--------|------|
| 1 | `quote` | 全书简短总结 |
| 2 | `text` | `主题：…`（多主题用 `bi_link` + `", "`，同 wolai-archive-app） |
| 3 | `text` | `CIP：…` |
| 4 | `text` | `ISBN：…` |
| 5 | `image` | 封面（见「封面」） |
| 6 | `divider` | |
| 7+ | 循环每**篇** | `heading` L1（篇名）→ `quote`（篇总结）→ 各章 `page` 子块 |
| 无篇时 | | 各章 `page` 子块直接挂在 divider 后 |

### 章子页面

```json
{ "type": "page", "content": "第一章 论劳动分工" }
```

- 子页 icon 可选 `emoji`（示例用 ✅ 表示已读；新建可省略或 📄）。
- `type: "page"` 的 **block id** 即章子页 `parent_id`（与 URL 中 page id 相同）。

### 章子页内小节（必做）

建完章 `page` 子块后，对该章 **`create_block(parent_id=<章 page id>, blocks=[...])`**，按目录顺序写入各**节**标题：

```json
{ "type": "heading", "level": 1, "content": "电影语言的起源" }
```

**去编号**（章内 heading 不含序号前缀）：

| 原目录 | 写入 content |
|--------|----------------|
| `1.1 电影语言的起源` | `电影语言的起源` |
| `第1节 电影语言的起源` | `电影语言的起源` |
| `Chapter 6 三个演员的对话`（若在章内） | `三个演员的对话` |

- 去掉 `\d+(\.\d+)+\s*`、`第[一二三四五六七八九十百\d]+节\s*`、`Chapter\s+\d+\s*` 等前缀；**保留**正文标点与引号。
- **有节**：每节一条 L1 `heading`，顺序与 TOC 一致。
- **无章内节**（如整章无 `x.y`）：写一条 L1，content 为**去掉「第X章」后的章名**（例：`第十章 运动结束后的剪接` → `运动结束后的剪接`）。
- **前置页**（出版前言 / 推荐序等）：若建子页，单条 L1 用篇名本身（无编号可去）。
- 节标题块下**留空**供用户笔记；校验：`get_page_outline(page_id=<章 id>)` 应列出各 L1 section。

### 篇标题层级

- 示例「国富论」第一篇为 `level: 2`、其余篇为 `level: 1`；**以原书目录层级为准**，同一本书内保持一致。
- 仅一层目录时章标题不要用假「篇」包装。

## 封面

1. **EPUB 内嵌 cover** / **PDF 首页或 skill 提取图** → 导出到 `.wolai-upload/{slug}/cover.jpg`（封面提取同样：**有 skill 走 skill**，无则自行导出）。
2. **无内嵌** → Open Library `https://covers.openlibrary.org/b/isbn/{isbn}-L.jpg` 或豆瓣/出版社图；下载到本地。
3. **写入 Wolai**：
   - 有 `create_upload_session`：空 `image` → session → PUT → `update_block(file_id)`（同 wolai-archive-figure）。
   - 无上传工具或内网 OSS 失败：临时 `image.link` 公网封面 URL，`caption` 注明来源；后续可改上传块。
4. 位置：**ISBN 之后、`divider` 之前**（示例页封面在文末亦可，本 skill 默认放元数据区）。

## 页面 icon（仅我来内置）

`update_doc(doc_id, icon={ type, icon })`：

| 允许 | 说明 |
|------|------|
| `type: "emoji"` | 单个 emoji（📕📗📘📙 等） |
| `type: "font_awesome"` | 我来内置 FA 名 |

**禁止**：`icon.type: "link"` 作页 icon。

## 外链（仅普通链接）

- **允许**：`text` / `quote` / `heading` 富文本 `{ title, link }`。
- **禁止**：`type: "bookmark"`。

## 父节点

默认挂工作空间根（「我的页面」顶级，与「闪念卡片盒」同级）：`list_docs()` 取任一顶级文档 `id` → `get_doc(doc_id)` 读 `document.parent_id`（`parent_type === "workspace"`）。用户指定其他父节点时改用其 `page_id`；**不要**默认放进数据库视图（示例国富论在 database 下，本 skill intentionally 放根目录）。

## 后续编辑

`get_page_outline` → `get_section_content` → `rewrite_section` / `insert_under_heading` / `insert_blocks_relative`；章内笔记在用户打开子页后另写。

## 校验 / 忌

`get_page_outline` 验篇/章 + 抽样章内 L1 节标题；`get_doc` 看 icon。忌：盲调 MCP；**本地文件未先读相关 skill 就写解析脚本**；**未核版本就建页**；**目录与文件不一致**；漏 CIP/ISBN 不说明；**篇下无总结 quote**；**章子页空壳、节标题未写入或仍带编号**；**用 bookmark 挂链**；**页 icon 用 link**；子页章名与 TOC 不符；把书放进非根 parent（除非用户要求）。
