# bookingSystem 技術文件索引

> 自動生成於 2026-01-18 | Meeting-Room-Booking-System 完整技術文件

## 專案概覽

| 屬性 | 值 |
|------|------|
| **專案名稱** | Meeting-Room-Booking-System (MRBS) |
| **專案類型** | Multi-part (Frontend + Backend) |
| **技術堆疊** | Vue.js 3.4 + Express.js 4.19 + MariaDB |
| **部署方式** | Docker + GitHub Actions CI/CD |
| **文件版本** | 1.0.0 |

---

## 文件目錄

### 核心技術文件

| 文件 | 說明 | 對象 |
|------|------|------|
| [source-tree-analysis.md](source-tree-analysis.md) | 完整源碼結構與註解 | 全體開發人員 |
| [integration-architecture.md](integration-architecture.md) | 前後端整合架構 | 架構師/後端開發 |
| [development-operations.md](development-operations.md) | 開發環境與運維指南 | 開發人員/運維 |

### 條件分析文件

| 文件 | 說明 | 對象 |
|------|------|------|
| [api-contracts-backend.md](api-contracts-backend.md) | REST API 完整合約 (22 端點) | 前後端開發 |
| [data-models-backend.md](data-models-backend.md) | 資料庫模型與 ERD (7 表格) | 後端開發/DBA |
| [component-inventory-frontend.md](component-inventory-frontend.md) | Vue 元件清單 (20+ 元件) | 前端開發 |

### 整合文件

| 文件 | 說明 | 對象 |
|------|------|------|
| [external-integrations.md](external-integrations.md) | 外部服務整合說明 | 架構師/運維 |

### 追蹤文件

| 文件 | 說明 |
|------|------|
| [project-scan-report.json](project-scan-report.json) | 文件生成進度追蹤 |

---

## 快速導航

### 我是新加入的開發人員...

1. 先讀 [source-tree-analysis.md](source-tree-analysis.md) 了解專案結構
2. 閱讀 [development-operations.md](development-operations.md) 設定開發環境
3. 根據負責領域閱讀：
   - 前端 → [component-inventory-frontend.md](component-inventory-frontend.md)
   - 後端 → [api-contracts-backend.md](api-contracts-backend.md) + [data-models-backend.md](data-models-backend.md)

### 我需要了解系統架構...

1. [integration-architecture.md](integration-architecture.md) - 前後端如何整合
2. [external-integrations.md](external-integrations.md) - 外部服務依賴

### 我需要部署或維護系統...

1. [development-operations.md](development-operations.md) - CI/CD 與部署流程
2. [external-integrations.md](external-integrations.md) - 環境變數與服務配置

### 我需要新增 API...

1. [api-contracts-backend.md](api-contracts-backend.md) - 現有 API 規範
2. [data-models-backend.md](data-models-backend.md) - 資料模型參考

---

## 專案統計

### 程式碼統計

| 類別 | 數量 |
|------|------|
| 後端 JS 檔案 | 18 |
| 前端 Vue 元件 | 20+ |
| API 端點 | 22 |
| 資料表 | 7 |
| 路由 | 11 |
| 測試檔案 | 7 |

### 技術堆疊

```
Frontend:
├── Vue.js 3.4
├── Vue Router 4.3
├── Vite 5.3
├── FullCalendar 6.1
├── Axios 1.6
└── Cypress 13 + Vitest 1.6

Backend:
├── Express.js 4.19
├── MariaDB 3.3
├── JWT (jsonwebtoken)
├── OAuth 2.0 (simple-oauth2)
├── Nodemailer
└── Mocha + Chai

DevOps:
├── Docker + Docker Compose
├── GitHub Actions
└── Docker Hub
```

---

## 原始專案文件

本專案原有的文件（位於 `Meeting-Room-Booking-System/doc/`）：

| 文件 | 說明 |
|------|------|
| `api.md` | 原始 API 文件 |
| `contribution.md` | 貢獻指南 |
| `workflow.md` | CI/CD 說明 |
| `test.md` | 測試架構 |
| `page.md` | 頁面說明 |

---

## 更新記錄

| 日期 | 版本 | 變更 |
|------|------|------|
| 2026-01-18 | 1.0.0 | 初次生成完整技術文件 |

---

## 生成資訊

- **生成工具**: document-project workflow (Deep Scan)
- **掃描範圍**: Full codebase analysis
- **生成檔案**: 8 個技術文件

