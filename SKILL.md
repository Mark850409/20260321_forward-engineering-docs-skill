---
name: forward-engineering-docs
description: |
  順向工程文件產出技能。從需求或構想出發，自動生成一整套完整的專案文件套件，
  包含：PRD 產品需求文件、SRD 系統需求規格、FRD 功能需求文件、API 規格文件、
  DB Schema 文件、操作手冊、安裝手冊，並支援套用自訂模板參數，
  所有文件均可無痛匯出為 Word (.docx) 格式。

  強制觸發情境——當使用者說出以下任何一項時，必須立即啟用此技能：
  - 「幫我寫 PRD」、「產出需求規格書」、「寫產品需求文件」
  - 「幫我寫 SRD」、「系統需求規格」、「功能需求文件」、「寫 FRD」
  - 「幫我寫 API 文件」、「產出 API spec」、「生成 API 規格」
  - 「幫我寫 DB 文件」、「資料庫設計文件」、「Schema 文件」
  - 「幫我寫操作手冊」、「用戶手冊」、「使用說明書」
  - 「幫我寫安裝手冊」、「安裝指南」、「部署文件」、「安裝說明」
  - 「產出一套完整文件」、「順向工程文件」、「從需求產文件」
  - 「我要開始一個新專案，幫我準備文件」、「專案文件套件」
  - 「套模板產出文件」、「用公司模板生成文件」
  - 「文件轉成 Word」、「文件輸出為 docx」
  即使使用者只說「幫我寫 [某文件類型]」而沒有明確說「文件」，也應觸發此技能。
---

# 順向工程文件產出技能

## 核心理念

從「目標與需求」出發，系統化生成業界標準的完整文件套件。
支援參數化模板、多格式輸出（Markdown + Word），一次設定、全套產出。

---

## 第一步：輸入收集與確認

### 必要輸入資訊

在產出文件前，先確認以下資訊（若不完整，主動詢問）：

| 資訊 | 範例 | 重要性 |
|------|------|--------|
| 專案名稱 | 「電商後台管理系統」 | 必填 |
| 專案簡介 | 一句話描述產品目的 | 必填 |
| 目標用戶 | 管理員、一般用戶、訪客 | 必填 |
| 需要哪些文件 | PRD / API / 安裝手冊… | 必填 |
| 模板偏好 | 公司模板 / 內建模板 / 無 | 選填 |
| 輸出格式 | Markdown / Word / 兩者都要 | 選填（預設兩者） |

### 快速啟動模式

若使用者提供的資訊充足（如貼上一份需求說明），直接開始產出，不需逐一確認。

---

## 第二步：文件套件說明

### 可產出的文件類型

| 文件代碼 | 全名 | 說明 | 主要受眾 |
|---------|------|------|---------|
| PRD | 產品需求文件 | 定義「做什麼」與「為什麼做」 | PM、設計師、開發者 |
| SRD | 系統需求規格 | 定義技術與非功能需求 | 架構師、Tech Lead |
| FRD | 功能需求文件 | 詳細的功能規格與 User Story | 開發者、測試人員 |
| API | API 規格文件 | Endpoint 定義、Request/Response | 前後端開發者 |
| DB | 資料庫設計文件 | Table Schema、ERD | 後端開發者、DBA |
| OPS | 操作手冊 | 系統操作說明、流程圖 | 終端用戶、客服 |
| INSTALL | 安裝手冊 | 環境建置與部署步驟 | DevOps、開發者 |

> 使用者說「全套文件」時，預設產出全部 7 份。

---

## 第三步：模板參數系統

### 支援的模板模式

| 模式 | 說明 | 觸發條件 |
|------|------|---------|
| `--template <file.docx>` | 套用使用者上傳的 Word 模板 | 使用者有公司模板 |
| `--builtin <n>` | 使用內建命名模板 | 選擇下方內建模板之一 |
| 無模板 | 使用預設乾淨樣式 | 未指定任何模板 |

### 內建模板清單

```
tech-spec   — 技術文件風格：深藍標題，Segoe UI 字型，適合 API / DB 文件
formal-doc  — 正式報告風格：深灰標題，Arial 字型，適合 PRD / SRD
user-guide  — 使用者手冊風格：友善綠色調，適合操作手冊 / 安裝手冊
minimal     — 極簡風格：黑白無色，適合需要客製化的文件
```

### 文件變數參數（Frontmatter）

每份文件頂端支援 YAML 變數替換：
```yaml
---
project: "{{project_name}}"
version: "{{version}}"
author: "{{author}}"
date: "{{date}}"
status: "{{status}}"
company: "{{company}}"
---
```

---

## 第四步：文件產出流程

### 4.1 生成 Markdown 內容

依序讀取對應的模板參考檔，填入專案資訊：

| 文件 | 參考模板路徑 | 說明 |
|------|------------|------|
| PRD | `references/prd-template.md` | 產品需求文件模板 |
| SRD | `references/srd-template.md` | 系統需求規格模板 |
| FRD | `references/frd-template.md` | 功能需求文件模板 |
| API | `references/api-spec-template.md` | API 規格模板 |
| DB  | `references/db-schema-template.md` | 資料庫文件模板 |
| OPS | `references/ops-manual-template.md` | 操作手冊模板 |
| INSTALL | `references/install-guide-template.md` | 安裝手冊模板 |

### 4.2 轉換為 Word

安裝依賴並使用轉換腳本：

```bash
pip install python-docx mistune --break-system-packages -q
```

呼叫轉換腳本（優先使用 markdown-to-word-template 技能）：

```bash
SCRIPT=/mnt/skills/user/markdown-to-word-template/scripts/md_to_word.py

# 使用使用者上傳的模板
python3 $SCRIPT /home/claude/docs/PRD.md \
  --template /mnt/user-data/uploads/template.docx \
  --mode clone \
  -o /home/claude/output/PRD.docx

# 使用內建模板
python3 $SCRIPT /home/claude/docs/PRD.md \
  --builtin tech-spec \
  -o /home/claude/output/PRD.docx
```

若 markdown-to-word-template 不可用，使用本技能的備用腳本：

```bash
python3 /home/claude/forward-engineering-docs/scripts/md_to_docx_fallback.py \
  /home/claude/docs/PRD.md -o /home/claude/output/PRD.docx
```

### 4.3 批次產出全套文件

```bash
for md_file in /home/claude/docs/*.md; do
  basename=$(basename "$md_file" .md)
  python3 $SCRIPT "$md_file" --builtin tech-spec \
    -o "/home/claude/output/${basename}.docx"
  echo "已產出 ${basename}.docx"
done
```

---

## 第五步：輸出結構

```
docs/
├── PRD.md          → PRD.docx
├── SRD.md          → SRD.docx
├── FRD.md          → FRD.docx
├── API-Spec.md     → API-Spec.docx
├── DB-Schema.md    → DB-Schema.docx
├── OPS-Manual.md   → OPS-Manual.docx
└── Install-Guide.md → Install-Guide.docx
```

產出後將所有 .docx 複製到 `/mnt/user-data/outputs/`，並用 `present_files` 工具提供下載。

---

## 第六步：品質檢查清單

- [ ] 每份文件開頭有版本資訊與修訂記錄表
- [ ] 所有章節標題編號一致
- [ ] 表格有完整的欄位說明
- [ ] API 文件每個 Endpoint 有 Request + Response 範例
- [ ] DB 文件有 ERD（Mermaid 語法）
- [ ] 安裝手冊有前置條件檢查清單
- [ ] 操作手冊有截圖佔位（以 `[截圖：XXX]` 標示）
- [ ] Word 輸出後驗證段落樣式正確

---

## 參考文件

| 文件 | 說明 | 使用時機 |
|------|------|---------|
| `references/prd-template.md` | PRD 完整模板 | 產出 PRD 前讀取 |
| `references/srd-template.md` | SRD 完整模板 | 產出 SRD 前讀取 |
| `references/frd-template.md` | FRD 完整模板 | 產出 FRD 前讀取 |
| `references/api-spec-template.md` | API 規格模板 | 產出 API 文件前讀取 |
| `references/db-schema-template.md` | DB Schema 模板 | 產出 DB 文件前讀取 |
| `references/ops-manual-template.md` | 操作手冊模板 | 產出操作手冊前讀取 |
| `references/install-guide-template.md` | 安裝手冊模板 | 產出安裝手冊前讀取 |
