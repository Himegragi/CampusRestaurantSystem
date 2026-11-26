# AI 使用紀錄索引

本資料夾收錄本專案使用 AI agent（Claude Code、Codex）的對話紀錄，依對話開始日期排列。時間皆為台北時間（UTC+08:00）。

| 日期 | 工具 | 紀錄 | 主題 |
| --- | --- | --- | --- |
| 2026-10-24 | Claude Code | [SRS 改寫為純需求規格](2026-10-24_Claude_SRS改寫為純需求規格.md) | 把 SRS 改寫成純需求規格，移出設計內容，精簡編號，定為 v0.5 |
| 2026-10-24 | Claude Code | [SRS 附錄 B 待決議事項](2026-10-24-165150-srsv05md-grill-with-docs-b.md) | 用 grilling 技能逐題決議 SRS v0.5 附錄 B（終端畫面匯出） |
| 2026-10-24 | Claude Code | [SAD 依賴圖與 SRS 驗收事項](2026-10-24_Claude_SAD依賴圖與SRS驗收事項.md) | 依賴圖的連線、SRS 是否放驗收事項、決議紀錄搬檔 |
| 2026-10-26 | Claude Code | [SAD v0.5 可讀性與邏輯檢查](2026-10-26_Claude_SAD_v0.5可讀性與邏輯檢查.md) | SAD v0.5 與模組文件的可讀性、邏輯缺口、圖號 |
| 2026-10-27 | Codex | [EGM 審查與文件流程討論](2026-10-27_Codex_EGM審查與文件流程討論.md) | 安裝 EGM skills、審查 SAD v0.5、審查人力與標準、文件機械感、Q1～Q7 改寫 |
| 2026-11-03 | Codex | [SAD v0.6 架構與先備知識問答](2026-11-05_SAD_v0.6_架構與先備知識問答.md) | Worker、先備知識、去重、訂單狀態、UML 總覽、圖 11 |
| 2026-11-05 | Codex | [術語對齊與商品、庫存模組精簡](2026-11-05_Codex_術語對齊與商品庫存模組精簡.md) | PRD/SRS/SAD 術語對齊，精簡 Catalog 與 Resources 模組，commit 與分支 |

## 未另外整理的紀錄

- Claude Code `82f1f2f6`：與 SRS 改寫那段對話（`ce8144de`）的前段相同，是同一段對話的早期分支。
- Codex `codex-auto-review` 子對話（2026-11-05、11-06、11-09 共三段）：Codex 在執行 commit 前自動判斷是否放行指令的內部紀錄，不是使用者對話。

## 原始紀錄位置

原始紀錄存在整理者的電腦上，其他成員無法直接開啟：

- Claude Code：`C:/Users/User/.claude/projects/d--Projects-CampusRestaurantSystem/*.jsonl`
- Codex：`C:/Users/User/.codex/sessions/2026/<月>/<日>/rollout-*.jsonl`，以檔案第一行的 `cwd` 判斷是否屬於本專案
