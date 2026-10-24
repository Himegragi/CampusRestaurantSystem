# 校園點餐系統｜商品與選項模組

> SAD v0.5｜需求基線 SRS v0.6｜設計審查稿

[回 SAD 索引](../校園點餐系統_SAD_v0.5.md) · [上一份：商家與營業模組](./02_商家與營業_Merchant.md) · [下一份：購物車與報價模組](./04_購物車與報價_CartQuote.md)

負責商品、分類、圖片、選項規則與價格版本。庫存數量由 Resources 管理，訂單歷史快照由 Orders 建立。

## 模組責任

| Module | 公開 Interface 範例 | 隱藏的主要實作 | 需求 |
| --- | --- | --- | --- |
| Catalog | `createProduct`、`updateProduct`、`configureOptions`、`validateSelections` | 商品版本、選項限制、圖片發布與上下架 | FR-007、008、010 |

完整分工與依賴規則見 [系統架構與模組分工](./00_系統架構與模組分工.md)；資料表見 [共用資料架構](./10_共用資料架構.md)，HTTP 契約見 [共用 API 與錯誤契約](./11_共用API與錯誤契約.md)。

## 商品與選項函式契約

| 函式／追溯 | 輸入 | 成功輸出 | 拒絕及一致性處理 |
| --- | --- | --- | --- |
| `createProduct`、`updateProduct`／FR-007 | 名稱、category 文字、basePrice、imageAssetId?、listed、expectedVersion | product、priceVersion、version | 名稱 1～50 字、分類 1～20 字且每店最多 20 類、售價 1～9999 整數；越界資產 404，圖片無效 `IMAGE_INVALID` |
| `configureOptions`／FR-008 | productId、groups、選項、minSelections、maxSelections、expectedVersion | 新版群組及商品版本 | 每商品最多 10 群組、每組 20 選項，名稱 1～30 字，加價 0～999；須符合 min/max 及選項數限制 |
| `validateSelections`／FR-008 | 商品權威設定、selectedOptionIds | normalizedSelections、optionExtra | 不存在、跨商品、重複、下架、未達最少或超過最多 → `OPTION_RULE_VIOLATION`；下架使可選數不足最少時，商品不可加入購物車 |

管理修改採欄位允許清單；不能以 body 覆寫 merchantId、reservedQuantity 或歷史快照。分類為商家自訂文字，可在儲存層正規化為表；任何商品／選項修改先鎖父商品，再更新版本。圖片先驗證實際內容並重新編碼，成功發布才可綁定商品；後端不抓任意輸入 URL。

## 共用管理流程

商品修改、版本衝突及稽核提交使用 [管理設定的共通流程](./02_商家與營業_Merchant.md)；商品與庫存類別關係見 [共用資料架構](./10_共用資料架構.md)。
