# API 規格文件

> **文件資訊**
> 
> | 欄位 | 內容 |
> |------|------|
> | 專案名稱 | {{project_name}} |
> | API 版本 | {{version}} |
> | 撰寫人 | {{author}} |
> | 日期 | {{date}} |
> | Base URL | `https://api.{{domain}}/v1` |

---

## 修訂記錄

| 版本 | 日期 | 修訂人 | 說明 |
|------|------|--------|------|
| 1.0 | {{date}} | {{author}} | 初版建立 |

---

## 1. API 總覽

### 1.1 基本資訊

| 項目 | 說明 |
|------|------|
| Base URL | `https://api.{{domain}}/v1` |
| 協議 | HTTPS only |
| 資料格式 | JSON (`Content-Type: application/json`) |
| 字元編碼 | UTF-8 |
| 時間格式 | ISO 8601（`2024-01-15T08:30:00Z`） |
| 版本策略 | URL Path Versioning（`/v1/`, `/v2/`） |

### 1.2 認證方式

使用 Bearer Token（JWT）認證：

```http
Authorization: Bearer <access_token>
```

Token 取得流程：
1. 呼叫 `POST /auth/login`
2. 從回應中取得 `access_token`
3. 在後續請求的 Header 帶入 `Authorization: Bearer <token>`

Token 有效期：`access_token` 30 分鐘，`refresh_token` 30 天

---

## 2. 通用規範

### 2.1 請求格式

```http
POST /v1/resource HTTP/1.1
Host: api.{{domain}}
Content-Type: application/json
Authorization: Bearer <token>
X-Request-ID: <uuid>

{
  "field": "value"
}
```

### 2.2 回應格式

**成功回應：**
```json
{
  "success": true,
  "data": { ... },
  "meta": {
    "request_id": "uuid",
    "timestamp": "2024-01-15T08:30:00Z"
  }
}
```

**分頁回應：**
```json
{
  "success": true,
  "data": [ ... ],
  "pagination": {
    "page": 1,
    "per_page": 20,
    "total": 100,
    "total_pages": 5
  }
}
```

**錯誤回應：**
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "輸入資料格式不正確",
    "details": [
      { "field": "email", "message": "Email 格式不正確" }
    ]
  },
  "meta": {
    "request_id": "uuid",
    "timestamp": "2024-01-15T08:30:00Z"
  }
}
```

### 2.3 分頁參數

| 參數 | 類型 | 預設值 | 說明 |
|------|------|--------|------|
| `page` | integer | 1 | 頁碼（從 1 開始） |
| `per_page` | integer | 20 | 每頁筆數（最大 100） |
| `sort` | string | - | 排序欄位，例：`created_at` |
| `order` | string | `desc` | 排序方向：`asc` / `desc` |

### 2.4 HTTP 狀態碼

| 狀態碼 | 說明 | 使用情境 |
|--------|------|---------|
| 200 | OK | 成功查詢、更新 |
| 201 | Created | 成功建立資源 |
| 204 | No Content | 成功刪除 |
| 400 | Bad Request | 請求格式錯誤、資料驗證失敗 |
| 401 | Unauthorized | 未登入或 Token 失效 |
| 403 | Forbidden | 無操作權限 |
| 404 | Not Found | 資源不存在 |
| 409 | Conflict | 資源衝突（如 Email 已存在） |
| 422 | Unprocessable Entity | 業務邏輯驗證失敗 |
| 429 | Too Many Requests | 超過 Rate Limit |
| 500 | Internal Server Error | 伺服器內部錯誤 |

### 2.5 錯誤碼定義

| 錯誤碼 | HTTP 狀態 | 說明 |
|--------|---------|------|
| `VALIDATION_ERROR` | 400 | 欄位驗證失敗 |
| `UNAUTHORIZED` | 401 | 未認證 |
| `TOKEN_EXPIRED` | 401 | Token 已過期 |
| `FORBIDDEN` | 403 | 無操作權限 |
| `NOT_FOUND` | 404 | 資源不存在 |
| `EMAIL_EXISTS` | 409 | Email 已被使用 |
| `RATE_LIMITED` | 429 | 請求頻率超限 |
| `INTERNAL_ERROR` | 500 | 伺服器內部錯誤 |

### 2.6 Rate Limiting

| 端點類型 | 限制 | 時間窗口 |
|---------|------|---------|
| 一般 API | 100 requests | 每分鐘 |
| 認證端點 | 10 requests | 每分鐘 |
| 上傳端點 | 5 requests | 每分鐘 |

Rate Limit 回應 Header：
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 87
X-RateLimit-Reset: 1705298400
```

---

## 3. 認證模組（Auth）

---

### POST /auth/register — 用戶註冊

**描述：** 以 Email + 密碼建立新用戶帳號

**認證：** 不需要

**Request：**
```http
POST /v1/auth/register
Content-Type: application/json
```

```json
{
  "email": "user@example.com",
  "password": "SecurePass123!",
  "name": "王小明"
}
```

| 欄位 | 類型 | 必填 | 說明 |
|------|------|------|------|
| `email` | string | ✅ | 有效 Email，最長 255 字元 |
| `password` | string | ✅ | 8-100 字元，含大小寫+數字 |
| `name` | string | ✅ | 用戶名稱，1-50 字元 |

**成功回應（201 Created）：**
```json
{
  "success": true,
  "data": {
    "message": "驗證信已寄出，請檢查您的信箱",
    "email": "user@example.com"
  }
}
```

**錯誤回應：**

| HTTP | 錯誤碼 | 情境 |
|------|--------|------|
| 400 | `VALIDATION_ERROR` | Email 格式不正確或密碼不符規則 |
| 409 | `EMAIL_EXISTS` | Email 已被使用 |

---

### POST /auth/login — 用戶登入

**描述：** 以 Email + 密碼登入，取得 JWT Token

**認證：** 不需要

**Request：**
```http
POST /v1/auth/login
Content-Type: application/json
```

```json
{
  "email": "user@example.com",
  "password": "SecurePass123!"
}
```

**成功回應（200 OK）：**
```json
{
  "success": true,
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refresh_token": "dGhpcyBpcyBhIHJlZnJlc2ggdG9rZW4...",
    "expires_in": 1800,
    "token_type": "Bearer",
    "user": {
      "id": "uuid",
      "email": "user@example.com",
      "name": "王小明",
      "role": "user"
    }
  }
}
```

**錯誤回應：**

| HTTP | 錯誤碼 | 情境 |
|------|--------|------|
| 400 | `VALIDATION_ERROR` | 欄位缺少或格式錯誤 |
| 401 | `UNAUTHORIZED` | Email 或密碼錯誤 |
| 423 | `ACCOUNT_LOCKED` | 帳號因多次失敗被鎖定 |

---

### POST /auth/refresh — 刷新 Token

**描述：** 使用 refresh_token 取得新的 access_token

**Request：**
```json
{
  "refresh_token": "dGhpcyBpcyBhIHJlZnJlc2ggdG9rZW4..."
}
```

**成功回應（200 OK）：**
```json
{
  "success": true,
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expires_in": 1800
  }
}
```

---

## 4. 用戶模組（Users）

---

### GET /users/me — 取得當前用戶資訊

**認證：** 需要（Bearer Token）

**成功回應（200 OK）：**
```json
{
  "success": true,
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "email": "user@example.com",
    "name": "王小明",
    "role": "user",
    "created_at": "2024-01-15T08:30:00Z",
    "updated_at": "2024-01-15T08:30:00Z"
  }
}
```

---

### PATCH /users/me — 更新用戶資訊

**認證：** 需要

**Request：**
```json
{
  "name": "王大明",
  "avatar_url": "https://cdn.example.com/avatar.jpg"
}
```

**成功回應（200 OK）：** 回傳更新後的用戶物件（同 GET /users/me 格式）

---

## 5. [下一個資源模組]

[依上方格式，以 REST 慣例定義每個 Endpoint]

---

## 附錄：資料模型

### User 物件

| 欄位 | 類型 | 說明 |
|------|------|------|
| `id` | uuid | 用戶唯一識別碼 |
| `email` | string | Email 地址 |
| `name` | string | 顯示名稱 |
| `role` | enum | `user` / `admin` |
| `created_at` | datetime | 建立時間（ISO 8601） |
| `updated_at` | datetime | 最後更新時間 |
