# 校園點餐系統｜商家與營業模組

> SAD v0.6｜2026-11-02｜需求基線 SRS v0.6｜設計採用版（待實作驗證）

[回 SAD 索引](../校園點餐系統_SAD_v0.6.md) · [上一份：身分與帳號模組](./01_身分與帳號_Identity.md) · [下一份：商品模組](./03_商品_Catalog.md)

負責商家資料、營業設定與接單資格。商家停用由應用層 MerchantLifecycle 協調本模組、Identity 與 Orders；時段容量的保存與占用由 Resources 負責。

## 模組責任

| Module | 公開 Interface 範例 | 隱藏的主要實作 | 需求 |
| --- | --- | --- | --- |
| Merchant | `createMerchant`、`updateMerchant`、`updateBusinessSchedule`、`canAcceptOrders` | 啟停、營業區間、例外日、接單設定 | FR-004、005 |

完整分工與依賴規則見 [系統架構與模組分工](./00_系統架構與模組分工.md)；資料表見 [共用資料架構](./10_共用資料架構.md)，HTTP 契約見 [共用 API 與錯誤契約](./11_共用API與錯誤契約.md)。

## 商家與營業函式契約

| 函式／追溯 | 輸入 | 成功輸出 | 拒絕及一致性處理 |
| --- | --- | --- | --- |
| `createMerchant`、`updateMerchant`／FR-004 | actor、名稱、位置、聯絡與取餐說明；修改帶 expectedVersion | merchant、version | 欄位錯誤、版本衝突；status 變更分派至停用／恢復工作流程，不能用一般欄位更新繞過 [訂單與交易模組](./06_訂單與交易_Orders.md) |
| `previewMerchantDisable`、`disableMerchant`（應用層 MerchantLifecycle）／FR-004、027 | merchantId；確認請求另帶 confirmed、商家版本與訂單 ID／版本集合摘要 | 預覽 activeOrderCount／confirmationToken；提交後商家與帳號停用、訂單取消結果 | 預覽不鎖定資源；確認時重驗資料，改變則回 `VERSION_CONFLICT` 並要求重新確認；不預設同意 |
| `updateBusinessSchedule`／FR-005 | weeklyHours、closedDates、acceptingOrders、expectedVersion | 設定、effectiveAccepting、unavailableReason、version | 星期、日期、時間錯誤，跨午夜或重疊拒絕；區段採起含迄不含，24:00 僅作當日結束邊界 |
| `canAcceptOrders`／FR-005 | 商家權威設定、serverNow | boolean、reason | 同時檢查商家啟用、接單開關、例外日與營業區間；前端快取不能當送單依據 |

## 時段與停用協作

設計方向：依 2026-11-02 指示先採用 [Q2：商家停用交易與跨模組協調](../校園點餐系統_SAD_v0.6_審查與待確認.md#q2) 的建議；實作及驗證仍須完成。

可預訂截止與時段設定見 [庫存與時段模組](./05_庫存與時段_Resources.md)；二次確認、整店取消及恢復行為見 [訂單模組的自動取消設計](./06_訂單與交易_Orders.md#自動取消與商家停用協作)。

MerchantLifecycle 是應用層協調服務，不新增業務模組。它開啟並持有單一 TransactionContext，依全域順序鎖住必要商家帳號、商家、管理員與學生、訂單及資源；取得全部前置鎖後重驗管理員有效性，再呼叫 Merchant／Identity 的交易內介面及 Orders 的取消介面，由 Orders 協調 Resources、Audit 與 Outbox。所有內部呼叫沿用同一交易，不自行提交。Orders 僅使用 Merchant 的只讀營業政策介面，Merchant 不反向呼叫 Orders。

confirmationToken 由後端簽發並綁 merchantId、商家版本與排序後的受影響訂單 ID／version 集合。新單、結單或版本變更導致不符時，整筆停用不生效，回衝突與最新受影響數量，請管理員重新確認；不可沿舊 token 擴大取消範圍。恢復商家也由此協調入口連動帳號及暫停接單設定。

## 圖 11 管理設定的共通更新

追溯：FR-004～007、009、026。

```mermaid
flowchart TD
  A["管理畫面提交 input 與 expectedVersion"] --> B{"目前角色與資料歸屬合法？"}
  B -->|否| E["拒絕，不修改設定"]
  B -->|是| C{"格式與該功能業務條件合法？"}
  C -->|否| E2["VALIDATION_ERROR，標示欄位"]
  C -->|是| D{"是否停用或恢復商家？"}
  D -->|是| SPECIAL["依專用工作流程<br/>停用須二次確認、連動帳號與系統取消"]
  D -->|否| TX["BEGIN，按共用順序鎖資料<br/>重新檢查版本與限制"]
  TX --> V{"條件仍成立？"}
  V -->|否| R["回滾，回衝突或業務錯誤"]
  V -->|是| W["儲存本次設定與必要稽核"]
  W --> COMMIT["COMMIT，確認成功才更新畫面"]
```

每次只更新所選功能。時段容量不得低於已占用份數，有進行中訂單不能改起迄，已使用或有引用的時段禁止刪除；商品與庫存仍各自檢查 [商品契約](./03_商品_Catalog.md#商品函式契約)及[庫存管理契約](./05_庫存與時段_Resources.md#庫存管理與每日初始化) 的條件。未知提交結果先重讀。
