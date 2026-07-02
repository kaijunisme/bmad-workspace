---
title: "BMad Automator（baut）模組"
date: 2026-07-02
tags: [bmad, module, baut, automation, story-cycle]
aliases: [bmad-baut-module]
---

baut 是 **Story 生命週期自動化** 模組，將 bmm 的 `CS → DS → CR → 下一個 CS` 迴圈包成單一自動化 workflow。適合 sprint 計畫已就緒、想讓 agent 自主跑完整個 story 週期而不需人工每步確認的情境。

---

## Overview

baut 為 external 模組，安裝方式為 GitHub 直接安裝（無獨立 npm 套件）。當前 workspace 追蹤 SHA `593f338`。只有 2 個 skill，兩個都在 `4-implementation` 階段，且都依賴 bmm 已完成 sprint planning。

---

## Core Concepts

### Skills 清單

| Code | Skill | Preceded-by | 用途 |
|---|---|---|---|
| `SA` | `bmad-story-automator` | `bmad-sprint-planning`（bmm `SP`） | 自動跑完 create → dev → QA → review → retrospective 全流程 |
| `SAR` | `bmad-story-automator-review` | `bmad-dev-story`（bmm `DS`） | 自動化 code review flow（含 auto-fix 與 sprint-status 同步） |

`SA` 是主入口，`SAR` 是 `SA` 內部呼叫的子流程，也可獨立使用。

---

### 完整前置條件

呼叫 `SA` 之前，專案應已完成：

```
1. 需求：bmad-prd (PRD) 或至少有明確 backlog
2. 架構：bmad-architecture (CA)
3. Stories：bmad-create-epics-and-stories (CE)
4. Readiness：bmad-check-implementation-readiness (IR)
5. Sprint plan：bmad-sprint-planning (SP)  ← baut SA 從這裡接手
```

---

### SA 執行流程

`bmad-story-automator` 內部依序：

```
Loop until sprint 完成：
  1. bmad-create-story (CS)        ← 取下一個 story
  2. （可選）bmad-testarch-atdd (AT) ← 若安裝 tea，先寫紅燈驗收測試
  3. bmad-dev-story (DS)            ← 實作
  4. bmad-story-automator-review    ← SAR：自動 code review + auto-fix
  5. 更新 sprint-status.yaml
  6. 若 epic 完成 → bmad-retrospective (ER)，否則回到步驟 1
```

---

### 何時該用 baut

**適合**：

- Sprint 已規劃、stories 已排序、context 已寫好
- 可容忍失敗（agent 卡住可回頭手動接手）
- 想加速 spike / prototype、或跑成本較低的 refactor 迴圈

**不適合**：

- Story 需求還在演進、每個 story 都要人工調整 context
- 高風險或不可逆的變更（生產資料、金流、通知）
- 希望每步人工把關的重要功能

---

### SAR 獨立使用

`bmad-story-automator-review`（`SAR`）也可以在**非自動化情境**單獨呼叫，做為 `bmad-dev-story` 之後的自動 code review：

```
（開發者手動跑 DS 完成 story 實作）
/bmad-story-automator-review
→ SAR：自動 review、若小 issue 自動 fix、同步 sprint-status
```

適合開發者對 `CR`（bmad-code-review）的互動式流程覺得慢時使用。

---

### 與 bmm、tea 的協作

baut 是**編排層**，本身不做 review 或實作，全部透過呼叫下游 skill 完成：

```
baut SA
├─ 呼叫 bmm 的 CS / DS / ER
├─ （選用）呼叫 tea 的 AT / TA
└─ 呼叫自家的 SAR（內含 bmm CR + auto-fix）
```

因此 baut 效能上限取決於 bmm / tea 的 skill 品質。

---

### 失敗處理

Agent 在 automation 中若卡住（例：DS 反覆修不好），會：

1. 停在當前 story，回報卡住原因
2. `sprint-status.yaml` 標記該 story 為 `blocked`
3. 使用者接手，可手動跑 `bmad-correct-course`（`CC`）或直接編輯 story

---

## References

- [[architecture]] — Orchestrator 型 workflow 的結構
- [[multi-agent]] — SA 內部的序列接力機制
- [[operations]] — Sprint 生命週期完整入口
- [[modules/bmm|BMad Method]] — SA 呼叫的下游 skill
- [[modules/tea|TEA 模組]] — 選用整合的 ATDD 流程
- [baut `module-help.csv`](../../_bmad/baut/module-help.csv) — 完整 skill 清單（SSOT）
