---
title: "BMad 操作手冊"
date: 2026-07-02
tags: [bmad, operations, cli, customize]
aliases: [bmad-operations]
---

本文彙整 workspace 使用 BMad 的日常操作：如何在 IDE 呼叫 skill、如何客製化 agent 行為、如何查詢當前安裝版本與變更設定。安裝與更新指令見 workspace 根目錄 [`README.md`](../README.md)。

---

## Overview

Skill 呼叫是 BMad 唯一的執行入口。使用者在 IDE 對話中以 skill 名稱或 menu-code 觸發，可選擇 Guided / Yolo / Headless 三種模式。設定客製化透過 `_bmad/custom/` 底下的 override 檔完成，安裝器不會覆蓋此區。

---

## Core Concepts

### 呼叫 Skill

以 Claude Code 為例（其他 IDE 對應目錄見 [`README.md`](../README.md) 的 IDE 整合表）：

```
使用者：/bmad-help
→ 列出所有可用 skill 與 menu-code

使用者：/bmad-prd
→ 啟動 Create PRD workflow（Guided 模式）

使用者：/bmad-prd -H
→ Headless 模式：跳過所有互動 prompt，用預設值一次跑完
```

也可以直接以 menu-code 呼叫（依 IDE 支援程度而定），例：對話中提及 `PRD` / `CS` / `DS` 讓 orchestrator 挑對應 skill。

---

### Menu-code 快捷表（常用）

完整清單見各模組 [`_bmad/<module>/module-help.csv`](../_bmad/)，以下列出使用率最高的幾個：

| Code | Skill | 用途 |
|---|---|---|
| `BH` | `bmad-help` | 列出所有可用 skill |
| `PM` | `bmad-party-mode` | 多 agent 討論 |
| `SP` | `bmad-spec` | 產生 SPEC.md 契約 |
| `FI` | `bmad-forge-idea` | 壓力測試點子 |
| `CB` | `bmad-product-brief` | 快速產品簡報 |
| `PRD` | `bmad-prd` | Create / Edit / Review PRD |
| `CA` | `bmad-architecture` | 系統架構 |
| `CS` | `bmad-create-story` | 建立可實作 story |
| `DS` | `bmad-dev-story` | 執行 story 實作 |
| `CR` | `bmad-code-review` | Code review |
| `QQ` | `bmad-quick-dev` | 意圖輸入 → 實作全流程 |
| `GPC` | `bmad-generate-project-context` | 掃描既有專案，產出 `project-context.md` |
| `DP` | `bmad-document-project` | 分析既有專案產生文件 |
| `BC` | `bmad-customize` | 修改 agent / workflow 行為 |
| `SA` | `bmad-story-automator` | Story 全自動化流程 |

---

### 執行模式參數

| Flag | 效果 | 適用 skill |
|---|---|---|
| （無） | Guided 模式：對話逐步確認 | 全部 |
| `-H` / `--headless` | 無互動、用預設值 | bmb / bmm 大多數 workflow |
| `--yolo` / 「just draft it」 | 產生草稿後細修 | Complex workflow（例：PRD、architecture） |
| `--solo` | Party Mode 降級：orchestrator 自行模擬所有 agent | `bmad-party-mode` |

---

### 客製化 Agent 與 Workflow

BMad 保留兩個安全區給使用者，安裝器**永遠不會覆蓋**：

| 檔案 | 用途 | 版控狀態 |
|---|---|---|
| `_bmad/custom/config.toml` | 團隊 / 專案共用 override | committed |
| `_bmad/custom/config.user.toml` | 個人 override | gitignored |

Override 檔採 TOML deep-merge 邏輯：定義的 table / key 會覆蓋 `_bmad/config.toml`，未定義的欄位保留原值。

#### 範例：改變 PM Agent 的產出風格

```toml
# _bmad/custom/config.toml
[agents.bmad-agent-pm]
description = "Prefers short, bulleted PRDs over narrative drafts."
```

#### 使用 `bmad-customize` skill

無需手寫 TOML — 呼叫 `BC`（`bmad-customize`）會掃描所有可客製化欄位，引導選擇正確的 scope（agent vs workflow）、產生 override 並驗證 merge 結果。

---

### 版本查詢

當前安裝的版本紀錄於 `_bmad/_config/manifest.yaml`（SSOT）。快速查詢：

```bash
# 讀取根版本
grep -E "^  version:" 'D:/_agent-workspace/workspace/bmad-workspace/_bmad/_config/manifest.yaml' | head -1

# 讀取所有模組版本
grep -B1 "version:" 'D:/_agent-workspace/workspace/bmad-workspace/_bmad/_config/manifest.yaml'
```

當前 workspace（main branch）安裝的版本：

| 模組 | 版本 |
|---|---|
| BMad 根版本 | `6.9.0` |
| core / bmm | `6.9.0` |
| bmb | `v2.1.0` |
| cis | `v0.2.1` |
| gds | `v0.6.0` |
| tea | `v1.19.0` |
| baut | `未固定版本，SHA 追蹤` |

> [!NOTE]
> 上表為安裝當下快照。實際以 `manifest.yaml` 為準。更新方式見 [`README.md`](../README.md) 的「更新」章節。

---

### Workflow 入口總覽

各模組的完整 skill 清單見對應 module 文件：

| 模組 | 文件 | 建議入口 |
|---|---|---|
| core | [[modules/core]] | `BH`（help）、`PM`（party mode） |
| bmm | [[modules/bmm]] | `DP`（brownfield）、`CB`（新專案）、`QQ`（快速實作） |
| bmb | [[modules/bmb]] | `SB`（初始化）、`IM → CM → VM`（新模組） |
| cis | [[modules/cis]] | `IS`（策略）、`BS`（腦力激盪） |
| gds | [[modules/gds]] | `BG`（遊戲點子）、`GDD`（設計文件） |
| tea | [[modules/tea]] | `TMT`（教學）、`TD → TF → CI`（新專案） |
| baut | [[modules/baut]] | `SA`（自動跑完 story cycle） |

---

### 輸出檔位置

Skill 產出寫入 `_bmad-output/`，依 skill 的 `output-location` 欄位分類：

```
_bmad-output/
├── planning-artifacts/       # brief / PRD / UX / architecture / epics-stories
├── implementation-artifacts/ # sprint-status / story / review / retrospective
├── test-artifacts/           # tea 模組產出
├── project-knowledge/        # bmad-document-project、bmad-generate-project-context
└── ...
```

---

## References

- [`README.md`](../README.md) — 安裝、更新、IDE 整合、branch 差異
- [[architecture]] — 四層架構與執行模式
- [[multi-agent]] — Party Mode 詳細機制
- [[modules/core]] — `bmad-customize` 完整說明
