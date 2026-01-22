# Integration Architecture - bookingSystem

> 自動生成於 2026-01-18 | Meeting-Room-Booking-System 整合架構分析

## 系統架構總覽

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              User Browser                                    │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │                    NCU Portal Authentication                         │   │
│   │                    (OAuth 2.0 with reCAPTCHA)                        │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
└───────────────────────────────────┬─────────────────────────────────────────┘
                                    │ HTTPS
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                           NCU Reverse Proxy                                  │
│                        (Path: /2fconference/*)                              │
└───────────────────────────────────┬─────────────────────────────────────────┘
                                    │ Port 3000
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                    Docker Container (ncu-mrbs)                               │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                     Express.js Server (Port 3000)                      │  │
│  │  ┌─────────────────────┐  ┌─────────────────────────────────────────┐ │  │
│  │  │   Static Files      │  │           API Routes                     │ │  │
│  │  │  (Vue.js SPA)       │  │   /api/login    /api/reservation         │ │  │
│  │  │  /MRBS-frontend/    │  │   /api/info     /api/room                │ │  │
│  │  │  dist/              │  │   /api/user     /api/violation           │ │  │
│  │  └─────────────────────┘  │   /api/log      /api/doc                 │ │  │
│  │                           └─────────────────────────────────────────┘ │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
└───────────────────────────────────┬─────────────────────────────────────────┘
                                    │ Port 3306
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                       MariaDB Database                                       │
│                    (Database: 2fconference)                                  │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐          │
│  │   User   │ │   Room   │ │Reservation│ │    Log   │ │Violation │          │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘          │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 前後端整合模式

### 1. 單一容器部署模式

本專案採用**單一容器部署**模式，Express.js 同時提供：
- **API 服務**: 處理所有 `/api/*` 路由
- **靜態檔案**: 托管 Vue.js 編譯後的 SPA

```javascript
// api/index.js - 關鍵整合程式碼

// API 路由註冊
app.use("/api/login", require("./api/login.js"));
app.use("/api/reservation", require("./api/reservation.js"));
// ... 其他 API 路由

// SPA 靜態檔案托管
const path = util.getParentPath(__dirname) + "/MRBS-frontend/dist";
app.use(express.static(path));

// History API Fallback (支援 Vue Router)
const history = require("connect-history-api-fallback");
app.use(history());
```

### 2. URL 路徑重寫

由於部署在 NCU 子路徑下，需要 URL 重寫：

```javascript
// 移除 /2fconference 前綴
app.use((req, res, next) => {
  const removeOnRoutes = '/2fconference';
  if (req.url.startsWith(removeOnRoutes)) {
    req.url = req.url.replace(removeOnRoutes, '') || '/';
    req.originalUrl = req.originalUrl.replace(removeOnRoutes, '') || '/';
  }
  next();
});
```

---

## 通訊協定

### API 呼叫流程

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│  Vue 元件    │     │  Fetch/Axios │     │  Express API │
│  (lobby.vue) │────▶│   Request    │────▶│  Controller  │
└──────────────┘     └──────────────┘     └──────┬───────┘
                                                  │
                     ┌──────────────┐     ┌──────┴───────┐
                     │   JSON       │◀────│    Model     │
                     │  Response    │     │   (MariaDB)  │
                     └──────────────┘     └──────────────┘
```

### 前端 API 配置

```javascript
// MRBS-frontend/src/config.js
export default {
  host: "",                        // 空字串 = 同源
  apiUrl: "/2fconference/api",     // API 基礎路徑
};
```

### API 呼叫範例

```javascript
// lobby.vue - 取得預約資料
eventApiUrl: (start, end) =>
  config.apiUrl + `/reservation?start_time=${start}&end_time=${end}`

// application.vue - 新增預約
await fetch(`${config.apiUrl}/reservation`, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ room_id, name, start_time, end_time, show, ext }),
  credentials: 'include'  // 包含 JWT Cookie
});
```

---

## 認證整合

### JWT Cookie 機制

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   Browser    │     │  NCU OAuth   │     │   Express    │
│              │     │   Server     │     │   Server     │
└──────┬───────┘     └──────┬───────┘     └──────┬───────┘
       │                    │                    │
       │  1. Click Login    │                    │
       │───────────────────▶│                    │
       │                    │                    │
       │  2. OAuth Redirect │                    │
       │◀───────────────────│                    │
       │                    │                    │
       │  3. Auth Code      │                    │
       │────────────────────────────────────────▶│
       │                    │                    │
       │                    │  4. Verify Token   │
       │                    │◀───────────────────│
       │                    │                    │
       │                    │  5. User Info      │
       │                    │───────────────────▶│
       │                    │                    │
       │  6. Set JWT Cookie │                    │
       │◀────────────────────────────────────────│
       │                    │                    │
```

### 前端認證守衛

```javascript
// router/index.js - 路由守衛
router.beforeEach(async (to, from, next) => {
  if (to.matched.some(record => record.meta.requiresManagerAuth)) {
    const res = await Auth.methods.login();
    if (!res.suc) {
      next({ name: 'failed_login', query: {error: res.reason} });
    } else if (res.privelege_level < 1) {
      next({ name: 'failed_login', query: {error: '權限不足'} });
    } else {
      next();
    }
  }
  // ...
});
```

### 後端 JWT 驗證

```javascript
// utilities/jwt.js
exports.verifyLogin = (req, res, next) => {
  const token = req.cookies.token;
  if (!token) return res.status(401).json({ error: 'Not authenticated' });

  jwt.verify(token, secret, (err, decoded) => {
    if (err) return res.status(403).json({ error: 'Invalid token' });
    req.identifier = decoded.identifier;
    next();
  });
};

exports.verifyAdmin = (req, res, next) => {
  // 先驗證登入，再檢查權限等級
  this.verifyLogin(req, res, async () => {
    const level = await User.getPrivilegeLevel(req.identifier);
    if (level < 1) return res.status(403).json({ error: 'Admin required' });
    next();
  });
};
```

---

## 資料流分析

### 預約功能資料流

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           Frontend (Vue.js)                              │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐                  │
│  │   lobby     │───▶│  calendar   │───▶│ application │                  │
│  │   .vue      │    │    .vue     │    │    .vue     │                  │
│  │             │◀───│             │◀───│             │                  │
│  │  - info     │    │ - events    │    │ - form data │                  │
│  │  - eventApi │    │ - FullCal   │    │ - submit    │                  │
│  └─────────────┘    └─────────────┘    └──────┬──────┘                  │
│                                               │                          │
│         props/emit                    POST /api/reservation              │
└───────────────────────────────────────────────┼──────────────────────────┘
                                                │
                                                ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                           Backend (Express.js)                           │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │                    JWT Middleware (verifyLogin)                  │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                    │                                     │
│                                    ▼                                     │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │              api/reservation.js (Controller)                     │    │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │    │
│  │  │ checkRules  │──│checkOverlap │──│   insert    │              │    │
│  │  │ (時間限制) │  │ (衝突檢查) │  │  (寫入DB)  │              │    │
│  │  └─────────────┘  └─────────────┘  └─────────────┘              │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                    │                                     │
│                                    ▼                                     │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │              model/reservation.js (Data Access)                  │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                    │                                     │
│                                    ▼                                     │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │                    MariaDB (Reservation Table)                   │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                    │                                     │
│                                    ▼                                     │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │              utilities/email.js (通知服務)                       │    │
│  │              - 成功預約通知                                       │    │
│  │              - 取消預約通知                                       │    │
│  └─────────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 安全機制

### HTTP 安全標頭

```javascript
// api/index.js
app.use((req, res, next) => {
  res.setHeader("Referrer-Policy", "same-origin");
  res.set("X-Content-Type-Options", "nosniff");
  res.setHeader("Content-Security-Policy",
    "default-src 'self' https://cdn.jsdelivr.net https://cdnjs.cloudflare.com; " +
    "img-src 'self' data:; " +
    "style-src 'self' 'unsafe-inline' https://unpkg.com https://cdn.quilljs.com https://cdn.jsdelivr.net; " +
    "script-src 'self' https://cdn.jsdelivr.net https://cdnjs.cloudflare.com https://unpkg.com https://cdn.quilljs.com; " +
    "font-src 'self' https://cdn.jsdelivr.net data:; " +
    "frame-ancestors 'self';"
  );
  next();
});
```

### Session 配置

```javascript
app.use(session({
  secret: config.session.secret,
  resave: false,
  saveUninitialized: true,
  cookie: {
    maxAge: 1000 * 60 * 60 * 24,  // 24 小時
    sameSite: 'strict',           // 防止 CSRF
    secure: true,                  // 僅 HTTPS
  },
}));
```

---

## 元件通訊模式

### Props 向下傳遞

```
┌─────────────┐
│   lobby     │
│   .vue      │
│  - info     │ ──────────────────┐
│  - eventApi │                   │
└──────┬──────┘                   │
       │                          │
       │ :info="info"             │ :eventApiUrl="eventApiUrl"
       ▼                          ▼
┌─────────────┐           ┌─────────────┐
│   header    │           │  calendar   │
│   .vue      │           │   .vue      │
│  - info     │           │ - eventApi  │
└─────────────┘           │ - info      │
                          └─────────────┘
```

### Events 向上傳遞

```
┌─────────────┐
│   lobby     │
│   .vue      │
│  setInfo()  │ ◀─────────────────┐
└──────┬──────┘                   │
       │                          │
       │                   @setInfo="setInfo"
       ▼                          │
┌─────────────┐           ┌─────────────┐
│   header    │           │   header    │
│   .vue      │           │   .vue      │
│  emit()     │ ──────────│  $emit()    │
└─────────────┘           └─────────────┘
```

---

## 整合關鍵點

| 整合點 | 前端位置 | 後端位置 | 說明 |
|--------|----------|----------|------|
| **API 基礎路徑** | `config.js` | `index.js` | `/2fconference/api` |
| **JWT Cookie** | `fetch` credentials | `jwt.js` middleware | 自動攜帶 |
| **路由歷史** | Vue Router history | `history()` middleware | SPA 支援 |
| **靜態資源** | `dist/` 目錄 | `express.static()` | 生產建構檔案 |
| **CORS** | 開發時跨域 | `cors()` middleware | 開發用 |

---

## 部署整合

### 建構流程

```bash
# 1. 建構前端
cd src/MRBS-frontend
npm run build            # 輸出至 dist/

# 2. Docker 映像包含前後端
cd src/
docker build .           # Dockerfile 複製 api/ 與 MRBS-frontend/dist/
```

### Dockerfile 整合

```dockerfile
FROM node:20-slim
WORKDIR /app
COPY api/ ./api/
COPY MRBS-frontend/dist/ ./MRBS-frontend/dist/
WORKDIR /app/api
RUN npm install
CMD ["npm", "start"]
```

