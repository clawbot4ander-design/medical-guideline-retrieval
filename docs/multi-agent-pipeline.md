# Conditional Multi-agent Pipeline

## 結論

醫學指南檢索適合使用條件式 multi-agent pipeline，而不是讓多個 agent 自由對話。

設計原則：

```text
窄任務、可觀測、可關閉、不要每次都跑全部 agent。
```

## 推薦 agents

### 1. Clinical Search Brain

搜尋前的大腦。

負責：

- 將使用者白話轉成臨床概念
- 決定 target chapters
- 產生 evidence targets
- 指定 avoid routes
- 補 required facets
- 產生 search queries

例：

```text
腳血管塞住
→ PAD / lower-extremity arterial disease / ASCVD
→ ADA S10 + ADA S12
→ antiplatelet, statin, BP, smoking cessation, revascularization, limb outcomes
```

### 2. Evidence Coverage Agent

搜尋後的覆蓋度檢查。

負責比較：

```text
required facets
vs
covered facets from selected guideline hits
```

這個 agent 是防止「看起來有片段，但其實沒有覆蓋核心問題」。

### 3. Retrieval Failure Analyzer Agent

debug/失敗分析時使用。

負責判斷：

- no candidates
- no selected hits
- selected hits missing required facets
- reranker dropped covered evidence
- LLM reranker / evidence review too conservative
- whole-section context missing

### 4. Regression Test Agent

部署前或修改 retrieval 後手動跑固定題組，避免修 A 壞 B。

建議題組：

- PAD / 下肢動脈阻塞藥物治療
- 視網膜病變分期與治療
- CGM 適用對象
- T2D + CKD + eGFR 25 用藥
- MASLD / MASH 合併糖尿病治療

## 為什麼先不加更多 agents

目前不建議一開始就加很多 agent：

- 每個 agent 都會增加 latency 或 debug 複雜度。
- 醫療問答最重要的是檢索證據，不是 agent 自由推理。
- Coverage / Failure / Regression 已經形成品質閉環。

下一階段可考慮：

- Citation Audit Agent：確認最終答案每個重點都有來源片段。
- Ontology Builder Agent：離線掃描新 guideline，自動更新 concept profile。
- Deployment Monitor Agent：部署後自動跑 health + regression。

