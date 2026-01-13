# 系統資料庫架構與欄位詳解 (Database Schema Documentation)

本文件依據實體關聯圖 (ERD) 將資料表分為七大模組進行詳細說明。

---

## 1. 身份與工作區 (Identity / Workspace)
此模組管理使用者身份、登入關聯以及組織架構。

### users (使用者核心表)
| 欄位名稱 | 資料型態 | 屬性 | 說明 |
| :--- | :--- | :--- | :--- |
| `id` | uuid | **PK** | 系統內部的唯一識別碼 (Internal User ID)。 |
| `clerk_user_id` | string | | 暫刪 |
| `email` | string | | 使用者的登入與通知信箱。 |
| `system_role` | enum | | 平台權限：`admin` (管理員) 或 `user` (一般用戶)。 |
| `password_hash` | | | 密碼雜湊值 |
| `created_at` | timestamp | | 建立時間。(確定不要??) |

### workspaces (工作區)
| 欄位名稱 | 資料型態 | 屬性 | 說明 |
| :--- | :--- | :--- | :--- |
| `id` | uuid | **PK** | 工作區識別碼。 |
| `owner_user_id` | uuid | **FK** | 工作區擁有者 ID。 |
| `name` | string | | 工作區名稱。 |
| `type` | enum | | 組織類型：`company` (公司) 或 `individual` (個人)。 |
| `created_at` | timestamp | | 建立時間。(確定不要???) |

### workspace_members (工作區成員)
| 欄位名稱 | 資料型態 | 屬性 | 說明 |
| :--- | :--- | :--- | :--- |
| `id` | uuid | **PK** | 關聯記錄 ID。 |
| `workspace_id` | uuid | **FK** | 所屬工作區。 |
| `user_id` | uuid | **FK** | 成員 ID。 |
| `role` | enum | | 角色：`owner` (擁有者) 或 `member` (成員)。 |
| `joined_at` | timestamp | | 加入時間。 |

---

## 2. 專案與參與者 (Project / Participants)
此模組定義專案的基本資訊與參與人員。

### projects (專案主表)
| 欄位名稱 | 資料型態 | 屬性 | 說明 |
| :--- | :--- | :--- | :--- |
| `id` | uuid | **PK** | 專案 ID。 |
| `workspace_id` | uuid | **FK** | 所屬工作區。 |
| `client_id` | uuid | **FK** | 業主 (Client) ID。(暫刪) |
| `freelancer_id` | uuid | **FK** | 接案者 (Freelancer) ID，若尚未指派則為空。(暫刪) |
| `title` | string | | 專案標題。 |
| `description` | text | | 專案詳細描述。 |
| `status` | enum | | 狀態：`draft`, `open`, `in_progress`, `completed`, `dispute`, `archived`。 |
| `created_at` | timestamp | | 建立時間。 |
| `published_at` | timestamp | | 公開發布時間。(暫刪) |

### project_members (專案協作者)
| 欄位名稱 | 資料型態 | 屬性 | 說明 |
| :--- | :--- | :--- | :--- |
| `id` | uuid | **PK** | 關聯 ID。 |
| `project_id` | uuid | **FK** | 所屬專案。 |
| `user_id` | uuid | **FK** | 使用者 ID。 |
| `role` | enum | | 角色：`client`, `freelancer`, `collaborator`, `viewer`。 |
| `joined_at` | timestamp | | 加入時間。 |

---

## 3. 規格與合約 (Spec / Contract)
此模組利用 AI 生成規格，並管理合約簽署。

### specs (需求規格書)
| 欄位名稱 | 資料型態 | 屬性 | 說明 |
| :--- | :--- | :--- | :--- |
| `id` | uuid | **PK** | 規格書 ID。 |
| `project_id` | uuid | **FK** | 所屬專案。 (暫刪)|
| `raw_input_json` | jsonb | | 原始需求輸入 (目標/用戶/功能/NFR)。 |
| `spec_doc_md` | text | | AI 生成的 Markdown 規格書內容。 |
| `coverage_score` | int | | 規格完整度分數。 |
| `version` | int | | 版本號。 |
| `created_at` | timestamp | | 生成時間。 |

### contracts (合約)
| 欄位名稱 | 資料型態 | 屬性 | 說明 |
| :--- | :--- | :--- | :--- |
| `id` | uuid | **PK** | 合約 ID。 |
| `project_id` | uuid | **FK** | 所屬專案。 |
| `terms_md` | text | | 合約條款內容 (Markdown)。 |
| `status` | enum | | 狀態：`draft`, `signed`, `void`。 |
| `created_at` | timestamp | | 建立時間。 |
| `signed_at` | timestamp | | 雙方完成簽署的時間。 |

### contract_signatures (合約簽章)
| 欄位名稱 | 資料型態 | 屬性 | 說明 |
| :--- | :--- | :--- | :--- |
| `id` | uuid | **PK** | 簽章 ID。 |
| `contract_id` | uuid | **FK** | 對應合約。 |
| `user_id` | uuid | **FK** | 簽署人。 |
| `signature_hash` | string | | 簽署摘要/雜湊 (數位證據)。 |
| `signed_at` | timestamp | | 簽署時間。 |

---

## 4. 里程碑與驗收 (Milestone / Acceptance)
此模組控制專案進度與付款節點。

### milestones (里程碑)
| 欄位名稱 | 資料型態 | 屬性 | 說明 |
| :--- | :--- | :--- | :--- |
| `id` | uuid | **PK** | 里程碑 ID。 |
| `project_id` | uuid | **FK** | 所屬專案。 |
| `seq` | int | | 排序順序。 |
| `title` | string | | 標題。 |
| `amount` | decimal | | 金額。 |
| `currency` | string | | 幣別。 |
| `due_date` | timestamp | | 截止日。 |
| `status` | enum | | 狀態：`draft`, `awaiting_payment`, `funded`, `submitted`, `accepted`, `released`, `dispute`, `refunded`。 |
| `acceptance_template_id`| uuid | **FK** | 綁定的驗收模板 ID。 |

### acceptance_templates (驗收模板)
| 欄位名稱 | 資料型態 | 屬性 | 說明 |
| :--- | :--- | :--- | :--- |
| `id` | uuid | **PK** | 模板 ID。 |
| `project_id` | uuid | **FK** | 所屬專案。 |
| `name` | string | | 模板名稱。 |
| `description` | text | | 描述。 |

### acceptance_items (驗收細項)
| 欄位名稱 | 資料型態 | 屬性 | 說明 |
| :--- | :--- | :--- | :--- |
| `id` | uuid | **PK** | 項目 ID。 |
| `template_id` | uuid | **FK** | 所屬模板。 |
| `content` | string | | 驗收項目具體描述。 |
| `is_required` | boolean | | 是否為必填項目。 |

### acceptance_reviews (驗收審核紀錄)
| 欄位名稱 | 資料型態 | 屬性 | 說明 |
| :--- | :--- | :--- | :--- |
| `id` | uuid | **PK** | 審核 ID。 |
| `milestone_id` | uuid | **FK** | 對應里程碑。 |
| `reviewer_id` | uuid | **FK** | 審核者 ID。 |
| `result` | enum | | 結果：`pass` (通過) 或 `fail` (失敗)。 |
| `comment` | text | | 評語。 |
| `item_results` | jsonb | | 細項勾選結果快照。 |
| `created_at` | timestamp | | 建立時間。 |

---

## 5. 資金託管 (Escrow / Money Flow)
此模組處理實際的金流進出。

### payments (入金)
| 欄位名稱 | 資料型態 | 屬性 | 說明 |
| :--- | :--- | :--- | :--- |
| `id` | uuid | **PK** | 支付 ID。 |
| `milestone_id` | uuid | **FK** | 對應里程碑。 |
| `stripe_intent_id`| string | | Stripe PaymentIntent ID。 |
| `amount` | decimal | | 金額。 |
| `status` | enum | | 狀態：`pending`, `succeeded`, `failed`。 |
| `created_at` | timestamp | | 建立時間。 |

### payouts (出金)
| 欄位名稱 | 資料型態 | 屬性 | 說明 |
| :--- | :--- | :--- | :--- |
| `id` | uuid | **PK** | 撥款 ID。 |
| `milestone_id` | uuid | **FK** | 對應里程碑。 |
| `stripe_transfer_id`| string | | Stripe Transfer ID。 |
| `amount` | decimal | | 金額。 |
| `created_at` | timestamp | | 建立時間。 |

### refunds (退款)
| 欄位名稱 | 資料型態 | 屬性 | 說明 |
| :--- | :--- | :--- | :--- |
| `id` | uuid | **PK** | 退款 ID。 |
| `payment_id` | uuid | **FK** | 對應的原始支付記錄。 |
| `amount` | decimal | | 退款金額。 |
| `reason` | string | | 原因。 |
| `created_at` | timestamp | | 建立時間。 |

---

## 6. 交付與證據 (Delivery / Evidence)
此模組儲存開發者的產出與附件。

### deliverables (交付物提交)
| 欄位名稱 | 資料型態 | 屬性 | 說明 |
| :--- | :--- | :--- | :--- |
| `id` | uuid | **PK** | 交付記錄 ID。 |
| `milestone_id` | uuid | **FK** | 對應里程碑。 |
| `submitter_id` | uuid | **FK** | 提交者 ID。 |
| `repo_url` | string | | 程式碼倉庫連結。 |
| `commit_hash` | string | | Git Commit Hash (防篡改證據)。 |
| `description` | text | | 交付說明。 |
| `submitted_at` | timestamp | | 提交時間。 |

### attachments (附件檔案)
| 欄位名稱 | 資料型態 | 屬性 | 說明 |
| :--- | :--- | :--- | :--- |
| `id` | uuid | **PK** | 附件 ID。 |
| `deliverable_id` | uuid | **FK** | 對應的交付記錄。 |
| `file_url` | string | | 檔案下載連結。 |
| `file_hash_sha256`| string | | 檔案內容雜湊值 (防篡改)。 |

---

## 7. 討論、爭議與審計 (Discussion / Dispute / Audit)
此模組負責溝通、爭議處理與系統紀錄。

### comments (留言)
| 欄位名稱 | 資料型態 | 屬性 | 說明 |
| :--- | :--- | :--- | :--- |
| `id` | uuid | **PK** | 留言 ID。 |
| `project_id` | uuid | **FK** | 所屬專案。 |
| `user_id` | uuid | **FK** | 留言者 ID。 |
| `content` | text | | 內容。 |
| `created_at` | timestamp | | 建立時間。 |

### disputes (爭議)
| 欄位名稱 | 資料型態 | 屬性 | 說明 |
| :--- | :--- | :--- | :--- |
| `id` | uuid | **PK** | 爭議 ID。 |
| `milestone_id` | uuid | **FK** | 相關里程碑。 |
| `opener_id` | uuid | **FK** | 發起爭議者 ID。 |
| `reason` | string | | 爭議理由。 |
| `status` | enum | | 狀態：`open` (處理中), `resolved` (已解決)。 |

### audit_logs (審計日誌)
| 欄位名稱 | 資料型態 | 屬性 | 說明 |
| :--- | :--- | :--- | :--- |
| `id` | uuid | **PK** | 日誌 ID。 |
| `project_id` | uuid | **FK** | 相關專案。 |
| `user_id` | uuid | **FK** | 執行動作的使用者。 |
| `action_type` | string | | 動作類型。 |
| `payload` | jsonb | | 變更內容資料。 |
| `ip_address` | string | | 來源 IP。 |
| `created_at` | timestamp | | 記錄時間。 |