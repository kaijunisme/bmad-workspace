---
title: "Core 模組"
date: 2026-07-02
tags: [bmad, module, core]
aliases: [bmad-core-module]
---

Core 是 BMad 的通用工具模組，提供不依賴特定領域的 primitives：多 agent 協作、文件審查、腦力激盪、客製化 override，以及 skill 註冊查詢。所有其他模組（bmm / bmb / cis / gds / tea / baut）都可假設 core skill 存在。

---

## Overview

Core 由 built-in 模組直接安裝，無獨立 npm 套件。當前 workspace 版本為 v6.9.0，共提供 12 個 skill，其中 `bmad-party-mode` 是多 agent 協作的統一入口。Core 沒有具名 agent 人格。

---

## Core Concepts

### Skills 清單

| Code | Skill | 用途 |
|---|---|---|
| `BH` | `bmad-help` | 列出所有已安裝 skill、對應 menu-code、推薦順序 |
| `ID` | `bmad-index-docs` | 讓 LLM 掌握可用文件清單，不必全載入 |
| `SD` | `bmad-shard-doc` | 大文件（>500 行）分片管理 |
| `BSP` | `bmad-brainstorming` | 使用多種技法進行腦力激盪 |
| `PM` | `bmad-party-mode` | 多 agent 討論、custom party 建立 |
| `EP` | `bmad-editorial-review-prose` | 逐段文字審稿（三欄式修訂表） |
| `ES` | `bmad-editorial-review-structure` | 結構性審稿（章節、段落組織） |
| `AR` | `bmad-review-adversarial-general` | 對抗性審查：找漏洞、找例外 |
| `ECH` | `bmad-review-edge-case-hunter` | 邊界案例獵人（與 AR 互補、方法驅動而非態度驅動） |
| `SP` | `bmad-spec` | 將任何意圖輸入蒸餾為 `SPEC.md` 契約 + 附件 |
| `FI` | `bmad-forge-idea` | 壓力測試點子：persona 反覆質問，最終選擇繼續 / 硬化 / 放棄 |
| `BC` | `bmad-customize` | 掃描可客製化欄位、產生 `_bmad/custom/` override |
| — | `bmad-advanced-elicitation` | 深度質問 primitive（Socratic、first principles、pre-mortem、red team） |

> [!NOTE]
> `bmad-advanced-elicitation` 為輔助 primitive，多被其他 skill 內部呼叫，不透過 menu-code 直接觸發。

---

### 三大主要用途

#### 1. 多 Agent 協作入口

`bmad-party-mode`（`PM`）是所有多 agent 討論的統一入口。它會讀取 `_bmad/_config/agent-manifest.csv` 並自動挑選 2–4 位合適的 agent。詳細機制見 [[multi-agent]]。

```
使用：/bmad-party-mode
「我想討論這個 API 設計是否符合 RESTful 慣例，並考量效能影響」
→ Orchestrator 自動 spawn Winston（架構師）+ Amelia（開發）+ Murat（測試）
```

#### 2. 內容品質保證

三個 review skill 各有分工：

| Skill | 定位 | 何時用 |
|---|---|---|
| `EP` (Editorial - Prose) | 逐字逐句 | 交付前的文案精修 |
| `ES` (Editorial - Structure) | 章節組織 | 多次補寫的長文件 |
| `AR` (Adversarial Review) | 找漏洞 | 交付前的品質守門 |
| `ECH` (Edge Case Hunter) | 邊界案例 | 需與 AR 並用取得正交覆蓋 |

其他模組的 `bmad-code-review`（`CR`）在內部會自動呼叫 `AR`。

#### 3. 契約與點子驗證

- `SP` (Spec)：任何 intent（brief、PRD、transcript、腦力激盪筆記）→ `SPEC.md` 契約 + companion 檔案。鎖定 WHAT 之後再談 HOW，適用範圍不限軟體
- `FI` (Forge Idea)：點子還沒成形時用。persona 反覆質問直到點子硬化、可 handoff 給 `bmad-spec` 或 `bmad-quick-dev`

---

### 客製化 override 流程

`BC`（`bmad-customize`）是 core 提供的 skill 客製化工具。詳細操作見 [[operations]]，此處僅示範最短路徑：

```
1. /bmad-customize
2. 選擇要客製化的目標（例：bmad-agent-pm）
3. 選擇 scope（team / user）
4. 描述想改變的行為（例：「PRD 用條列式而非敘事風格」）
5. Skill 自動產生 TOML → 寫入 _bmad/custom/config.toml
6. 驗證 merge 結果
```

安全區檔案：

| 檔案 | 對象 | 版控 |
|---|---|---|
| `_bmad/custom/config.toml` | 團隊共用 | committed |
| `_bmad/custom/config.user.toml` | 個人 | gitignored |

---

### 與其他模組的關係

Core skills 是全模組共用的 primitives：

- **`bmad-party-mode`** 使用所有已安裝模組的 agent 花名冊
- **`bmad-help`** 彙整所有模組的 `module-help.csv`
- **`bmad-review-adversarial-general`** 被 `bmm`、`gds` 的 `bmad-code-review` 內部呼叫
- **`bmad-customize`** 可 override 任何模組的 agent / workflow

---

## References

- [[architecture]] — 框架四層與 Compaction Survival
- [[multi-agent]] — Party Mode 完整機制
- [[operations]] — 客製化 override 詳細流程與參數
- [Core `module-help.csv`](../../_bmad/core/module-help.csv) — 完整 skill 清單（SSOT）
- [BMad 官方文件](https://docs.bmad-method.org/)
