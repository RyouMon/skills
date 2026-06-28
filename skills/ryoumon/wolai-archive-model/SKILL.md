---
name: wolai-archive-model
description: "Archives an AI model into Wolai after web research; collects task, type, base model, version, vendor, links (HF/Civitai/GitHub/download/paper), intro, usage, examples, and community reviews; sets page icon via update_page using only built-in emoji or font_awesome. Example media uploaded via create_upload_session when available. URLs as rich-text inline link, never bookmark blocks. Topics: comma-separated bi_link. Triggers: 归档模型、AI模型归档、Wolai 模型、Checkpoint、LORA、LLM 归档、归档xx模型到wolai."
---

# wolai-archive-model

## 常量

- **parent_id**（新页父节点 = 工作空间根）：`list_docs()` 取任一顶级文档 `id` → `get_doc(doc_id)` 读 `document.parent_id`（`parent_type === "workspace"`）。用户指定父节点时用其 ID。
- **风格**：SKILL 正文压缩；写入 Wolai 的正文可正常中文，避免废话段。

## 前置

1. Wolai MCP 已启用（常见 server 名 `user-wolai`；以 IDE MCP 列表为准）。
2. **先读工具 JSON**（schema）再 `call_mcp_tool`：`mcps/<wolai-server>/tools/*.json` 或等价路径。
3. MCP 工具名以 schema 为准（常见：`create_doc`/`update_doc`/`search_docs` 或 `create_page`/`update_page`/`search_pages`）。

## 流程

**搜索确认 → 多源调研 → 写稿 → MCP。** 定款 → 定 `parent_id` → `create_page`（或 `create_doc`）→ **`update_page`/`update_doc` 设 icon** → `create_block` / `rewrite_section`。

### 第一步：搜索并确认模型（必做）

用户给出名称（中文/英文/缩写/作者+模型名均可）后，**先搜再归档**，禁止未定位具体版本就建页。

1. **多源搜索**（并行尽量多开）：
   - **Hugging Face**：`site:huggingface.co {关键词}`；读 Model Card、README、Files、License。
   - **Civitai**：`site:civitai.com {关键词}`；读 Base Model、Type、Trigger Words、Gallery。
   - **GitHub**：官方/作者仓库 README、Releases、Model Card。
   - **官网/博客**：厂商模型页（OpenAI、Anthropic、Google、Black Forest Labs、Stability、Qwen 等）。
   - **论文**：arXiv、OpenReview、厂商技术报告。
   - **评测/社区**：LMSYS Arena、Open LLM Leaderboard、Artificial Analysis、Reddit、知乎、B 站/YouTube 测评。
2. **整理候选列表**，每条至少含：完整模型名、版本/变体、任务类型、供应商、**主链 URL**（HF 或 Civitai 或官网其一）。
3. **用户确认**：
   - **唯一明确匹配** → 简述依据（供应商 + 全名 + 版本），再进入调研。
   - **多个候选 / 不确定**（如 Flux Dev vs Schnell、同 LoRA 多版本）→ 编号列表展示（附链接），**等用户选定后再继续**。
   - **零结果** → 回报已搜渠道，请用户补充 HF/Civitai 链接、作者名或精确版本号。
4. 定款后 `search_pages`/`search_docs`(query=模型常用名)：已有同名页则更新，勿重复建。

`title` = **模型常用中文名或社区通称**；无稳定中文名时用官方英文名，副标题可在正文「版本」行注明。

### 页面 icon（仅我来内置）

建页后立刻 `update_page`/`update_doc`(icon={ type, icon })。

| 允许 | 说明 |
|------|------|
| `type: "emoji"` | 单个 Unicode emoji。 |
| `type: "font_awesome"` | 我来内置 Font Awesome 图标名。 |

**禁止**：`icon.type: "link"` 作页面图标。

**选型**：LLM/对话→🤖；图像生成→🎨；视频→🎬；音频→🎵；Embedding→📊；多模态→🧠。与模型任务一致。

## 归档前：深度调研（必做）

目标：**摸透模型再下笔**，禁止只抄 HF 卡片首段就归档。

### 信息源（按模型类型加权）

| 类型 | 优先源 |
|------|--------|
| **LLM / VLM** | 官网技术页、HF Model Card、GitHub、论文、LMSYS/Open LLM Leaderboard、API 文档 |
| **Diffusion（SD/Flux 等）** | Civitai 模型页、HF、官方 repo、ComfyUI/A1111 文档、社区 Gallery |
| **LoRA / LyCORIS 等** | Civitai（Base Model + Trigger + 推荐权重）、作者说明、示例图 |
| **ControlNet / IP-Adapter** | HF/Civitai、原论文、示例工作流 |
| **视频 / 音频** | HF、GitHub、官方 Demo、论文 |

### 字段收集（内化后写入 Wolai）

| 字段 | 要求 |
|------|------|
| 一句话总结 | 顶格 `quote`；谁用、解决啥、相对同类的定位 |
| 任务 | 具体 AI 用途：文本生成、对话、图像生成、图生视频、TTS、Embedding 等；可多任务逗号分隔 |
| 类型 | 模型文件/形态：Checkpoint、LoRA、LyCORIS、Textual Inversion、ControlNet、GGUF、API-only 等；纯 API 无权重写「API 服务」 |
| 基础模型 | 训练所基于的底模（LoRA 必填）；原生底模写「—」或自指（如 Flux.1 Dev） |
| 版本 | 官方版本号/变体名（Dev、Schnell、v1.5、0723 等） |
| 供应商 | 作者、团队或组织 |
| 主页 | 官网模型介绍页；无则留空 |
| HF | Hugging Face 模型页 URL |
| CIVITAI | Civitai 模型页 URL；非 SD 生态通常无 |
| Github | 官方仓库 URL |
| 下载地址 | **多格式分行**（见下「下载地址」节）；每格式一行，未发布写「暂无」 |
| 论文 | arXiv/技术报告链接；无则写「无」或省略行 |

缺项：**该行仍保留标签**，值写「暂无」或「不适用（{原因}）」，勿静默删字段。

### 下载地址（多格式，必做）

**禁止**只写一条 Civitai/HF 总链就结束。调研同一**版本/变体**在作者官方渠道发布的**全部权重形态**，每种占一行：

```
下载地址（{格式标签}）：{富文本 link 或「暂无」}
```

**调研顺序**：作者 Civitai 同版本名下所有 Model 页 → 作者 Collection → HF/ModelScope 官方 repo → 社区镜像（仅作补充并标注非官方）。

**格式标签**（按模型生态选 applicable 项，不适用可省略该行）：

| 生态 | 常见格式标签（示例） |
|------|---------------------|
| **DaSiWa / Wan 视频** | TrueVision Safetensors、Lightspeed Safetensors、Lightspeed NVFP4 Safetensors、GGUF（Q8/Q4 等）、FP16 Safetensors |
| **SD/Flux 图像** | Safetensors、FP8/FP16、GGUF、Pickle（legacy） |
| **LLM** | GGUF（注明量化级）、Safetensors/FP16、GPTQ、AWQ、MLX、API |
| **LoRA 等** | Safetensors（通常唯一形态；仍写一行） |
| **API-only** | API 文档 / Playground（替代权重行） |

**Wan 2.2 I2V 注意**：High + Low 成对；TrueVision 与 Lightspeed 是**不同 Civitai Model 页**，GGUF 常为独立页（如 `{版本名} | Lightspeed | GGUF`），勿与 Safetensors 页混为一谈。

行内可附简短说明（体积、step 建议、是否需 ComfyUI-GGUF），用字面量括号写在 link 后。

## 外链（仅普通链接）

- **允许**：`text` / `heading` / `quote` / 列表等富文本 `content` 里行内 `{ title: string, link: string }`（及 `bi_link`）。
- **禁止**：`create_block` 的 **`type: "bookmark"`**。

## 页面模板 → blocks

正文须按此顺序与格式：

```markdown
> （一句话总结该模型）

任务：（使用的 AI 任务，如图片生成、视频生成、文本生成）
类型：（模型文件类型，如 Checkpoint Merge、LoRA 等）
基础模型：（基于哪个模型训练的）
版本：官方版本
供应商：作者或组织
主页：模型的官网主页（如果有的话）
HF：huggingface 链接
CIVITAI：civitai 链接
Github：GitHub 链接
下载地址（TrueVision Safetensors）：
下载地址（Lightspeed Safetensors）：
下载地址（Lightspeed NVFP4 Safetensors）：
下载地址（GGUF）：
下载地址（其他，如 HF/ModelScope/API）： 
论文：论文链接，如果有的话

# 介绍

# 使用说明
运行环境，提示词技巧等

# 示例
使用该模型生成的作品，一般和媒体相关的模型贴上生成的图片或视频
LLM 展示对话示例
媒体类的可以考虑放一些 comfyui 工作流

# 评价
| 一句话总结

下面具体放：
网络上社区中对该模型的评价
该模型在当前领域的排名
各大博主的评测
```

### 骨架映射

| 顺序 | 块类型 | 内容 |
|------|--------|------|
| 1 | `quote` | 一句话总结 |
| 2 | `text` | `任务：…` |
| 3 | `text` | `类型：…` |
| 4 | `text` | `基础模型：…` |
| 5 | `text` | `版本：…` |
| 6 | `text` | `供应商：…` |
| 7 | `text` | `主页：` + 有则富文本 `link`，无则「暂无」 |
| 8 | `text` | `HF：` + link 或「暂无」 |
| 9 | `text` | `CIVITAI：` + link 或「暂无」 |
| 10 | `text` | `Github：` + link 或「暂无」 |
| 11+ | `text`×N | **下载地址多行**：见上「下载地址（多格式）」；至少覆盖该模型已发布的全部官方格式；每行 `下载地址（{格式}）：` + link 或「暂无」 |
| 12+N | `text` | `论文：` + link 或「暂无」 |
| 13+N | `text` | `主题：…`（见下「主题」；可省略若用户未要求） |
| 14+N | `divider` | — |
| 15+N | `heading` L1 | `介绍` |
| 16+N | `text`/`bullet` | 架构、能力边界、许可、与其他版本差异；**实质段落**，非 README 全文粘贴 |
| 17+N | `heading` L1 | `使用说明` |
| 18+N | `text`/`heading` L2 | 见下「使用说明节」 |
| 19+N | `heading` L1 | `示例` |
| 20+N | 见下「示例节」 | — |
| 21+N | `heading` L1 | `评价` |
| 22+N | `quote` 或 `text` | 评价一句话总结（可与顶格 quote 呼应但侧重口碑） |
| 23+N | `text`/`bullet`/`table` | 社区评价、排行榜、博主评测 |

标签行可用富文本 `bold: true`；URL 一律富文本 `link`，**勿 bookmark**。

### 主题（可多关联、逗号分隔）

同 wolai-archive-app：`主题：` + 多个 `{ type: "bi_link", title, block_id }`，项间 `", "`。例：`主题：人工智能, Stable Diffusion`。无 ID 时暂纯文本。

### 介绍节

- 模型背景、核心能力、适用/不适用场景。
- 许可（商用与否）、上下文长度 / 分辨率上限等硬指标。
- 与同系列其他变体（Dev vs Schnell、Instruct vs Base）的差异。

### 使用说明节

按模型类型选写，**至少覆盖「运行环境」+ 一项「使用技巧」**：

| 类型 | 须写 |
|------|------|
| **LLM** | 推理框架（vLLM、llama.cpp、Ollama 等）、推荐量化、上下文与采样参数、System Prompt 要点 |
| **Diffusion Checkpoint** | 推荐 UI（ComfyUI/A1111/Forge）、采样器/步数/CFG、分辨率、VAE 是否另配 |
| **LoRA** | 推荐权重（0.6–1.0 等）、Trigger Words、搭配底模、易翻车设置 |
| **ControlNet / 插件** | 预处理、权重、与底模组合 |
| **视频/音频** | 显存/时长限制、帧率、参考官方 CLI 或 Gradio 参数 |
| **API-only** | 端点、认证、定价层级、速率限制 |

提示词：给 1–3 条**可复现**示例 prompt（图像写英文 prompt 常见；LLM 写中文/英文均可）。

### 示例节

按类型分支（可组合）：

**媒体类（图像/视频/音频）**

1. 从官方 Gallery、Civitai 示例、HF Model Card、作者推文/博客收集 **2–6 张**代表产出。
2. **优先上传 Wolai**（见「示例图/视频」）；每张 `caption` 可写 prompt 摘要或来源。
3. 视频：无上传能力时行内链 B 站/YouTube 官方或高质量测评 Demo；有缩略图则上传缩略图 + 视频链。
4. **ComfyUI 工作流**：有官方/作者 JSON → `code` 块贴 JSON 或行内链 Civitai/HF 工作流文件；注明底模与 LoRA 节点。

**LLM / 对话类**

1. `code` 或 `quote` 块展示 **1–2 轮**有代表性的对话（含 system/user/assistant 或等价格式）。
2. 展示模型特长（推理、代码、长上下文、多语言）；勿编造能力——示例应来自官方 Demo、Model Card 或公开测评。

**纯 Embedding / 分类等小模型**

- 简短输入→输出示例或官方 benchmark 表格摘要。

### 评价节

结构：

1. **一句话总结**（`quote`）：社区共识式概括（优点 + 主要槽点）。
2. **社区评价**：Reddit/知乎/V2EX/Discord 等**可溯源**观点摘要；标注倾向（ praise / 争议 / 过时）。
3. **领域排名**：若有公开榜单则写名次 + 榜单名 + 调研日期（如 LMSYS Arena Elo、Open LLM Leaderboard、Civitai 下载量档位）；无权威榜则写「暂无统一排名」+ 相对定位（第一梯队/性价比/ niche）。
4. **博主/媒体评测**：B 站/YouTube/技术博客链接列表，每条一行富文本 `link`，标题含 UP/频道/站点名。

忌：无来源的主观吹贬；把训练数据传闻当事实。

### 示例图/视频

**优先规则**（同 wolai-archive-figure）：

1. 从 **官方/Civitai/HF 示例** 取图 URL → 下载到 `.wolai-upload/{slug}/` → `create_upload_session` → PUT → `update_block(file_id)`。
2. **禁止**默认用 Civitai CDN / 聚合站直链作永久 `image.link`（与 figure skill 一致）。
3. 无 `create_upload_session` 或内网 OSS 失败：临时 `image.link` 公网示例 URL，`caption` 注明来源与「待直传」；或仅保留评测视频链接。

## 后续编辑

`get_page_outline` → `rewrite_section` / `insert_under_heading`；新版本发布时更新「版本」「下载地址」与「评价」。

## 校验 / 忌

`get_page_outline` 验结构；`get_page`/`get_doc` 看 icon；**下载地址 ≥2 种格式行**（若作者仅发一种则其余行写「暂无」并说明）。忌：盲调 MCP；**未搜索确认就建页**；**多候选未让用户选**；**下载地址只写一条总链**；元数据缺行；README 无结构全文粘贴；**bookmark 挂外链**；**页 icon 用 link**；LoRA 不写基础模型与 trigger；LLM 示例编造对话；评价无来源；媒体示例零图零链。
