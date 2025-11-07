# COMP3134 BI & CRM Project – Group 6

## Project Structure
- `Group_6.ipynb`: Main analysis notebook (data loading → validation → transformation → EDA → feature engineering → modeling → insights → export)
- `sales_6.csv`, `customers_6.csv`, `products_6.csv`: Original datasets
- `rfm_clusters.csv`: Exported customer RFM features with cluster labels
- `cluster_profile.csv`: Mean feature values per cluster
- `merged_sample.csv`: Sample of merged transactional detail (first 2000 rows) for reference

## Environment Setup
### Recommended Versions
- Python >= 3.9

### Install Dependencies
Using pip (Windows PowerShell):
```powershell
pip install -r requirements.txt
```

If you plan to use association rules later:
```powershell
pip install mlxtend
```

## Reproducibility Steps
1. Place all three source CSV files in the same folder as the notebook.
2. Open and run `Group_6.ipynb` top to bottom (Restart & Run All recommended).
3. Outputs (`rfm_clusters.csv`, `cluster_profile.csv`) will be generated near the end.

## Data Processing Overview
| Phase | Key Actions |
|-------|-------------|
| Validation | Format checks (IDs), missing values, date parsing |
| Transformation | Split product list to line items, join products & customers, compute line and invoice totals |
| Feature Engineering | RFM (Recency, Frequency, Monetary) + Category variety + Avg invoice + Preferred mall |
| Modeling | KMeans clustering (silhouette score to pick k) |
| EDA | Mall/category sales, invoice amount distribution, demographic distributions |
| Export | Cluster assignments + profiles saved as CSV |

## Modeling Details
- Scaling: `StandardScaler`
- Algorithm: `KMeans` (auto n_init)
- k selection: silhouette score across range 2–6
- Profiling: mean feature values per cluster

## Interpreting Clusters (Example Template)
| Cluster | Traits | Possible Strategy |
|---------|-------|------------------|
| 0 | High Monetary, Low Recency (recent buyers) | Early access / premium loyalty program |
| 1 | High Frequency, Moderate Monetary | Bundle offers to lift order value |
| 2 | Low Frequency, Low Monetary | Acquisition push: first-purchase incentive |

Adjust after reviewing actual profile numbers.

## Marketing Strategy Draft
- Acquisition Tactic: Target low-frequency segment with welcome bundles combining top 2 cross-sell categories.
- Retention Tactic: Offer tiered loyalty perks to high-Monetary cluster (exclusive previews, upsell electronics accessories).

## Extending the Analysis
- Association rules (Apriori/FP-Growth) for product bundling.
- Time-series forecasting (monthly revenue) using SARIMA or Prophet.
- Classification model predicting future high-value customers.

## Troubleshooting
| Issue | Cause | Fix |
|-------|-------|-----|
| Date column all NaT | Locale/format mismatch | Verify format `%d/%m/%Y` |
| Empty clusters or poor silhouette | k too high/low | Rerun k selection range or add features |
| Memory issues | Large dataset | Sample or chunk read |

## Team Information
Add group number and member student IDs here for submission.

## License / Academic Integrity
This work is for academic purposes only. Ensure compliance with course guidelines.
