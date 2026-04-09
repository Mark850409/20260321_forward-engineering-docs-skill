# 順向工程文件產出（forward-engineering-docs）

從需求或構想出發，依標準模板產出 **PRD／SRD／FRD／API／DB／操作手冊／安裝手冊** 等專案文件，並可匯出 **Markdown** 與 **Word（.docx）** 的 **Agent Skill**。

## 核心理念

由「目標與需求」系統化生成文件套件；支援參數化模板與多格式輸出，一次設定即可批次產出。

## 可產出文件

| 代碼 | 全名 | 主要受眾 |
|------|------|----------|
| PRD | 產品需求文件 | PM、設計、開發 |
| SRD | 系統需求規格 | 架構、Tech Lead |
| FRD | 功能需求文件 | 開發、測試 |
| API | API 規格文件 | 前後端 |
| DB | 資料庫設計文件（含 Schema／ERD） | 後端、DBA |
| OPS | 操作手冊 | 終端用戶、客服 |
| INSTALL | 安裝手冊 | DevOps、開發 |

使用者若指定「**全套文件**」，預設產出上述 **7 份**（見 [`SKILL.md`](SKILL.md)）。

## 產出前需確認的資訊

專案名稱、簡介、目標用戶、要產哪些文件為必填；模板偏好與輸出格式（Markdown／Word／兩者）為選填。資訊充足時可依「快速啟動模式」直接產出。

## 模板與 Word 樣式

| 模式 | 說明 |
|------|------|
| `--template <file.docx>` | 使用者提供的 Word 模板（建議搭配 `markdown-to-word-template` 的 clone 模式） |
| `--builtin <名稱>` | 內建命名樣式 |
| 無模板 | 乾淨預設樣式 |

**內建樣式名稱**（與 `SKILL.md` 一致）：`tech-spec`、`formal-doc`、`user-guide`、`minimal`。

每份 Markdown 頂端可含 YAML **frontmatter**，支援變數如 `project`、`version`、`author`、`date`、`status`、`company` 等（見 [`SKILL.md`](SKILL.md)）。

## Markdown 模板（產出前必讀）

| 文件 | 參考檔 |
|------|--------|
| PRD | [`references/prd-template.md`](references/prd-template.md) |
| SRD | [`references/srd-template.md`](references/srd-template.md) |
| FRD | [`references/frd-template.md`](references/frd-template.md) |
| API | [`references/api-spec-template.md`](references/api-spec-template.md) |
| DB | [`references/db-schema-template.md`](references/db-schema-template.md) |
| OPS | [`references/ops-manual-template.md`](references/ops-manual-template.md) |
| INSTALL | [`references/install-guide-template.md`](references/install-guide-template.md) |

## 轉成 Word

1. 安裝：`python-docx`、`mistune`（見 `SKILL.md`）。  
2. **優先**使用同工作區 [`markdown-to-word-template`](../markdown-to-word-template) 的 `scripts/md_to_word.py`。  
3. 若該技能／腳本不可用，可使用本技能備援腳本 [`scripts/md_to_docx_fallback.py`](scripts/md_to_docx_fallback.py)（同樣支援內建樣式與變數替換）。

`SKILL.md` 中的 `/mnt/...`、`/home/claude/...` 路徑僅對應特定執行環境，本機請改為實際路徑。

## 輸出檔名對應（摘要）

```
docs/PRD.md           → PRD.docx
docs/SRD.md           → SRD.docx
docs/FRD.md           → FRD.docx
docs/API-Spec.md      → API-Spec.docx
docs/DB-Schema.md     → DB-Schema.docx
docs/OPS-Manual.md    → OPS-Manual.docx
docs/Install-Guide.md → Install-Guide.docx
```

完整流程、批次指令與品質檢查清單見 [`SKILL.md`](SKILL.md)。

## 目錄結構

```
forward-engineering-docs-skill/
├── README.md
├── SKILL.md
├── references/          # 各類文件 Markdown 模板
└── scripts/
    └── md_to_docx_fallback.py   # Markdown → docx 備援
```

## 何時啟用

例如：撰寫 PRD／SRD／FRD、API／DB 文件、操作／安裝手冊、新專案文件套件、套公司模板、輸出 docx 等。完整觸發語句見 [`SKILL.md`](SKILL.md) 的 frontmatter `description`。

## 授權與貢獻

若本技能隸屬於上層儲存庫，請依該儲存庫的授權與貢獻指南為準。
