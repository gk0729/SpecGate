# AI 通用組件接口標準規範（範例）

版本：v1.0.0  
狀態：Draft for Public Adoption  
授權：MIT（依本倉庫 LICENSE）

---

## 0. 文件定位

本文件為可公開的通用規範範例，目標是降低 AI 生成代碼在「組件對接」時的歧義與缺陷。

- 本文件是「規範 + 實作指南」合併型文檔。
- 本文件不綁定特定品牌、供應商、框架或模型。
- 本文件採用去識別化術語，避免暴露內部實作細節。

### 0.1 規範詞彙

- MUST：必須遵守。
- SHOULD：建議遵守，除非有明確理由。
- MAY：可選。

---

## 1. 設計目標與原則

### 1.1 設計目標

1. 統一組件間接口契約。
2. 支持 AI 代理、SDK、CLI、服務層的可互操作。
3. 在版本演進下保持兼容與可遷移。
4. 可被測試、可觀測、可治理。

### 1.2 設計原則

1. **分層解耦**：上層不直接依賴下層內部狀態。
2. **契約優先**：先定義接口、輸入輸出、錯誤模型。
3. **可替換實作**：演算法與供應商可替換，不影響接口。
4. **最小必要暴露**：僅暴露跨組件必要資訊。
5. **可觀測性內建**：每個關鍵操作可追蹤。

---

## 2. 七層參考架構（L0-L6）

> 說明：保留完整多層模型，但所有層級均採中立命名。

| 層級 | 名稱 | 主要責任 |
|---|---|---|
| L0 | 核心引擎層 | 統一 ID、序列化、壓縮、檢索核心能力 |
| L1 | 語義增強層 | 語言處理、文本轉換、多語支援 |
| L2 | 協作同步層 | 多端同步、衝突解決、安全傳輸 |
| L3 | 本地存儲層 | 持久化、快取、索引管理 |
| L4 | 推理分析層 | 關聯推理、圖分析、路徑搜尋 |
| L5 | 對外接口層 | CLI、SDK、API 對外契約 |
| L6 | 監控治理層 | 日誌、指標、審計、告警 |

### 2.1 依賴規則

1. L(n) MUST 僅依賴 L(n-1) 或平台通用基礎庫。
2. 橫向層間調用 SHOULD 透過明確接口與事件，不得直接改寫內部存儲。
3. L5/L6 不得繞過 L0-L4 契約直接操作資料底層。

---

## 3. 核心接口契約（L0）

### 3.1 統一標識符接口（Unified Identifier）

```python
class UnifiedId:
    def __init__(self, raw_id: str, timestamp_ns: int, source_id: str, seq: int, checksum: str): ...
    @property
    def to_dict(self) -> dict: ...
    def serialize(self) -> bytes: ...
    @staticmethod
    def deserialize(data: bytes) -> "UnifiedId": ...

def generate_id() -> UnifiedId: ...
def parse_id(text: str) -> UnifiedId: ...
def is_valid_id(text: str) -> bool: ...
```

**規範要求**

1. `generate_id` MUST 具備線程安全。
2. 序列化格式 MUST 可往返還原。
3. `is_valid_id` MUST 提供純格式檢查，不能觸發副作用。

### 3.2 數據編解碼接口

```python
def encode(payload: object) -> str: ...
def decode(serialized: str) -> object: ...
```

**規範要求**

1. 編解碼 MUST 100% 可往返（對可序列化型別）。
2. 解碼失敗 MUST 拋出明確錯誤類型，不得吞錯。
3. 格式可為 JSON/YAML/自定語法，但 MUST 在實作文件明確聲明。

### 3.3 檢索接口

```python
from typing import Literal, Optional

def search(
    query: str,
    method: Literal["time", "hash", "vector", "hybrid"] = "hybrid",
    time_range: Optional[tuple[int, int]] = None,
    limit: int = 10,
) -> list[dict]: ...
```

**規範要求**

1. `hybrid` MUST 定義可重現的排序策略。
2. `limit` MUST 有上限保護。
3. 返回結果 MUST 含 `id`, `score`, `method`, `timestamp`。

---

## 4. L1-L6 參考接口要求

### 4.1 L1 語義增強

```python
def transform_text(id_value: UnifiedId, target_lang: str) -> UnifiedId: ...
def supported_languages() -> list[str]: ...
```

- 轉換失敗 SHOULD 回傳原 ID 並附可追蹤錯誤資訊。
- 語言清單 MUST 可由程式獲取，不可僅寫死在文檔。

### 4.2 L2 協作同步

```python
def sync_push(ids: list[UnifiedId] | None = None) -> bool: ...
def sync_pull(since_ns: int | None = None) -> int: ...
def sync_status() -> dict: ...
```

- 傳輸 MUST 使用 TLS。
- 身份驗證 MUST 支持可輪換憑證。
- 衝突解決策略 MUST 具可重放性與審計紀錄。

### 4.3 L3 本地存儲

```python
def initialize_storage(path: str) -> bool: ...
def add_node(id_value: UnifiedId, content: str, metadata: dict | None = None) -> bool: ...
def get_node(id_text: str) -> dict: ...
def delete_node(id_text: str) -> bool: ...
```

- 批量操作 MUST 支持交易（transaction）。
- 索引策略 SHOULD 對常用查詢欄位建立索引。
- 快取資料 MUST 可設定 TTL。

### 4.4 L4 推理分析

```python
def infer_relations(id_value: UnifiedId, depth: int = 2) -> list[dict]: ...
def find_related(id_value: UnifiedId, limit: int = 10) -> list[str]: ...
```

- 推理深度 MUST 有安全上限。
- 推理結果 SHOULD 附可解釋依據（來源節點或規則）。

### 4.5 L5 對外接口（CLI/SDK）

```python
class GatewayClient:
    def learn(self, text: str, metadata: dict | None = None) -> UnifiedId: ...
    def search(self, query: str, method: str = "hybrid", limit: int = 10) -> list[dict]: ...
    def recall(self, id_text: str) -> dict: ...
    def stats(self) -> dict: ...
```

- SDK MUST 保持執行緒安全或清楚聲明非執行緒安全。
- CLI MUST 提供非零退出碼以表示失敗。

### 4.6 L6 監控治理

```python
def log_event(level: str, component: str, action: str, details: dict | None = None) -> None: ...
def metrics_snapshot() -> dict: ...
```

- 關鍵操作 MUST 具 trace_id / request_id。
- 指標至少 SHOULD 包含延遲、錯誤率、吞吐量、命中率。

---

## 5. 通用資料模型

### 5.1 Knowledge Node

```json
{
  "id": "UID-...",
  "content": "...",
  "content_hash": "...",
  "timestamp_ns": 1730000000000000000,
  "tags": ["tag-a", "tag-b"],
  "metadata": {
    "source": "manual|api|agent",
    "confidence": 0.95
  }
}
```

### 5.2 Search Result

```json
{
  "id": "UID-...",
  "score": 0.91,
  "method": "hybrid",
  "preview": "...",
  "timestamp_ns": 1730000000000000000
}
```

### 5.3 Error Payload

```json
{
  "error": {
    "code": "INDEX_TIMEOUT",
    "message": "vector index timeout",
    "retryable": true,
    "trace_id": "req-123"
  }
}
```

---

## 6. 錯誤模型與處理策略

### 6.1 錯誤分類

1. `ValidationError`：輸入非法或格式錯誤。
2. `StorageError`：存儲、索引、快取失敗。
3. `ProcessingError`：編解碼、壓縮、向量或推理失敗。
4. `SyncError`：同步、網路、認證、衝突處理失敗。
5. `ConfigurationError`：配置缺失或衝突。

### 6.2 錯誤處理規範

1. 所有錯誤 MUST 映射到穩定 `error.code`。
2. 可恢復錯誤 SHOULD 支持有限重試（含退避）。
3. 降級行為 MUST 可觀測、可審計。

---

## 7. 擴展接口（插件與提供者）

```python
class ExtensionProvider:
    def initialize(self, config: dict) -> bool: ...
    def shutdown(self) -> bool: ...
    @property
    def name(self) -> str: ...
    @property
    def version(self) -> str: ...

def register_provider(provider_cls: type[ExtensionProvider]) -> None: ...
def list_providers() -> list[str]: ...
```

**規範要求**

1. 插件生命週期 MUST 明確：初始化、運行、關閉。
2. 插件掛鉤執行失敗 MUST 不得拖垮核心流程。
3. 插件版本 SHOULD 遵循語義化版本。

---

## 8. 版本與兼容策略

### 8.1 SemVer

採用 `MAJOR.MINOR.PATCH`：

- MAJOR：不兼容變更
- MINOR：向後兼容新增
- PATCH：向後兼容修復

### 8.2 兼容承諾

1. 核心接口（L0）在同一 MAJOR 內 MUST 保持兼容。
2. 上層接口（L1-L6）在同一 MINOR 內 SHOULD 保持源碼兼容。
3. 實驗接口 MUST 使用 `experimental_` 前綴。

### 8.3 棄用政策

1. 棄用 MUST 先公告至少一個 MINOR 週期。
2. 棄用接口 MUST 提供替代方案與遷移指引。

---

## 9. SLO 與一致性驗證（不綁定固定數字）

### 9.1 SLO 等級

- **SLO-A（核心）**：ID 生成、編解碼、主檢索路徑。
- **SLO-B（擴展）**：推理、同步、翻譯、插件。
- **SLO-C（背景）**：報表、批處理、低優先任務。

### 9.2 驗證要求

1. MUST 提供往返測試（編碼→解碼一致）。
2. MUST 提供並發安全測試（競態與資料一致性）。
3. MUST 提供錯誤注入測試（超時、斷線、無效輸入）。
4. SHOULD 提供基準壓測方法（資料量、查詢混合比例、硬體條件）。

---

## 10. 安全與隱私最低要求

1. 傳輸與儲存中的敏感資料 MUST 支持加密。
2. 憑證 MUST 不得硬編碼在代碼或示例配置中。
3. 日誌 MUST 支持敏感欄位遮罩。
4. 審計事件 MUST 保留操作主體、時間、動作與結果。

---

## 11. 實作導入建議（最小落地路徑）

1. 先落地 L0/L3/L5（可用核心）。
2. 再加 L6（可觀測性），確保可運維。
3. 最後擴充 L1/L2/L4（語義、協作、推理）。
4. 每次新增功能時，同步更新：接口契約、錯誤碼、遷移指南、測試基準。

---

## 12. 非重製聲明

本文件為通用化、去識別化範例標準，僅保留可公開的架構思路與接口治理方法；
未包含任何私有品牌命名、內部識別符設計細節、供應商綁定參數或不可公開內容。
