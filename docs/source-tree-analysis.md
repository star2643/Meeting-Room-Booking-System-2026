# Source Tree Analysis - bookingSystem

> 自動生成於 2026-01-18 | Meeting-Room-Booking-System 完整源碼結構

## 專案架構概覽

```
Meeting-Room-Booking-System/           ← 專案根目錄
├── .github/                           ← CI/CD 配置
│   └── workflows/
│       ├── no_vpn_pipeline.yaml       ★ 主要部署管線 (GitHub Actions → Docker Hub → NCU VM)
│       └── npm-publish-github-packages.yml
│
├── doc/                               ← 原始專案文件
│   ├── api.md                         ★ REST API 完整文件
│   ├── contribution.md                開發貢獻指南
│   ├── page.md                        頁面說明
│   ├── test.md                        測試架構說明
│   └── workflow.md                    CI/CD 工作流程說明
│
├── README.md                          ★ 專案入口文件
│
└── src/                               ← 源碼目錄
    ├── docker-compose.yml             ★ Docker 編排配置
    ├── Dockerfile                     ★ 容器建構配置
    │
    ├── api/                           ← [BACKEND] Express.js API 服務
    │   └── (詳見後端結構)
    │
    ├── MRBS-frontend/                 ← [FRONTEND] Vue.js 前端應用
    │   └── (詳見前端結構)
    │
    └── templates/                     ← 舊版模板檔案 (已棄用)
        ├── js/                        JavaScript 模板
        └── image/                     圖片資源
```

---

## [BACKEND] API 服務結構

```
src/api/                               ← Express.js 後端服務
│
├── index.js                           ★ [ENTRY POINT] 應用程式入口
│                                        - 設定 Express 伺服器
│                                        - 載入中介層 (helmet, session, cors)
│                                        - 註冊 API 路由
│                                        - 監聽 Port 3000
│
├── package.json                       ★ 依賴管理
│   └── 主要依賴: express@4.19, mariadb@3.3, jsonwebtoken, oauth, nodemailer
│
├── api/                               ← [CONTROLLERS] API 路由處理器
│   ├── doc.js                         文件管理 API
│   ├── info.js                        使用者資訊 API
│   ├── log.js                         操作日誌 API
│   ├── login.js                       ★ NCU OAuth SSO 登入
│   ├── reservation.js                 ★ [CORE] 預約管理 CRUD
│   ├── room.js                        會議室管理 API
│   ├── user.js                        使用者管理 API
│   └── violation.js                   違規紀錄 API
│
├── model/                             ← [DATA LAYER] 資料模型
│   ├── conn.js                        ★ MariaDB 連線池管理
│   ├── schema.sql                     ★ [DATABASE SCHEMA] 完整資料表定義
│   ├── doc.js                         Doc 資料模型
│   ├── info.js                        Info 資料模型
│   ├── log.js                         Log 資料模型
│   ├── operator.js                    操作類型定義
│   ├── reservation.js                 ★ Reservation 資料模型 (核心業務邏輯)
│   ├── room.js                        Room 資料模型
│   ├── user.js                        User 資料模型
│   └── violation.js                   Violation 資料模型
│
├── utilities/                         ← [UTILITIES] 工具函式
│   ├── config.js                      ★ [CONFIG] 環境設定 (DB, Email, Session)
│   ├── email.js                       ★ 郵件發送服務 (Nodemailer)
│   ├── info.js                        NCU Portal 資訊查詢
│   ├── jwt.js                         ★ [AUTH] JWT 驗證中介層
│   ├── main.js                        主要工具函式
│   ├── oauth.js                       ★ NCU OAuth 整合
│   └── test.js                        測試輔助函式
│
└── testing/                           ← [TESTS] 後端測試
    ├── main.js                        測試入口 (Mocha)
    ├── reservation.js                 預約 API 測試
    └── violation.js                   違規 API 測試
```

### 後端關鍵檔案說明

| 檔案 | 重要性 | 說明 |
|------|--------|------|
| `index.js` | ★★★ | 應用程式入口，設定所有中介層與路由 |
| `api/reservation.js` | ★★★ | 預約核心業務邏輯，包含衝突檢查與權限控制 |
| `utilities/jwt.js` | ★★★ | JWT 驗證，`verifyLogin` 與 `verifyAdmin` 中介層 |
| `utilities/config.js` | ★★★ | 所有環境設定集中管理 |
| `model/schema.sql` | ★★★ | 完整資料庫結構定義 |
| `api/login.js` | ★★☆ | NCU OAuth SSO 流程處理 |
| `model/conn.js` | ★★☆ | 資料庫連線池，所有查詢入口 |

---

## [FRONTEND] Vue.js 前端結構

```
src/MRBS-frontend/                     ← Vue.js 3.4 前端應用
│
├── package.json                       ★ 依賴管理
│   └── 主要依賴: vue@3.4, vue-router@4.3, vite@5.3
│
├── vite.config.js                     ★ Vite 建構配置 (base: /2fconference/)
├── vitest.config.js                   Vitest 測試配置
├── cypress.config.js                  Cypress E2E 配置
│
├── src/                               ← 前端源碼
│   │
│   ├── main.js                        ★ [ENTRY POINT] Vue 應用入口
│   ├── App.vue                        ★ 根元件
│   ├── config.js                      前端設定
│   │
│   ├── router/
│   │   └── index.js                   ★ [ROUTING] Vue Router 設定
│   │                                    - 11 個路由定義
│   │                                    - beforeEach 導航守衛
│   │                                    - requiresAuth / requiresManagerAuth 元資料
│   │
│   ├── middleware/
│   │   └── auth.vue                   ★ [AUTH] 認證檢查元件
│   │                                    - JWT 驗證
│   │                                    - 權限等級檢查
│   │
│   ├── views/                         ← [PAGES] 頁面視圖
│   │   ├── homeView.vue               首頁
│   │   └── aboutView.vue              關於頁面
│   │
│   ├── components/                    ← [COMPONENTS] 功能元件
│   │   │
│   │   ├── lobby.vue                  ★ [CORE] 使用者大廳 - 預約主介面
│   │   ├── calendar.vue               ★ [CORE] FullCalendar 日曆元件
│   │   ├── application.vue            ★ [CORE] 預約申請表單
│   │   │
│   │   ├── conference.vue             ★ 會議管理 (管理員)
│   │   ├── reservation.vue            預約 CRUD 操作
│   │   │
│   │   ├── board.vue                  看板管理
│   │   ├── preview.vue                看板預覽
│   │   ├── demo.vue                   看板展示 (公開)
│   │   │
│   │   ├── rule.vue                   規則編輯器 (Quill)
│   │   ├── ruleDemo.vue               規則展示 (公開)
│   │   │
│   │   ├── privilege.vue              權限管理
│   │   ├── log.vue                    操作日誌
│   │   │
│   │   ├── header.vue                 頁面標頭 (可重用)
│   │   ├── rootMenu.vue               管理員側邊選單
│   │   ├── welcome.vue                歡迎頁面
│   │   ├── failedLogin.vue            登入失敗頁面
│   │   ├── helloWorld.vue             範例元件
│   │   │
│   │   └── icons/                     ← 圖示元件
│   │       ├── IconCommunity.vue
│   │       ├── IconDocumentation.vue
│   │       ├── IconEcosystem.vue
│   │       ├── IconSupport.vue
│   │       └── IconTooling.vue
│   │
│   └── assets/                        ← 靜態資源
│       └── (CSS, 圖片等)
│
├── public/                            ← 公開靜態資源
│   └── licence.md
│
├── dist/                              ← 建構輸出 (已編譯)
│   └── assets/
│       └── index-*.js                 生產版本 bundle
│
├── cypress/                           ← [E2E TESTS] Cypress 測試
│   ├── e2e/
│   │   ├── example.cy.js
│   │   └── lobby.cy.js                ★ 大廳功能 E2E 測試
│   ├── fixtures/
│   │   └── example.json
│   └── support/
│       ├── commands.js
│       └── e2e.js
│
├── __test__/                          ← [UNIT TESTS] Vitest 單元測試
│   ├── Header.test.js
│   ├── Preview.test.js
│   ├── RootMenu.test.js
│   └── Welcome.test.js
│
└── .vite/                             ← Vite 快取 (自動生成)
    └── deps/
```

### 前端關鍵檔案說明

| 檔案 | 重要性 | 說明 |
|------|--------|------|
| `src/main.js` | ★★★ | Vue 應用入口，初始化 Router |
| `src/router/index.js` | ★★★ | 所有路由定義與認證守衛 |
| `src/middleware/auth.vue` | ★★★ | 認證與權限驗證邏輯 |
| `src/components/lobby.vue` | ★★★ | 核心預約介面 |
| `src/components/calendar.vue` | ★★★ | FullCalendar 整合 |
| `src/components/application.vue` | ★★☆ | 預約申請表單 |
| `vite.config.js` | ★★☆ | 建構設定，包含 base path |

---

## 整合點分析

### Frontend → Backend 通訊

```
┌─────────────────────────────────────────────────────────────────┐
│                        FRONTEND (Vue.js)                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │   lobby.vue  │  │ calendar.vue │  │ conference.vue│          │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘          │
│         │                 │                 │                   │
│         └────────────────┬┴─────────────────┘                   │
│                          │ Axios HTTP Requests                  │
│                          │ (with JWT Cookie)                    │
└──────────────────────────┼──────────────────────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────────────────────┐
│                        BACKEND (Express.js)                      │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                     JWT Middleware                         │  │
│  │   jwt.verifyLogin() / jwt.verifyAdmin()                   │  │
│  └───────────────────────────────────────────────────────────┘  │
│                              │                                   │
│         ┌────────────────────┼────────────────────┐             │
│         ▼                    ▼                    ▼             │
│  ┌─────────────┐     ┌─────────────┐     ┌─────────────┐        │
│  │/api/reservation│   │/api/room    │     │/api/user    │        │
│  └──────┬──────┘     └──────┬──────┘     └──────┬──────┘        │
│         │                   │                   │               │
│         └───────────────────┴───────────────────┘               │
│                             │                                    │
│                             ▼                                    │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                    MariaDB Database                        │  │
│  └───────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────┘
```

### 認證流程

```
┌─────────┐     ┌─────────┐     ┌─────────┐     ┌─────────┐
│ Browser │────▶│ Frontend│────▶│ Backend │────▶│NCU OAuth│
└─────────┘     └─────────┘     └─────────┘     └─────────┘
     │               │               │               │
     │  1. Click Login               │               │
     │──────────────▶│               │               │
     │               │ 2. Redirect   │               │
     │               │──────────────▶│               │
     │               │               │ 3. OAuth Flow │
     │               │               │──────────────▶│
     │               │               │◀──────────────│
     │               │               │ 4. Get Token  │
     │               │ 5. Set JWT Cookie             │
     │◀──────────────────────────────│               │
     │               │               │               │
```

---

## 部署結構

```
┌─────────────────────────────────────────────────────────────┐
│                    GitHub Repository                         │
│  └── .github/workflows/no_vpn_pipeline.yaml                 │
└─────────────────────────┬───────────────────────────────────┘
                          │ Push to main
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                    GitHub Actions                            │
│  1. Build Docker Image                                       │
│  2. Push to Docker Hub                                       │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                    NCU VM Server                             │
│  ┌─────────────────────────────────────────────────────────┐│
│  │              docker-compose.yml                          ││
│  │  ┌─────────────────┐  ┌─────────────────┐               ││
│  │  │  MRBS Container │  │ MariaDB Container│               ││
│  │  │  (Frontend+API) │◀▶│    (Database)    │               ││
│  │  │    Port 3000    │  │    Port 3306     │               ││
│  │  └─────────────────┘  └─────────────────┘               ││
│  └─────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────┘
```

---

## 檔案統計

| 類別 | 數量 | 說明 |
|------|------|------|
| **後端 JS 檔案** | 18 | API + Model + Utilities |
| **前端 Vue 元件** | 20+ | Views + Components |
| **測試檔案** | 7 | Mocha + Vitest + Cypress |
| **設定檔案** | 8 | package.json, vite.config, etc. |
| **文件檔案** | 5 | doc/ 目錄下 |
| **CI/CD 配置** | 2 | GitHub Actions workflows |

