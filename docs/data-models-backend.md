# Data Models - Backend

> 自動生成於 2026-01-18 | Meeting-Room-Booking-System 資料庫架構

## 資料庫概覽

| 屬性 | 值 |
|------|---|
| **資料庫** | MariaDB / MySQL |
| **字元集** | utf8mb4 |
| **Collation** | utf8mb4_unicode_ci |
| **表格數** | 7 |

## 實體關係圖 (ERD)

```
┌─────────────┐     ┌─────────────┐
│  Operation  │     │    Room     │
│─────────────│     │─────────────│
│ operation_id│     │ room_id     │
│ operation_  │     │ room_name   │
│   name      │     │ room_status │
└──────┬──────┘     └──────┬──────┘
       │                   │
       │                   │
       ▼                   ▼
┌─────────────┐     ┌─────────────┐
│    User     │◄────│ Reservation │
│─────────────│     │─────────────│
│ identifier  │     │ reserve_id  │
│ chinesename │     │ identifier  │──► User
│ email       │     │ room_id     │──► Room
│ mobilePhone │     │ name        │
│ unit        │     │ start_time  │
│ status      │     │ end_time    │
│ privilege_  │     │ show        │
│   level     │     │ ext         │
└──────┬──────┘     │ status      │
       │            └─────────────┘
       │
       ├──────────────────┐
       ▼                  ▼
┌─────────────┐    ┌─────────────┐
│     Log     │    │  Violation  │
│─────────────│    │─────────────│
│ log_id      │    │ violation_id│
│ identifier  │    │ identifier  │──► User
│ datetime    │    │ datetime    │
│ IP          │    │ reason      │
│ operation_id│    │ remark      │
└─────────────┘    │ status      │
                   └─────────────┘

┌─────────────┐
│     Doc     │ (獨立)
│─────────────│
│ name        │
│ blocks      │
│ id_content  │
└─────────────┘
```

---

## 表格詳細定義

### 1. User（使用者）

| 欄位 | 類型 | 約束 | 說明 |
|------|------|------|------|
| `identifier` | varchar(12) | PK | 使用者識別碼（NCU 學號/員工編號）|
| `chinesename` | varchar(20) | | 中文姓名 |
| `email` | varchar(35) | | 電子郵件 |
| `mobilePhone` | varchar(12) | | 手機號碼 |
| `unit` | varchar(20) | | 所屬單位 |
| `status` | tinyint(1) | DEFAULT 1 | 帳號狀態 |
| `privilege_level` | tinyint(1) | DEFAULT 0 | 權限等級 (0=一般, 1+=管理員) |

### 2. Room（會議室）

| 欄位 | 類型 | 約束 | 說明 |
|------|------|------|------|
| `room_id` | int | PK, AUTO_INCREMENT | 會議室 ID |
| `room_name` | varchar(20) | NOT NULL | 會議室名稱 |
| `room_status` | boolean | NOT NULL | 是否啟用 |

### 3. Reservation（預約）

| 欄位 | 類型 | 約束 | 說明 |
|------|------|------|------|
| `reserve_id` | int | PK, AUTO_INCREMENT | 預約 ID |
| `identifier` | varchar(12) | FK → User | 申請人 |
| `room_id` | int | FK → Room | 會議室 |
| `name` | varchar(40) | NOT NULL | 會議名稱 |
| `start_time` | datetime | NOT NULL | 開始時間 |
| `end_time` | datetime | NOT NULL | 結束時間 |
| `show` | boolean | NOT NULL | 是否公開顯示 |
| `ext` | varchar(10) | | 分機號碼 |
| `status` | tinyint(1) | DEFAULT 0 | 預約狀態 |

### 4. Operation（操作類型）

| 欄位 | 類型 | 約束 | 說明 |
|------|------|------|------|
| `operation_id` | int | PK, AUTO_INCREMENT | 操作 ID |
| `operation_name` | varchar(40) | NOT NULL | 操作名稱 |

**預設資料**: `login successfully`, `login failed`, `logout`

### 5. Log（操作日誌）

| 欄位 | 類型 | 約束 | 說明 |
|------|------|------|------|
| `log_id` | int | PK, AUTO_INCREMENT | 日誌 ID |
| `identifier` | varchar(12) | FK → User | 操作者 |
| `datetime` | datetime | DEFAULT CURRENT_TIMESTAMP | 時間 |
| `IP` | varchar(15) | NOT NULL | IP 位址 |
| `operation_id` | int | NOT NULL | 操作類型 |

**索引**: `identifier_time` on (`identifier`, `datetime`)

### 6. Violation（違規紀錄）

| 欄位 | 類型 | 約束 | 說明 |
|------|------|------|------|
| `violation_id` | int | PK, AUTO_INCREMENT | 違規 ID |
| `identifier` | varchar(12) | FK → User | 違規者 |
| `datetime` | datetime | DEFAULT CURRENT_TIMESTAMP | 時間 |
| `reason` | varchar(200) | NOT NULL | 原因 |
| `remark` | varchar(200) | | 備註 |
| `status` | tinyint(1) | DEFAULT 0 | 狀態 |

### 7. Doc（文件）

| 欄位 | 類型 | 約束 | 說明 |
|------|------|------|------|
| `name` | varchar(40) | PK | 文件名稱 |
| `blocks` | longtext | NOT NULL | 內容區塊 (JSON) |
| `id_content` | varchar(200) | | ID 對應內容 |

---

## 關聯關係

| 關係 | 類型 | 說明 |
|------|------|------|
| User → Reservation | 1:N | 一個使用者可有多筆預約 |
| User → Log | 1:N | 一個使用者可有多筆日誌 |
| User → Violation | 1:N | 一個使用者可有多筆違規 |
| Room → Reservation | 1:N | 一個會議室可有多筆預約 |

---

## 業務規則

### 預約衝突檢查
```sql
-- 檢查時間重疊
SELECT * FROM Reservation
WHERE room_id = ?
AND status = 0
AND ((start_time <= ? AND end_time > ?)
  OR (start_time < ? AND end_time >= ?)
  OR (start_time >= ? AND end_time <= ?))
```

### 權限控制
- `privilege_level = 0`: 一般使用者，限制預約時段
- `privilege_level >= 1`: 管理員，完整權限
