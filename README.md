# Sybil Lung Cancer Risk Prediction Pipeline

本專案實作了基於 Sybil 深度學習模型的胸部低劑量電腦斷層掃描（LDCT）肺癌風險預測管線。專案包含自動化 DICOM 影像品質控管、多病例批次推論、臨床結構化資料連結，以及後續的探索性統計分析與風險軌跡視覺化。

## 專案結構與檔案說明

* **`dicom\_sybil.ipynb`**: 主程式 Jupyter Notebook，涵蓋資料探索、DICOM QC、模型推論、資料合併與視覺化分析。
* **`requirements.txt`**: 專案 Python 依賴套件清單。
* **`metadata\_obtained/` \& `QC\_Decisions\_Report/`**: 存放 DICOM Metadata 與品質控管報告（自動產生）。
* **`inference\_results/`**: 存放 Sybil 批次推論結果與斷點續傳暫存檔（自動產生）。
* **`data/`**: 存放臨床結構化資料（如 `nlst\_780\_prsn\_idc\_20210527.csv`）及最終分析資料集（自動產生）。

## 系統需求與環境建置

本專案設計了自動化環境建置腳本，確保分析環境具備高度可重現性。

1. **硬體建議**：強烈建議使用支援 NVIDIA GPU 的環境（安裝 CUDA 11.8 或以上）。若系統無可用 GPU，程式將自動回退至 CPU 模式執行。
2. **一鍵安裝與啟動**：
請於 Windows 環境下點擊執行 **`run\_project.bat`**。該腳本將自動執行以下流程：

   * 下載並安裝 Portable Miniconda 至專案目錄（不影響系統原有環境）。
   * 建立獨立的 `sybil-env` Python 3.9 虛擬環境。
   * 根據 `requirements.txt` 安裝 PyTorch、Sybil 及其餘依賴套件。
   * 自動下載 `sybil\_ensemble` 模型權重，支援後續完全離線推論。
   * 自動啟動 Jupyter Lab 並開啟 `01\_dicom\_exploration.ipynb`。

## 管線執行步驟

請於 Jupyter Lab 中開啟 `01\_dicom\dicom_sybil.ipynb`，並依序執行各個 Cell：

### Step 1. 資料探索與品質控管 (DICOM QC)

* **資料載入**：透過彈出式 UI 選擇包含 TCIA NLST DICOM 檔案的根目錄，程式將自動掃描並擷取 Metadata。
* **雙重 QC 機制**：

  * **切片層級 (Level 1)**：自動排除非胸部 CT、Scout / Localizer 影像、切片厚度 > 5.0mm 及 DERIVED / MIP 衍生影像。
  * **Series 層級 (Level 2)**：執行 3D 空間 Z 軸幾何排序，檢查切片總數（至少 30 張）及切片間距一致性，嚴格篩選出符合 Sybil 推論標準的影像。

### Step 2. 批次推論與預測 (Batch Inference)

* **自動化推論**：載入 Sybil 預訓練模型進行多病例批次推論，輸出 Y1 至 Y6 的肺癌風險機率。
* **斷點續傳機制**：推論過程會即時將進度寫入 `inference\_results/wip\_checkpoint.csv`。若執行中斷，重新啟動時將自動跳過已成功預測的病例。

### Step 3. 臨床資料整合與分析資料集 (Clinical Data Linkage)

* **降維與代表作挑選**：針對同一病患同日的多次掃描，程式會依據 Convolution Kernel 優先級（STANDARD > LUNG > B30F > FC01）進行智慧篩選，確保每人每時間點僅保留一筆最佳紀錄。
* **計算追蹤終點**：自動將推論結果與 `nlst\_780\_prsn\_idc\_20210527.csv` 合併。利用 `candx\_days` 與 `scr\_days` 計算精確的肺癌發生時間；針對未罹癌者，則處理 `fup\_days` 與 `canc\_free\_days` 轉換為觀察終點，最終匯出完整的分析資料集。

### Step 4. 探索性分析與視覺化 (EDA \& Visualization)

* **描述性統計**：產出 Y1、Y3、Y6 風險預測的 Mean、SD、Median、IQR 及極值統計表。
* **個人風險曲線 UI**：提供互動式下拉選單，使用者可選擇特定病患與掃描日期，動態繪製 Y1–Y6 風險折線圖。
* **風險變化軌跡 (Spaghetti Plot)**：視覺化不同病患隨追蹤時間點（TIMEPOINT）的 Y6 預測風險走勢，並依據實際臨床診斷結果（Cancer / No Cancer）分色。
* **ROC 曲線評估**：針對「第一年內發生肺癌」計算並繪製 ROC 曲線（含 AUC 數值），以評估模型在現有樣本中的鑑別力。

## AI 與外部工具使用聲明

根據評估規範，本專案在開發與撰寫過程中使用以下工具協助：

* **使用的 AI 工具**：(例如：ChatGPT, Gemini, GitHub Copilot 等)
* **主要用途**：輔助撰寫批次處理與自動化環境建置腳本、優化 Pandas 資料處理效能，以及潤飾英文報告段落
* **本人確認之重要決策**：
  * 所有針對 DICOM 的 QC 邏輯（如 Series 切片間距檢查）皆由本人根據 Sybil 模型限制與文獻要求進行設計與確認。
  * 臨床資料（如 `candx\_days` 與 `fup\_days`）的計算規則與未知結局（Censoring）的處理邏輯，由本人理解資料字典後親自制定並驗證其正確性。
  * 外部驗證的論文研究設計與結果判讀，均代表本人的科學推理與觀點。



│  dicom_sybil.ipynb
│  analysis of lung cancer prediction using sybil.docx
│  analysis of lung cancer prediction using sybil.pdf
│  analysis of lung cancer prediction using sybil3.pdf
│  README.md
│  requirements.txt
│  run_project.bat
│  
├─data
│      analysis_dataset.csv
│      
├─inference_results
│      final_results_20261015_031939.csv
│      results_20261015_031939.csv
│      
├─metadata_obtained
│      dicom_metadata_20261015_030703.csv
│      
├─nst-ogfile
│      nlst_780_canc_idc_20210527.csv
│      nlst_780_ctabc_idc_20210527.csv
│      nlst_780_ctab_idc_20210527.csv
│      nlst_780_prsn_idc_20210527.csv
│      nlst_780_screen_idc_20210527.csv
│      
└─QC_Decisions_Report
        dicom_QC_Decisions_Report_20261015_030703.csv
        series_QC_20261015_030703.csv
