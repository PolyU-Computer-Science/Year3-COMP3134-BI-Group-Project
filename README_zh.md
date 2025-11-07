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

## 常見問題排查
| 問題 | 原因 | 解決方法 |
|------|------|----------|
| 日期欄全 NaT | 地區/格式不符 | 檢查格式 `%d/%m/%Y` |
| 分群效果差 | k 選擇不當/特徵不足 | 重跑 k 範圍或增加特徵 |
| 記憶體不足 | 數據量大 | 取樣或分批讀取 |

## 組員信息
請在此處填寫組號及所有成員學號。

## 版權與學術誠信
本專案僅供學術用途，請遵守課程規範。