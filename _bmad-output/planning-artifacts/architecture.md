---
stepsCompleted: [1, 2, 3, 4, 5, 6]
inputDocuments:
  - _bmad-output/planning-artifacts/prd.md
  - docs/index.md
  - docs/api-contracts-backend.md
  - docs/data-models-backend.md
  - docs/component-inventory-frontend.md
  - docs/source-tree-analysis.md
  - docs/development-operations.md
  - docs/integration-architecture.md
  - docs/external-integrations.md
workflowType: 'architecture'
project_name: 'bookingSystem'
user_name: 'enci'
date: '2026-01-22'
---

# Architecture Decision Document

_This document builds collaboratively through step-by-step discovery. Sections are appended as we work through each architectural decision together._

## Project Context Analysis

### Requirements Overview

**Functional Requirements:**
- 27 項功能需求，涵蓋週期預約完整生命週期
- 核心能力：4 種週期模式（每週/隔週/每月固定日/每月固定週次）
- 衝突處理：建立時批次檢測，用戶選擇跳過或取消
- 編輯彈性：支援單次、此後所有、整個週期三種編輯範圍
- 管理功能：代理操作與操作日誌記錄

**Non-Functional Requirements:**
- 效能：衝突檢查 ≤3s、日曆載入 ≤2s、批次操作 ≤5s
- 資料一致性：交易處理確保原子性，防止競爭條件
- 整合：與現有 API 風格一致、繼承權限控制機制

**Scale & Complexity:**
- Primary domain: Full-stack Web Application
- Complexity level: Medium
- Estimated new architectural components: 4-6

### Technical Constraints & Dependencies

**現有系統約束：**
- 前端：Vue.js 3.4 SPA，無集中狀態管理
- 後端：Express.js 4.19，JWT Cookie 認證
- 資料庫：MariaDB，現有 7 表格
- 部署：Docker + GitHub Actions CI/CD

**必須保持相容：**
- 現有預約 API 介面風格
- privilege_level 權限機制（0=一般用戶，1+=管理員）
- Email 通知機制（Nodemailer）
- 前端元件通訊模式（props/emit）

### Cross-Cutting Concerns Identified

1. **認證與授權**：繼承現有 JWT 機制，管理員代理操作需記錄
2. **交易管理**：批次建立/編輯/刪除需原子性保證
3. **通知整合**：週期預約事件需觸發 Email 通知
4. **效能優化**：批次衝突檢查需優化查詢策略
5. **資料關聯**：週期預約與單次預約的關聯追蹤

## Starter Template Evaluation

### Primary Technology Domain

Full-stack Web Application (Brownfield Extension)

### Starter Options Considered

**N/A - 棕地專案**

此專案是現有 Vue.js 3.4 + Express.js 4.19 + MariaDB 系統的功能擴展，不適用起始模板選擇。

### Existing Technical Foundation

**已確立的技術決策：**

| 決策領域 | 現有選擇 | 新功能遵循 |
|----------|----------|------------|
| 前端框架 | Vue.js 3.4 (Options API) | ✅ |
| 建構工具 | Vite 5.3 | ✅ |
| 後端框架 | Express.js 4.19 | ✅ |
| 資料庫 | MariaDB | ✅ |
| 認證機制 | JWT Cookie | ✅ |
| API 風格 | RESTful JSON | ✅ |
| 測試框架 | Vitest + Cypress + Mocha | ✅ |

**開發模式約束：**

- 前端元件使用 Options API 風格
- 狀態管理透過 Props/Emit，不引入新狀態庫
- API 端點遵循 `/api/{resource}` 命名慣例
- 資料模型放置於 `model/` 目錄
- 前端元件放置於 `components/` 目錄

**Note:** 週期性預約功能將作為現有系統的模組擴展實作。

## Core Architectural Decisions

### Decision Priority Analysis

**Critical Decisions (Block Implementation):**
- 資料模型設計（展開儲存 + RecurringSeries 表）
- 週期規則格式（RRULE）
- API 端點設計（混合模式）

**Important Decisions (Shape Architecture):**
- 衝突檢測策略（單次批次查詢）
- 前端元件組織（獨立 recurringForm.vue）
- 日曆顯示方式（視覺標記）
- 交易策略（單一交易）

**Deferred Decisions (Post-MVP):**
- 即時日曆更新（WebSocket/SSE）
- 日曆匯出整合（Google Calendar/Outlook）

### Data Architecture

**ADR-001: 週期預約資料模型**
- **決策**: 展開儲存模式
- **內容**: 每個週期場次儲存為獨立 Reservation 記錄
- **理由**: 與現有衝突檢測邏輯相容、查詢簡單、實作複雜度低

**ADR-002: 資料表結構**
- **決策**: 新增 RecurringSeries 表
- **內容**: 獨立表儲存週期規則，Reservation 表新增 `series_id` 外鍵
- **理由**: 資料正規化、支援「此後所有」編輯時的系列分裂

**ADR-003: 週期規則格式**
- **決策**: iCalendar RRULE (RFC 5545)
- **內容**: 使用標準 RRULE 字串儲存週期規則
- **理由**: 業界標準、未來日曆整合可直接匯出

**資料表設計：**

```sql
-- 新增表：RecurringSeries
CREATE TABLE RecurringSeries (
  series_id INT PRIMARY KEY AUTO_INCREMENT,
  identifier VARCHAR(12) NOT NULL,        -- 建立者
  room_id INT NOT NULL,                    -- 會議室
  name VARCHAR(40) NOT NULL,               -- 會議名稱
  rrule VARCHAR(255) NOT NULL,             -- RRULE 規則
  start_time TIME NOT NULL,                -- 開始時間
  end_time TIME NOT NULL,                  -- 結束時間
  show BOOLEAN NOT NULL,                   -- 是否公開
  ext VARCHAR(10),                         -- 分機
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (identifier) REFERENCES User(identifier),
  FOREIGN KEY (room_id) REFERENCES Room(room_id)
);

-- 修改表：Reservation 新增欄位
ALTER TABLE Reservation ADD COLUMN series_id INT NULL;
ALTER TABLE Reservation ADD COLUMN occurrence_date DATE NULL;
ALTER TABLE Reservation ADD FOREIGN KEY (series_id)
  REFERENCES RecurringSeries(series_id) ON DELETE SET NULL;
```

### API & Communication Patterns

**ADR-004: API 端點設計**
- **決策**: 混合設計
- **內容**: 週期建立用獨立端點，單筆操作複用現有端點
- **理由**: 職責分離清晰、現有功能不受影響

**API 端點規格：**

| 方法 | 端點 | 說明 |
|------|------|------|
| POST | `/api/recurring-reservation` | 建立週期預約 |
| GET | `/api/recurring-reservation/:id` | 取得週期系列資訊 |
| PUT | `/api/recurring-reservation/:id` | 修改週期規則 |
| DELETE | `/api/recurring-reservation/:id` | 刪除整個系列 |
| PUT | `/api/reservation/:id?scope={single\|future\|all}` | 修改場次 |
| DELETE | `/api/reservation/:id?scope={single\|future\|all}` | 刪除場次 |

**ADR-005: 衝突檢測策略**
- **決策**: 單次批次查詢
- **內容**: 展開所有場次時間後，一次 SQL 查詢所有衝突
- **理由**: 效能最佳（<1s）、輕鬆滿足 NFR1（≤3s）

### Frontend Architecture

**ADR-006: 週期預約 UI 元件**
- **決策**: 獨立元件
- **內容**: 新增 `recurringForm.vue`，由 `application.vue` 引入
- **理由**: 符合 PRD 要求、程式碼模組化、易於維護

**ADR-007: 日曆顯示方式**
- **決策**: 視覺標記
- **內容**: 週期預約加上圖示或特殊樣式
- **理由**: 滿足 FR18 辨識需求、FullCalendar 原生支援

**元件結構：**
```
components/
├── application.vue      # 修改：引入 recurringForm
├── recurringForm.vue    # 新增：週期設定表單
├── calendar.vue         # 修改：週期預約樣式
└── reservation.vue      # 修改：scope 參數支援
```

### Transaction & Consistency

**ADR-008: 批次操作交易策略**
- **決策**: 單一交易
- **內容**: 整個批次操作在一個 transaction 內完成
- **理由**: 完全滿足 NFR7 原子性、52 筆記錄無效能問題

### Decision Impact Analysis

**Implementation Sequence:**
1. 資料庫 Schema 變更（RecurringSeries 表 + Reservation 欄位）
2. 後端 Model 層（recurringSeries.js + reservation.js 擴展）
3. 後端 API 層（recurring-reservation.js）
4. 前端元件（recurringForm.vue）
5. 前端整合（application.vue, calendar.vue 修改）
6. 測試（單元測試 + E2E）

**Cross-Component Dependencies:**
- RecurringSeries 表必須先建立，才能開發 API
- RRULE 解析邏輯需同時用於後端展開和前端顯示
- scope 參數需前後端同步實作

## Implementation Patterns & Consistency Rules

### Pattern Categories Defined

**Critical Conflict Points Identified:** 8 個需要一致性規範的領域

### Naming Patterns

**Database Naming Conventions（沿用現有）：**

| 項目 | 慣例 | 新增範例 |
|------|------|----------|
| 表名 | PascalCase | `RecurringSeries` |
| 欄位名 | snake_case | `series_id`, `occurrence_date` |
| 外鍵 | {table}_id | `series_id` |

**API Naming Conventions（沿用現有）：**

| 項目 | 慣例 | 新增範例 |
|------|------|----------|
| 端點 | lowercase, kebab-case | `/api/recurring-reservation` |
| 查詢參數 | snake_case | `start_time`, `scope` |
| 路徑參數 | :id 格式 | `/api/recurring-reservation/:id` |

**Code Naming Conventions（沿用現有）：**

| 項目 | 慣例 | 新增範例 |
|------|------|----------|
| 後端 Model | camelCase.js | `recurringSeries.js` |
| 後端 API | camelCase.js | `recurringReservation.js` |
| 前端元件 | camelCase.vue | `recurringForm.vue` |
| 函式名 | camelCase | `createRecurringSeries`, `expandOccurrences` |
| 變數名 | camelCase | `seriesId`, `occurrenceDate` |

### Format Patterns

**API Response Format:**
- 成功：直接回傳資料（沿用現有）
- 錯誤：欄位擴展模式

```javascript
// 成功回應
{ reserve_id: 1, series_id: 5, ... }

// 一般錯誤
{ error: '會議室不存在' }

// 衝突錯誤（擴展欄位）
{
  error: '部分場次與現有預約衝突',
  conflicts: [
    { date: '2026-02-05', existingName: '系會' },
    { date: '2026-02-19', existingName: '讀書會' }
  ]
}
```

**Date/Time Format:**
- API 傳輸：`YYYY-MM-DD HH:mm:ss`（沿用現有）
- RRULE 內部格式由後端處理，不影響 API 介面

### Process Patterns

**Loading State Management:**
細分狀態模式，提供清晰的使用者回饋：

```javascript
// recurringForm.vue
data() {
  return {
    isExpanding: false,        // 展開場次預覽中
    isCheckingConflicts: false, // 檢測衝突中
    isSubmitting: false         // 建立預約中
  }
}
```

**Edit Scope Handling:**
點擊後彈窗模式，使用 SweetAlert2：

```javascript
// reservation.vue - 編輯/刪除週期預約場次
async handleEditRecurring(reservation) {
  if (!reservation.series_id) return this.handleEdit(reservation);

  const { value: scope } = await Swal.fire({
    title: '修改範圍',
    input: 'radio',
    inputOptions: {
      single: '僅此次',
      future: '此次及之後',
      all: '整個週期'
    },
    inputValidator: (value) => !value && '請選擇修改範圍'
  });

  if (scope) {
    // 依據 scope 呼叫對應 API
  }
}
```

### Enforcement Guidelines

**All AI Agents MUST:**

1. **命名一致性**：嚴格遵循現有命名慣例（PascalCase 表名、snake_case 欄位、camelCase 程式碼）
2. **API 相容性**：新端點遵循 `/api/{resource}` 格式，回應格式與現有 API 一致
3. **錯誤處理**：使用 `{ error: 'message' }` 格式，需要額外資訊時擴展欄位
4. **元件風格**：使用 Options API，狀態透過 Props/Emit 傳遞
5. **交易處理**：批次操作必須使用 transaction 確保原子性

**Anti-Patterns to Avoid:**

| 避免 | 正確做法 |
|------|----------|
| `recurring_series` (表名) | `RecurringSeries` |
| `seriesId` (欄位名) | `series_id` |
| `/api/recurringSeries` | `/api/recurring-reservation` |
| `{ success: false, error: {} }` | `{ error: 'message' }` |
| Composition API | Options API |
| Pinia/Vuex | Props/Emit |

## Project Structure & Boundaries

### Complete Project Directory Structure

```
Meeting-Room-Booking-System/
├── .github/
│   └── workflows/
│       └── no_vpn_pipeline.yaml          # CI/CD（現有）
├── doc/                                   # 原始文件（現有）
├── src/
│   ├── docker-compose.yml                 # Docker 編排（現有）
│   ├── Dockerfile                         # 容器建構（現有）
│   │
│   ├── api/                               # [後端] Express.js
│   │   ├── index.js                       # 應用入口（現有）
│   │   ├── package.json                   # 依賴管理（現有）
│   │   │
│   │   ├── api/                           # API 路由處理器
│   │   │   ├── doc.js                     # （現有）
│   │   │   ├── info.js                    # （現有）
│   │   │   ├── log.js                     # （現有）
│   │   │   ├── login.js                   # （現有）
│   │   │   ├── reservation.js             # （修改）scope 參數支援
│   │   │   ├── recurringReservation.js    # 【新增】週期預約 API
│   │   │   ├── room.js                    # （現有）
│   │   │   ├── user.js                    # （現有）
│   │   │   └── violation.js               # （現有）
│   │   │
│   │   ├── model/                         # 資料模型
│   │   │   ├── conn.js                    # 連線池（現有）
│   │   │   ├── schema.sql                 # Schema（修改）新增表/欄位
│   │   │   ├── doc.js                     # （現有）
│   │   │   ├── info.js                    # （現有）
│   │   │   ├── log.js                     # （現有）
│   │   │   ├── operator.js                # （現有）
│   │   │   ├── reservation.js             # （修改）series 相關查詢
│   │   │   ├── recurringSeries.js         # 【新增】週期系列模型
│   │   │   ├── room.js                    # （現有）
│   │   │   ├── user.js                    # （現有）
│   │   │   └── violation.js               # （現有）
│   │   │
│   │   ├── utilities/                     # 工具函式
│   │   │   ├── config.js                  # （現有）
│   │   │   ├── email.js                   # （修改）週期預約通知
│   │   │   ├── info.js                    # （現有）
│   │   │   ├── jwt.js                     # （現有）
│   │   │   ├── main.js                    # （現有）
│   │   │   ├── oauth.js                   # （現有）
│   │   │   ├── rrule.js                   # 【新增】RRULE 解析工具
│   │   │   └── test.js                    # （現有）
│   │   │
│   │   └── testing/                       # 後端測試
│   │       ├── main.js                    # （現有）
│   │       ├── reservation.js             # （修改）新增測試案例
│   │       ├── recurringReservation.js    # 【新增】週期預約測試
│   │       └── violation.js               # （現有）
│   │
│   └── MRBS-frontend/                     # [前端] Vue.js
│       ├── package.json                   # （現有）
│       ├── vite.config.js                 # （現有）
│       ├── vitest.config.js               # （現有）
│       ├── cypress.config.js              # （現有）
│       │
│       ├── src/
│       │   ├── main.js                    # （現有）
│       │   ├── App.vue                    # （現有）
│       │   ├── config.js                  # （現有）
│       │   │
│       │   ├── router/
│       │   │   └── index.js               # （現有）
│       │   │
│       │   ├── middleware/
│       │   │   └── auth.vue               # （現有）
│       │   │
│       │   ├── components/
│       │   │   ├── lobby.vue              # （現有）
│       │   │   ├── calendar.vue           # （修改）週期預約樣式
│       │   │   ├── application.vue        # （修改）引入 recurringForm
│       │   │   ├── recurringForm.vue      # 【新增】週期設定表單
│       │   │   ├── conference.vue         # （修改）管理員週期操作
│       │   │   ├── reservation.vue        # （修改）scope 選擇邏輯
│       │   │   ├── header.vue             # （現有）
│       │   │   └── ...                    # （其他現有元件）
│       │   │
│       │   ├── assets/
│       │   │   └── recurring.css          # 【新增】週期預約樣式
│       │   │
│       │   └── views/                     # （現有）
│       │
│       ├── cypress/
│       │   └── e2e/
│       │       ├── lobby.cy.js            # （修改）週期預約測試
│       │       └── recurring.cy.js        # 【新增】週期預約 E2E
│       │
│       └── __test__/
│           ├── RecurringForm.test.js      # 【新增】週期表單測試
│           └── ...                        # （其他現有測試）
│
└── docs/                                  # 技術文件（現有）
```

### Architectural Boundaries

**API Boundaries:**

| 邊界 | 端點 | 職責 |
|------|------|------|
| 週期管理 | `/api/recurring-reservation/*` | 週期系列 CRUD |
| 單筆操作 | `/api/reservation/*` | 個別場次操作（含 scope） |
| 現有功能 | `/api/room`, `/api/user`, ... | 不變 |

**Data Boundaries:**

```
RecurringSeries (週期規則)
       │
       │ 1:N
       ▼
Reservation (個別場次)
       │
       │ N:1
       ▼
Room / User (現有關聯)
```

**Component Boundaries:**

```
application.vue (預約表單)
       │
       │ 引入
       ▼
recurringForm.vue (週期設定)
       │
       │ emit
       ▼
application.vue → API 呼叫
```

### Requirements to Structure Mapping

**Feature Mapping:**

| 需求類別 | FR 編號 | 對應檔案 |
|----------|---------|----------|
| 週期建立 | FR1-FR6 | `recurringReservation.js`, `recurringSeries.js`, `recurringForm.vue`, `rrule.js` |
| 衝突管理 | FR7-FR10 | `recurringReservation.js`, `reservation.js`, `recurringForm.vue` |
| 編輯取消 | FR11-FR16 | `reservation.js` (API), `reservation.vue` (UI) |
| 日曆顯示 | FR17-FR18 | `calendar.vue`, `recurring.css` |
| 管理功能 | FR19-FR22 | `conference.vue`, `recurringReservation.js` |
| 規則整合 | FR23-FR24 | `recurringReservation.js` |
| 通知 | FR25-FR27 | `email.js` |

### New Files Summary

| 層級 | 檔案 | 用途 |
|------|------|------|
| **後端 API** | `api/recurringReservation.js` | 週期預約 REST 端點 |
| **後端 Model** | `model/recurringSeries.js` | 週期系列資料操作 |
| **後端工具** | `utilities/rrule.js` | RRULE 解析與展開 |
| **後端測試** | `testing/recurringReservation.js` | 週期預約單元測試 |
| **前端元件** | `components/recurringForm.vue` | 週期設定 UI |
| **前端樣式** | `assets/recurring.css` | 週期預約視覺樣式 |
| **前端測試** | `__test__/RecurringForm.test.js` | 元件單元測試 |
| **E2E 測試** | `cypress/e2e/recurring.cy.js` | 端對端測試 |

### Modified Files Summary

| 層級 | 檔案 | 修改內容 |
|------|------|----------|
| **資料庫** | `model/schema.sql` | 新增 RecurringSeries 表、Reservation 欄位 |
| **後端 API** | `api/reservation.js` | 新增 scope 參數處理 |
| **後端 Model** | `model/reservation.js` | 新增 series 相關查詢 |
| **後端工具** | `utilities/email.js` | 新增週期預約通知模板 |
| **前端元件** | `components/application.vue` | 引入 recurringForm |
| **前端元件** | `components/calendar.vue` | 週期預約樣式標記 |
| **前端元件** | `components/reservation.vue` | scope 選擇彈窗 |
| **前端元件** | `components/conference.vue` | 管理員週期操作 |
