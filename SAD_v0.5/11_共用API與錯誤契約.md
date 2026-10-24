# 校園點餐系統｜共用 API 與錯誤契約

> SAD v0.5｜需求基線 SRS v0.6｜設計審查稿

[回 SAD 索引](../校園點餐系統_SAD_v0.5.md) · [上一份：共用資料架構](./10_共用資料架構.md) · [下一份：前端架構](./12_前端架構.md)

本章定義共用介面契約，欄位限制與政策以 SRS v0.6 為準。函式為設計契約，不代表已有程式實作。主要入口對應 [共用 API 與錯誤契約](./11_共用API與錯誤契約.md) 的 API，內部協作函式不另開公開端點。actor 與 now 由後端提供；tx 表示呼叫端持有的同一資料庫交易，內部函式不得自行提交。

## HTTP 共通契約

**D-05**：路徑前綴 `/api/v1`，Cookie Session 驗證；寫入請求具 CSRF 防護。`actorId`、`studentId` 及商家操作身分由伺服器建立。需並行保護的修改使用 `expectedVersion`；狀態命令使用 `commandId`。

成功回應為 `{data, requestId}`；列表為 `{data:{items,nextCursor},requestId}`。採穩定排序與 cursor，含同時間資料的 ID 次序。`requestId` 用於除錯，`Idempotency-Key` 用於同次送單去重，`commandId` 用於同次狀態命令去重，三者不可混用。

| 類型 | HTTP | 代表 code／行為 |
| --- | --- | --- |
| JSON 格式錯誤／欄位非法 | 400／422 | `VALIDATION_ERROR`，保留可修正內容 |
| 未登入／帳號停用／越權 | 401／403 | `UNAUTHENTICATED`、`ACCOUNT_DISABLED`、`FORBIDDEN` |
| 無資源或不在可見範圍 | 404 | `RESOURCE_NOT_FOUND`，不揭露他人資料是否存在 |
| 條件或版本衝突 | 409 | `QUOTE_CHANGED`、`SLOT_FULL`、`INSUFFICIENT_STOCK`、`VERSION_CONFLICT` |
| 狀態或取消不合法 | 409 | `INVALID_ORDER_TRANSITION`、`CANCELLATION_NOT_ALLOWED` |
| 核碼或交付確認失敗 | 422 | `PICKUP_CODE_MISMATCH`、`HANDOVER_REQUIRED` |
| 尚在處理 | 202 | `PROCESSING`，回傳原操作查詢路徑 |
| 超過請求頻率 | 429 | `RATE_LIMITED`，提供稍後重試資訊 |
| 已過期操作 | 410 | `SUBMISSION_EXPIRED`、`COMMAND_EXPIRED`，查看現有訂單，不重跑舊操作 |
| 進行中訂單達上限 | 409 | `ACTIVE_ORDER_LIMIT_REACHED`，引導訂單頁 |
| 取餐碼核對鎖定 | 429 | `PICKUP_VERIFICATION_LOCKED`，提供 retryAfter，不暴露碼 |
| 暫時不可用／非預期錯誤 | 503／500 | 結果可能未知時沿用原 key 查詢，不宣告已回滾 |

## 端點索引

表中省略 `/api/v1`。完整欄位與 OpenAPI 檔待實作階段產製；不得僅因知道路徑而跳過資源授權。

| 領域 | 端點 | 存取與重點 |
| --- | --- | --- |
| 登入 | `POST /auth/email/challenges`、`POST /auth/email/verify`、`POST /auth/logout`、`GET /me` | 受限申請頻率、一次性驗證；登出撤銷 Session |
| 公開菜單 | `GET /merchants`、`GET /merchants/{id}/menu`、`GET /merchants/{id}/pickup-slots` | 僅已公開資料；瀏覽不保證送單可用 |
| 購物車 | `GET/PUT /cart`、`DELETE /cart` | 本人；跨店替換必須確認；以版本防止舊畫面覆寫 |
| 報價與送單 | `POST /quotes`、`POST /orders`、`GET /order-submissions/{key}` | 本人；後端計價；建立訂單須 Idempotency-Key |
| 個資刪除 | `DELETE /me`，body 含 `confirmed:true` | 本人學生，全部訂單終止才可執行；見 [身分與帳號模組](./01_身分與帳號_Identity.md) |
| 命令結果 | `GET /orders/{id}/commands/{commandId}` | 原命令操作者且目前仍可讀該訂單；逾期回 410 |
| 學生訂單 | `GET /orders`、`GET /orders/{id}`、`POST /orders/{id}/cancel` | 本人；只可取消待確認訂單 |
| 商家設定 | `PUT /merchant/business-settings`、`POST /merchant/pickup-slots`、`PATCH /merchant/pickup-slots/{id}` | 所屬商家；具版本與容量保護 |
| 商品與圖片 | `POST /merchant/products`、`PATCH /merchant/products/{id}`、`PUT /merchant/products/{id}/options`、`POST /merchant/images` | 所屬商家；選項更新同步商品版本 |
| 庫存 | `PATCH /merchant/products/{id}/inventory` | 不能覆寫保留數量；遵循共用鎖規則 |
| 接單清單 | `GET /merchant/orders`、`GET /merchant/overview` | 所屬商家；待確認數量與狀態清單 |
| 商家命令 | `POST /merchant/orders/{id}/accept`、`/ready`、`/reject`、`/cancel`、`/complete`、`/uncollected` | 每次帶 commandId、expectedVersion；拒單／取消需原因 |
| 站內通知 | `GET /notifications` | 目前有權限的本人通知；訂單詳細內容另行驗權 |
| 商家停用預覽 | `GET /admin/merchants/{id}/disable-preview` | 回受影響數與確認 token；不產生異動 |
| 帳號／商家管理 | `POST /admin/accounts`、`PATCH /admin/accounts/{id}/status`、`POST /admin/merchants`、`PATCH /admin/merchants/{id}` | 管理員；商家停用須二次確認並連動帳號與系統取消 |
| 管理查詢 | `GET /admin/orders`、`GET /admin/statistics`、`GET /admin/audit-events` | 受控查詢；統計含訂單、取消四類、未取、完成、進行中及商品備餐數量 |

商家統計使用 `GET /merchant/statistics`，後端固定所屬商家；管理員查詢可涵蓋全部商家。API 統一採 `/quotes`、`/merchant/products/{id}/options`、`/merchant/overview`，不另保留 v0.4 的 `/order-quotes`、`/option-groups`、`/order-board` 別名。

## 輸入型別、錯誤與限流契約

| 型別 | 契約 |
| --- | --- |
| ID | 伺服器核發、不透明識別；格式正確不代表有權讀取 |
| String | 逐欄定義必填與長度；必填文字 trim 後非空；字數提案按 Unicode 碼點計，前後端使用同一函式 |
| Integer／Money | 嚴格整數及範圍檢查，不把字串、小數或溢位值隱式轉成有效輸入；Money 為 TWD 整數元 |
| Boolean | 只接受 JSON true／false，不接受 0、1 或字串替代 |
| Date／Time／Timestamp | 真實存在的日期與時間，時間點帶偏移；營業判斷使用伺服器時鐘與 Asia/Taipei |
| Enum／Array | 未知列舉拒絕；陣列逐項驗證，檢查空集合、重複、混店及整單總份數 |
| PATCH | 缺漏代表不修改；null 只有明確可清除的選填欄位可用，不可清除必填資料；拒絕未定義欄位 |
| AuthContext | 由 Session 解析；不接受 body 提供的 role、studentId、permissions 作授權依據 |

錯誤回應使用 `{error:{code,message,fields?,retryable},requestId}`；fields 只含可修正欄位路徑與理由，不含其他人資料、SQL 或堆疊。`retryable` 只表示同一次操作是否可安全重試，不能解讀為自動產生新 key。未知結果先查原識別；資料變更後須重新確認，才建立新操作。

除 [共用 API 與錯誤契約](./11_共用API與錯誤契約.md) 外補充代碼：`OPTION_RULE_VIOLATION`／422、`CART_MERCHANT_CONFLICT`／409、`PRODUCT_UNAVAILABLE`／409、`MERCHANT_UNAVAILABLE`／409、`SLOT_UNAVAILABLE`／409、`PICKUP_WINDOW_NOT_ENDED`／409、`COMMAND_ID_REUSED`／409、`QUERY_FAILED`／500。畫面保留可修正輸入；409 提示重讀與確認，500 不以空資料或「已回滾」掩蓋未知結果。

限流的非業務固定值採設計起始值：每 IP 登入申請 30 次／分鐘、驗證 60 次／分鐘；每商家帳號圖片上傳 10 次／分鐘；管理查詢每帳號 60 次／分鐘、同時最多 3 個高成本查詢。共用持久計數器、回 429 與 Retry-After，依負載與濫用測試調整；不得放寬 FR-001 的信箱配額或 FR-019 的逐訂單錯碼鎖定。
