# 安裝手冊

> **文件資訊**
> 
> | 欄位 | 內容 |
> |------|------|
> | 系統名稱 | {{project_name}} |
> | 文件版本 | {{version}} |
> | 適用系統版本 | [系統版本號] |
> | 撰寫人 | {{author}} |
> | 日期 | {{date}} |
> | 適用環境 | Development / Staging / Production |

---

## 修訂記錄

| 版本 | 日期 | 修訂人 | 說明 |
|------|------|--------|------|
| 1.0 | {{date}} | {{author}} | 初版建立 |

---

## 前言

本手冊說明 {{project_name}} 的完整安裝與部署流程。請按步驟執行，並確保所有前置條件已就緒。

> ⚠️ **重要提醒：**
> - 在正式環境操作前，請先在測試環境完整驗證
> - 確保有完整的備份與回滾計畫
> - 建議在維護時間視窗內執行部署

---

## 第 1 章：前置條件檢查

### 1.1 硬體需求

| 元件 | 最低規格 | 建議規格（Production） |
|------|---------|---------------------|
| CPU | 2 核心 | 4 核心 以上 |
| RAM | 4 GB | 16 GB 以上 |
| 磁碟 | 50 GB SSD | 200 GB SSD |
| 網路 | 100 Mbps | 1 Gbps |

### 1.2 軟體需求

| 軟體 | 最低版本 | 確認指令 |
|------|---------|---------|
| Operating System | Ubuntu 22.04 LTS | `lsb_release -a` |
| Docker | 24.0 | `docker --version` |
| Docker Compose | 2.20 | `docker compose version` |
| Git | 2.40 | `git --version` |
| Node.js | 20 LTS | `node --version` |
| npm | 10 | `npm --version` |
| PostgreSQL | 16 | `psql --version` |

### 1.3 網路需求

確保以下連接埠可用：

| 連接埠 | 服務 | 說明 |
|--------|------|------|
| 80 | HTTP | Web 服務 |
| 443 | HTTPS | Web 服務（SSL） |
| 5432 | PostgreSQL | 資料庫（僅內網） |
| 6379 | Redis | 快取（僅內網） |

### 1.4 前置條件確認清單

請在開始安裝前確認以下項目：

- [ ] 作業系統版本符合需求
- [ ] Docker 與 Docker Compose 已安裝
- [ ] 目標連接埠未被佔用
- [ ] 有足夠的磁碟空間（`df -h`）
- [ ] 具備 sudo 權限
- [ ] DNS 設定已完成（正式環境）
- [ ] SSL 憑證已準備（正式環境）
- [ ] 已取得所有必要的 API 金鑰與密鑰

---

## 第 2 章：安裝步驟

### 2.1 取得原始碼

```bash
# 建立部署目錄
mkdir -p /opt/{{project_name}}
cd /opt/{{project_name}}

# 從版本控制系統取得程式碼
git clone https://github.com/{{org}}/{{repo}}.git .
git checkout v{{version}}

# 確認版本
git log --oneline -1
```

### 2.2 設定環境變數

```bash
# 複製環境變數範本
cp .env.example .env

# 編輯設定檔
nano .env
```

**必填環境變數說明：**

```bash
# === 應用程式設定 ===
APP_NAME="{{project_name}}"
APP_ENV=production           # development / staging / production
APP_URL=https://yourdomain.com
APP_SECRET_KEY=              # ⚠️ 必填：產生方式見下方

# === 資料庫設定 ===
DATABASE_URL=postgresql://user:password@localhost:5432/dbname
# 或分開設定：
DB_HOST=localhost
DB_PORT=5432
DB_NAME={{project_name}}_prod
DB_USER=app_user
DB_PASSWORD=                 # ⚠️ 必填：使用強密碼

# === 快取設定 ===
REDIS_URL=redis://localhost:6379/0

# === 郵件設定 ===
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=noreply@yourdomain.com
SMTP_PASSWORD=               # ⚠️ 必填

# === 第三方整合 ===
# GOOGLE_CLIENT_ID=
# GOOGLE_CLIENT_SECRET=
```

**產生 Secret Key：**
```bash
# 產生隨機 Secret Key
python3 -c "import secrets; print(secrets.token_hex(32))"
# 或
openssl rand -hex 32
```

### 2.3 資料庫初始化

```bash
# 建立資料庫用戶（以 postgres 超級用戶執行）
sudo -u postgres psql << 'EOF'
CREATE USER app_user WITH PASSWORD 'your_secure_password';
CREATE DATABASE {{project_name}}_prod OWNER app_user;
GRANT ALL PRIVILEGES ON DATABASE {{project_name}}_prod TO app_user;
