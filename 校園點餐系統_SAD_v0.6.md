# 校園點餐系統｜系統架構設計文件

> Software Architecture Design（SAD）｜v0.6｜2026-11-02｜設計採用版（待實作驗證）

本文件為 SAD 入口，內容依九個業務模組及共通設計拆成獨立 Markdown 檔。需求基線為 [SRS v0.6](./校園點餐系統_SRS_v0.6.md)。系統維持模組化單體架構。

先讀 [系統架構與模組分工](./SAD_v0.6/00_系統架構與模組分工.md)，再依功能開啟模組文件；需要跨模組協作時，直接使用文中的連結。函式、相關規則與圖形已放到各自模組，Mermaid 圖維持內嵌，省略關聯數量標示。

快速閱讀：[SAD 精簡版](./SAD_v0.6/校園點餐系統_SAD_v0.6_精簡版.md)；審查討論：[審查結果與待確認事項](./SAD_v0.6/校園點餐系統_SAD_v0.6_審查與待確認.md)。

## 本版採用內容

2026-11-05 範圍更新：依使用者決議移除客製化選項功能，FR-008／AC-008 已廢止。Catalog 改為商品模組；計價、快照、API、資料類別、測試設計與追溯同步調整。同日再精簡商品模組，移除商品圖片、分類與獨立售完設定，合併為單一商品資料版本；後續統一所有商品為有限庫存，移除庫存模式、模式切換與略過保留的例外。名詞與欄位以 [商品模組對照](./SAD_v0.6/03_商品_Catalog.md#名詞與欄位對照)及 [領域詞彙](./CONTEXT.md)為準。

2026-11-05 時段規則再精簡：預訂固定提前 10 分鐘截止、容量不得低於已占用份數、已使用時段禁止刪除，取代原封存方案；每日可用量重置規則維持。

整併已採用的建議：個資刪除與獨立復原日誌、商家生命週期協調、建單時確認報價、已使用時段刪除保護，以及分角色 Session 效期。所有問題保留實際情境與處理理由，圖號、圖名及章節連結維持一致。

Session 採學生最長 24 小時／閒置 30 分鐘、商家 12 小時／60 分鐘、管理員 8 小時／15 分鐘，任一先到即失效，自動輪詢不續期。Q5 的部署資源與責任人、Q7 的具體時間參數及驗收界線仍列為後續定版事項；採用設計不等於完成實作驗收。

## 模組文件

| 文件 | 模組 | 內容 |
| --- | --- | --- |
| [身分與帳號模組](./SAD_v0.6/01_身分與帳號_Identity.md) | Identity | 驗證、Session、授權、帳號狀態與個資刪除 |
| [商家與營業模組](./SAD_v0.6/02_商家與營業_Merchant.md) | Merchant | 商家資料、營業、接單資格與停用協作 |
| [商品模組](./SAD_v0.6/03_商品_Catalog.md) | Catalog | 商品名稱、商品單價、商品上下架與商品資料版本 |
| [購物車與報價模組](./SAD_v0.6/04_購物車與報價_CartQuote.md) | CartQuote | 單店購物車、計價與價格確認 |
| [庫存與時段模組](./SAD_v0.6/05_庫存與時段_Resources.md) | Resources | 每日庫存、時段、保留、耗用與釋回 |
| [訂單與交易模組](./SAD_v0.6/06_訂單與交易_Orders.md) | Orders | 建單、快照、狀態命令、自動取消與交易 |
| [通知與工作台模組](./SAD_v0.6/07_通知與工作台_Notifications.md) | Notifications | 站內通知、商家提示與背景投遞 |
| [查詢與統計模組](./SAD_v0.6/08_查詢與統計_AdministrationQuery.md) | AdministrationQuery | 管理訂單查詢與基本營運統計 |
| [操作紀錄模組](./SAD_v0.6/09_操作紀錄_Audit.md) | Audit | 重要異動紀錄、保護與查閱 |

## 共通設計

| 文件 | 內容 |
| --- | --- |
| [系統架構與模組分工](./SAD_v0.6/00_系統架構與模組分工.md) | 全系統 UML、需求依據、技術選型、模組依賴與決策摘要 |
| [共用資料架構](./SAD_v0.6/10_共用資料架構.md) | 領域類別圖、資料表、金額與時間型別、限制及索引 |
| [共用 API 與錯誤契約](./SAD_v0.6/11_共用API與錯誤契約.md) | 共用 HTTP 契約、端點、錯誤與輸入規則 |
| [前端架構](./SAD_v0.6/12_前端架構.md) | 畫面狀態、前景輪詢、斷線與恢復 |
| [部署維運與驗證](./SAD_v0.6/13_部署維運與驗證.md) | 部署、備份、監控、效能、驗證與交付條件 |
| [需求追溯](./SAD_v0.6/14_需求追溯.md) | 有效 FR-001～007、009～028 的設計落點、FR-008 廢止紀錄，以及 NFR 與其他需求的設計落點 |

## 圖表索引

圖號採全套文件共用編號：圖 0 為全系統總覽，圖 1～18 為各視角。圖名及所在文件的連結均直接定位到同名圖表標題；每個標題下都有 Mermaid 圖。商品模組與操作紀錄模組共用其他文件的圖，未另列不存在的專屬圖。

| 圖號 | 圖名 | 圖型 | 所在文件 | 原始碼 |
| --- | --- | --- | --- | --- |
| 圖 0 | [全系統 UML 架構總覽](./SAD_v0.6/00_系統架構與模組分工.md#圖-0-全系統-uml-架構總覽) | UML 類別與相依圖 | [系統架構與模組分工](./SAD_v0.6/00_系統架構與模組分工.md#圖-0-全系統-uml-架構總覽) | [Mermaid](./SAD_v0.6_diagrams/00_system_architecture_uml.mmd) |
| 圖 1 | [角色與功能架構圖](./SAD_v0.6/00_系統架構與模組分工.md#圖-1-角色與功能架構圖) | 架構示意 | [系統架構與模組分工](./SAD_v0.6/00_系統架構與模組分工.md#圖-1-角色與功能架構圖) | [Mermaid](./SAD_v0.6_diagrams/01_roles.mmd) |
| 圖 2 | [部署架構圖](./SAD_v0.6/00_系統架構與模組分工.md#圖-2-部署架構圖) | 架構示意 | [系統架構與模組分工](./SAD_v0.6/00_系統架構與模組分工.md#圖-2-部署架構圖) | [Mermaid](./SAD_v0.6_diagrams/02_deployment.mmd) |
| 圖 3 | [模組依賴圖](./SAD_v0.6/00_系統架構與模組分工.md#圖-3-模組依賴圖) | 架構示意 | [系統架構與模組分工](./SAD_v0.6/00_系統架構與模組分工.md#圖-3-模組依賴圖) | [Mermaid](./SAD_v0.6_diagrams/03_modules.mmd) |
| 圖 4 | [帳號、菜單與資源類別圖](./SAD_v0.6/10_共用資料架構.md#圖-4-帳號菜單與資源類別圖) | UML 類別圖 | [共用資料架構](./SAD_v0.6/10_共用資料架構.md#圖-4-帳號菜單與資源類別圖) | [Mermaid](./SAD_v0.6_diagrams/04_catalog_classes.mmd) |
| 圖 5 | [訂單、快照與保留類別圖](./SAD_v0.6/10_共用資料架構.md#圖-5-訂單快照與保留類別圖) | UML 類別圖 | [共用資料架構](./SAD_v0.6/10_共用資料架構.md#圖-5-訂單快照與保留類別圖) | [Mermaid](./SAD_v0.6_diagrams/05_order_classes.mmd) |
| 圖 6 | [訂單狀態圖](./SAD_v0.6/06_訂單與交易_Orders.md#圖-6-訂單狀態圖) | UML 狀態圖 | [訂單與交易_Orders](./SAD_v0.6/06_訂單與交易_Orders.md#圖-6-訂單狀態圖) | [Mermaid](./SAD_v0.6_diagrams/06_order_state.mmd) |
| 圖 7 | [送單識別與重播循序圖](./SAD_v0.6/06_訂單與交易_Orders.md#圖-7-送單識別與重播循序圖) | UML 循序圖 | [訂單與交易_Orders](./SAD_v0.6/06_訂單與交易_Orders.md#圖-7-送單識別與重播循序圖) | [Mermaid](./SAD_v0.6_diagrams/07_submission_sequence.mmd) |
| 圖 8 | [建立訂單交易循序圖](./SAD_v0.6/06_訂單與交易_Orders.md#圖-8-建立訂單交易循序圖) | UML 循序圖 | [訂單與交易_Orders](./SAD_v0.6/06_訂單與交易_Orders.md#圖-8-建立訂單交易循序圖) | [Mermaid](./SAD_v0.6_diagrams/08_order_transaction_sequence.mmd) |
| 圖 9 | [通知投遞循序圖](./SAD_v0.6/07_通知與工作台_Notifications.md#圖-9-通知投遞循序圖) | UML 循序圖 | [通知與工作台_Notifications](./SAD_v0.6/07_通知與工作台_Notifications.md#圖-9-通知投遞循序圖) | [Mermaid](./SAD_v0.6_diagrams/09_notification_sequence.mmd) |
| 圖 10 | [登入與一次性驗證](./SAD_v0.6/01_身分與帳號_Identity.md#圖-10-登入與一次性驗證) | UML 循序圖 | [身分與帳號_Identity](./SAD_v0.6/01_身分與帳號_Identity.md#圖-10-登入與一次性驗證) | [Mermaid](./SAD_v0.6_diagrams/10_login_sequence.mmd) |
| 圖 11 | [管理設定的共通更新](./SAD_v0.6/02_商家與營業_Merchant.md#圖-11-管理設定的共通更新) | 函式流程圖 | [商家與營業_Merchant](./SAD_v0.6/02_商家與營業_Merchant.md#圖-11-管理設定的共通更新) | [Mermaid](./SAD_v0.6_diagrams/11_management_flow.mmd) |
| 圖 12 | [購物車至價格確認](./SAD_v0.6/04_購物車與報價_CartQuote.md#圖-12-購物車至價格確認) | 函式流程圖 | [購物車與報價_CartQuote](./SAD_v0.6/04_購物車與報價_CartQuote.md#圖-12-購物車至價格確認) | [Mermaid](./SAD_v0.6_diagrams/12_cart_quote_flow.mmd) |
| 圖 13 | [庫存與容量的一次保留](./SAD_v0.6/05_庫存與時段_Resources.md#圖-13-庫存與容量的一次保留) | 函式流程圖 | [庫存與時段_Resources](./SAD_v0.6/05_庫存與時段_Resources.md#圖-13-庫存與容量的一次保留) | [Mermaid](./SAD_v0.6_diagrams/13_reserve_resources_flow.mmd) |
| 圖 14 | [狀態命令檢查與提交](./SAD_v0.6/06_訂單與交易_Orders.md#圖-14-狀態命令檢查與提交) | 函式流程圖 | [訂單與交易_Orders](./SAD_v0.6/06_訂單與交易_Orders.md#圖-14-狀態命令檢查與提交) | [Mermaid](./SAD_v0.6_diagrams/14_order_command_flow.mmd) |
| 圖 15 | [依原狀態處理資源](./SAD_v0.6/05_庫存與時段_Resources.md#圖-15-依原狀態處理資源) | 函式流程圖 | [庫存與時段_Resources](./SAD_v0.6/05_庫存與時段_Resources.md#圖-15-依原狀態處理資源) | [Mermaid](./SAD_v0.6_diagrams/15_resource_effects_flow.mmd) |
| 圖 16 | [接單與學生取消的競爭](./SAD_v0.6/06_訂單與交易_Orders.md#圖-16-接單與學生取消的競爭) | UML 循序圖 | [訂單與交易_Orders](./SAD_v0.6/06_訂單與交易_Orders.md#圖-16-接單與學生取消的競爭) | [Mermaid](./SAD_v0.6_diagrams/16_accept_cancel_sequence.mmd) |
| 圖 17 | [交付核碼與未取的分流](./SAD_v0.6/06_訂單與交易_Orders.md#圖-17-交付核碼與未取的分流) | 函式流程圖 | [訂單與交易_Orders](./SAD_v0.6/06_訂單與交易_Orders.md#圖-17-交付核碼與未取的分流) | [Mermaid](./SAD_v0.6_diagrams/17_pickup_flow.mmd) |
| 圖 18 | [管理查詢、統計與稽核查閱](./SAD_v0.6/08_查詢與統計_AdministrationQuery.md#圖-18-管理查詢統計與稽核查閱) | 函式流程圖 | [查詢與統計_AdministrationQuery](./SAD_v0.6/08_查詢與統計_AdministrationQuery.md#圖-18-管理查詢統計與稽核查閱) | [Mermaid](./SAD_v0.6_diagrams/18_admin_query_flow.mmd) |

共 19 張圖；內嵌定義與原始碼同步維護。流程圖是協作示意；類別、狀態與循序圖使用各自 UML 表達。
