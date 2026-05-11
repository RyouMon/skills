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

**Go（需已安装 Go）**

```bash
go install github.com/runkids/skillshare/cmd/skillshare@latest
```

**升级 CLI 本身**

```bash
skillshare upgrade
```

验证：

```bash
skillshare version
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

**交互式（推荐首次）**

```bash
skillshare init
```

**仅预览**

```bash
skillshare init --dry-run
```

**绑定已有技能 Git 仓库（远端）**

```bash
skillshare init --remote git@github.com:你的账号/your-skills.git
```

若仓库根目录不全是技能（还有 README、CI 等），可用 **`--subdir`** 把源指到子目录（例如 `skills/`）。非交互示例见官方 [init 文档](https://skillshare.runkids.cc/docs/reference/commands/init)。

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

## 3. 安装技能：开源一律建议 `--track`

**`--track`（简写 `-t`）**：克隆时**保留 `.git`**，把整段仓库作为「可跟踪的技能源」放进源目录；之后可用 `skillshare check` / `skillshare update` 拉上游更新。被跟踪的仓库在源目录里会以 **`_` 前缀**命名（嵌套层级用 `__` 分隔），例如 `_team__frontend__ui`。

对**来自开源仓库、希望随上游更新**的技能集合，推荐始终：

```bash
skillshare install github.com/组织或用户/仓库名 --track
```

可与分支、子目录、项目模式组合，例如：

```bash
skillshare install github.com/team/skills --track --branch develop --all
skillshare install github.com/team/skills --track -p
```

**单技能路径安装**（不整仓 track，适合只装某一个目录下的技能）：

```bash
skillshare install owner/repo/skills/pdf
```

**浏览仓库内多个技能再选**（不提供到具体 skill 的路径时进入 discovery）：

```bash
skillshare install anthropics/skills
```

**无参数 `install`（从 `config.yaml` 清单安装）**  
会安装配置里列出的远程技能；其中 `tracked: true` 与命令行 `--track` 等价。适合团队复现同一套技能：

```bash
skillshare install
skillshare install -p
```

安装只改**源目录**；要让各 CLI 生效需再执行 **`skillshare sync`**。

其他常用标志：`--skill` / `-s`（多技能仓库里指定名称或 glob），`--all` / `-y`，`--into`（安装到源下子目录），`--force`，`--update` / `-u`，`--branch` / `-b`，`--dry-run` / `-n`，`--project` / `-p`。完整说明见 [install 命令文档](https://skillshare.runkids.cc/docs/reference/commands/install)。

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
| 装开源技能集（建议跟踪） | `skillshare install github.com/org/repo --track` |
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
