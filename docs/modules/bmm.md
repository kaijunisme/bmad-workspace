---
title: "BMad Method（bmm）模組"
date: 2026-07-02
tags: [bmad, module, bmm, workflow]
aliases: [bmad-bmm-module]
---

bmm 是 BMad 的主流程模組，涵蓋從需求分析到程式碼交付的完整開發生命週期。它以 **四階段（analysis → planning → solutioning → implementation）** 組織 workflow，並提供 6 位具名 agent 對應軟體團隊的常見角色。

---

## Overview

當前 workspace 版本為 v6.9.0（built-in）。bmm 提供 26 個 skill、6 位 agent。新專案建議入口為 `CB`（Product Brief）或 `WB`（PRFAQ Challenge）；既有專案（brownfield）建議先跑 `GPC`（Generate Project Context）產出 `project-context.md`，讓 agent 快速掌握程式碼庫。

---

## Core Concepts

### 6 位 Agent

| Agent | 代號 | 角色 | 專長 |
|---|---|---|---|
| **Mary** | `bmad-agent-analyst` | Business Analyst | 市場研究、競品分析、需求挖掘 |
| **Paige** | `bmad-agent-tech-writer` | Technical Writer | 技術文件、Mermaid 圖表、術語解釋 |
| **John** | `bmad-agent-pm` | Product Manager | PRD 撰寫、使用者訪談、優先序 |
| **Sally** | `bmad-agent-ux-designer` | UX Designer | 使用者研究、UI / 互動設計 |
| **Winston** | `bmad-agent-architect` | Architect | 系統架構、API 設計、技術決策 |
| **Amelia** | `bmad-agent-dev` | Developer | Story 實作、TDD |

任一 agent 都可用「跟 <名字> 說話」或 skill 名稱直接呼叫，亦可在 Party Mode 中被 orchestrator spawn。

---

### 四階段流程

```
1-analysis      → 市場 / 技術 / 領域研究、腦力激盪、產品簡報
2-planning      → PRD 建立與驗證、UX 設計
3-solutioning   → 系統架構、Epic / Story 規劃、實作準備度確認
4-implementation → Sprint 規劃 → Story 建立 → 開發 → Code Review → 回顧
```

---

### 1-analysis：需求分析

| Code | Skill | 說明 |
|---|---|---|
| `IN` | `bmad-investigate` | 對輸入做鑑識級調查，產生 evidence-graded 案卷 |
| `BP` | `bmad-brainstorming` | 專家引導的腦力激盪（bmm 版，區別於 core `BSP`） |
| `MR` | `bmad-market-research` | 市場分析、競品情報、趨勢 |
| `DR` | `bmad-domain-research` | 產業領域深度研究、術語 |
| `TR` | `bmad-technical-research` | 技術可行性、架構選項 |
| `CB` | `bmad-product-brief` | 快速確立產品概念的簡報（Create Brief） |
| `WB` | `bmad-prfaq` | Amazon Working Backwards 壓力測試（PRFAQ Challenge） |

> [!TIP]
> `CB` 適合概念已明確、只需結構化的情境；`WB` 適合需要挑戰假設、pressure-test 的情境。

---

### 2-planning：產品規劃

| Code | Skill | 說明 |
|---|---|---|
| `PRD` | `bmad-prd` | Create / Edit / Review PRD 三合一（Complex Workflow，多階段引導） |
| `CU` | `bmad-ux` | UX / UI 設計文件 |

`bmad-prd` 支援三種呼叫模式：全新建立、以 change signal 修改既有 PRD、對照 checklist 驗證。輸出含 HTML findings 報告。

---

### 3-solutioning：架構與 Story 拆分

| Code | Skill | 說明 |
|---|---|---|
| `CA` | `bmad-architecture` | 建立技術架構文件（Complex Workflow） |
| `CE` | `bmad-create-epics-and-stories` | 從架構產出 Epics 與 Stories |
| `IR` | `bmad-check-implementation-readiness` | 確認 PRD / UX / 架構 / Stories 是否對齊 |

`CA` 為 spine architecture — 只寫下必要的 invariants，可 scale 至 full architecture。Brownfield 專案用 `CA` 會 ratify 現有程式碼庫。

---

### 4-implementation：Sprint 與 Story 週期

| Code | Skill | 說明 |
|---|---|---|
| `SP` | `bmad-sprint-planning` | 產生 Sprint 計畫（`sprint-status.yaml`） |
| `SS` | `bmad-sprint-status` | 查看 Sprint 進度與下一步建議 |
| `CS` | `bmad-create-story` | 準備下一個可開發 Story（含完整 context） |
| `VS` | `bmad-create-story` (action=validate) | 驗證 Story readiness |
| `DS` | `bmad-dev-story` | 執行 Story 實作（Dev Story） |
| `CR` | `bmad-code-review` | Code Review（內部呼叫 `AR`） |
| `CK` | `bmad-checkpoint-preview` | 引導式 commit / branch / PR 檢視 |
| `QA` | `bmad-qa-generate-e2e-tests` | 為已實作程式產生 API / E2E 自動化測試 |
| `ER` | `bmad-retrospective` | Epic 結束後回顧總結 |

**典型迴圈**：`CS → VS → DS → CR → CS → ...`。若 `CR` 有 issue，回到 `DS` 修正；若 issue 過大，觸發 `CC`（Correct Course）。Epic 結束後跑 `ER`，重大變動則用 `CC`。

---

### Anytime skills（無階段限制）

| Code | Skill | 說明 |
|---|---|---|
| `DP` | `bmad-document-project` | 分析既有專案產生文件（brownfield 起手式） |
| `GPC` | `bmad-generate-project-context` | 掃描 codebase 產出 LLM 友善的 `project-context.md` |
| `QQ` | `bmad-quick-dev` | 一站式：意圖輸入 → 規劃 → 實作 → 審查 → 呈現 |
| `CC` | `bmad-correct-course` | 應付重大變動：可能建議從頭、更新 PRD、重做架構或修 stories |

> [!TIP]
> **Brownfield 起手三部曲**：`DP`（產文件）→ `GPC`（產 context）→ 依需求進入四階段。

---

### Tech Writer Sub-actions

Paige（`bmad-agent-tech-writer`）有 5 個獨立 action：

| Code | Action | 說明 |
|---|---|---|
| `WD` | `write` | 依需求撰寫指定文件；multi-turn 對話，可 spawn subprocess 做研究 |
| `US` | `update-standards` | 更新 tech-writer 的 `documentation-standards.md` 記憶檔 |
| `MG` | `mermaid` | 依描述產生 Mermaid 圖 |
| `VD` | `validate` | 對照文件標準審查指定文件，返回優先順序建議 |
| `EC` | `explain` | 產生技術概念的清晰解釋（含範例 + 圖） |

---

### 與其他模組的接軌

- 完成 `bmad-create-story:create`（`CS`）後，可 handoff 到 tea 的 `AT`（ATDD）產出紅燈驗收測試
- 完成 `bmad-sprint-planning`（`SP`）後，可 handoff 到 baut 的 `SA`（Story Automator）全自動跑完 Story 週期
- 任一階段皆可用 core 的 `PM`（Party Mode）拉多位 agent 討論

---

## References

- [[architecture]] — 四層架構與 Complex Workflow 機制
- [[multi-agent]] — Party Mode 與 agent 呼叫
- [[operations]] — Skill 呼叫、Menu-code 快捷
- [[modules/tea|TEA 模組]] — ATDD 與 story 週期整合
- [[modules/baut|BMad Automator]] — Story 週期全自動化
- [bmm `module-help.csv`](../../_bmad/bmm/module-help.csv) — 完整 skill 清單（SSOT）
- [BMad Method 官方文件](https://docs.bmad-method.org/)
