# skills

个人技能集合。本仓库技能通过 **[skillshare](https://github.com/runkids/skillshare)**（命令行 `skillshare`）安装、更新并同步到各 AI CLI。下文「skillshare」均指该官方工具；文档站：<https://skillshare.runkids.cc/docs/>。

---

## 为何用 skillshare

- **单一来源**：技能放在 skillshare 的「源目录」，再 `sync` 到 Claude、Cursor、Codex 等 60+ 目标。
- **安装与更新**：从 GitHub/GitLab 等安装；带 `--track` 的仓库可用 `check` / `update` 拉取上游。
- **跨机器**：源目录用 git 管理时，可用 `push` / `pull` 与远端同步后再 `sync`。

路径约定（官方）：

| 平台 | 全局技能源 | 项目技能源 |
|------|------------|------------|
| Windows | `%AppData%\skillshare\skills\` | 当前仓库 `.skillshare\skills\` |
| macOS / Linux | `~/.config/skillshare/skills/` | 当前仓库 `.skillshare/skills/` |

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
  Enter subdirectory name: skills/
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

与官方 [install 文档](https://skillshare.runkids.cc/docs/reference/commands/install) 一致的常见用法示例：

```bash
# GitHub 简写：装到具体技能路径
skillshare install anthropics/skills/skills/pdf

# 只给仓库，进入发现模式（交互挑选）
skillshare install anthropics/skills

# 本地路径
skillshare install ~/Downloads/my-skill

# 无参数：按 config.yaml 中的 skills 清单安装
skillshare install
skillshare install -p
```

多技能仓库可非交互指定名称或 glob、或一次装全：

```bash
skillshare install anthropics/skills -s pdf,commit
skillshare install anthropics/skills --all
```

**`--track`（简写 `-t`）**：整仓克隆并保留 `.git`，便于之后用 `skillshare check` / `skillshare update` 拉上游；适合团队共享的固定仓库。可与分支、子目录、项目模式组合，例如：

```bash
skillshare install github.com/team/skills --track
skillshare install github.com/team/skills --track --branch develop --all
skillshare install github.com/team/skills --track -p
```

配置里 `tracked: true` 与命令行 `--track` 等价。

安装只改**源目录**；要让各 CLI 生效需再执行 **`skillshare sync`**。

其他常用标志：`--skill` / `-s`，`--all` / `-y`，`--into`，`--force`，`--update` / `-u`，`--branch` / `-b`，`--dry-run` / `-n`，`--project` / `-p`。完整说明见上文 install 文档链接。

---

## 4. 检查与更新技能

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

- 对 **tracked 仓库**：在源目录内 `git pull`，并有安全审计门禁；本地有未提交改动时可先提交或用 `--force`。
- 对 **带元数据的远程单技能**：按记录的来源重新安装。

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

## 5. 同步（`sync`）与反向收集（`collect`）

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

## 6. 跨机器：Git 远端与 `push` / `pull`

当技能源目录是带 remote 的 git 仓库时：

```bash
skillshare push -m "Add or update skills"
skillshare pull
```

`pull` = `git pull` 后对目标执行 `sync`。与「无参数 `install` 按 URL 重装」是互补关系，详见官方 [Cross-Machine Sync](https://skillshare.runkids.cc/docs/how-to/sharing/cross-machine-sync)。

---

## 7. 开发自己的技能

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

---

## 8. 日常推荐流程（速查）

| 步骤 | 命令 |
|------|------|
| 新机 / 新仓库 | `skillshare init` 或 `skillshare init -p` |
| 从仓库安装技能 | `skillshare install owner/repo` 或 `owner/repo/路径/到/技能` |
| 装完推到各 CLI | `skillshare sync` |
| 看是否有更新 | `skillshare check` |
| 拉上游 | `skillshare update <名或仓库>或 skillshare update --all` |
| 再同步 | `skillshare sync` |
| 自研新技能 | `skillshare new <name>` → 编辑 → `skillshare sync` → `skillshare push` |
| 另一台电脑取最新 | `skillshare pull` |
| 安全扫描 | `skillshare audit` |
| 排障 | `skillshare doctor` |

---

## 9. 参考链接

- 仓库与发行版：<https://github.com/runkids/skillshare>
- 文档首页：<https://skillshare.runkids.cc/docs/>
- 命令总览：<https://skillshare.runkids.cc/docs/reference/commands>
- GitHub Action：`runkids/setup-skillshare@v1`（CI 里安装并 `sync`）
