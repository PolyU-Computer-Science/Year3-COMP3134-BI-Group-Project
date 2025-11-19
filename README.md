# COMP3134 BI & CRM Project – Group 6

RetailX Customer Analytics and Promotion Strategy – Project Documentation

This README reflects the current scope of `Group_6.ipynb`. The notebook focuses on integrating RetailX data, running exploratory analytics, computing RFM metrics, and applying KMeans clustering.

---

## 1. Repository Structure

```
Group Project/Jupyter/
├── Group_6.ipynb                  # Main analysis notebook (data process, EDA, RFM, KMeans)
├── README.md                      # This document
├── requirements.txt               # Python dependencies (pandas, numpy, matplotlib, seaborn, scikit-learn)
├── data/                          # Raw input data
│   ├── sales_6.csv                # Transaction-level invoice data (invoice no, customer id, product list, date, mall)
│   ├── customers_6.csv            # Customer profile data (gender, age, payment method, etc.)
│   └── products_6.csv             # Product catalog (product id, category, price)
└── output/                        # Notebook-generated outputs
    ├── csv/
    │   └── merged_data.csv        # Exploded & merged transaction data (joined with customers & products)
    └── image/                     # All figures for slides
        ├── age_distribution_by_gender.png
        ├── cluster_rfm_means_zscore.png
        ├── customer_clusters_pca.png
        ├── invoice_amount_distribution.png
        ├── invoice_amount_distribution_by_cluster.png
        ├── payment_method_counts.png
        ├── sales_by_category.png
        ├── sales_by_mall.png
        ├── seasonal_sales_pattern.png
        ├── total_price_by_category_and_cluster.png
        ├── total_price_by_payment_method_and_cluster.png
        └── total_price_by_shopping_mall_and_cluster.png
```

---

## 2. Environment Setup & Execution

- **Python**: 3.9 or later is recommended.
- **Install dependencies**:

  ```powershell
  pip install -r requirements.txt
  ```
- **Run the analysis**:

  ```powershell
  jupyter nbconvert --to notebook --execute Group_6.ipynb --output executed_Group_6.ipynb --ExecutePreprocessor.timeout=600
  ```

  or open the notebook in **JupyterLab/VS Code** and use "Restart & Run All".
- **Validate outputs**: reruns should refresh `output/csv/merged_data.csv` and overwrite the eight PNG files listed above. If the files do not update, check that the script has permission to write to the directory.

---

## 3. Data Pipeline

**1. Loading and Validating Input**

- Use `pathlib.Path` to establish paths, so reruns will not fail even if the folder already exists.
- All three CSV files are loaded using `pandas`; calls to `.info()` and `.isna().sum()` will expose data quality issues.
- Regular expressions are used to validate the format of invoice (`INV\d{5}`), customer (`C\d{5}`), and product (`P\d{5}`) IDs; invalid rows will trigger console warnings.
- Use `pd.to_datetime(..., format="%d/%m/%Y", errors="coerce")` to parse invoice dates to ensure reliable downstream date calculations.

**2. Expand and Merge Product Lists**

- Each invoice line contains a comma-separated list of product IDs; `.str.split(',').explode()` converts them into line items.
- Product attributes and customer demographics are merged to produce a unified dataset, with each line containing price, store, payment method, age, and gender.
- The merged row-level table is exported to `output/csv/merged_data.csv` for external reference outside the notebook.

**3. Exploratory Analysis**

All visualizations are stored in `output/image/` and can be used directly as slides:

- `sales_by_mall.png`: Total revenue broken down by store, highlighting the top-performing stores.
- `sales_by_category.png`: Category breakdown showing that fashion and electronics sales dominate.
- `payment_method_counts.png`: A bar chart of customer payment methods (credit card, mobile payment, cash).
- `invoice_amount_distribution.png`: Invoice amount distribution chart, used to identify large purchases.
- `age_distribution_by_gender.png`: Box plot, showing the age composition by gender.
- `seasonal_sales_pattern.png`: Monthly trend line, revealing peak sales seasons.
- `total_price_by_shopping_mall_and_cluster.png`: Total spending by shopping mall and cluster, showing which customer groups contributed the most revenue in each shopping mall.
- `total_price_by_category_and_cluster.png`: Total spending by product category and cluster, highlighting which customer groups favor each category.
- `total_price_by_payment_method_and_cluster.png`: Total spending by payment method and cluster, revealing the preferred payment methods for each customer group.
- `invoice_amount_distribution_by_cluster.png`: Invoice amount distribution by cluster, comparing typical and high-value spending for different customer groups.

---

## 4. RFM Features and KMeans Segmentation

1. **RFM Calculation**

- Snapshot date = Maximum invoice date + 1 day.
- Grouped by customer, it yields currency (total unit price), frequency (unique invoice), and last purchase date (days since the last purchase).
- The generated `rfm` data frame will be retained in the notebook for subsequent modeling.

2. **Scaling and Model Selection**

- `RobustScaler` can handle skewed currency values without logarithmic transformation.
- KMeans models for `k` ∈ [2, 8] are compared using silhouette coefficients; the notebook currently selects **best_k = 3**.
- Final clustering uses the `KMeans(n_clusters=best_k, random_state=42, n_init=20)` algorithm, and cluster labels are added to the `rfm` archive.

3. **Diagnosis and Interpretation**

- `customer_clusters_pca.png` plots the clustering based on the first two principal components and highlights the cluster centers.
- `cluster_rfm_means_zscore.png` compares the mean z-scores for most recent purchase time, purchase frequency, and purchase amount in each cluster.
- The Markdown cells summarize the business analysis:

  - Cluster 0 – Customers who purchase infrequently and spend little
  - Cluster 1 – Customers with stable, moderate spending
  - Cluster 2 – Customers who purchase frequently and spend a lot
