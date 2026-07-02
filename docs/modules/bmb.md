---
title: "BMad Builder（bmb）模組"
date: 2026-07-02
tags: [bmad, module, bmb, builder]
aliases: [bmad-bmb-module]
---

bmb 是「建造工具的工具」— 用來建立、分析、轉換自訂的 agent、workflow、module。當你需要把常用流程封裝為可重用 skill、把外部 skill 轉為 BMad 規範、或打包整組 skill 成可安裝模組時，都由 bmb 負責。

---

## Overview

bmb 為 external 模組，npm 套件為 [`bmad-builder`](https://github.com/bmad-code-org/bmad-builder)。當前 workspace 版本為 v2.1.0。核心產出物是三種：agent skill、workflow / utility skill、可安裝的 module。所有 skill 皆支援 `-H`（headless）參數。

---

## Core Concepts

### Skills 清單

| Code | Skill (action) | 用途 |
|---|---|---|
| `SB` | `bmad-bmb-setup` (configure) | 安裝 / 更新 bmb 模組設定與 help entries |
| `BA` | `bmad-agent-builder` (build-process) | 建立、編輯、重建 agent skill |
| `AA` | `bmad-agent-builder` (quality-analysis) | 對現有 agent 進行品質分析 |
| `BW` | `bmad-workflow-builder` (build-process) | 建立、編輯、重建 workflow / utility skill |
| `AW` | `bmad-workflow-builder` (quality-analysis) | 對現有 workflow 進行品質分析 |
| `CW` | `bmad-workflow-builder` (convert-process) | 將任意 skill 轉為符合 BMad 規範的等效品，含 before / after HTML 對照 |
| `IM` | `bmad-module-builder` (ideate-module) | 從模糊想法出發，七階段對話引導產出 module 規劃 |
| `CM` | `bmad-module-builder` (create-module) | 將 `skills/` 資料夾打包為可安裝的 BMad module |
| `VM` | `bmad-module-builder` (validate-module) | 驗證 module 結構完整性 |

---

### Agent 三型（Builder 支援）

`bmad-agent-builder` 建立的 agent 可選三型，差別在於狀態與自主性：

#### Stateless Agent

單一 `SKILL.md`，無跨會話記憶。適合單次任務型專家。

#### Memory Agent

在 Stateless 基礎上加 **Sanctum** 記憶結構 + **First Breath** 首次啟動引導：

```
my-agent/
├── SKILL.md
└── sanctum/
    ├── first-breath.md        # 首次啟動的問候與偏好搜集
    ├── preferences.md         # 累積的使用者偏好
    └── domain-knowledge.md    # 專案 / 領域知識
```

適合長期協作的顧問型 agent。

#### Autonomous Agent

在 Memory Agent 基礎上加 **PULSE** 機制，可在會話之間主動執行。需搭配外部排程（Claude Code Cron 等），BMad 本身不提供排程。

---

### Workflow 三型（Builder 支援）

`bmad-workflow-builder` 建立的 skill 可選三型：

| 類型 | 結構 | 適合情境 |
|---|---|---|
| **Simple Utility** | 單一 `SKILL.md` | 無狀態單次工具 |
| **Simple Workflow** | `SKILL.md` + inline 步驟 | 步驟少、流程固定的引導式任務 |
| **Complex Workflow** | `SKILL.md` + `references/` step 檔案 | 多階段、需 Compaction Survival |

Complex Workflow 每個 step 為獨立 markdown 檔，`SKILL.md` 依序 include。這是 bmm `bmad-prd`、`bmad-architecture` 採用的結構。

---

### 執行模式（Guided / Yolo / Headless）

Builder skill 都可帶模式參數：

| 模式 | 觸發 | 用途 |
|---|---|---|
| Guided | 直接呼叫 | 對話式引導，逐步確認 |
| Yolo | 對話說「just draft it」 | 一次性草稿產出，事後細修 |
| Headless | 加 `-H` / `--headless` | 全預設，適合自動化 |

---

### Module Builder 三條路徑

`bmad-module-builder` 拆為 IM → CM → VM 三段，可獨立呼叫也可串聯：

| 路徑 | 用途 | 輸入 | 輸出 |
|---|---|---|---|
| **IM** (Ideate Module) | 從模糊想法規劃 module 架構 | 初始描述 | Module 計畫（含 agent 數量、workflow 類型） |
| **CM** (Create Module) | Skills 資料夾 → 可安裝 module | `skills/` 資料夾或單一 `SKILL.md` | Module 基礎架構（含 `module-help.csv`、config） |
| **VM** (Validate Module) | 檢查結構與註冊完整性 | Module 或 skill 路徑 | 驗證報告 |

---

### 完整建立路徑

從想法到可安裝 module 的完整流程：

```
Step 1：IM（Ideate Module）
   ↓ 確立架構、agent 數量、workflow 類型
Step 2：BA × N（Agent Builder）
   ↓ 逐一建立各 agent，選定 Stateless / Memory / Autonomous
Step 3：BW × N（Workflow Builder）
   ↓ 逐一建立各 workflow，選定類型與執行模式
Step 4：更新 agent-manifest.csv
   ↓ 讓 Party Mode 能挑選新 agent
Step 5：CM（Create Module）
   ↓ 打包為 module 結構
Step 6：VM（Validate Module）
   ↓ 驗證後可發佈或跨專案使用
```

---

### Skill 轉換（CW）

`CW`（Convert a Skill）將任意來源的 skill（含外部 URL）轉為 outcome-driven 的 BMad 等效品：

- **輸入**：`--convert <path-or-URL>`
- **輸出**：轉換後 skill + before / after HTML 對照報告
- **用途**：把非 BMad 生態的 skill 或 prompt 引入 workspace

---

### 快速上手

若只是想試建一個 agent 或 workflow，最短路徑：

```
1. /bmad-agent-builder（Guided）
2. 描述 agent 的角色、原則、專長
3. Builder 生成 SKILL.md 到 _bmad-output/agents/
4. /bmad-agent-builder --action quality-analysis --path <生成路徑>
   （對剛做出來的 agent 做品質分析）
```

想直接進 module 規劃：

```
/bmad-module-builder --action ideate-module "我想做一個處理法遵報告的模組"
```

---

## References

- [[architecture]] — Agent 三型、Workflow 三型的抽象定義
- [[multi-agent]] — 自訂 agent 加入 Party Mode
- [[operations]] — Headless / Yolo / Guided 參數
- [bmb `module-help.csv`](../../_bmad/bmb/module-help.csv) — 完整 skill 清單（SSOT）
- [bmad-builder GitHub](https://github.com/bmad-code-org/bmad-builder)
- [BMad Builder 官方文件](https://bmad-builder-docs.bmad-method.org/)
