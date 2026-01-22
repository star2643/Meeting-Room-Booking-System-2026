# API Contracts - Backend

> 自動生成於 2026-01-18 | Meeting-Room-Booking-System API

## 概覽

| 屬性 | 值 |
|------|---|
| **Base URL** | `http://localhost:3000` |
| **認證方式** | JWT Cookie (`token`) |
| **內容類型** | `application/json` |

## API 端點總覽

| 模組 | 端點數 | 說明 |
|------|--------|------|
| Login | 1 | NCU OAuth SSO |
| Info | 1 | 使用者資訊 |
| Reservation | 6 | 預約管理 CRUD |
| Room | 2 | 會議室管理 |
| User | 5 | 使用者管理 |
| Violation | 3 | 違規紀錄 |
| Log | 1 | 操作日誌 |
| Doc | 3 | 文件管理 |

---

## 1. 認證 (Login)

### GET `/api/login/sso`
NCU OAuth SSO 登入，將 OAuth 回傳的 identifier 以 JWT sign 後放入 cookie。

---

## 2. 使用者資訊 (Info)

### GET `/api/info/chinesename`
- **認證**: 需要
- **回應**: `{"data": "王冠權"}`

---

## 3. 預約管理 (Reservation)

### GET `/api/reservation`
取得指定時間範圍內的預約。

| 參數 | 類型 | 說明 |
|------|------|------|
| `start_time` | string | 開始時間 (YYYY-MM-DD) |
| `end_time` | string | 結束時間 (YYYY-MM-DD) |

### POST `/api/reservation`
新增預約。

| 欄位 | 類型 | 必填 | 說明 |
|------|------|------|------|
| `room_id` | int | ✅ | 會議室 ID |
| `name` | string | ✅ | 會議名稱 (≤20字) |
| `start_time` | datetime | ✅ | 開始時間 |
| `end_time` | datetime | ✅ | 結束時間 |
| `show` | boolean | ✅ | 是否公開顯示 |
| `ext` | string | ✅ | 分機號碼 |

**業務規則**:
- 時間衝突檢查
- 一般使用者: 8:00-18:00、週一至週五、最長 7 天
- 成功後寄送 Email 通知

### PUT `/api/reservation`
更新預約（需驗證權限）。

### DELETE `/api/reservation`
刪除預約，刪除後寄送取消通知。

### GET `/api/reservation/show`
取得公開顯示的預約（用於看板）。

### GET `/api/reservation/inrange`
- **認證**: 需要管理員權限
- 取得指定時間的預約。

---

## 4. 會議室 (Room)

### GET `/api/room`
取得所有會議室資訊。

### POST `/api/room`
新增會議室。

| 欄位 | 類型 | 說明 |
|------|------|------|
| `name` | string | 會議室名稱 |
| `status` | boolean | 狀態 |

---

## 5. 使用者 (User)

### GET `/api/user`
取得所有使用者及違規次數。

### GET `/api/user/self`
取得當前使用者資訊。

### GET `/api/user/privilege`
取得當前使用者權限等級。

### PUT `/api/user/privilege`
更新使用者權限等級。

### PUT `/api/user/status`
更新使用者狀態。

---

## 6. 違規紀錄 (Violation)

### GET `/api/violation`
取得所有違規紀錄。

### POST `/api/violation`
新增違規紀錄。

| 欄位 | 類型 | 說明 |
|------|------|------|
| `reserve_id` | int | 預約 ID |
| `reason` | string | 原因 |
| `remark` | string | 備註 |

### DELETE `/api/violation`
刪除違規紀錄。

---

## 7. 操作日誌 (Log)

### GET `/api/log`
取得操作日誌。

| 參數 | 類型 | 說明 |
|------|------|------|
| `offset` | int | 偏移量 |
| `num` | int | 筆數 |
| `day_limit` | int | 天數範圍 |

---

## 8. 文件管理 (Doc)

### GET `/api/doc`
取得指定文件內容。

### GET `/api/doc/all`
取得所有文件名稱。

### POST `/api/doc`
新增/更新文件（最大 5MB）。

---

## 認證與權限

### 權限等級 (privilege_level)
| 等級 | 角色 | 權限 |
|------|------|------|
| 0 | 一般使用者 | 基本預約（8:00-18:00，週一至週五） |
| 1+ | 管理員 | 完整 CRUD、任意時間預約 |

### JWT 驗證
- `jwt.verifyLogin`: 驗證登入狀態
- `jwt.verifyAdmin`: 驗證管理員權限
