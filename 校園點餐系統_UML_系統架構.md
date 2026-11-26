# 校園點餐系統｜UML 系統架構圖

> 2026-11-11｜依據 SAD v0.6｜設計方案，程式尚未開始寫

這張圖用 UML 類別圖的表示法畫出系統的主要部分，以及它們之間誰依賴誰。後端三組用 `<<package>>` 表示，組內列出包含的模組。只畫到「有哪幾塊、誰依賴誰」這一層；模組內部的函式與資料，見 SAD v0.6 的〈系統架構與模組分工〉圖 0。

```mermaid
classDiagram
direction TB

class Student["學生"]
<<actor>> Student
Student : 看菜單、下單、取餐

class MerchantStaff["商家人員"]
<<actor>> MerchantStaff
MerchantStaff : 管菜單庫存、接單

class Admin["管理員"]
<<actor>> Admin
Admin : 管帳號、查訂單統計

class WebApp["前端網站"]
<<boundary>> WebApp
WebApp : 三種角色共用，登入後畫面不同

class AccountMerchant["帳號與商家"]
<<package>> AccountMerchant
AccountMerchant : 身分 Identity
AccountMerchant : 商家 Merchant

class OrderingAcceptance["點餐與接單"]
<<package>> OrderingAcceptance
OrderingAcceptance : 商品 Catalog
OrderingAcceptance : 購物車與報價 CartQuote
OrderingAcceptance : 庫存與時段 Resources
OrderingAcceptance : 訂單 Orders

class NotifyAdmin["通知與管理"]
<<package>> NotifyAdmin
NotifyAdmin : 通知 Notifications
NotifyAdmin : 查詢統計 AdministrationQuery
NotifyAdmin : 操作紀錄 Audit

class Worker["背景程式"]
<<process>> Worker
Worker : 自動取消逾時訂單
Worker : 每日重置庫存
Worker : 送通知

class Database["資料庫"]
<<database>> Database
Database : 帳號、商品、庫存、訂單

class MailService["寄信服務"]
<<external>> MailService
MailService : 寄登入驗證碼

Student ..> WebApp : 使用
MerchantStaff ..> WebApp : 使用
Admin ..> WebApp : 使用

WebApp ..> AccountMerchant : 登入、商家設定
WebApp ..> OrderingAcceptance : 點餐、接單
WebApp ..> NotifyAdmin : 看通知、查資料

OrderingAcceptance ..> AccountMerchant : 檢查身分與營業狀態
OrderingAcceptance ..> NotifyAdmin : 寫入事件與操作紀錄
AccountMerchant ..> MailService : 寄驗證碼

Worker ..> OrderingAcceptance : 逾時取消、重置庫存
Worker ..> NotifyAdmin : 產生通知

AccountMerchant ..> Database : 讀寫
OrderingAcceptance ..> Database : 讀寫
NotifyAdmin ..> Database : 讀寫
```

## 怎麼讀

- **虛線箭頭**（`..>`）是 UML 的「依賴」：A 指向 B，表示 A 要用到 B。例如「點餐與接單」指向「帳號與商家」，是因為成立訂單前要先確認學生身分、商家有在營業。
- **`<< >>` 裡的字**是 UML 的「構造型」（stereotype），標示這個方塊是哪一類東西：

| 構造型 | 意思 |
| --- | --- |
| `<<actor>>` | 使用系統的人 |
| `<<boundary>>` | 使用者直接接觸的介面 |
| `<<package>>` | 一組模組，方塊內列出包含哪些模組 |
| `<<process>>` | 獨立執行的程式 |
| `<<database>>` | 資料庫 |
| `<<external>>` | 系統外部的服務 |

## 後端三組

| 組 | 包含的模組 | 負責什麼 |
| --- | --- | --- |
| 帳號與商家 | 身分、商家 | 登入、確認你能做什麼，以及商家的營業設定 |
| 點餐與接單 | 商品、購物車與報價、庫存與時段、訂單 | 從看菜單、算價格、扣庫存、成立訂單，到商家確認訂單和交付餐點 |
| 通知與管理 | 通知、查詢統計、操作紀錄 | 通知學生和商家、讓管理員查資料、記下誰做了什麼 |

**技術**：前端 React、後端 NestJS、資料庫 PostgreSQL。
