# Contributing to SpecGate

感謝你願意參與 SpecGate。本文定義最小可執行的提案與合併規則，確保規範可演進且不破壞兼容性。

## 1) 貢獻範圍

歡迎以下類型的貢獻：

- 規範文字修正（錯字、歧義、術語一致性）
- 接口契約補充（不破壞既有兼容）
- 測試與驗證方法補充（SLO/一致性）
- 雙語內容對齊（zh-TW / en）

不建議：

- 直接提交品牌綁定、私有命名、憑證欄位或不可公開資訊

## 2) 分支與提交

- 建議使用功能分支：`feat/*`、`fix/*`、`docs/*`
- 提交訊息建議：
  - `docs: clarify L0 search contract`
  - `spec: add deprecation migration requirement`

## 3) 提案流程（PR）

每個 PR 請至少包含：

1. **變更動機**：要解決什麼問題。
2. **影響範圍**：涉及哪個章節/層級（L0-L6）。
3. **兼容性評估**：是否影響既有實作。
4. **驗證方式**：如何證明修改可行（示例、測試方法或對照）。

## 4) 兼容性守則（必看）

- L0 核心契約：同一 MAJOR 版本內不得破壞兼容。
- L1-L6 參考接口：同一 MINOR 版本內應盡量保持源碼兼容。
- 若需破壞性調整：
  1. 必須先標記棄用（deprecation）
  2. 必須提供替代方案
  3. 必須提供遷移說明

## 5) 文檔規範

- 規範詞彙請使用：`MUST` / `SHOULD` / `MAY`
- 新增術語請同步更新兩份文件：
  - `docs/specs/ai-component-interface-standard-example.zh-TW.md`
  - `docs/specs/ai-component-interface-standard-example.en.md`
- 避免使用私有或品牌化命名，優先中立術語。

## 6) PR 檢查清單

提交前請確認：

- [ ] 變更有清楚動機與範圍
- [ ] 沒有引入私有資訊或品牌綁定
- [ ] 兼容性說明完整
- [ ] 中英文件已同步（如適用）
- [ ] 範例或驗證方式可重現

---

若你不確定變更是否屬於破壞性調整，請先開 Discussion 或 Draft PR。
