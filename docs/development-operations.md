# Development & Operations Guide - bookingSystem

> 自動生成於 2026-01-18 | Meeting-Room-Booking-System 開發與運維指南

## 快速啟動

### 環境需求

| 項目 | 版本 | 說明 |
|------|------|------|
| **Node.js** | 20.10.0+ | 後端與前端建構 |
| **npm** | 10+ | 套件管理 |
| **Docker** | 20+ | 容器化部署 |
| **MariaDB** | 10+ | 資料庫 |

### 本地開發

```bash
# 1. 安裝後端依賴
cd Meeting-Room-Booking-System/src/api
npm install

# 2. 啟動後端服務
npm start                     # 監聽 Port 3000

# 3. 安裝前端依賴 (新終端機)
cd Meeting-Room-Booking-System/src/MRBS-frontend
npm install

# 4. 啟動前端開發伺服器
npm run dev                   # Vite 開發伺服器
```

---

## 建構指令

### Frontend (Vue.js)

| 指令 | 說明 |
|------|------|
| `npm run dev` | 啟動 Vite 開發伺服器 (HMR) |
| `npm run build` | 建構生產版本至 `dist/` |
| `npm run preview` | 預覽生產版本 |

### Backend (Express.js)

| 指令 | 說明 |
|------|------|
| `npm start` | 啟動 Express 伺服器 (Port 3000) |
| `npm test` | 執行後端測試 (Mocha) |

---

## 測試架構

### 測試類型總覽

| 類型 | 框架 | 位置 | 指令 |
|------|------|------|------|
| **Frontend Unit** | Vitest | `MRBS-frontend/__test__/` | `npm run test:unit` |
| **Backend Unit** | Mocha + Chai | `api/testing/` | `npm test` |
| **E2E** | Cypress | `MRBS-frontend/cypress/e2e/` | `npm run test:e2e` |

### Frontend 單元測試

```bash
cd src/MRBS-frontend
npm run test:unit
```

測試檔案：
- `RootMenu.test.js` - 管理者左側選單
- `Header.test.js` - 頁面標頭
- `Preview.test.js` - 看板播放邏輯
- `Welcome.test.js` - 歡迎頁面

### Backend 單元測試

```bash
cd src/api
npm test
```

測試檔案：
- `reservation.js` - 預約 CRUD 與重複檢查
- `violation.js` - 違規紀錄管理

### E2E 測試

```bash
cd src/MRBS-frontend

# 執行 E2E 測試 (無頭模式)
npm run test:e2e

# 開發模式 (開啟 Cypress UI)
npm run test:e2e:dev
```

測試情境：
- 登入流程 (NCU Portal + reCAPTCHA)
- 會議室預約 CRUD
- 違規事項新增
- 文字編輯器操作

---

## 環境設定

### 環境變數

| 變數 | 說明 | 範例 |
|------|------|------|
| `db_host` | 資料庫主機 | `mysql` 或 `localhost` |
| `db_port` | 資料庫埠 | `3306` |
| `db_name` | 資料庫名稱 | `2fconference` |
| `db_username` | 資料庫帳號 | `2fconference` |
| `db_password` | 資料庫密碼 | (機密) |
| `host` | 應用程式主機 | `https://example.ncu.edu.tw` |
| `oauth_client_id` | NCU OAuth Client ID | (機密) |
| `oauth_client_secret` | NCU OAuth Secret | (機密) |
| `sender_mail_account` | 寄件者郵件帳號 | `mrbs@ncu.edu.tw` |
| `sender_mail_password` | 寄件者郵件密碼 | (機密) |

### 設定檔位置

| 檔案 | 用途 |
|------|------|
| `api/utilities/config.js` | 後端設定集中管理 |
| `MRBS-frontend/src/config.js` | 前端設定 |
| `MRBS-frontend/vite.config.js` | Vite 建構設定 |

---

## CI/CD 管線

### 流程概覽

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   Test Job   │────▶│  Build Job   │────▶│ Deploy Job   │
│  (Node 20)   │     │   (Docker)   │     │  (SSH + DC)  │
└──────────────┘     └──────────────┘     └──────────────┘
       │                    │                    │
       ▼                    ▼                    ▼
   npm install         docker build        docker compose
   npm test           docker push              pull & up
```

### GitHub Actions 工作流程

觸發條件：PR 合併至 `main` 分支

**Job 1: Test**
```yaml
- uses: actions/checkout@v2
- uses: actions/setup-node@v2 (node 20.10.0)
- run: cd src/api && npm install
- run: cd src/api && npm test
```

**Job 2: Build**
```yaml
- docker login
- docker build . --tag $REGISTRY/$IMAGE_NAME
- docker push $REGISTRY/$IMAGE_NAME
```

**Job 3: Deploy**
```yaml
- SSH 連線至 NCU VM
- cd $WORK_DIR && docker compose pull
- docker compose up -d
```

### GitHub Secrets 設定

| Secret | 說明 |
|--------|------|
| `DOCKER_USERNAME` | Docker Hub 帳號 |
| `DOCKER_PASSWORD` | Docker Hub 密碼 |
| `SSH_USER` | 部署主機 SSH 使用者 |
| `SSH_HOST` | 部署主機 IP/Domain |
| `SSH_PRIVATE_KEY` | SSH 私鑰 |
| `WORK_DIR` | 部署目錄 |

### GitHub Variables 設定

| Variable | 說明 |
|----------|------|
| `REGISTRY` | Docker Registry (e.g., `docker.io`) |
| `IMAGE_NAME` | 映像名稱 (e.g., `tommygood/ncu-mrbs`) |

---

## Docker 部署

### 容器建構

```bash
cd src/
docker build . --tag ncu-mrbs:latest
```

### Docker Compose 配置

```yaml
# docker-compose.yml
version: "3.0"

services:
  ncu-mrbs:
    image: tommygood/ncu-mrbs:latest
    ports:
      - 3000:3000
    environment:
      - db_host=mysql
      - db_name=2fconference
      - db_port=3306
      - db_username=2fconference
      - db_password=${DB_PASSWORD}
      - host=${HOST}
      - oauth_client_id=${OAuth_Client_ID}
      - oauth_client_secret=${OAuth_Client_Secret}
      - sender_mail_account=${Sender_Mail_Account}
      - sender_mail_password=${Sender_Mail_Password}
```

### 部署指令

```bash
# 拉取最新映像
docker compose pull

# 啟動服務
docker compose up -d

# 查看日誌
docker compose logs -f

# 停止服務
docker compose down
```

---

## 資料庫設定

### 初始化資料庫

```bash
# 連線至 MariaDB
mysql -u root -p

# 建立資料庫
CREATE DATABASE 2fconference CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

# 建立使用者
CREATE USER '2fconference'@'%' IDENTIFIED BY 'your_password';
GRANT ALL PRIVILEGES ON 2fconference.* TO '2fconference'@'%';

# 匯入 Schema
mysql -u 2fconference -p 2fconference < src/api/model/schema.sql
```

### 資料庫連線測試

```javascript
// 測試連線 (Node.js)
const mariadb = require('mariadb');
const pool = mariadb.createPool({
  host: 'localhost',
  user: '2fconference',
  password: 'your_password',
  database: '2fconference'
});

pool.getConnection()
  .then(conn => {
    console.log('Connected!');
    conn.release();
  })
  .catch(err => console.error(err));
```

---

## 除錯指南

### 常見問題

| 問題 | 可能原因 | 解決方案 |
|------|----------|----------|
| 登入失敗 | OAuth 設定錯誤 | 檢查 `oauth_client_id/secret` |
| 資料庫連線失敗 | 主機/密碼錯誤 | 檢查 `db_*` 環境變數 |
| 前端 404 | Base path 錯誤 | 確認 `vite.config.js` base 設定 |
| Email 發送失敗 | SMTP 設定錯誤 | 檢查郵件帳密與 SMTP 主機 |

### 日誌查看

```bash
# Docker 日誌
docker compose logs -f ncu-mrbs

# 後端日誌 (本地)
cd src/api && npm start 2>&1 | tee app.log
```

---

## 維護檢查清單

### 部署前檢查

- [ ] 所有測試通過 (`npm test`, `npm run test:unit`)
- [ ] 生產建構成功 (`npm run build`)
- [ ] 環境變數已正確設定
- [ ] 資料庫連線正常
- [ ] OAuth 回調 URL 已設定

### 部署後驗證

- [ ] 服務可正常存取 (Port 3000)
- [ ] 登入流程正常運作
- [ ] 預約功能可正常使用
- [ ] Email 通知發送正常

