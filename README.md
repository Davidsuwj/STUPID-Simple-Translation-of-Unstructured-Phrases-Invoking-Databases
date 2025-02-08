## 版本資訊
**Versions:**

**V1** : 2025-2-8  **Template, Views**設計完成尚未整合。

---

# STUPID-Simple-Translation-of-Unstructured-Phrases-Invoking-Databases

## STUPID 的涵義與目的

**STUPID** 是本研究提出的創新解決方案的縮寫，其完整含義為 **Simple Translation of Unstructured Phrases Invoking Databases**。  
本方案旨在解決企業級 **Data Mart** 中因表格數量激增與 Schema 多變所帶來的查詢挑戰，並彌補傳統 **Text-to-SQL** 方案在多表、多 Schema 環境下的不足。主要目標包括：

- **簡單易用**：讓非技術使用者無需掌握 SQL 語法也能輕鬆查詢數據。
- **動態 Schema 選取**：根據使用者選擇的表格，自動載入並注入必要 Schema 資訊，降低 Context 超載。
- **自動 SQL 生成與錯誤修正**：利用 LLM 自動生成 SQL 查詢，並在執行失敗時自動檢查並修正錯誤，降低人工干預需求。
- **自然語言回覆**：將 SQL 查詢結果轉換為自然語言回覆，完全隱藏 SQL 細節，提升使用體驗。
- **企業級擴展性**：適應多表、多 Schema 的動態數據環境，滿足大規模 Data Mart 的需求。

---

## 1. 摘要（Abstract）

隨著企業級 **Data Mart** 的發展，資料庫表格數量持續增長，傳統的 **Text-to-SQL** 方案難以擴展至多表、多 Schema 環境。現有研究（如 **Spider Dataset**）大多聚焦於靜態 Schema，且 **生成 SQL 查詢後仍需使用者進行手動修正**，影響查詢體驗與準確性。

本研究提出一種 **無需使用者理解 SQL 語法的 Text-to-SQL 解決方案**，透過：

1. **動態 Schema 選取機制**
2. **上下文 Schema 內嵌式 SQL 生成**
3. **SQL 查詢結果 LLM 轉換為自然語言回覆**
4. **SQL 執行失敗時的動態錯誤修正與查詢優化**
5. **全流程隱藏 SQL，使用者僅接收自然語言回應**

該方案相比傳統 **Text-to-SQL**，在企業級環境下具備 **更高的擴展性、準確率與使用者體驗**。

---

## 2. 研究動機（Introduction）

- **Text-to-SQL 的應用潛力**  
  透過自然語言查詢資料，無需 SQL 知識即可獲取資訊，提高企業數據利用率。

- **現有方案的局限性**：
  - **Spider Dataset**：僅適用於靜態 Schema，難以擴展到企業級 Data Mart。
  - **Vanna AI + RAG**：需建立大量問答對應，**成本高且難以即時適應動態 Schema**。
  - **傳統 Text-to-SQL**：使用者需自行檢視 SQL，執行錯誤時無法自動修正，使用體驗較差。

- **企業級 Data Mart 的挑戰**：
  - **數百張表格，結構多變**
  - **非技術人員無法手動修正 SQL**
  - **需要降低 SQL 生成錯誤並提升查詢體驗**

本研究提出一種**完全 SQL 隱藏化**的 Text-to-SQL 方案，使非技術使用者也能流暢查詢數據。

---

## 3. 相關研究（Related Work）

### 3.1. Spider Dataset & 靜態 Schema 方案

- 多數研究使用 **Spider Dataset** 進行 Text-to-SQL 測試，但該數據集：
  - **固定 Schema，無法動態適應不同表格**
  - **查詢範圍受限，難以擴展至企業級應用**

### 3.2. Retrieval-Augmented Generation（RAG）方案

- **Vanna AI** 採用 RAG（向量檢索增強生成），透過 **歷史問答匹配 SQL 查詢**：
  - 優點：能提供較準確的 SQL 查詢
  - 缺點：
    - **需蒐集大量問答數據，時間與成本高**
    - **無法處理全新查詢，擴展性受限**

### 3.3. SQL 生成 & LLM

- GPT-4、CodeX（OpenAI）等模型具備 **Text-to-SQL 能力**，但：
  - **執行錯誤時無法自行修正**
  - **SQL 查詢結果仍需人工解讀**
  - **非技術人員難以直接使用**

---

## 4. 研究方法（Methodology）

本研究提出 **企業級高擴展性 Text-to-SQL 方案**，涵蓋 **SQL 生成、錯誤修正、自然語言回覆** 三大核心模組。

### 4.1. 系統架構

**完整流程：**

1. **使用者選擇所需的表格名稱，後端自動載入對應的 Schema 並準備相關表格資訊（Schema 選取）**
2. **Schema 內嵌於 Prompt，LLM 生成 SQL 查詢**
3. **執行 SQL，取得查詢結果**
4. **LLM 解析查詢結果並以自然語言回覆**（隱藏 SQL）
5. **SQL 執行失敗時，系統自動檢查錯誤並修正**
6. 若再次失敗，會詢問使用者相關欄位與條件是否正確？
7. **確認修正後再次執行 SQL，最終回覆相關數據給使用者**

<img src="https://github.com/user-attachments/assets/b3ecec38-b35a-439d-a09d-64277b4e9735" width="1800" height="200"/>

*（上圖示意：查詢流程中各階段的操作）*

### 4.2. 核心技術

- **動態 Schema 植入（Context Injection）**
  - **使用者選擇的表格動態載入**，降低 Context 超載
  - SQL Prompt 僅包含 **必要表格資訊**，提升查詢準確率

- **錯誤檢測 & SQL 修正**
  - **語法檢查（Syntax Check）**
  - **Schema 比對（Schema Validation）**
  - **使用者條件確認（Condition Validation）**
  - **錯誤修正後重新生成 SQL**

- **SQL 結果轉換為自然語言**
  - **LLM 解析數據後，自然語言回覆**
  - **隱藏 SQL，讓使用者完全無感**

---

## 5. 實驗與評估（Experiments & Evaluation）

### 5.1. 測試數據集

- **企業級 Data Mart**
  - 超過 **100+ 張表**
  - 涵蓋 **多領域（財務、行銷、庫存等）**
  - **動態 Schema，每次查詢不同表格組合**

---