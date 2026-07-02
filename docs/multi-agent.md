---
title: "多 Agent 協作機制"
date: 2026-07-02
tags: [bmad, party-mode, orchestrator, multi-agent]
aliases: [bmad-multi-agent]
---

BMad 內建兩種多 Agent 協作模式：**並行討論（Party Mode）** 與 **序列接力**。兩者底層都依賴 IDE 的 subagent 機制（Claude Code 為 `Agent` 工具）spawn 出獨立 context 的子 agent。本文說明兩種模式的原理、Token 控制策略，以及使用限制。

---

## Overview

Party Mode 讓多個 agent 真實獨立思考。Orchestrator（主對話）將問題精煉為約 400 字 context 後平行 spawn 多個 subagent，避免對話累積導致的 Token 爆增。序列接力則有四種實作路徑，適合流程有明確順序時使用。

---

## Core Concepts

### Party Mode（並行模式）

由 core 模組的 `bmad-party-mode` 觸發（menu-code `PM`）。呼叫時 Orchestrator 讀取 `_bmad/_config/agent-manifest.csv` 中的 agent 花名冊，依問題性質自動挑選 2–4 位最相關的 agent。

```
使用者問題
    ↓
Orchestrator（主對話 Claude）
    ↓ 平行 spawn
┌──────────┐  ┌──────────┐  ┌──────────┐
│  Mary    │  │ Winston  │  │  Amelia  │
│ 獨立思考 │  │ 獨立思考 │  │ 獨立思考 │
└──────────┘  └──────────┘  └──────────┘
    ↓
 Orchestrator 依序呈現各 Agent 完整回應
```

**Solo 模式（降級機制）**：當 subagent 不可用（例：IDE 不支援、額度不足）時，加入 `--solo` 參數可讓 Orchestrator 在同一個回應內依序模擬所有 agent。

**自訂 Party**：`bmad-party-mode` 支援建立 custom party 與 persona，可用於 AI focus-group 或跨模組研討。

---

### 四種序列接力方式

#### 方式一：Workflow Step 檔案

同一 skill 內部依序載入 `references/` 底下的 step 檔案，每步驟以使用者確認（輸入 `C`）進入下一步。上下文連續，適合短流程（3–5 步）。

- **代表 skill**：`bmad-prd`、`bmad-architecture`、`bmad-product-brief`
- **Token 特性**：累積式（context 逐步變長）

#### 方式二：Skill 鏈（人工觸發）

`module-help.csv` 的 `preceded-by` / `followed-by` 欄位定義推薦順序，使用者手動觸發下一個 skill。每次 skill 啟動都是新 context，Token 消耗最低。

- **代表流程**：`bmad-product-brief → bmad-prd → bmad-ux → bmad-architecture`
- **Token 特性**：每 skill 獨立、最省 token

#### 方式三：Orchestrator Skill（自動序列）

自訂 orchestrator skill，依序 spawn 子 agent，將前一個 agent 的產出作為下一個的輸入。支援條件分支，全自動執行。

- **代表 skill**：`bmad-quick-dev`（QQ）、`bmad-story-automator`（SA）
- **Token 特性**：Orchestrator context 控制在摘要層級，可跑長流程

#### 方式四：混合模式

Orchestrator 在特定步驟切換為並行（Party Mode），其他步驟序列執行。例：`bmad-code-review` 可 spawn 多個 review agent 平行審查後合併結論。

---

### 各方式比較

| 面向 | Workflow Steps | Skill 鏈 | Orchestrator | Party Mode |
|---|:-:|:-:|:-:|:-:|
| 執行方式 | 序列 | 序列（人工） | 序列（自動） | 並行 |
| 子 Agent 獨立性 | ❌ | ❌ | ✅ | ✅ |
| Token 消耗 | 高（累積） | 最低 | 中（可控） | 中 |
| 需人工介入 | 每步確認 | 每步手動 | 不需要 | 少 |
| 支援條件分支 | ❌ | ❌ | ✅ | ❌ |

---

### Orchestrator 的 Token 控制原則

Orchestrator 若把完整對話塞給下一個 agent，long-running flow 會迅速吃光 context。BMad 的正確做法是「摘要 + 產出文件參照」：

```
❌ 錯誤：把所有對話記錄塞給下一個 Agent
✅ 正確：
   1. 子 Agent 完成 → 產出寫入檔案（含 status frontmatter）
   2. Orchestrator 只保留精煉摘要
   3. spawn 下一個 Agent 時只傳：任務說明 + 摘要 + 必要背景 + 產出檔案路徑
```

Party Mode 明確規定每個子 agent 的 context 上限為約 **400 字**。

---

### LLM 依從性的限制

BMad 是 **提示詞驅動** 框架，無法硬保證 LLM 一定執行 spawn 或遵守步驟順序：

| 風險 | 現況 |
|---|---|
| 長對話後 LLM 忘記 spawn，改自行回應 | 靠使用者發現並糾正 |
| Orchestrator 未精煉就傳完整 context | 靠 `SKILL.md` 指令品質 |
| Party Mode 只 spawn 一個 agent（節省 token 傾向） | 需明確要求「請 spawn 三位」 |

> [!IMPORTANT]
> BMad 是最佳努力（best-effort）系統，不是確定性系統。若流程有不可逆操作（資料庫寫入、發送通知、金融交易），應搭配人工確認節點，或改用 LangGraph 等具強制執行機制的框架。

---

### 可用 Agent 花名冊

以 v6.9.0 為準，安裝所有模組時可 spawn 的 agent（詳見 [[modules/bmm|bmm]]、[[modules/cis|cis]]、[[modules/gds|gds]]、[[modules/tea|tea]]、[[modules/bmb|bmb]]）：

| 模組 | Agent | 角色 |
|---|---|---|
| bmm | Mary | Business Analyst |
| bmm | Paige | Technical Writer |
| bmm | John | Product Manager |
| bmm | Sally | UX Designer |
| bmm | Winston | Architect |
| bmm | Amelia | Developer |
| tea | Murat | Test Architect |
| bmb | — | Agent Builder |
| cis | Carson | Brainstorming Coach |
| cis | Dr. Quinn | Creative Problem Solver |
| cis | Maya | Design Thinking Coach |
| cis | Victor | Innovation Strategist |
| cis | Caravaggio | Presentation Master |
| cis | Sophia | Storyteller |
| gds | Cloud Dragonborn | Game Architect |
| gds | Samus Shepard | Game Designer |
| gds | Link Freeman | Game Developer |

---

## References

- [[architecture]] — 四層架構、Agent 三型、Workflow 三型
- [[operations]] — Party Mode 觸發、`--solo` / `--headless` 參數
- [[modules/bmb|BMad Builder]] — 建立自訂 Agent 與 Party 成員
- [BMad 官方文件](https://docs.bmad-method.org/)
