# External Integrations - bookingSystem

> 自動生成於 2026-01-18 | Meeting-Room-Booking-System 外部整合

## 整合總覽

| 服務 | 類型 | 用途 | 必要性 |
|------|------|------|--------|
| **NCU OAuth** | 認證 | 單一登入 (SSO) | 必要 |
| **NCU Portal** | 資料 | 使用者資訊查詢 | 必要 |
| **SMTP (Outlook)** | 通知 | 郵件通知 | 必要 |
| **Docker Hub** | 部署 | 映像倉庫 | CI/CD |
| **GitHub Actions** | CI/CD | 自動化流程 | CI/CD |

---

## 1. NCU OAuth 2.0 整合

### 概述

本系統使用中央大學 OAuth 服務進行身分驗證，實現單一登入 (SSO)。

### 整合流程

```
┌──────────┐    ┌────────────┐    ┌────────────┐
│  使用者   │───▶│  本系統    │───▶│ NCU OAuth  │
│          │    │  /api/login │    │  Server    │
└──────────┘    └────────────┘    └────────────┘
     │                                   │
     │   1. 點擊登入                      │
     │──────────────────────────────────▶│
     │                                   │
     │   2. NCU 登入頁面                  │
     │◀──────────────────────────────────│
     │                                   │
     │   3. 輸入帳密                      │
     │──────────────────────────────────▶│
     │                                   │
     │   4. reCAPTCHA 驗證               │
     │──────────────────────────────────▶│
     │                                   │
     │   5. 回傳 Auth Code               │
     │◀──────────────────────────────────│
     │                                   │
     │   6. 設定 JWT Cookie              │
     │◀──────────────────────────────────│
```

### 配置要求

| 設定項 | 環境變數 | 說明 |
|--------|----------|------|
| Client ID | `oauth_client_id` | OAuth 應用程式 ID |
| Client Secret | `oauth_client_secret` | OAuth 應用程式密鑰 |
| Callback URL | 系統自動 | `{host}/api/login/sso` |

### 程式碼位置

- **後端處理**: `api/api/login.js`
- **OAuth 工具**: `api/utilities/oauth.js`
- **JWT 簽發**: `api/utilities/jwt.js`

---

## 2. NCU Portal API

### 概述

系統透過 NCU Portal API 取得使用者基本資訊。

### 使用情境

- 新使用者首次登入時自動建立帳號
- 取得使用者中文姓名、單位等資訊

### 程式碼位置

- `api/utilities/info.js`

### 資料欄位

| 欄位 | 說明 |
|------|------|
| `identifier` | 學號/員工編號 |
| `chinesename` | 中文姓名 |
| `email` | 電子郵件 |
| `unit` | 所屬單位 |
| `mobilePhone` | 手機號碼 |

---

## 3. SMTP 郵件服務

### 概述

系統使用 Nodemailer 透過 Outlook SMTP 發送通知郵件。

### 觸發情境

| 事件 | 收件者 | 內容 |
|------|--------|------|
| 預約成功 | 申請者 | 預約確認通知 |
| 預約取消 | 申請者 | 取消通知 |

### 配置要求

| 設定項 | 環境變數 | 說明 |
|--------|----------|------|
| 寄件帳號 | `sender_mail_account` | Outlook 帳號 |
| 寄件密碼 | `sender_mail_password` | 應用程式密碼 |
| SMTP 主機 | 程式碼內建 | `smtp.office365.com` |
| SMTP 埠 | 程式碼內建 | `587` (TLS) |

### 程式碼位置

- `api/utilities/email.js`

### 使用範例

```javascript
// 預約成功通知
Email.sendUser(
  identifier,
  Email.subject.succeessful_reservation,
  Email.text.succeessful_reservation(identifier, start_time, end_time, room_id)
);
```

---

## 4. CDN 依賴

### 前端 CDN 載入

```javascript
// lobby.vue
const cdn = [
  'https://cdn.jsdelivr.net/npm/fullcalendar@6.1.11/index.global.min.js',
  'https://cdn.jsdelivr.net/npm/axios@1.6.7/dist/axios.min.js',
  'https://cdnjs.cloudflare.com/ajax/libs/dompurify/2.4.0/purify.min.js',
  'https://cdn.jsdelivr.net/npm/sweetalert2@11.7.3/dist/sweetalert2.all.min.js'
];
```

### CDN 服務清單

| 服務 | 用途 | 網域 |
|------|------|------|
| jsDelivr | FullCalendar, Axios, SweetAlert2 | cdn.jsdelivr.net |
| cdnjs | DOMPurify | cdnjs.cloudflare.com |
| unpkg | Quill Editor | unpkg.com |

---

## 5. Docker Hub

### 概述

CI/CD 流程將建構的映像推送至 Docker Hub。

### 配置要求

| 設定項 | GitHub Secret | 說明 |
|--------|---------------|------|
| 帳號 | `DOCKER_USERNAME` | Docker Hub 使用者 |
| 密碼 | `DOCKER_PASSWORD` | Docker Hub 密碼 |
| 映像名稱 | `IMAGE_NAME` (Variable) | 如 `tommygood/ncu-mrbs` |
| Registry | `REGISTRY` (Variable) | 如 `docker.io` |

### 映像位置

- [Docker Hub Repository](https://hub.docker.com/repository/docker/tommygood/ncu-mrbs/)

---

## 6. GitHub Actions

### 工作流程配置

- **檔案**: `.github/workflows/no_vpn_pipeline.yaml`
- **觸發**: PR 合併至 main 分支

### 必要 Secrets

| Secret | 說明 |
|--------|------|
| `DOCKER_USERNAME` | Docker Hub 帳號 |
| `DOCKER_PASSWORD` | Docker Hub 密碼 |
| `SSH_USER` | 部署主機 SSH 使用者 |
| `SSH_HOST` | 部署主機位址 |
| `SSH_PRIVATE_KEY` | SSH 私鑰 |
| `WORK_DIR` | 部署目錄 |

---

## 整合健康檢查

### 檢查清單

| 整合 | 驗證方式 | 預期結果 |
|------|----------|----------|
| NCU OAuth | 點擊登入按鈕 | 重導向至 NCU 登入頁 |
| NCU Portal | 登入後檢查使用者資訊 | 顯示中文姓名 |
| SMTP | 新增預約 | 收到確認郵件 |
| Database | 存取任意 API | 正常回應 |

### 故障排除

| 問題 | 可能原因 | 解決方案 |
|------|----------|----------|
| OAuth 失敗 | Client ID/Secret 錯誤 | 檢查環境變數 |
| 郵件未發送 | SMTP 憑證問題 | 重新產生應用程式密碼 |
| CDN 載入失敗 | CSP 阻擋 | 檢查 Content-Security-Policy |

