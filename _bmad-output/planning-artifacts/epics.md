---
stepsCompleted:
  - step-01-validate-prerequisites
  - step-02-design-epics
  - step-03-create-stories
  - step-04-final-validation
status: complete
completedAt: '2026-01-25'
inputDocuments:
  - _bmad-output/planning-artifacts/prd.md
  - _bmad-output/planning-artifacts/architecture.md
---

# bookingSystem - Epic Breakdown

## Overview

This document provides the complete epic and story breakdown for bookingSystem, decomposing the requirements from the PRD, UX Design if it exists, and Architecture requirements into implementable stories.

## Requirements Inventory

### Functional Requirements

**週期預約建立**
- FR1: 使用者可以建立每週重複的預約
- FR2: 使用者可以建立隔週重複的預約
- FR3: 使用者可以建立每月固定日期的預約
- FR4: 使用者可以建立每月固定週次的預約（如：每月第一個週五）
- FR5: 使用者可以設定週期預約的結束日期
- FR6: 使用者可以設定週期預約的重複次數

**衝突管理**
- FR7: 系統可以在建立週期預約時檢測所有場次的時段衝突
- FR8: 使用者可以查看週期預約的衝突清單
- FR9: 使用者可以選擇跳過衝突場次並建立其他可用場次
- FR10: 使用者可以選擇取消整個週期預約建立

**週期預約編輯與取消**
- FR11: 使用者可以編輯單一場次的週期預約
- FR12: 使用者可以編輯某場次及其之後所有場次
- FR13: 使用者可以編輯整個週期預約系列
- FR14: 使用者可以取消單一場次的週期預約
- FR15: 使用者可以取消某場次及其之後所有場次
- FR16: 使用者可以取消整個週期預約系列

**日曆顯示**
- FR17: 使用者可以在日曆上查看週期預約的所有場次
- FR18: 使用者可以識別哪些預約屬於同一個週期系列

**管理員功能**
- FR19: 管理員可以查看所有使用者的週期預約
- FR20: 管理員可以代理編輯使用者的週期預約
- FR21: 管理員可以代理取消使用者的週期預約
- FR22: 系統可以記錄管理員對週期預約的操作日誌

**規則整合**
- FR23: 系統可以將現有預約規則（時段限制）套用至週期預約
- FR24: 一般使用者的週期預約受限於工作日 8:00-18:00

**通知**
- FR25: 系統可以在週期預約建立成功時發送 Email 通知
- FR26: 系統可以在週期預約修改時發送 Email 通知
- FR27: 系統可以在週期預約取消時發送 Email 通知

### NonFunctional Requirements

**效能**
- NFR1: 週期預約建立時的衝突檢查應在 3 秒內完成（最多 52 週）
- NFR2: 日曆載入週期預約顯示應在 2 秒內完成
- NFR3: 批次編輯/取消操作應在 5 秒內完成

**整合**
- NFR4: 週期預約功能應與現有預約 API 保持一致的介面風格
- NFR5: 週期預約應完整整合現有 Email 通知機制
- NFR6: 週期預約應遵循現有權限控制機制（privilege_level）

**資料一致性**
- NFR7: 週期預約的批次操作應使用資料庫交易，確保原子性
- NFR8: 系統應防止週期預約建立時的競爭條件（race condition）

### Additional Requirements

**架構決策（來自 Architecture.md）**

- **ADR-001 資料模型**：採用展開儲存模式，每個週期場次儲存為獨立 Reservation 記錄
- **ADR-002 資料表結構**：新增 RecurringSeries 表儲存週期規則，Reservation 表新增 `series_id` 外鍵
- **ADR-003 週期規則格式**：使用 iCalendar RRULE (RFC 5545) 標準格式
- **ADR-004 API 端點設計**：混合設計，週期建立用獨立端點 `/api/recurring-reservation`，單筆操作複用現有端點
- **ADR-005 衝突檢測策略**：展開所有場次後，一次 SQL 批次查詢所有衝突
- **ADR-006 週期預約 UI 元件**：新增獨立元件 `recurringForm.vue`，由 `application.vue` 引入
- **ADR-007 日曆顯示方式**：週期預約加上視覺標記（圖示或特殊樣式）
- **ADR-008 批次操作交易策略**：整個批次操作在單一 transaction 內完成

**技術約束**

- 前端：Vue.js 3.4 (Options API)，無集中狀態管理
- 後端：Express.js 4.19，JWT Cookie 認證
- 資料庫：MariaDB
- 命名慣例：PascalCase 表名、snake_case 欄位、camelCase 程式碼

**實作順序（來自架構文件）**

1. 資料庫 Schema 變更（RecurringSeries 表 + Reservation 欄位）
2. 後端 Model 層（recurringSeries.js + reservation.js 擴展）
3. 後端 API 層（recurringReservation.js）
4. 前端元件（recurringForm.vue）
5. 前端整合（application.vue, calendar.vue 修改）
6. 測試（單元測試 + E2E）

### FR Coverage Map

| FR 編號 | Epic | 說明 |
|---------|------|------|
| FR1 | Epic 1 | 建立每週重複的預約 |
| FR2 | Epic 1 | 建立隔週重複的預約 |
| FR3 | Epic 1 | 建立每月固定日期的預約 |
| FR4 | Epic 1 | 建立每月固定週次的預約 |
| FR5 | Epic 1 | 設定週期預約的結束日期 |
| FR6 | Epic 1 | 設定週期預約的重複次數 |
| FR7 | Epic 1 | 建立時檢測所有場次的時段衝突 |
| FR8 | Epic 1 | 查看週期預約的衝突清單 |
| FR9 | Epic 1 | 選擇跳過衝突場次並建立其他可用場次 |
| FR10 | Epic 1 | 選擇取消整個週期預約建立 |
| FR11 | Epic 2 | 編輯單一場次的週期預約 |
| FR12 | Epic 2 | 編輯某場次及其之後所有場次 |
| FR13 | Epic 2 | 編輯整個週期預約系列 |
| FR14 | Epic 2 | 取消單一場次的週期預約 |
| FR15 | Epic 2 | 取消某場次及其之後所有場次 |
| FR16 | Epic 2 | 取消整個週期預約系列 |
| FR17 | Epic 3 | 在日曆上查看週期預約的所有場次 |
| FR18 | Epic 3 | 識別哪些預約屬於同一個週期系列 |
| FR19 | Epic 4 | 管理員查看所有使用者的週期預約 |
| FR20 | Epic 4 | 管理員代理編輯使用者的週期預約 |
| FR21 | Epic 4 | 管理員代理取消使用者的週期預約 |
| FR22 | Epic 4 | 記錄管理員對週期預約的操作日誌 |
| FR23 | Epic 1 | 將現有預約規則套用至週期預約 |
| FR24 | Epic 1 | 一般使用者週期預約受限於工作日 8:00-18:00 |
| FR25 | Epic 1 | 週期預約建立成功時發送 Email 通知 |
| FR26 | Epic 2 | 週期預約修改時發送 Email 通知 |
| FR27 | Epic 2 | 週期預約取消時發送 Email 通知 |

## Epic List

### Epic 1：週期預約建立與衝突處理
用戶可以建立各種模式的週期預約，並在有衝突時獲得清晰的處理選項。

**FRs covered:** FR1, FR2, FR3, FR4, FR5, FR6, FR7, FR8, FR9, FR10, FR23, FR24, FR25

### Epic 2：週期預約編輯與取消
用戶可以靈活地修改或取消週期預約，選擇影響單次、此後所有、或整個週期。

**FRs covered:** FR11, FR12, FR13, FR14, FR15, FR16, FR26, FR27

### Epic 3：日曆週期預約顯示
用戶可以在日曆上清楚查看並識別週期預約。

**FRs covered:** FR17, FR18

### Epic 4：管理員週期預約管理
管理員可以查看、代理操作並追蹤所有用戶的週期預約。

**FRs covered:** FR19, FR20, FR21, FR22

## Epic 1：週期預約建立與衝突處理

用戶可以建立各種模式的週期預約，並在有衝突時獲得清晰的處理選項。

### Story 1.1：週期預約資料架構與基礎建立

As a 使用者,
I want 建立每週重複的會議室預約,
So that 我可以一次設定整學期的固定會議時間，省去每週重複操作。

**Acceptance Criteria:**

**Given** 使用者已登入系統並進入預約頁面
**When** 使用者選擇會議室、設定時間、勾選「週期預約」並選擇「每週」模式
**Then** 系統顯示週期設定表單，包含結束條件選項
**And** 使用者可以預覽將產生的所有場次日期

**Given** 使用者填寫完整的週期預約資訊（每週模式）
**When** 使用者點擊「建立預約」
**Then** 系統在 RecurringSeries 表建立週期規則記錄
**And** 系統在 Reservation 表建立所有展開的場次記錄，每筆記錄包含 series_id 關聯
**And** 所有場次在單一資料庫交易中完成（NFR7）

**Given** 後端收到週期預約建立請求
**When** 系統處理 RRULE 規則展開場次
**Then** 使用 iCalendar RRULE (RFC 5545) 標準格式儲存規則
**And** 正確計算每週重複的所有日期

**技術備註：**
- 建立 RecurringSeries 資料表（schema.sql）
- 修改 Reservation 表新增 series_id, occurrence_date 欄位
- 建立 recurringSeries.js Model
- 建立 rrule.js 工具（每週模式）
- 建立 recurringReservation.js API（POST 端點）
- 建立 recurringForm.vue 前端元件（基礎版）

---

### Story 1.2：支援所有週期模式

As a 使用者,
I want 選擇不同的週期重複模式（隔週、每月固定日、每月固定週次）,
So that 我可以根據實際會議需求設定最適合的重複規則。

**Acceptance Criteria:**

**Given** 使用者在週期預約表單中
**When** 使用者選擇「隔週」模式
**Then** 系統正確產生隔週重複的場次日期
**And** RRULE 規則使用 INTERVAL=2 參數

**Given** 使用者選擇「每月固定日期」模式（如每月 15 日）
**When** 使用者指定日期並建立預約
**Then** 系統正確產生每月該日期的場次
**And** 若該月無此日期（如 2 月 30 日），則跳過該月

**Given** 使用者選擇「每月固定週次」模式（如每月第一個週五）
**When** 使用者指定週次和星期幾並建立預約
**Then** 系統正確計算每月對應的日期
**And** RRULE 規則使用 BYDAY 參數（如 1FR 表示第一個週五）

**Given** 使用者預覽任一週期模式的場次
**When** 展開場次計算完成
**Then** 前端顯示所有將產生的日期清單
**And** 計算時間符合 NFR1（≤3 秒）

**技術備註：**
- 擴展 rrule.js 支援 INTERVAL、BYMONTHDAY、BYDAY 參數
- 更新 recurringForm.vue 新增模式選擇 UI
- 新增場次預覽功能

---

### Story 1.3：週期預約結束條件設定

As a 使用者,
I want 設定週期預約的結束條件（指定日期或重複次數）,
So that 我可以精確控制週期預約的範圍。

**Acceptance Criteria:**

**Given** 使用者選擇「結束日期」作為結束條件
**When** 使用者指定一個結束日期並建立預約
**Then** 系統產生從開始日到結束日之間所有符合週期規則的場次
**And** RRULE 使用 UNTIL 參數

**Given** 使用者選擇「重複次數」作為結束條件
**When** 使用者指定次數（如 10 次）並建立預約
**Then** 系統精確產生指定次數的場次
**And** RRULE 使用 COUNT 參數

**Given** 使用者設定的結束條件會產生超過 52 週的場次
**When** 系統計算場次數量
**Then** 系統仍正常處理（無硬性上限）
**And** 衝突檢查仍在 3 秒內完成（NFR1）

**技術備註：**
- 擴展 rrule.js 支援 UNTIL 和 COUNT 參數
- 更新 recurringForm.vue 結束條件 UI
- 確保長期週期預約的效能

---

### Story 1.4：批次衝突檢測與顯示

As a 使用者,
I want 在建立週期預約前看到所有場次的衝突情況,
So that 我可以了解哪些時段已被佔用，做出適當的決定。

**Acceptance Criteria:**

**Given** 使用者設定完週期預約參數並點擊「檢查可用性」或「建立預約」
**When** 系統展開所有場次日期
**Then** 系統執行單次批次 SQL 查詢檢測所有場次的衝突
**And** 檢測在 3 秒內完成（NFR1）

**Given** 部分場次與現有預約時間重疊
**When** 衝突檢測完成
**Then** 系統顯示衝突清單，包含：衝突場次的日期、衝突的現有預約名稱、衝突時段
**And** 清單以日期順序排列

**Given** 所有場次皆無衝突
**When** 衝突檢測完成
**Then** 系統顯示「所有場次皆可預約」訊息
**And** 使用者可直接確認建立

**Given** 兩個使用者同時檢測相同時段
**When** 系統處理衝突檢測
**Then** 使用資料庫交易鎖防止競爭條件（NFR8）

**技術備註：**
- 實作批次衝突檢測 SQL 查詢（單次查詢所有場次）
- API 回傳衝突清單格式：`{ conflicts: [{ date, existingName, time }] }`
- 前端顯示衝突清單 UI

---

### Story 1.5：衝突處理選項

As a 使用者,
I want 在發現衝突時選擇如何處理,
So that 我可以決定是跳過衝突場次還是取消整個週期預約。

**Acceptance Criteria:**

**Given** 系統檢測到部分場次有衝突
**When** 衝突清單顯示給使用者
**Then** 系統提供兩個選項：「跳過衝突場次，建立其他可用場次」、「取消建立」

**Given** 使用者選擇「跳過衝突場次」
**When** 使用者確認建立
**Then** 系統僅建立無衝突的場次
**And** 建立完成後顯示成功訊息，說明已建立 N 次、跳過 M 次
**And** 所有操作在單一交易中完成（NFR7）

**Given** 使用者選擇「取消建立」
**When** 使用者確認取消
**Then** 系統不建立任何預約
**And** 返回預約表單，保留已填寫的資料

**Given** 衝突場次數量等於總場次數量（全部衝突）
**When** 衝突清單顯示
**Then** 系統顯示警告「所有場次皆有衝突」
**And** 僅提供「取消建立」選項

**技術備註：**
- 前端衝突處理 UI（SweetAlert2 或自定義 Modal）
- API 支援 `skipConflicts: true` 參數
- 回傳建立結果統計

---

### Story 1.6：現有規則整合

As a 一般使用者,
I want 週期預約自動套用現有的預約規則,
So that 我不會建立違反系統規定的預約。

**Acceptance Criteria:**

**Given** 一般使用者（privilege_level = 0）建立週期預約
**When** 設定的時間超出工作日 8:00-18:00 範圍
**Then** 系統拒絕建立並顯示錯誤訊息「一般使用者僅能預約工作日 8:00-18:00」

**Given** 一般使用者建立週期預約
**When** 週期規則產生的某些場次落在週末
**Then** 系統自動排除週末場次
**And** 顯示提示訊息說明已排除的場次

**Given** 管理員（privilege_level ≥ 1）建立週期預約
**When** 設定任意時間
**Then** 系統允許建立，不受工作日時段限制

**Given** 系統有其他現有預約規則（如會議室特殊限制）
**When** 建立週期預約
**Then** 所有現有規則皆套用至週期預約的每個場次

**技術備註：**
- 複用現有 reservation.js 的規則驗證邏輯
- 在 recurringReservation.js 建立前呼叫規則檢查
- 前端顯示規則限制提示

---

### Story 1.7：週期預約建立通知

As a 使用者,
I want 在週期預約建立成功後收到 Email 通知,
So that 我有預約的確認記錄。

**Acceptance Criteria:**

**Given** 使用者成功建立週期預約
**When** 所有場次建立完成
**Then** 系統發送一封彙總 Email 給使用者
**And** Email 包含：週期預約標題、會議室名稱、週期規則說明、總場次數量、起迄日期範圍、若有跳過的衝突場次則列出跳過的日期

**Given** 使用者跳過部分衝突場次後建立
**When** Email 發送
**Then** Email 明確說明「已建立 N 次，跳過 M 次衝突場次」
**And** 列出被跳過的具體日期

**Given** Email 發送失敗
**When** 系統無法寄出通知
**Then** 預約仍然成功建立（Email 失敗不影響預約）
**And** 記錄 Email 發送失敗的 log

**技術備註：**
- 擴展 email.js 新增週期預約通知模板
- 彙整建立結果產生 Email 內容
- 非同步發送，不阻塞 API 回應

---

## Epic 2：週期預約編輯與取消

用戶可以靈活地修改或取消週期預約，選擇影響單次、此後所有、或整個週期。

### Story 2.1：Scope 選擇 UI 與單一場次編輯

As a 使用者,
I want 點擊週期預約場次時選擇編輯範圍,
So that 我可以只修改這一次的預約而不影響其他場次。

**Acceptance Criteria:**

**Given** 使用者點擊一個屬於週期系列的預約場次
**When** 選擇「編輯」操作
**Then** 系統顯示範圍選擇彈窗，包含三個選項：「僅此次」、「此次及之後」、「整個週期」

**Given** 使用者選擇「僅此次」編輯
**When** 修改預約資訊（時間、會議名稱等）並儲存
**Then** 系統僅更新該單一場次的 Reservation 記錄
**And** 其他同系列場次不受影響
**And** 該場次仍保留 series_id 關聯

**Given** 使用者編輯單一場次的時間
**When** 新時間與其他預約衝突
**Then** 系統顯示衝突警告
**And** 使用者需選擇其他時間或取消編輯

**技術備註：**
- 修改 reservation.vue 新增 scope 選擇邏輯（SweetAlert2）
- 擴展 PUT /api/reservation/:id 支援 `?scope=single` 參數
- 單一場次編輯直接更新 Reservation 記錄

---

### Story 2.2：單一場次取消

As a 使用者,
I want 取消週期預約中的單一場次,
So that 我可以處理特殊情況而不影響整個週期。

**Acceptance Criteria:**

**Given** 使用者點擊週期預約場次並選擇「取消」
**When** 選擇「僅此次」範圍
**Then** 系統顯示確認對話框

**Given** 使用者確認取消單一場次
**When** 系統處理取消請求
**Then** 該場次從 Reservation 表中刪除
**And** 其他同系列場次不受影響
**And** RecurringSeries 記錄保持不變

**Given** 週期系列只剩最後一個場次
**When** 使用者取消該場次
**Then** 系統同時刪除 RecurringSeries 記錄
**And** 清理孤立的系列資料

**技術備註：**
- 擴展 DELETE /api/reservation/:id 支援 `?scope=single` 參數
- 實作孤立系列清理邏輯

---

### Story 2.3：此後所有場次編輯（系列分裂）

As a 使用者,
I want 編輯某場次及其之後的所有場次,
So that 我可以從某個時間點開始調整週期預約，而不影響已過去的場次。

**Acceptance Criteria:**

**Given** 使用者選擇「此次及之後」編輯範圍
**When** 修改預約資訊（如時間從 14:00 改為 15:00）
**Then** 系統執行「系列分裂」：原系列保留該場次之前的所有記錄、建立新的 RecurringSeries 記錄包含新設定、該場次及之後的場次更新 series_id 指向新系列
**And** 所有操作在單一交易中完成（NFR7）

**Given** 使用者編輯的場次是系列的第一個場次
**When** 選擇「此次及之後」
**Then** 行為等同於「整個週期」編輯
**And** 不產生系列分裂，直接更新原系列

**Given** 編輯後的新時間與現有預約衝突
**When** 系統檢測到衝突
**Then** 顯示衝突的場次清單
**And** 使用者可選擇跳過衝突場次或取消編輯

**Given** 批次編輯涉及大量場次（如 30 個以上）
**When** 系統處理編輯請求
**Then** 操作在 5 秒內完成（NFR3）

**技術備註：**
- 實作系列分裂邏輯（split series）
- PUT /api/reservation/:id?scope=future
- 更新 RecurringSeries 的 RRULE（調整 UNTIL）
- 建立新 RecurringSeries 並重新關聯場次

---

### Story 2.4：此後所有場次取消

As a 使用者,
I want 取消某場次及其之後的所有場次,
So that 我可以提前結束週期預約而保留歷史記錄。

**Acceptance Criteria:**

**Given** 使用者選擇「此次及之後」取消範圍
**When** 確認取消操作
**Then** 系統刪除該場次及之後的所有 Reservation 記錄
**And** 更新原 RecurringSeries 的 RRULE，將結束日期調整為該場次的前一次
**And** 所有操作在單一交易中完成（NFR7）

**Given** 使用者取消的場次是系列的第一個場次
**When** 選擇「此次及之後」
**Then** 行為等同於「整個週期」取消
**And** 刪除所有場次和 RecurringSeries 記錄

**Given** 取消後原系列仍有場次
**When** 取消完成
**Then** 原 RecurringSeries 記錄保留
**And** RRULE 的 UNTIL 或 COUNT 更新為實際剩餘場次

**Given** 批次取消涉及大量場次
**When** 系統處理取消請求
**Then** 操作在 5 秒內完成（NFR3）

**技術備註：**
- DELETE /api/reservation/:id?scope=future
- 批次刪除 SQL 語句
- 更新 RecurringSeries RRULE

---

### Story 2.5：整個週期編輯與取消

As a 使用者,
I want 編輯或取消整個週期預約系列,
So that 我可以一次性調整或取消所有相關場次。

**Acceptance Criteria:**

**Given** 使用者選擇「整個週期」編輯範圍
**When** 修改預約資訊（如會議名稱、時間）
**Then** 系統更新 RecurringSeries 記錄
**And** 更新所有關聯的 Reservation 記錄
**And** 所有操作在單一交易中完成（NFR7）

**Given** 使用者編輯整個週期的時間
**When** 新時間與現有預約衝突
**Then** 顯示所有衝突場次清單
**And** 使用者可選擇跳過衝突場次或取消編輯

**Given** 使用者選擇「整個週期」取消範圍
**When** 確認取消操作
**Then** 系統刪除 RecurringSeries 記錄
**And** 刪除所有關聯的 Reservation 記錄
**And** 顯示取消成功訊息，說明共取消 N 個場次

**Given** 整個週期編輯/取消涉及大量場次
**When** 系統處理請求
**Then** 操作在 5 秒內完成（NFR3）

**技術備註：**
- PUT /api/reservation/:id?scope=all 或 PUT /api/recurring-reservation/:id
- DELETE /api/recurring-reservation/:id（刪除整個系列）
- 批次更新/刪除 SQL 語句
- CASCADE 刪除或手動清理關聯記錄

---

### Story 2.6：編輯與取消通知

As a 使用者,
I want 在週期預約被修改或取消時收到 Email 通知,
So that 我能及時了解預約的變更情況。

**Acceptance Criteria:**

**Given** 使用者編輯週期預約（任何範圍）
**When** 編輯操作完成
**Then** 系統發送修改通知 Email
**And** Email 包含：修改前後的差異、影響的場次範圍說明、修改後的場次清單摘要

**Given** 使用者編輯「僅此次」
**When** Email 發送
**Then** Email 說明「已修改單一場次：[日期]」

**Given** 使用者編輯「此次及之後」或「整個週期」
**When** Email 發送
**Then** Email 說明影響的場次數量和日期範圍

**Given** 使用者取消週期預約（任何範圍）
**When** 取消操作完成
**Then** 系統發送取消通知 Email
**And** Email 包含：取消的場次範圍、被取消的具體日期清單

**Given** Email 發送失敗
**When** 系統無法寄出通知
**Then** 編輯/取消操作仍然成功
**And** 記錄 Email 發送失敗的 log

**技術備註：**
- 擴展 email.js 新增修改和取消通知模板
- 根據 scope 參數產生不同的 Email 內容
- 非同步發送，不阻塞 API 回應

---

## Epic 3：日曆週期預約顯示

用戶可以在日曆上清楚查看並識別週期預約。

### Story 3.1：日曆顯示週期預約場次

As a 使用者,
I want 在日曆上查看週期預約的所有場次,
So that 我可以清楚了解我的週期預約分布情況。

**Acceptance Criteria:**

**Given** 使用者進入日曆頁面
**When** 日曆載入預約資料
**Then** 週期預約的每個場次都正確顯示在對應日期
**And** 載入時間在 2 秒內完成（NFR2）

**Given** 使用者切換日曆檢視（月/週/日）
**When** 檢視範圍改變
**Then** 該範圍內的週期預約場次正確顯示
**And** 顯示效能不受週期預約數量影響

**Given** 週期預約跨越多個月份
**When** 使用者切換到不同月份
**Then** 每個月份都正確顯示該月的場次

**Given** 使用者點擊日曆上的週期預約場次
**When** 預約詳情顯示
**Then** 顯示該場次的完整資訊
**And** 標示此預約屬於週期系列

**技術備註：**
- 修改 calendar.vue 的資料載入邏輯
- 週期預約場次使用現有 Reservation 查詢（因為採用展開儲存）
- FullCalendar 事件渲染

---

### Story 3.2：週期系列視覺識別

As a 使用者,
I want 能夠識別哪些預約屬於同一個週期系列,
So that 我可以快速區分週期預約和單次預約。

**Acceptance Criteria:**

**Given** 日曆上顯示週期預約場次
**When** 使用者查看日曆
**Then** 週期預約場次有明顯的視覺標記（如重複圖示或特殊邊框）
**And** 視覺標記與單次預約有明顯區別

**Given** 同一週期系列的多個場次顯示在日曆上
**When** 使用者查看這些場次
**Then** 同系列場次使用相同的視覺標記或顏色
**And** 使用者可識別它們屬於同一系列

**Given** 使用者 hover 或點擊週期預約場次
**When** 顯示預約資訊
**Then** 顯示週期資訊，包含：「週期預約」標籤、週期規則說明、總場次數量、此場次在系列中的位置

**Given** 日曆顯示混合的週期預約和單次預約
**When** 使用者瀏覽日曆
**Then** 兩種預約類型視覺上可明確區分

**技術備註：**
- 新增 recurring.css 定義週期預約樣式
- FullCalendar eventClassNames 或 eventContent 自訂
- 前端根據 series_id 判斷是否為週期預約
- Tooltip 或 Popover 顯示週期詳情

---

## Epic 4：管理員週期預約管理

管理員可以查看、代理操作並追蹤所有用戶的週期預約。

### Story 4.1：管理員查看所有週期預約

As a 管理員,
I want 查看所有使用者的週期預約,
So that 我可以掌握會議室的週期預約使用情況並協調衝突。

**Acceptance Criteria:**

**Given** 管理員（privilege_level ≥ 1）登入系統
**When** 進入管理後台的預約管理頁面
**Then** 可以查看所有使用者的週期預約列表
**And** 列表包含：建立者、會議室、週期規則、場次數量、起迄日期

**Given** 管理員查看週期預約列表
**When** 點擊某個週期系列
**Then** 展開顯示該系列的所有場次
**And** 每個場次顯示日期、時間、狀態

**Given** 管理員需要篩選週期預約
**When** 使用篩選功能
**Then** 可以依據以下條件篩選：建立者、會議室、日期範圍、週期模式

**Given** 管理員查看特定會議室的日曆
**When** 有多個週期預約重疊
**Then** 管理員可以清楚看到重疊情況
**And** 識別需要協調的衝突

**技術備註：**
- 擴展 conference.vue 新增週期預約檢視區塊
- GET /api/recurring-reservation（管理員專用，返回所有系列）
- 支援分頁和篩選參數

---

### Story 4.2：管理員代理編輯與取消

As a 管理員,
I want 代理編輯或取消使用者的週期預約,
So that 我可以協助使用者處理週期預約問題或協調會議室使用。

**Acceptance Criteria:**

**Given** 管理員查看某使用者的週期預約
**When** 選擇代理編輯
**Then** 系統顯示與一般使用者相同的編輯介面
**And** 管理員可選擇編輯範圍（僅此次/此次及之後/整個週期）

**Given** 管理員代理編輯週期預約
**When** 修改完成並儲存
**Then** 系統執行編輯操作（與一般使用者相同邏輯）
**And** 原使用者收到修改通知 Email
**And** Email 註明「由管理員 [管理員名稱] 代為修改」

**Given** 管理員代理取消週期預約
**When** 確認取消
**Then** 系統執行取消操作
**And** 原使用者收到取消通知 Email
**And** Email 註明「由管理員 [管理員名稱] 代為取消」

**Given** 管理員編輯/取消時間不受規則限制
**When** 管理員設定任意時間
**Then** 系統允許操作，不受工作日 8:00-18:00 限制

**技術備註：**
- 複用現有編輯/取消 API，增加管理員權限檢查
- API 記錄操作者身份（operator_id）
- Email 模板區分自行操作和代理操作

---

### Story 4.3：管理員操作日誌記錄

As a 管理員,
I want 系統記錄所有管理員對週期預約的操作,
So that 我可以追蹤和審核管理員的操作歷史。

**Acceptance Criteria:**

**Given** 管理員對週期預約執行任何操作（查看除外）
**When** 操作完成
**Then** 系統在操作日誌中記錄：操作時間、操作者、操作類型、操作範圍、目標週期系列 ID、受影響的使用者、變更前後的差異

**Given** 管理員需要查看操作日誌
**When** 進入日誌查詢頁面
**Then** 可以查看週期預約相關的所有操作記錄
**And** 支援依日期範圍、操作者、操作類型篩選

**Given** 管理員代理操作產生日誌
**When** 查看日誌詳情
**Then** 可以看到完整的操作脈絡
**And** 包含操作前後的資料快照

**技術備註：**
- 複用現有 Log 表或新增欄位支援週期預約
- 在 API 層統一記錄操作日誌
- log.js 擴展週期預約操作類型
