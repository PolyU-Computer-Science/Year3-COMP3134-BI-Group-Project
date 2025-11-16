# COMP3134 BI & CRM Project – Group 6

RetailX Customer Analytics and Promotion Strategy – Project Documentation

This README reflects the current scope of `Group_6.ipynb`. The notebook focuses on integrating RetailX data, running exploratory analytics, computing RFM metrics, and applying KMeans clustering.

---

## 1. Project Overview

**Business Scenario**

RetailX operates several shopping malls. It wants to consolidate invoice, customer, and product data so that BI analysts can:

- Spot behaviour patterns across malls, product categories, and payment channels
- Quantify customer value using Recency–Frequency–Monetary metrics
- Build actionable customer segments for CRM and promotion planning

**Current Objectives**

- Clean and merge the three source tables into a trustworthy transaction view
- Produce descriptive visuals that highlight mall performance, category mix, payment mix, and seasonality
- Derive RFM features, select the number of clusters via silhouette analysis, and interpret KMeans segments with PCA and z-score charts
- Package the merged dataset and figures so slides can be updated quickly

---

## 2. Repository Structure

```
Group Project/Jupyter/
├── Group_6.ipynb          # Main analysis notebook
├── README.md              # This document
├── requirements.txt       # Python dependencies
├── data/
│   ├── sales_6.csv
│   ├── customers_6.csv
│   └── products_6.csv
└── output/
    ├── csv/
    │   └── merged_data.csv
    └── image/
        ├── age_distribution_by_gender.png
        ├── cluster_rfm_means_zscore.png
        ├── customer_clusters_pca.png
        ├── invoice_amount_distribution.png
        ├── payment_method_counts.png
        ├── sales_by_category.png
        ├── sales_by_mall.png
        └── seasonal_sales_pattern.png
```

---

## 3. Data Pipeline

**1. Load & Validate Inputs**

- Paths are created with `pathlib.Path` so reruns do not fail if folders already exist.
- All three CSVs are loaded with `pandas`; `.info()` and `.isna().sum()` calls surface data-quality issues.
- Regular expressions confirm invoice (`INV\d{5}`), customer (`C\d{5}`), and product (`P\d{5}`) ID formats; invalid rows trigger console warnings.
- Invoice dates are parsed via `pd.to_datetime(..., format="%d/%m/%Y", errors="coerce")` so downstream recency math is reliable.

**2. Explode Product Lists & Merge**

- Each invoice row contains a comma-separated list of product IDs; `.str.split(',').explode()` converts this into line items.
- Product attributes and customer demographics are joined in, yielding a unified dataset with price, mall, payment method, age, and gender on every row.
- The merged line-level table is exported to `output/csv/merged_data.csv` for reference outside the notebook.

**3. Exploratory Analytics**

All visuals are saved to `output/image/` for direct slide use:

- `sales_by_mall.png`: Total revenue by mall highlights the top-performing location.
- `sales_by_category.png`: Category breakdown shows fashion and electronics dominating sales.
- `payment_method_counts.png`: Bar chart of how customers pay (credit card vs. e-wallet vs. cash).
- `invoice_amount_distribution.png`: Distribution of invoice totals to spot big-ticket purchases.
- `age_distribution_by_gender.png`: Boxplots illustrating the age mix across genders.
- `seasonal_sales_pattern.png`: Monthly trend line to reveal peak seasons.

---

## 4. RFM Features & KMeans Segmentation

1. **RFM Calculation**

   - Snapshot date = max invoice date + 1 day.
   - Group by customer to derive Monetary (sum of prices), Frequency (unique invoices), and Recency (days since last purchase).
   - The resulting `rfm` dataframe stays inside the notebook for further modeling.
2. **Scaling & Model Selection**

   - `RobustScaler` handles the skewed Monetary values without requiring a log transform.
   - KMeans models with `k` ∈ [2, 8] are compared using silhouette scores; the notebook currently selects **best_k = 3**.
   - Final clustering uses `KMeans(n_clusters=best_k, random_state=42, n_init=20)` and appends the cluster label to `rfm`.
3. **Diagnostics & Interpretation**

   - `customer_clusters_pca.png` plots the clusters on the first two PCA components with centroids highlighted.
   - `cluster_rfm_means_zscore.png` compares z-scored Recency, Frequency, and Monetary means across clusters.
   - A markdown cell summarises the business interpretation:
     - Cluster 0 – infrequent, low-spend shoppers
     - Cluster 1 – steady mid-tier spenders
     - Cluster 2 – frequent, high-value customers

---

## 5. Environment Setup & Execution

- **Python**: 3.9 or later is recommended.
- **Install dependencies**:

  ```powershell
  pip install -r requirements.txt
  ```
- **Run the analysis**:

  ```powershell
  jupyter nbconvert --to notebook --execute Group_6.ipynb --output executed_Group_6.ipynb --ExecutePreprocessor.timeout=600
  ```

  or open the notebook in JupyterLab/VS Code and use "Restart & Run All".
- **Validate outputs**: reruns should refresh `output/csv/merged_data.csv` and overwrite the eight PNG files listed above. If the files do not update, check that the script has permission to write to the OneDrive directory.

---

## 6. Key Notes & Troubleshooting

| Issue                                  | Likely Cause                                    | Suggested Fix                                                                                    |
| -------------------------------------- | ----------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| Dates become `NaT`                   | Source file not in `DD/MM/YYYY`               | Update the `format` argument or pre-convert the column before loading.                         |
| Missing `output/csv/merged_data.csv` | Product list explode failed (e.g., null string) | Ensure `Product id list` is non-null; apply `.fillna('')` before splitting if necessary.     |
| Silhouette scores all low (<0.2)       | Features not scaled or too few transactions     | Confirm `RobustScaler` ran and consider trimming outliers before clustering.                   |
| PNGs not saved                         | `output/image/` removed or read-only          | Recreate the folder (`Path(...).mkdir(parents=True, exist_ok=True)`) and rerun plotting cells. |

---

## 7. Scope Boundaries

- The current notebook stops after RFM-based clustering. Persona write-ups, association rules, and additional CSV/model artifacts that appeared in older documentation are intentionally omitted until corresponding code exists.
- Any future features (e.g., Apriori mining or persona export) should update this README again so the listed deliverables always match the notebook outputs.
