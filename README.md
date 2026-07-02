# bmad-workspace

個人維護的 BMad Method 工作空間，作為 AI 驅動軟體與遊戲開發流程的基底。

- 官方站點：<https://bmadcode.com/>
- 官方文件：<https://docs.bmad-method.org/>
- 官方社群：<https://discord.gg/gk8jAdXWmj>

---

## 這個 Workspace 是什麼

BMad Method 是一套「AI Agent 團隊 + Skill/Workflow 資產」的方法論。它把產品規劃、架構設計、開發、測試、遊戲設計等角色拆成獨立 agent skill，再由多個 IDE（Claude Code、Cursor、Codex 等）呼叫執行。

這個 repo 是 BMad 安裝後的完整目錄快照，涵蓋：

- `_bmad/` — BMad 核心與模組資產（安裝器管理）
- `.claude/skills/`、`.agents/skills/`、`.kiro/skills/` 等 — 各 IDE 對應的 skill 佈署位置
- `_bmad-output/` — 執行 workflow 產出的檔案（規劃、測試、實作 artifact）

---

## 模組總覽

BMad 分為 **built-in** 與 **external** 兩類模組。built-in 由 `bmad-method` npm 主包提供；external 由獨立 npm 套件 + GitHub repo 分發，`bmad-method` 安裝器會自動下載。

| Module | 全名 | 用途 | 來源 |
|---|---|---|---|
| `core` | BMad Core | 底層共用能力（brainstorming 等） | built-in |
| `bmm` | BMad Method | 主流程：文件、規劃、架構、開發、故事 | built-in |
| `bmb` | BMad Builder | 建置/客製 skill 與 agent 的工具鏈 | [`bmad-builder`](https://github.com/bmad-code-org/bmad-builder) |
| `cis` | Creative Intelligence Suite | 創新策略、設計思考、簡報敘事 | [`bmad-creative-intelligence-suite`](https://github.com/bmad-code-org/bmad-module-creative-intelligence-suite) |
| `gds` | Game Dev Studio | 遊戲開發（Unity / Unreal / Godot） | [`bmad-game-dev-studio`](https://github.com/bmad-code-org/bmad-module-game-dev-studio) |
| `tea` | Test Architecture Enterprise | 測試策略、Playwright / Pact / k6 自動化 | [`bmad-method-test-architecture-enterprise`](https://github.com/bmad-code-org/bmad-method-test-architecture-enterprise) |
| `baut` | BMad Automator | Story 生命週期自動化（sprint → dev → QA → retro） | external |

各模組的完整 skill 清單見對應目錄下的 `module-help.csv`（例如 `_bmad/bmm/module-help.csv`）。實際安裝版本記錄在 `_bmad/_config/manifest.yaml`。

---

## 前置需求

| 工具 | 用途 | 備註 |
|---|---|---|
| Node.js ≥ 18 | 執行 `npx bmad-method` | 建議 LTS |
| npx | 呼叫 `bmad-method` CLI | 隨 Node.js 附帶 |
| Git | 版本控制、external 模組 clone | — |
| `uv` | 執行 BMad 內建 Python 腳本 | 選用，越來越多 workflow 依賴 `uv run` |

---

## 安裝

首次安裝（完整七模組）：

```bash
npx --yes bmad-method install \
  --directory <target-dir> \
  --channel stable \
  --tools claude-code,cursor \
  -y
```

只安裝主流程（bmm + bmb）：

```bash
npx --yes bmad-method install \
  --directory <target-dir> \
  --channel stable \
  --modules bmm,bmb \
  --tools claude-code \
  -y
```

參數說明（節錄，完整清單見 `npx bmad-method install --help`）：

| Flag | 用途 |
|---|---|
| `--directory <path>` | 指定安裝目錄（不用先 `cd`） |
| `--modules <ids>` | 逗號分隔模組 ID，省略則安裝全部 |
| `--tools <ids>` | 逗號分隔 IDE ID，`--list-tools` 可查全清單 |
| `--channel stable\|next` | `stable` 綁最新 release tag；`next` 綁 main HEAD |
| `--pin <code>=<tag>` | 將指定模組鎖到特定版本，例：`--pin bmb=v1.8.0` |
| `--set <module>.<key>=<value>` | 非互動式設定模組選項，`--list-options` 可查 |
| `-y` / `--yes` | 全部使用預設，跳過互動 prompt |
| `--action install\|update\|quick-update` | 安裝新環境 / 更新既有 / 快速更新 |

---

## 更新

在已安裝的目錄執行：

```bash
npx --yes bmad-method install \
  --directory <target-dir> \
  --action update \
  --channel stable \
  -y
```

只更新既有的部分模組（例：僅 bmm + bmb）：

```bash
npx --yes bmad-method install \
  --directory <target-dir> \
  --action update \
  --channel stable \
  --modules bmm,bmb \
  -y
```

安裝器會將被覆寫的檔案存為 `*.bak`。若已用 Git 版控，可直接清掉：

```bash
find <target-dir> -name '*.bak' -type f -delete
```

安全保留區（安裝器不會覆寫）：

- `_bmad/custom/config.toml` — 團隊共用 override（committed）
- `_bmad/custom/config.user.toml` — 個人 override（gitignored）

---

## 目錄結構

```
bmad-workspace/
├── _bmad/                          # 核心資產（安裝器管理）
│   ├── _config/manifest.yaml       # 安裝版本記錄（SSOT）
│   ├── config.toml                 # 主設定（installer-managed，勿手改）
│   ├── config.user.toml            # 個人答案（installer-managed）
│   ├── custom/                     # 使用者 override（安全區）
│   ├── core/  bmm/  bmb/  cis/  gds/  tea/  baut/
│   │                               # 各模組資產（skills / workflows / templates）
│   └── scripts/                    # 共用 Python 腳本
├── .claude/skills/                 # Claude Code 佈署位置
├── .agents/skills/                 # github-copilot / codex / cursor / gemini / windsurf / openclaw / opencode
├── .agent/skills/                  # antigravity 佈署位置
├── .kiro/skills/                   # Kiro 佈署位置
├── .github/agents/                 # GitHub Copilot commands
├── _bmad-output/                   # workflow 產出
│   ├── planning-artifacts/
│   ├── implementation-artifacts/
│   └── test-artifacts/
└── docs/                           # 專案文件（bmm project_knowledge）
```

---

## IDE 整合

安裝器一次會將 skill 佈署到所有支援 IDE 的目錄。目前支援：

| IDE | Skill 佈署位置 |
|---|---|
| Claude Code | `.claude/skills/` |
| GitHub Copilot | `.agents/skills/` + `.github/agents/`（commands） |
| Codex | `.agents/skills/` |
| Cursor | `.agents/skills/` |
| Gemini | `.agents/skills/` |
| Windsurf | `.agents/skills/` |
| OpenClaw | `.agents/skills/` |
| OpenCode | `.agents/skills/` |
| Antigravity | `.agent/skills/` |
| Kiro | `.kiro/skills/` |

指定要佈署哪些 IDE：`--tools claude-code,cursor,kiro`。查全清單：`npx bmad-method install --list-tools`。

---

## 常用工作流入口

在 IDE 中呼叫 `bmad-help` skill 可列出當前所有可用 workflow。以下為主要模組的代表性 skill（menu-code 對應 `module-help.csv` 中的 `menu-code` 欄）：

| 模組 | menu-code | Skill | 用途 |
|---|---|---|---|
| bmm | DP | `bmad-document-project` | 分析既有專案產生文件 |
| bmm | — | `bmad-plan-project` / `bmad-create-story` | 規劃與 story 建立 |
| bmb | SB | `bmad-bmb-setup` | 建置/更新 builder 模組設定 |
| bmb | — | `bmad-agent-builder` | 建立新 agent skill |
| cis | IS | `bmad-cis-innovation-strategy` | 識別破壞式機會、商業模式創新 |
| gds | DP | `gds-document-project` | 分析既有遊戲專案 |
| tea | TMT | `bmad-teach-me-testing` | 測試基礎教學（7 堂 session） |
| baut | SA | `bmad-story-automator` | Story 生命週期自動化 |

---

## 分支說明

| Branch | 模組組合 | 用途 |
|---|---|---|
| `main` | core + bmm + bmb + cis + gds + tea + baut | 完整安裝，作為 upstream 樣板 |
| `bmb` | core + bmm + bmb + baut | 精簡工作流，聚焦軟體開發主線 |

切換分支後，`_bmad/`、`.claude/skills/` 等目錄會隨之切換，無需重跑安裝器。
