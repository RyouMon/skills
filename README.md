# skills

个人技能集合。本仓库技能通过 **[skillshare](https://github.com/runkids/skillshare)**（命令行 `skillshare`）安装、更新并同步到各 AI CLI。下文「skillshare」均指该官方工具；文档站：<https://skillshare.runkids.cc/docs/>。

下列流程与官方文档中的 [Commands](https://skillshare.runkids.cc/docs/reference/commands)、[init](https://skillshare.runkids.cc/docs/reference/commands/init)、[install](https://skillshare.runkids.cc/docs/reference/commands/install) 及 [Cross-Machine Sync](https://skillshare.runkids.cc/docs/how-to/sharing/cross-machine-sync) 等章节一致，便于从「装机」到「日常」一眼看完。

---

## 为何用 skillshare

- **单一来源**：技能放在 skillshare 的「源目录」，再 `sync` 到 Claude、Cursor、Codex 等 60+ 目标。
- **安装与更新**：从 GitHub/GitLab 等安装；用 `check` / `update` 或 `install --update` 拉上游（**不推荐 `--track`**，见 §3）。
- **跨机器**：源目录用 git 管理时，可用 `push` / `pull` 与远端同步后再 `sync`。

---

## 1. 安装命令行

任选其一（官方 README）：

**macOS / Linux**

```bash
curl -fsSL https://raw.githubusercontent.com/runkids/skillshare/main/install.sh | sh
```

**Windows PowerShell**

```powershell
irm https://raw.githubusercontent.com/runkids/skillshare/main/install.ps1 | iex
```

**Homebrew（macOS / Linux）**

```bash
brew install skillshare
```

**升级 CLI 本身**

```bash
skillshare upgrade
```

**可选：Shell 补全**

```bash
skillshare completion bash --install
skillshare completion zsh --install
skillshare completion powershell --install
```

---

## 2. 初始化（`init`）

首次在本机建立「源目录 + 目标（各 AI CLI 的技能目录）+ `config.yaml`」，并可选初始化 git、绑定远端。

**绑定已有技能 Git 仓库（远端）**

推荐先建立仓库，然后使用远端仓库初始化：

```bash
skillshare init --remote https://github.com/RyouMon/skills.git
```

若仓库根目录不全是技能（还有 README、CI 等），可用 **`--subdir`** 把源指到子目录（例如 `skills/`）。非交互示例见官方 [init 文档](https://skillshare.runkids.cc/docs/reference/commands/init)。

**推荐初始化流程**
```
→ Source directory stores your skills (single source of truth)
  Default: C:\Users\Wen\AppData\Roaming\skillshare\skills
  Customize source path? [y/N]: Y
  Enter source path: C:\Users\Wen\Projects\skills
✓ Source path: C:\Users\Wen\Projects\skills (Windows) ~\Projects\skills (Linux/MacOS)  # 推荐选择一个自己常用的项目目录

Initialize from existing skills?
─────────────────────────────────────────
  Copy skills from an existing directory to the shared source?

  [1] Copy from witsy (56 skills)
  [2] Copy from universal (56 skills)
  [3] Copy from dexto (56 skills)
  [4] Copy from warp (56 skills)
  [5] Copy from cline (56 skills)
  [6] Copy from codex (1 skills)
  [7] Copy from claude (38 skills)
  [8] Start fresh (empty source)

  Enter choice [1]: 8
→ Starting with empty source
✓ Added 3 target(s): claude, cursor, universal  # 只选择自己用的，避免同步时耗时过长。

Sync mode preference
─────────────────────────────────────────
  1) merge    — per-skill symlinks, preserves local skills
  2) copy     — real files, recommended if unsure whether your AI CLI supports symlinks
  3) symlink  — entire directory linked

  Enter choice [1]: 1
✓ Sync mode: merge

→ If a tool doesn't support symlinks, switch to copy mode:
  skillshare config --mode copy && skillshare sync
→ Git already initialized in source directory
→ Git remote already configured: https://github.com/RyouMon/skills.git

→ Specifying a subdirectory as the source will store skills in the subdirectory (e.g. skills/) instead of in the root
  Specify a subdirectory as the source (e.g. skills)? [y/N]: Y
  Enter subdirectory name: skills/  # 为了与 skills.sh 兼容
```

**项目模式（技能跟代码仓走，提交 `.skillshare/`）**

```bash
skillshare init -p
skillshare init -p --targets claude,cursor
```

**已装过 skillshare，新装了某个 AI CLI**

```bash
skillshare init --discover
skillshare init -p --discover
```

常用标志摘要：`--source` / `-s`，`--project` / `-p`，`--remote`，`--subdir`，`--git` / `--no-git`，`--targets` / `-t`，`--all-targets`，`--mode` / `-m`（`merge` | `copy` | `symlink`），`--discover` / `-d`，`--select`。

初始化后通常：

```bash
skillshare sync
```

---

## 3. 安装技能

安装只改**源目录**；装完后务必 **`skillshare sync`** 才会推到各 AI CLI。完整标志见官方 [install 文档](https://skillshare.runkids.cc/docs/reference/commands/install)。

### 最常见安装命令

| 场景 | 命令 |
|------|------|
| 装单个技能（GitHub 简写 + 路径） | `skillshare install anthropics/skills/skills/pdf` |
| 浏览仓库、交互挑选 | `skillshare install anthropics/skills` |
| 非交互：指定几个技能 | `skillshare install anthropics/skills -s pdf,commit` |
| 非交互：装仓库里全部 | `skillshare install anthropics/skills --all` |
| 装到子目录（按仓库/分类分组） | `skillshare install owner/repo -s pdf --into anthropics` |
| 更新已装技能 | `skillshare install pdf --update` 或 `skillshare update pdf` |
| 本地路径 | `skillshare install ~/Downloads/my-skill` |
| 按 config 清单批量装 | `skillshare install` |
| 装完同步到 Cursor 等 | `skillshare sync` |

```bash
# 最常用：挑技能 → 同步
skillshare install anthropics/skills -s pdf,commit
skillshare sync

# 多技能仓库一次装全（不用 --track）
skillshare install garrytan/gstack --all
skillshare sync

# 预览不写入
skillshare install anthropics/skills --all --dry-run
```

> **不推荐 `--track`**：`--track` 整仓克隆后，`sync` 会把仓库内路径扁平化成很长前缀（如 `_gstack__agents__skills__browse`）。**Claude Code 等 Agent 会直接忽略这类过长名称的技能**。日常安装请用 `-s` / `--all` 按技能逐个拷贝，源目录用 `--into` 分组即可；更新用 `skillshare update <技能名>` 或 `skillshare install <技能名> --update`。

### 多技能仓库

```bash
skillshare install anthropics/skills -s pdf,commit      # 指定名称
skillshare install anthropics/skills -s "core-*"        # glob（引号防 shell 展开）
skillshare install anthropics/skills --all              # 全部
skillshare install anthropics/skills --all --exclude "test-*"  # 排除
```

### 安装时指定子目录（`--into`，推荐用于分组）

```bash
skillshare install anthropics/skills -s pdf --into frontend
# → 源目录 frontend/pdf/

skillshare install anthropics/skills -s pdf --into frontend/react
# → 源目录 frontend/react/pdf/

# 按来源仓库分组（目标名较短，Agent 可识别）
skillshare install garrytan/gstack --all --into gstack
# → 源目录 gstack/browse/ … sync 后为 gstack__browse
```

`sync` 后各 CLI 目标里路径会**自动扁平化**，用 `__` 连接。嵌套越深、前缀越长，越容易被 Agent 忽略——**尽量保持 1～2 层目录**，技能目录名本身宜短（如 `pdf`、`browse`）。

| 源目录路径 | sync 后的目标名 | Agent 友好度 |
|------------|-----------------|--------------|
| `pdf/` | `pdf` | 最佳 |
| `gstack/browse/` | `gstack__browse` | 通常可用 |
| `_gstack/agents/skills/browse/`（`--track`） | `_gstack__agents__skills__browse` | 易被 Claude Code 忽略 |

---

## 4. 整理已安装的技能

技能变多后，可在**源目录**里用文件夹分组；skillshare 在 `sync` 时自动扁平化，AI CLI 仍得到扁平结构。详见官方 [Organizing Skills](https://skillshare.runkids.cc/docs/how-to/daily-tasks/organizing-skills)。

### 分组原则

| 方式 | 说明 |
|------|------|
| **`--into` + `-s` / `--all`**（推荐） | 按仓库或领域分子目录，每个技能单独安装；目标名短，Agent 能加载 |
| **源目录根下平铺** | 技能少时最简单，sync 后目标名即技能名 |
| **`--track` 整仓**（不推荐） | 保留仓库深层路径，sync 后前缀过长，Claude Code 等会忽略 |

### 推荐目录结构（多仓库混合）

```text
源目录（例如 ~/Projects/skills/skills/）
├── frontend/                 # 自研 / 按领域
│   └── vue-best-practices/
├── anthropics/               # skillshare install anthropics/skills -s pdf --into anthropics
│   └── pdf/
└── gstack/                   # skillshare install garrytan/gstack --all --into gstack
    ├── browse/
    └── qa/
    # sync → gstack__browse、gstack__qa（可接受）
    # 勿用 --track → _gstack__agents__skills__browse（过长）
```

### 新技能：安装时直接进文件夹

不必先装再 `mv`，用 `--into` 一步到位（见上文 §3）。

### 已有扁平目录：手动迁移

若技能已在源目录根下平铺，可建文件夹后移动，再 `sync`：

```bash
cd ~/Projects/skills/skills   # 或你的源目录

mkdir -p frontend/react utils
mv react-best-practices frontend/react/
mv remotion utils/

skillshare sync   # 更新各 CLI 的链接；旧扁平名会被清理
```

### 查看与管理嵌套技能

```bash
skillshare list -g            # 按文件夹分组列出
skillshare check              # 检查更新（显示相对路径）
skillshare update frontend/react/react-best-practices   # 完整路径
skillshare update react-best-practices                  # 短名（唯一时）
skillshare update --all && skillshare sync
```

只有含 `SKILL.md` 的目录才算技能；中间文件夹（如 `frontend/`）仅用于组织，不必自带 `SKILL.md`。

### 从 `--track` 迁移

若源目录里已有 `_repo-name/` 整仓跟踪，建议卸载后按技能重装，缩短 sync 目标名：

```bash
skillshare uninstall _garrytan-gstack
skillshare install garrytan/gstack --all --into gstack
skillshare sync
```

---

## 5. 检查与更新技能

**只检查、不写盘**

```bash
skillshare check
skillshare check --json
skillshare check -p
```

**应用更新**

```bash
skillshare update my-skill
skillshare update team-skills
skillshare update --all
skillshare update --all --diff
```

- 对 **普通安装的技能**（推荐）：`skillshare update <技能名>` 或 `skillshare install <技能名> --update`，按元数据中的来源重新拉取。
- 对 **历史遗留的 tracked 仓库**（`--track`）：在源目录内 `git pull`；新安装请勿再用 `--track`。

上游删除技能后本地可能变 **stale**，可用：

```bash
skillshare update --all --prune
```

更新后务必：

```bash
skillshare sync
```

**升级 CLI 与内置技能**（与「更新仓库里的技能」不同）：

```bash
skillshare upgrade
```

---

## 6. 同步（`sync`）与反向收集（`collect`）

**源 → 各 AI CLI 目标**

```bash
skillshare sync
skillshare sync --dry-run
skillshare sync --all
```

`sync` 只分发技能；`sync agents` 只同步 agents；`sync --all` 可包含 skills、agents、extras（需在 config 中配置）。若某环境不支持符号链接，可对单目标改为 copy 模式后重 sync：

```bash
skillshare target cursor --mode copy
skillshare sync
```

**在目标里改写了技能，想收回源目录**

```bash
skillshare collect claude
skillshare collect --all
skillshare sync
```

---

## 7. 跨机器：Git 远端与 `push` / `pull`

当技能源目录是带 remote 的 git 仓库时：

```bash
skillshare push -m "Add or update skills"
skillshare pull
```

`pull` = `git pull` 后对目标执行 `sync`。与「无参数 `install` 按 URL 重装」是互补关系，详见官方 [Cross-Machine Sync](https://skillshare.runkids.cc/docs/how-to/sharing/cross-machine-sync)。

---

## 8. 开发自己的技能

**脚手架创建目录与 `SKILL.md` 模板**

```bash
skillshare new my-skill
skillshare new my-skill -p
skillshare new my-reviewer -P reviewer
```

命名规则：小写字母、数字、`-`、`_`，且以字母或 `_` 开头。

**编辑** 源目录下该技能的 `SKILL.md`（`description` 建议写清「做什么 + 何时触发」）。可选补充 `references/`、`scripts/` 等。

**发布到本机各 CLI**

```bash
skillshare sync
```

**备份到远端（若源已 `git init` 且配置 remote）**

```bash
skillshare push -m "Describe your skill change"
```

也可用 Web UI：`skillshare ui`（项目模式加 `-p`）。

### 组织级技能与团队共享

团队共用技能时，**源目录本身用 git 管理**（`init --remote` + `push` / `pull`），成员从各上游仓库用 `-s` / `--all` 安装到源目录，**不要用 `--track`**，以免 sync 后技能名过长、Agent 无法加载。

**负责人**：维护本仓库（或独立 skills 仓），在 `config.yaml` 的 `skills:` 里记录上游来源。

**成员：初次**

```bash
skillshare init --remote https://github.com/org/skills.git
skillshare install          # 按 config 清单安装
skillshare sync
```

**成员：日常更新**

```bash
skillshare pull             # 同步团队对源目录的 git 变更
skillshare update --all     # 拉各上游技能更新
skillshare sync
```

官方 [Organization-Wide Skills](https://skillshare.runkids.cc/docs/how-to/sharing/organization-sharing) 文档以 `--track` 为例；本仓库因 Claude Code 兼容性**改用按技能安装 + 源目录 git**。

---

## 9. 日常推荐流程（速查）

| 步骤 | 命令 |
|------|------|
| 新机 / 新仓库 | `skillshare init` 或 `skillshare init -p` |
| 装单个技能 | `skillshare install owner/repo/path/to/skill` |
| 浏览 / 批量装 | `skillshare install owner/repo` 或 `-s a,b` / `--all` |
| 按目录分组装 | `skillshare install owner/repo -s pdf --into repo-name` |
| ~~整仓 `--track`~~ | 不推荐：sync 名过长，Claude Code 会忽略 |
| 装完推到各 CLI | `skillshare sync` |
| 整理已有技能 | 源目录 `mkdir` + `mv` → `skillshare sync` |
| 看是否有更新 | `skillshare check` / `skillshare list -g` |
| 拉上游 | `skillshare update <名>` 或 `skillshare update --all` |
| 再同步 | `skillshare sync` |
| 自研新技能 | `skillshare new <name>` → 编辑 → `skillshare sync` → `skillshare push` |
| 另一台电脑取最新 | `skillshare pull` |
| 安全扫描 | `skillshare audit` |
| 排障 | `skillshare doctor` |

---

## 10. 参考链接

- 仓库与发行版：<https://github.com/runkids/skillshare>
- 文档首页：<https://skillshare.runkids.cc/docs/>
- 安装命令：<https://skillshare.runkids.cc/docs/reference/commands/install>
- 文件夹整理：<https://skillshare.runkids.cc/docs/how-to/daily-tasks/organizing-skills>
- 跟踪仓库（官方能力，本仓库不推荐）：<https://skillshare.runkids.cc/docs/understand/tracked-repositories>
- 组织级技能（团队共享）：<https://skillshare.runkids.cc/docs/how-to/sharing/organization-sharing>
- 命令总览：<https://skillshare.runkids.cc/docs/reference/commands>
- GitHub Action：`runkids/setup-skillshare@v1`（CI 里安装并 `sync`）
