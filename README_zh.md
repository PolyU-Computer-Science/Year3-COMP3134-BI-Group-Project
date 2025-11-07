# COMP3134 BI & CRM 專案 – 第6組

## 專案結構
- `Group_6.ipynb`：主分析 Notebook（數據載入 → 驗證 → 轉換 → EDA → 特徵工程 → 建模 → 洞察 → 結果匯出）
- `sales_6.csv`、`customers_6.csv`、`products_6.csv`：原始數據集
- `rfm_clusters.csv`：帶有分群標籤的客戶 RFM 特徵匯出
- `cluster_profile.csv`：各分群的特徵均值
- `merged_sample.csv`：交易明細樣本（前2000行）供參考

## 環境設置
### 建議版本
- Python >= 3.9

### 安裝依賴
在 Windows PowerShell 下執行：
```powershell
pip install -r requirements.txt
```

如需使用關聯規則分析：
```powershell
pip install mlxtend
```

## 可重現步驟
1. 將三個原始 CSV 文件放入 Notebook 同一資料夾（或 data 文件夾）。
2. 打開並自上而下運行 `Group_6.ipynb`（建議 Restart & Run All）。
3. 結果文件（如 `rfm_clusters.csv`、`cluster_profile.csv`）將在分析末端自動生成。

### 已安裝套件（核心片段）——由當前環境生成
下面展示開發與測試 Notebook 時使用環境中的核心套件片段（僅包含 Notebook 使用的主要套件）。您可以使用文後命令在本機重新生成相同片段。

```
matplotlib==3.9.2
mlxtend==0.23.4
numpy==1.26.4
pandas==2.2.2
scikit-learn==1.5.1
seaborn @ file:///... (請以 `pip freeze` 結果為準)
```

在 Windows PowerShell 中重新生成該片段的命令：

```powershell
pip freeze | findstr /R "pandas numpy scikit-learn matplotlib seaborn python-pptx mlxtend" > requirements_core.txt
cat requirements_core.txt
```

若要鎖定精確版本以利重現，請將 `requirements_core.txt` 中的行複製至 `requirements.txt`，然後在乾淨的虛擬環境中運行 `pip install -r requirements.txt`。

## 數據處理流程
| 階段 | 主要操作 |
|------|----------|
| 驗證 | 格式檢查（ID）、缺失值、日期解析 |
| 轉換 | 商品列表拆分、商品/客戶合併、計算明細與發票金額 |
| 特徵工程 | RFM（最近一次消費、消費頻率、累積金額）+ 品類多樣性 + 平均客單價 + 偏好商場 |
| 建模 | KMeans 分群（用 silhouette 分數選 k） |
| EDA | 商場/品類銷售、客單價分佈、人口統計分佈 |
| 匯出 | 分群標籤與特徵均值保存為 CSV |

## 建模細節
- 標準化：`StandardScaler`
- 演算法：`KMeans`（自動 n_init）
- k 選擇：silhouette 分數，範圍2–6
- 分群分析：各群特徵均值

## 分群解讀（範例模板）
| 分群 | 特徵 | 建議策略 |
|------|------|----------|
| 0 | 高金額、低 Recency（近期活躍） | VIP提前體驗/高級會員方案 |
| 1 | 高頻率、中等金額 | 組合促銷提升客單價 |
| 2 | 低頻率、低金額 | 新客引流/首購優惠 |

請根據實際 profile 數值調整。

## 行銷策略草案
- 獲客策略：針對低頻群推 Welcome Bundle，結合前2大交叉品類。
- 留客策略：高金額群推分層會員福利（新品搶先、電子配件加購）。

## 延伸分析
- 關聯規則（Apriori/FP-Growth）挖掘商品組合
- 時序預測（月營收）可用 SARIMA 或 Prophet
- 分類模型預測未來高價值客戶

## 清理決策（紀錄）
以下列出 `Group_6.ipynb` 中實際套用的資料清理與回退規則，便於審閱者重現或按需調整。

- 缺少 `LineAmount` / 價格缺失：當 `LineAmount` 或 `Price` 欄位缺失時，Notebook 以每筆明細 `1.0` 為替代值，確保發票彙總可成功執行。程式碼參考：`merged['LineAmount'] = 1.0`。

- `ProductList`（多商品）拆分：若交易列含以逗號分隔的商品清單（欄名 `Product id list`），先以 `str.split(',')` 拆分，再使用 `explode()` 展開為多列，並用 `str.strip()` 去除空白；展開後與 `products` 表依商品 id 合併以取得商品屬性。

- 發票總額 (`InvoiceAmount`) 缺失：若原資料沒有 `InvoiceAmount`，會以 `LineAmount` 按 `Invoice no` 做 group sum 重新計算並回填，確保每張發票有一致的總額。

- 行為特徵缺值填補（如 `AvgInterpurchaseDays`、`MallDiversity`、`PaymentDiversity`、`CategoryConcentration`）：以該特徵在客戶間的中位數填補（保守策略）。程式碼參考：`rfm_enhanced[c] = rfm_enhanced[c].fillna(rfm_enhanced[c].median())`。

- 離群值處理與轉換：
	- 對金額類特徵採用 `log1p` 轉換（例如 `MonetaryLog = np.log1p(Monetary)` 與 `AvgInvoiceLog = np.log1p(AvgInvoice)`）來降低偏態影響，而非直接截斷。
	- 若偏好使用截斷，可在 log 前對金額做 99 百分位上限截斷，例如：`cap = df['Monetary'].quantile(0.99); df['Monetary'] = df['Monetary'].clip(upper=cap)`。

- 日期解析：`Invoice date` 使用 `format='%d/%m/%Y'` 並設 `errors='coerce'`，格式錯誤的日期會成為 `NaT`，建議在計算 RFM 前檢查是否有 `NaT` 並做人工處理或排除。

以上規則旨在採用保守處理（中位數填補、log1p 轉換），若您的資料有大量缺值或極端值，請在 Notebook 中說明改動並重新執行分析。

---

## 在乾淨環境中運行 Notebook（推薦命令）
以下為在 Windows PowerShell 下建立虛擬環境、安裝相依套件並以 `nbconvert` 非互動方式執行 Notebook 的建議指令。若執行時間過長或資料量較大，請調整 timeout。

```powershell
# 建立並啟用虛擬環境（PowerShell）
python -m venv .venv
.\.venv\Scripts\Activate.ps1

# 更新 pip 並安裝需求
python -m pip install --upgrade pip
pip install -r requirements.txt

# 執行 Notebook（自上而下）。必要時增加 --ExecutePreprocessor.timeout 的秒數。
jupyter nbconvert --to notebook --execute Group_6.ipynb --output executed_Group_6.ipynb --ExecutePreprocessor.timeout=600
```

預期執行時間與記憶體需求（近似值）：
- 以此次開發用的小型資料（數千筆交易）為例：在一般筆電（4 核、8 GB RAM）上約需 1–5 分鐘；峰值記憶體通常低於 2–4 GB。
- 若資料量上升到數萬或數十萬筆，執行時間可能擴展為 10–30+ 分鐘，且記憶體需求可能超過 8 GB。此時建議增加機器記憶體或採分批/取樣方式執行。

若 `nbconvert` 在某步驟逾時（timeout），建議以互動方式在 Jupyter 中增大 timeout 或逐段執行（Restart & Run All）。

## 常見問題排查
| 問題 | 原因 | 解決方法 |
|------|------|----------|
| 日期欄全 NaT | 地區/格式不符 | 檢查格式 `%d/%m/%Y` |
| 分群效果差 | k 選擇不當/特徵不足 | 重跑 k 範圍或增加特徵 |
| 記憶體不足 | 數據量大 | 取樣或分批讀取 |

## 組員信息
請在此處填寫組號及所有成員學號與分工，並於提交前把下面的範本替換為實際資訊。

- 組號：第6組
- 課程：COMP3134 BI & CRM
- 學期：例如 2025/26 Semester 1
- GitHub 倉庫： https://github.com/PolyU-Computer-Science/Year3-COMP3134-BI-Group-Project

- 組員範例（請用實際姓名與學號替換）：
	- 學生A (A1234567) — 角色：Notebook 與資料清理，聯絡：youremail@domain.edu
	- 學生B (B2345678) — 角色：EDA 與視覺化，聯絡：youremail@domain.edu
	- 學生C (C3456789) — 角色：建模與分群，聯絡：youremail@domain.edu
	- 學生D (D4567890) — 角色：簡報與報告撰寫，聯絡：youremail@domain.edu

- 報告與簡報分工（4 分鐘示例）：
	- Speaker 1: 0:00–1:30 — 專案與資料介紹
	- Speaker 2: 1:30–2:30 — 方法與建模結果
	- Speaker 3: 2:30–3:30 — 洞察與行銷策略
	- Speaker 4: 3:30–4:00 — 結論與 Q&A 準備

請在最終提交前把占位資訊換成真實姓名、學號與 email。

## Personas（購買者角色說明）
Notebook (`Group_6.ipynb`) 會產生 `output/personas_table.csv`，此檔案將每個分群簡潔化為 persona 並提供建議的獲客與留客策略。下面為中文 persona 範本與說明：

- 高價值（近期活躍）
	- 特徵：高於中位數的消費（Monetary），近期有購買（Recency 低）。通常為忠實、高消費客群。
	- 獲客建議：非主要對象；可透過推薦計畫或邀請現有 VIP 推薦相似使用者。
	- 留客建議：VIP 方案、搶先看新品、個人化高階促銷、專屬組合包。
	- 建議 KPI：平均訂單金額、留存率、客戶終身價值（CLTV）。

- 高頻率但低金額
	- 特徵：購買頻率高但累積金額低，可能常買低價或配件類。
	- 獲客建議：推單次加購或交叉銷售（例如：加購配件打 10% 折扣）。
	- 留客建議：消費達標回饋、點數機制或套裝折扣以提高客單價。
	- 建議 KPI：單次訂單品項數、客單價、Upsell 轉換率。

- 低價值 / 休眠
	- 特徵：購買頻率低、消費金額低、距上次購買時間較長（Recency 高）。
	- 獲客建議：Welcome / 再激活優惠 — 時限性折扣或由關聯規則挑選的跨品類歡迎組合。
	- 留客建議：重啟購買的 Email/SMS 推送 + 個人化推薦與優惠券；贏回活動（win-back）。
	- 建議 KPI：再啟動率、贏回活動轉換率、再啟動的 CAC（成本）。

說明與提醒：
- persona 標籤是基於群體中點（中位數）啟發式判定，請根據 `cluster_profile.csv` 或 `cluster_profile_enhanced.csv` 裡的實際數值微調標籤與策略。
- 若有產生 `assoc_rules_top.csv`，可用裡面的 antecedent/consequent 組合直接填到簡報的 cross-sell 範例中（更具體、說服力更強）。

如需我代為把組員名單填入（你提供姓名與學號清單即可），或把 personas 轉成一頁 PPT 範本，也可告訴我我來幫你完成。

## 版權與學術誠信
本專案僅供學術用途，請遵守課程規範。