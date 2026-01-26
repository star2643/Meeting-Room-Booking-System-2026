# Story 1.1: 週期預約資料架構與基礎建立

Status: complete

## Story

As a 使用者,
I want 建立每週重複的會議室預約,
so that 我可以一次設定整學期的固定會議時間，省去每週重複操作。

## Acceptance Criteria

### AC1: 週期預約表單顯示

**Given** 使用者已登入系統並進入預約頁面
**When** 使用者選擇會議室、設定時間、勾選「週期預約」並選擇「每週」模式
**Then** 系統顯示週期設定表單，包含結束條件選項
**And** 使用者可以預覽將產生的所有場次日期

### AC2: 週期預約建立（資料層）

**Given** 使用者填寫完整的週期預約資訊（每週模式）
**When** 使用者點擊「建立預約」
**Then** 系統在 RecurringSeries 表建立週期規則記錄
**And** 系統在 Reservation 表建立所有展開的場次記錄，每筆記錄包含 series_id 關聯
**And** 所有場次在單一資料庫交易中完成（NFR7）

### AC3: RRULE 規則處理

**Given** 後端收到週期預約建立請求
**When** 系統處理 RRULE 規則展開場次
**Then** 使用 iCalendar RRULE (RFC 5545) 標準格式儲存規則
**And** 正確計算每週重複的所有日期

## Tasks / Subtasks

- [x] Task 1: 資料庫 Schema 變更 (AC: #2, #3)
  - [x] 1.1 建立 RecurringSeries 資料表
  - [x] 1.2 修改 Reservation 表新增 series_id, occurrence_date 欄位
  - [x] 1.3 新增外鍵約束

- [x] Task 2: 後端 Model 層 (AC: #2, #3)
  - [x] 2.1 建立 recurringSeries.js Model（CRUD 操作）
  - [x] 2.2 建立 rrule.js 工具（每週模式解析與展開）
  - [x] 2.3 擴展 reservation.js Model（series 相關查詢）

- [x] Task 3: 後端 API 層 (AC: #2)
  - [x] 3.1 建立 recurringReservation.js API（POST 端點）
  - [x] 3.2 實作建立週期預約的交易邏輯
  - [x] 3.3 整合 JWT 驗證中介層

- [x] Task 4: 前端元件 (AC: #1)
  - [x] 4.1 建立 recurringForm.vue 元件（基礎版）
  - [x] 4.2 修改 application.vue 引入週期設定
  - [x] 4.3 實作場次預覽功能

- [x] Task 5: 測試 (AC: all)
  - [x] 5.1 後端單元測試（recurringReservation.js）
  - [x] 5.2 前端元件測試（RecurringForm.test.js）

## Dev Notes

### 關鍵架構決策

| 決策 | 內容 | 參考 |
|------|------|------|
| ADR-001 | 展開儲存模式：每個週期場次儲存為獨立 Reservation 記錄 | [architecture.md#ADR-001] |
| ADR-002 | RecurringSeries 表儲存週期規則，Reservation 新增 series_id | [architecture.md#ADR-002] |
| ADR-003 | 使用 iCalendar RRULE (RFC 5545) 標準格式 | [architecture.md#ADR-003] |
| ADR-008 | 批次操作使用單一資料庫交易 | [architecture.md#ADR-008] |

### 資料庫 Schema（必須完全遵循）

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

### API 端點規格

| 方法 | 端點 | 說明 |
|------|------|------|
| POST | `/api/recurring-reservation` | 建立週期預約 |

**Request Body:**
```json
{
  "room_id": 1,
  "name": "研究室週會",
  "start_time": "14:00",
  "end_time": "16:00",
  "rrule": "FREQ=WEEKLY;BYDAY=WE;UNTIL=20260630",
  "show": true,
  "ext": "1234"
}
```

**Success Response:**
```json
{
  "series_id": 5,
  "created_count": 18,
  "occurrences": ["2026-02-05", "2026-02-12", ...]
}
```

**Error Response:**
```json
{
  "error": "會議室不存在"
}
```

### RRULE 格式（本故事僅支援每週模式）

```
RRULE:FREQ=WEEKLY;BYDAY=WE;UNTIL=20260630
```

| 參數 | 說明 | 範例 |
|------|------|------|
| FREQ | 頻率（本故事固定為 WEEKLY） | `FREQ=WEEKLY` |
| BYDAY | 星期幾（MO/TU/WE/TH/FR/SA/SU） | `BYDAY=WE` |
| UNTIL | 結束日期（YYYYMMDD 格式） | `UNTIL=20260630` |
| COUNT | 重複次數（與 UNTIL 二擇一） | `COUNT=10` |

### 推薦函式庫

**npm 套件：** `rrule` (v2.8.1)
- 授權：BSD-3-Clause
- 用途：解析與生成 RFC 5545 RRULE
- 安裝：`npm install rrule`

**使用範例：**
```javascript
const { RRule } = require('rrule');

// 從 RRULE 字串解析並展開日期
const rule = RRule.fromString('RRULE:FREQ=WEEKLY;BYDAY=WE;UNTIL=20260630');
const dates = rule.all();

// 建立新 RRULE
const newRule = new RRule({
  freq: RRule.WEEKLY,
  byweekday: [RRule.WE],
  until: new Date(2026, 5, 30)
});
const rruleString = newRule.toString();
```

### Project Structure Notes

**新增檔案：**

| 檔案路徑 | 用途 |
|----------|------|
| `src/api/model/recurringSeries.js` | RecurringSeries 資料模型 |
| `src/api/utilities/rrule.js` | RRULE 解析與展開工具 |
| `src/api/api/recurringReservation.js` | 週期預約 REST API |
| `src/api/testing/recurringReservation.js` | 後端單元測試 |
| `src/MRBS-frontend/src/components/recurringForm.vue` | 週期設定 UI |
| `src/MRBS-frontend/__test__/RecurringForm.test.js` | 前端元件測試 |

**修改檔案：**

| 檔案路徑 | 修改內容 |
|----------|----------|
| `src/api/model/schema.sql` | 新增 RecurringSeries 表、Reservation 欄位 |
| `src/api/model/reservation.js` | 新增 series 相關查詢方法 |
| `src/api/index.js` | 註冊新 API 路由 |
| `src/MRBS-frontend/src/components/application.vue` | 引入 recurringForm 元件 |

### 實作順序建議

1. **資料庫 Schema** → 先建立表結構，後續開發才能進行
2. **rrule.js 工具** → RRULE 解析是核心功能
3. **recurringSeries.js Model** → 資料層 CRUD
4. **recurringReservation.js API** → 後端 API 端點
5. **前端元件** → UI 實作
6. **測試** → 驗證功能正確性

### 命名慣例（必須遵循）

| 項目 | 慣例 | 範例 |
|------|------|------|
| 資料表 | PascalCase | `RecurringSeries` |
| 欄位名 | snake_case | `series_id`, `occurrence_date` |
| 後端 Model | camelCase.js | `recurringSeries.js` |
| 後端 API | camelCase.js | `recurringReservation.js` |
| 前端元件 | camelCase.vue | `recurringForm.vue` |
| 函式名 | camelCase | `createRecurringSeries` |

### 反模式清單（禁止）

| 禁止 | 正確做法 |
|------|----------|
| `recurring_series` (表名) | `RecurringSeries` |
| `seriesId` (欄位名) | `series_id` |
| `/api/recurringSeries` (端點) | `/api/recurring-reservation` |
| Composition API | Options API |
| Vuex/Pinia | Props/Emit |
| 多次個別 INSERT | 單一交易批次 INSERT |

### 技術約束

- **前端**：Vue.js 3.4 (Options API)，無集中狀態管理
- **後端**：Express.js 4.19，CommonJS (`require`/`module.exports`)
- **資料庫**：MariaDB，使用 `model/conn.js` 連線池
- **認證**：JWT Cookie，使用 `jwt.verifyLogin` 中介層

### 效能要求

- **NFR7**：批次建立必須使用資料庫交易，確保原子性
- 展開場次計算應在 1 秒內完成

### References

- [Source: _bmad-output/planning-artifacts/architecture.md#Core-Architectural-Decisions]
- [Source: _bmad-output/planning-artifacts/epics.md#Story-1.1]
- [Source: _bmad-output/planning-artifacts/prd.md#FR1-FR6]
- [Source: _bmad-output/project-context.md#Framework-Specific-Rules]
- [External: npm rrule package](https://www.npmjs.com/package/rrule)
- [External: RFC 5545 iCalendar](https://datatracker.ietf.org/doc/html/rfc5545)

## Dev Agent Record

### Agent Model Used

claude-opus-4-5-20251101

### Debug Log References

N/A

### Completion Notes List

- All 5 tasks completed successfully
- Database schema updated with RecurringSeries table and Reservation modifications
- Backend API endpoint `/api/recurring-reservation` implemented with batch transaction support (NFR7)
- Frontend recurring form component integrated with occurrence preview
- Unit tests created for both backend and frontend

### File List

**新增檔案：**
- `Meeting-Room-Booking-System/src/api/model/recurringSeries.js`
- `Meeting-Room-Booking-System/src/api/utilities/rrule.js`
- `Meeting-Room-Booking-System/src/api/api/recurringReservation.js`
- `Meeting-Room-Booking-System/src/api/testing/recurringReservation.js`
- `Meeting-Room-Booking-System/src/MRBS-frontend/src/components/recurringForm.vue`
- `Meeting-Room-Booking-System/src/MRBS-frontend/__test__/RecurringForm.test.js`

**修改檔案：**
- `Meeting-Room-Booking-System/src/api/model/schema.sql`
- `Meeting-Room-Booking-System/src/api/model/reservation.js`
- `Meeting-Room-Booking-System/src/api/index.js`
- `Meeting-Room-Booking-System/src/MRBS-frontend/src/components/application.vue`
