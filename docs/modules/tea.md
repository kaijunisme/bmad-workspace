---
title: "Test Architecture Enterprise（tea）模組"
date: 2026-07-02
tags: [bmad, module, tea, testing, atdd, ci, nfr]
aliases: [bmad-tea-module]
---

tea 是 BMad 的**企業級測試架構**模組，涵蓋從測試策略、框架初始化、CI/CD 品質流水線到 ATDD、非功能需求評估與涵蓋追蹤的完整測試生命週期。同時內建 TEA Academy 的 7 堂課程，可作為測試基礎教學工具。

---

## Overview

tea 為 external 模組，npm 套件為 [`bmad-method-test-architecture-enterprise`](https://github.com/bmad-code-org/bmad-method-test-architecture-enterprise)。當前 workspace 版本為 v1.19.0。提供 1 位具名 agent（Murat）與 9 個 workflow skill，分佈於 learning、solutioning、implementation 三階段。

---

## Core Concepts

### Agent

| Agent | 代號 | 角色 |
|---|---|---|
| **Murat** | `bmad-tea` | Master Test Architect and Quality Advisor |

Murat 是 tea 的統一入口。呼叫方式：「跟 Murat 說話」或直接呼叫 `bmad-tea`。

---

### 三階段流程

```
0-learning      → 測試基礎教學（TEA Academy）
3-solutioning   → Test Design、Test Framework 初始化、CI Setup
4-implementation → ATDD、Test Automation、Review、NFR、Traceability
```

---

### 0-learning：TEA Academy

| Code | Skill | 說明 |
|---|---|---|
| `TMT` | `bmad-teach-me-testing` | 7 堂 session 教學：測試基礎、pyramid、mutation、ATDD 等 |

輸出含 progress file、session notes、certificate。適合團隊新人 onboarding。

---

### 3-solutioning：測試策略與框架

| Code | Skill | 說明 |
|---|---|---|
| `TD` | `bmad-testarch-test-design` | Risk-based 測試規劃：identify risks、決定測試層級、產出 test design document |
| `TF` | `bmad-testarch-framework` | 初始化 production-ready 測試框架（多語言 / 多引擎支援） |
| `CI` | `bmad-testarch-ci` | 設定 CI/CD 品質流水線：gate、report、trend 追蹤 |

推薦順序：`TD → TF → CI`。此三步應在 sprint 開始前完成。

---

### 4-implementation：ATDD 與涵蓋追蹤

| Code | Skill | 說明 |
|---|---|---|
| `AT` | `bmad-testarch-atdd` | Acceptance Test Driven Development：實作前產生紅燈驗收測試骨架 + checklist |
| `TA` | `bmad-testarch-automate` | 擴大自動化測試涵蓋（story 實作完成後） |
| `RV` | `bmad-testarch-test-review` | 品質稽核（0-100 分制的評分報告） |
| `NR` | `bmad-testarch-nfr` | NFR Evidence Audit：非功能需求（效能、安全、可靠性）證據稽核 |
| `TR` | `bmad-testarch-trace` | 涵蓋 traceability matrix + gate decision |

**典型 story 週期**：

```
（story 建立後）
bmad-create-story:create (bmm CS)
    ↓
bmad-testarch-atdd (AT)          ← tea 介入：先寫紅燈驗收測試
    ↓
bmad-dev-story (bmm DS)          ← 實作，讓紅燈變綠
    ↓
bmad-testarch-automate (TA)      ← 擴大自動化涵蓋
    ↓
bmad-testarch-test-review (RV)   ← 稽核測試品質
    ↓
bmad-testarch-nfr (NR)           ← NFR 稽核
    ↓
bmad-testarch-trace (TR)         ← 產出 traceability + gate 決策
```

---

### 與 bmm 的整合點

tea 的 skill 都是為了嵌入 bmm 的四階段而設計，`preceded-by` 欄位明確標示：

| tea skill | 接在誰之後 |
|---|---|
| `AT` (ATDD) | `bmad-create-story:create`（bmm `CS`） |
| `NR` (NFR) | `bmad-testarch-automate`（tea `TA`） |
| `TR` (Trace) | `bmad-testarch-test-review`（tea `RV`） |

---

### 與其他測試 skill 的區別

BMad 內有多處 skill 名稱含「test」，容易混淆：

| Skill | 模組 | 用途 |
|---|---|---|
| `bmad-qa-generate-e2e-tests`（`QA`） | bmm | 一次性為既有程式產生 API / E2E 測試 |
| `bmad-testarch-*` | tea | 企業級測試架構，強調流程與涵蓋 |
| `gds-test-*` | gds | 遊戲專屬測試（含 playtest、performance） |

若專案採用 tea，通常會用 tea 的一整套取代 bmm 的 `QA`。

---

### NFR 稽核範圍

`NR`（NFR Evidence Audit）稽核的四類非功能需求：

- **Performance**：latency、throughput、resource usage
- **Reliability**：availability、error rate、recovery
- **Security**：authn / authz、data protection、vulnerability
- **Scalability**：load、concurrency、data volume

產出 NFR report 含各項證據等級（evidence-graded），並標示缺口。

---

## References

- [[architecture]] — Compaction Survival 與 status frontmatter
- [[multi-agent]] — Murat 加入 Party Mode 討論
- [[operations]] — Skill 呼叫、preceded-by 順序
- [[modules/bmm|BMad Method]] — tea 整合點（`CS` → `AT`、`DS` → `TA`）
- [tea `module-help.csv`](../../_bmad/tea/module-help.csv) — 完整 skill 清單（SSOT）
- [tea GitHub](https://github.com/bmad-code-org/bmad-method-test-architecture-enterprise)
- [tea 官方文件](https://bmad-code-org.github.io/bmad-method-test-architecture-enterprise/)
