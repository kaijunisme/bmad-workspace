---
title: "Creative Intelligence Suite（cis）模組"
date: 2026-07-02
tags: [bmad, module, cis, creativity, strategy]
aliases: [bmad-cis-module]
---

cis 提供**創意策略與人本設計**類的顧問型 agent 與 workflow：破壞性創新、系統性問題解決、設計思考、腦力激盪、敘事框架、簡報視覺化。與 bmm 專注於軟體交付不同，cis 面向的是「決策階段的思考品質」。

---

## Overview

cis 為 external 模組，npm 套件為 [`bmad-creative-intelligence-suite`](https://github.com/bmad-code-org/bmad-module-creative-intelligence-suite)。當前 workspace 版本為 v0.2.1。提供 5 個 workflow skill 與 6 位具名 agent，任一 skill 都可用 anytime 觸發。

---

## Core Concepts

### Skills 清單

| Code | Skill | 用途 |
|---|---|---|
| `IS` | `bmad-cis-innovation-strategy` | 辨識破壞式機會、設計商業模式創新 |
| `PS` | `bmad-cis-problem-solving` | 系統性問題解決方法論攻關複雜挑戰 |
| `DT` | `bmad-cis-design-thinking` | 同理心驅動的人本設計流程 |
| `BS` | `bmad-brainstorming` | 使用多種技法引導腦力激盪（cis 版） |
| `ST` | `bmad-cis-storytelling` | 使用經典故事框架打造敘事 |

> [!NOTE]
> cis 的 `bmad-brainstorming`（`BS`）與 core 的 `BSP`、bmm 的 `BP` 名稱相同但實作不同：cis 版聚焦創意技法組合，bmm 版聚焦專案脈絡導向。

---

### 6 位 Agent

| Agent | 代號 | 角色 |
|---|---|---|
| **Carson** | `bmad-cis-agent-brainstorming-coach` | Elite Brainstorming Specialist |
| **Dr. Quinn** | `bmad-cis-agent-creative-problem-solver` | Master Problem Solver |
| **Maya** | `bmad-cis-agent-design-thinking-coach` | Design Thinking Maestro |
| **Victor** | `bmad-cis-agent-innovation-strategist` | Disruptive Innovation Oracle |
| **Caravaggio** | `bmad-cis-agent-presentation-master` | Visual Communication Expert |
| **Sophia** | `bmad-cis-agent-storyteller` | Master Storyteller |

呼叫方式：「跟 <名字> 說話」或直接呼叫對應 skill / agent。

---

### 五類創意任務對應

| 任務類型 | 建議 Agent | 建議 Skill |
|---|---|---|
| 需要突破現有市場框架、找 disruption | Victor | `IS` |
| 卡在複雜問題不知從何下手 | Dr. Quinn | `PS` |
| 產品初期需要理解使用者 | Maya | `DT` |
| 需要大量發散想法 | Carson | `BS` |
| 產品 / 提案需要打動人心的敘事 | Sophia | `ST` |
| 準備 pitch deck 或視覺化簡報 | Caravaggio | — |

---

### 使用範例

#### 破壞式創新分析

```
/bmad-cis-innovation-strategy
「我們的 SaaS 訂閱模式面臨 usage-based 定價競爭者的壓力，
 想探索三個潛在的破壞式應對策略」
→ Victor 引導完成 Jobs-to-be-Done、Disruption Matrix、Business Model Canvas
→ 產出 innovation strategy 文件到 output_folder
```

#### 使用者訪談前準備

```
/bmad-cis-design-thinking
「準備下週 5 位目標使用者訪談，主題是 legacy CRM 遷移體驗」
→ Maya 引導完成 Empathy Map、訪談腳本、觀察維度
```

#### 為 PRD 補上故事

```
（先跑 bmm 的 bmad-prd 完成需求文件）
/bmad-cis-storytelling
「以剛完成的 PRD 為輸入，產出面向 C-level 的 3 分鐘 pitch 敘事」
→ Sophia 套用 Hero's Journey / SCQA / StoryBrand 三選一
```

---

### 與 bmm 的協作模式

cis 常被安排在 bmm 主流程的 **前** 或 **側**：

```
（前）
bmad-cis-innovation-strategy (IS) → bmad-product-brief (CB) → bmad-prd (PRD) → ...

（側）
bmad-prd (PRD) 完成後
  ├→ bmm 內部：bmad-ux (CU) → bmad-architecture (CA)
  └→ cis 補強：bmad-cis-storytelling (ST) 產出對 stakeholder 的敘事
```

---

## References

- [[architecture]] — Agent 與 Workflow 層次
- [[multi-agent]] — Party Mode 可拉入 Victor、Maya 等做跨領域討論
- [[operations]] — Anytime skill 呼叫方式
- [[modules/bmm|BMad Method]] — cis 常與 bmm 主流程接軌
- [cis `module-help.csv`](../../_bmad/cis/module-help.csv) — 完整 skill 清單（SSOT）
- [cis GitHub](https://github.com/bmad-code-org/bmad-module-creative-intelligence-suite)
- [cis 官方文件](https://cis-docs.bmad-method.org/)
