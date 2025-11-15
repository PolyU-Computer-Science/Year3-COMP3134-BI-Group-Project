# COMP3134 BI & CRM Project – Group 6

This README is a flow guide for `Group_6.ipynb`: every step, what the code does, and the files generated. 

## Environment Setup (retained)

- Python ≥ 3.9
- Install dependencies (PowerShell):
  ```powershell
  pip install -r requirements.txt
  ```
- Core package versions used during development:
  ```
  matplotlib==3.9.2
  mlxtend==0.23.4
  numpy==1.26.4
  pandas==2.2.2
  scikit-learn==1.5.1
  seaborn==0.13.2
  ```

## Notebook Flow (Step-by-step)

1. **Setup paths & style**

- Ensure `data/`, `output/csv`, `output/image`, and `output/model` exist (`Path(...).mkdir(exist_ok=True)`).
- Apply consistent visualization defaults via `sns.set(style='whitegrid')` so every chart looks uniform.

2. **Load raw CSVs**

- Read `sales_6.csv`, `customers_6.csv`, `products_6.csv` with `pd.read_csv`.
- Print shapes and `display(... .head())` to verify columns before transformation.

3. **Data validation**

- Count missing values (`df.isna().sum()`), validate IDs with regex (e.g., `str.match(r'^INV\d{5}$')`).
- Parse invoice dates using `pd.to_datetime(..., errors='coerce')` and inspect unique categorical levels.

4. **Product explode & merge**

- Split `Product id list` on commas, strip whitespace, `explode` into line items.
- Merge with product and customer tables, then save the enriched detail as `output/csv/merged_data.csv`.

5. **Exploratory analysis**

- Build mall/category revenue bars, payment-method distribution, invoice histogram, age-by-gender boxplot, and monthly trend line.
- Export every Matplotlib/Seaborn figure to `output/image/` for direct slide use.

6. **RFM calculation**

- Aggregate per customer to get Monetary, Frequency, and most recent purchase date.
- Compute Recency from `snapshot_date`, apply `np.log1p` to Monetary, and assign 1–5 quantile scores via `pd.qcut`.

7. **KMeans clustering**

- Scale R, F, and log Monetary with `RobustScaler`, sweep k=2..8, pick the highest silhouette, and fit `KMeans(n_clusters=best_k, n_init=20)`.
- Output `cluster_profile.csv`, `customers_with_cluster.csv`, PCA visuals, and persist scaler/PCA/model under `output/model/`.

8. **Cluster diagnostics**

- Plot z-scored R/F/M means per cluster to highlight behavioral differences.
- Measure distances in PCA space to flag near-boundary customers and export `output/csv/boundary_customers.csv`.

9. **Persona & high-value view**

- Merge clusters with demographics, compute annualized spend, label personas, and map acquisition/retention tactics.
- Produce `personas_table.csv`, `customers_with_cluster_enriched.csv`, `high_value_customers.csv`, and top-category charts/CSVs per cluster.

10. **Association rules**

- Encode invoices into baskets with `TransactionEncoder`, run `apriori(min_support=0.01)`, derive rules with `association_rules(..., metric='confidence', min_threshold=0.3)`, filter by `lift > 1.2`, and save to `association_rules.csv`.

11. **Deliverables recap**

- Summarize artifact locations: all CSV exports in `output/csv/`, charts in `output/image/`, and serialized models in `output/model/` for presentation packaging.

### Directory quick reference

- `data/`: raw CSV inputs (sales/customers/products).
- `output/csv/`: merged data, cluster/persona exports, boundary list, association rules, etc.
- `output/image/`: all Matplotlib/Seaborn figures saved during notebook run.
- `output/model/`: `kmeans_model.joblib`, `scaler.joblib`, `pca.joblib`.

## Cleaning & Modeling Notes

- **ID validation**: invoice/customer/product IDs must match `INV\d{5}`, `C\d{5}`, `P\d{5}`; mismatches flagged early.
- **Date parsing**: `pd.to_datetime(..., format='%d/%m/%Y', errors='coerce')`; rows with `NaT` drop from recency calculations automatically.
- **Product split**: `.str.split(',')` + `.apply(lambda lst: [pid.strip() for pid in lst])` ensures product IDs join cleanly to the master table.
- **Monetary skew**: `np.log1p(Monetary)` plus `RobustScaler` stabilizes feature space before clustering.
- **KMeans config**: `n_init=20`, silhouette-driven k (2–8). PCA plot stored as `customer_clusters_pca.png` for visual inspection.

## How to Re-run Everything

1. Place `sales_6.csv`, `customers_6.csv`, `products_6.csv` inside `data/`.
2. Install dependencies (see setup above).
3. Run the notebook from start to finish (Jupyter “Restart & Run All” or CLI):
   ```powershell
   jupyter nbconvert --to notebook --execute Group_6.ipynb --output executed_Group_6.ipynb --ExecutePreprocessor.timeout=600
   ```
4. Confirm timestamps on files within `output/` to ensure they reflect your run; copy these into the submission package.

## Troubleshooting Cheatsheet

| Symptom                         | Cause                          | Fix                                                                 |
| ------------------------------- | ------------------------------ | ------------------------------------------------------------------- |
| All parsed dates become `NaT` | CSV not in `DD/MM/YYYY`      | Adjust `format` argument or convert before reading                |
| Silhouette very low             | Inadequate k / missing signals | Extend k range or add behavior features (e.g., average basket size) |
| Association rules empty         | Thresholds too strict          | Lower `min_support` or `confidence` and rerun                   |
| Notebook crashes from memory    | Dataset >> sample provided     | Chunk the sales file or sample invoices for prototype runs          |

## Personas & Strategy Use

- `personas_table.csv` + persona visualizations feed the “Target segment / positioning / value prop” slide.
- `top_categories_per_persona.csv` + `association_rules.csv` provide concrete acquisition/retention tactics (bundle ideas, coupon targets).
- `high_value_customers.csv` highlights VIP IDs for loyalty proposals.
