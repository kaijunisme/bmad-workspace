---
title: "Game Dev Studio（gds）模組"
date: 2026-07-02
tags: [bmad, module, gds, game-dev, unity, unreal, godot]
aliases: [bmad-gds-module]
---

gds 是專門處理**遊戲開發**的模組，將 bmm 的軟體交付流程改寫為 pre-production → design → technical → production 四階段，並加入 gametest 專用測試 workflow。支援 Unity、Unreal Engine、Godot 三大引擎的專案脈絡。

---

## Overview

gds 為 external 模組，npm 套件為 [`bmad-game-dev-studio`](https://github.com/bmad-code-org/bmad-module-game-dev-studio.git)。當前 workspace 版本為 v0.6.0。提供 3 位具名 agent、23 個 skill（含 gametest 專屬）。新專案入口為 `BG`（Brainstorm Game）或 `GB`（Game Brief）；既有專案用 `DP`（Document Project）+ `PC`（Project Context）。

---

## Core Concepts

### 3 位 Agent

| Agent | 代號 | 角色 |
|---|---|---|
| **Cloud Dragonborn** | `gds-agent-game-architect` | Game Systems Architect（引擎、系統、網路、基礎架構） |
| **Samus Shepard** | `gds-agent-game-designer` | Game Designer（GDD、玩法、敘事、關卡） |
| **Link Freeman** | `gds-agent-game-dev` | Game Developer（story 實作、程式碼、QA、sprint 主持） |

Link Freeman 是「合併型」dev agent，同時承擔 code、QA、scrum master 三個角色。

---

### 四階段流程

```
1-preproduction → 遊戲點子腦力激盪、領域研究、Game Brief
2-design        → GDD、Narrative、UX、PRD（可選）
3-technical     → Architecture、Epics/Stories、Test Framework/Design、Project Context
4-production    → Sprint Planning、Story 週期、Code Review、Retrospective
（gametest 為獨立階段：Test Automate、E2E、Playtest、Performance、Test Review）
```

---

### 1-preproduction：概念發想

| Code | Skill | 說明 |
|---|---|---|
| `BG` | `gds-brainstorm-game` | 遊戲專屬腦力激盪（含遊戲類型、機制、目標受眾） |
| `DR` | `gds-domain-research` | 遊戲產業領域深度研究 |
| `GB` | `gds-create-game-brief` | 互動式建立 game brief，定義遊戲願景 |

---

### 2-design：設計文件

| Code | Skill | 說明 |
|---|---|---|
| `GDD` | `gds-gdd` | Game Design Document（GDD）— 遊戲的主設計 artifact |
| `ND` | `gds-create-narrative` | 敘事設計：故事結構、角色弧、世界觀 |
| `CU` | `gds-ux` | Game UX：`DESIGN.md` + `EXPERIENCE.md`，含 UI / HUD / 輸入 / 玩家旅程 |
| `PRD` | `gds-prd` | Game PRD（常從 GDD 衍生，或為與外部工具整合準備） |

**GDD 涵蓋**：核心 pillars、遊戲機制、進度系統、關卡、美術、音效、開發 epics。是 gds 最關鍵的 artifact。

---

### 3-technical：架構與 Stories

| Code | Skill | 說明 |
|---|---|---|
| `PC` | `gds-generate-project-context` | 產生 `project-context.md` 給選定的 agentic tool |
| `GA` | `gds-game-architecture` | Scale-adaptive 遊戲架構：引擎系統、網路、技術設計 |
| `CE` | `gds-create-epics-and-stories` | 從 GDD 需求產出 Epics 與 Stories |
| `IR` | `gds-check-implementation-readiness` | 確認 GDD / UX / Architecture / Stories 對齊 |
| `TF` | `gds-test-framework` | 初始化 Unity / Unreal / Godot 測試框架 |
| `TD` | `gds-test-design` | 產生涵蓋玩法、進度、品質需求的測試方案 |

---

### 4-production：Sprint 與 Story 週期

| Code | Skill | 說明 |
|---|---|---|
| `SP` | `gds-sprint-planning` | 從 epic 檔案生成 / 更新 `sprint-status.yaml` |
| `SS` | `gds-sprint-status` | 查看 sprint 進度、風險、下一步建議 |
| `CS` | `gds-create-story` | 建立含完整 context 的 story |
| `DS` | `gds-dev-story` | 執行 story 實作 |
| `CR` | `gds-code-review` | Clean-context QA code review（僅對 Ready for Review 的 story） |
| `ER` | `gds-retrospective` | Epic 結束後回顧 |
| `IN` | `gds-investigate` | Forensic 案卷調查（bug、incident、重建陌生程式碼） |

---

### gametest：測試專屬階段

gds 為遊戲測試設計了獨立階段，`phase = gametest`：

| Code | Skill | 說明 |
|---|---|---|
| `TA` | `gds-test-automate` | 產生自動化遊戲測試 |
| `ES` | `gds-e2e-scaffold` | 建立 E2E 測試基礎架構 |
| `PP` | `gds-playtest-plan` | 結構化 playtest 計畫 |
| `PT` | `gds-performance-test` | 設計效能測試策略 |
| `TR` | `gds-test-review` | 審查測試品質與涵蓋 |

推薦順序：`TA → ES → PP → PT → TR`。

---

### Anytime skills

| Code | Skill | 說明 |
|---|---|---|
| `DP` | `gds-document-project` | 分析既有遊戲專案產生文件（brownfield 起手） |
| `QD` | `gds-quick-dev` | 一站式：任何 intent 或 change request 走完 clarify → plan → implement → review → present |
| `CC` | `gds-correct-course` | 應付 sprint 中的重大 off-track |

> [!TIP]
> **Brownfield 遊戲專案起手**：`DP`（產文件）→ `PC`（產 context）→ 依需求進入四階段。

---

### 與其他模組的協作

- gds 的四階段是**獨立**的替代版本，非 bmm 的擴充。同一專案通常只選其中一個
- 但 gds 專案可用 core 的 `PM`（Party Mode）拉入 bmm agent（例：Winston 討論架構）
- 敘事類 skill 可與 cis 的 `ST`（Storytelling）配合
- 測試 skill 可與 tea 配合（不過 gds 內建的 gametest phase 通常已足夠）

---

## References

- [[architecture]] — Complex Workflow 與四階段結構
- [[multi-agent]] — 混合 gds + bmm agent 討論
- [[operations]] — Anytime skill 呼叫
- [[modules/cis|CIS 模組]] — 敘事框架補強
- [[modules/tea|TEA 模組]] — 進階測試架構（可選）
- [gds `module-help.csv`](../../_bmad/gds/module-help.csv) — 完整 skill 清單（SSOT）
- [gds GitHub](https://github.com/bmad-code-org/bmad-module-game-dev-studio)
- [gds 官方文件](https://game-dev-studio-docs.bmad-method.org/)
