---
title: "BMad 框架架構"
date: 2026-07-02
tags: [bmad, architecture, agent, workflow, skill]
aliases: [bmad-architecture]
---

BMad 是一套以 **提示詞驅動** 的 AI Agent 開發框架，在 Claude Code / Copilot / Cursor 等 IDE 內以「Skill 呼叫」為執行單位。本文整理 workspace 內採用的四層架構、Agent 與 Workflow 的分類，以及支撐長對話存活的 Compaction Survival 機制。

---

## Overview

BMad 採用 **Module > Agent > Workflow > Skill** 四層封裝。每一層對應不同的抽象邊界：Module 是可安裝的資產包、Agent 是有人格的 LLM 人物、Workflow 是有順序的多步任務、Skill 是最小可執行單元（一份 `SKILL.md`）。當前 workspace 安裝 v6.9.0，共 7 個模組。

---

## Core Concepts

### 四層架構

| 層次 | 定義 | 檔案體現 |
|---|---|---|
| **Module** | 最大封裝單位。一組針對特定領域的 skills、agents、templates 集合，可用 `npm` 安裝或用 `bmb` 打包 | `_bmad/<module>/` |
| **Agent** | 有名字、角色與行為原則的 AI 人格。可被 Party Mode 或 Orchestrator spawn 為 subagent | `_bmad/<module>/.../bmad-agent-<name>/SKILL.md` |
| **Workflow** | 由多個步驟組成的引導式流程。每步驟可為單一 skill、外部指令或使用者確認 | `_bmad/<module>/<phase>/<skill>/SKILL.md`（含 step 檔案） |
| **Skill** | 最小執行單位。每個 skill 是一份 `SKILL.md`，內含 identity、principles、workflow steps | `_bmad/<module>/**/SKILL.md` |

所有 skill 都會在安裝時被同步佈署到 `.claude/skills/`、`.agents/skills/` 等 IDE 目錄，各 IDE 由該處讀取。

---

### Agent 三型

BMad Builder（bmb 模組）支援建立三種類型的 agent，差別在於是否具備跨會話記憶與自主排程能力。

#### Stateless Agent（無狀態）

結構最精簡，所有定義集中於單一 `SKILL.md`。每次對話獨立，無跨會話記憶。適合單次任務型專家（例：`bmad-agent-analyst` 進行一次性市場研究）。

#### Memory Agent（記憶型）

在 Stateless 基礎上加入 **Sanctum** 記憶結構與 **First Breath** 首次啟動引導。能累積使用者偏好與專案知識，跨會話保持一致性。

```
my-agent/
├── SKILL.md
└── sanctum/
    ├── first-breath.md
    ├── preferences.md
    └── domain-knowledge.md
```

#### Autonomous Agent（自主型）

在 Memory Agent 基礎上加入 **PULSE** 機制，Agent 可在會話之間自主執行。需搭配 Claude Code Cron 或其他外部排程機制。

> [!WARNING]
> BMad 本身不提供定時排程，Autonomous Agent 需搭配外部排程工具才能運作。

---

### Workflow 三型

| 類型 | 結構 | 適合情境 |
|---|---|---|
| **Simple Utility** | 單一 `SKILL.md` | 無狀態的單次工具（例：`bmad-help`） |
| **Simple Workflow** | `SKILL.md` + inline 步驟 | 步驟少、流程固定的引導式任務（例：`bmad-product-brief`） |
| **Complex Workflow** | `SKILL.md` + `references/` step 檔案 | 多階段、長時間執行、需 Compaction Survival（例：`bmad-prd`、`bmad-architecture`） |

---

### 三種執行模式

同一個 workflow skill 可在不同模式下呼叫，控制互動深度。

| 模式 | 說明 | 觸發方式 |
|---|---|---|
| **Guided** | 預設模式。對話式引導，每步驟需使用者確認 | 直接呼叫 skill |
| **Yolo** | 一次性草稿產出，跳過中間確認 | 對話中說「just draft it」或 `--yolo` |
| **Headless** | 無互動自動執行，使用合理預設值 | 呼叫時帶 `-H` / `--headless` |

多數 bmb / bmm skill 的 `args` 欄支援 `-H`，見各模組 `module-help.csv`。

---

### Compaction Survival

長對話會被 IDE 自動壓縮（compaction），過往回應會被摘要化。BMad 的解法是 **「文件本身就是狀態」**（Document-Itself Pattern）：每個 workflow 的產出文件都含 frontmatter 記錄執行進度。

```yaml
---
title: "分析報告：市場研究"
status: "step-3-completed"
stepsCompleted: [1, 2, 3]
updated: "2026-07-02T11:30:00Z"
---
```

Context 被壓縮後，下一步的 LLM 只需讀取輸出文件的 frontmatter，即可還原所有狀態，不依賴對話記憶。此模式在 `bmad-prd`、`bmad-architecture`、`bmad-create-story` 等 Complex Workflow 全面採用。

---

### 輸出資料夾（output-location）

每個 skill 的產出寫入預定資料夾，見 `module-help.csv` 的 `output-location` 欄。workspace 常用的輸出根目錄：

| 資料夾 | 用途 |
|---|---|
| `_bmad-output/planning-artifacts/` | 規劃階段產出（brief、PRD、UX、architecture、epics/stories） |
| `_bmad-output/implementation-artifacts/` | 實作階段產出（sprint status、story、review report） |
| `_bmad-output/test-artifacts/` | 測試相關產出（tea 模組） |
| `_bmad-output/project-knowledge/` | 專案知識（`bmad-document-project`、`bmad-generate-project-context` 產出） |

---

## References

- [[multi-agent]] — Party Mode、Orchestrator 與序列接力
- [[operations]] — Skill 呼叫、custom config、workflow 入口
- [[modules/bmb|BMad Builder]] — 建立自訂 Agent、Workflow、Module
- [[modules/bmm|BMad Method]] — 主流程模組實作範例
- [BMad 官方文件](https://docs.bmad-method.org/)
- [BMad Builder 文件](https://bmad-builder-docs.bmad-method.org/)
