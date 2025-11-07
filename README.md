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
Fill in your group's details below before submission. Replace the placeholders with real names/IDs and update presenter assignments.

- Group number: Group 6
- Course: COMP3134 BI & CRM
- Semester: [e.g., 2025/26 Semester 1]
- GitHub repository: https://github.com/PolyU-Computer-Science/Year3-COMP3134-BI-Group-Project

- Members (example template):
	- Student A (A1234567) — Role: Notebook & Data ETL, Contact: youremail@domain.edu
	- Student B (B2345678) — Role: EDA & Visuals, Contact: youremail@domain.edu
	- Student C (C3456789) — Role: Modeling & Clustering, Contact: youremail@domain.edu
	- Student D (D4567890) — Role: Presentation & Report, Contact: youremail@domain.edu

- Presenter allocation (4-minute presentation):
	- Speaker 1: 0:00–1:30 — Project overview & data
	- Speaker 2: 1:30–2:30 — Methods & modeling results
	- Speaker 3: 2:30–3:30 — Insights & marketing tactics
	- Speaker 4: 3:30–4:00 — Conclusion & Q&A handling

Make sure to replace placeholders with actual names, student IDs and contact emails before final submission.

## Personas (generated from notebook `Group_6.ipynb`)
The notebook generates a `personas_table.csv` (in `output/`) summarising each cluster into a concise persona and recommended tactics. Below are the persona templates and how to interpret them.

- High-Value (Recent)
	- Traits: above-median Monetary, low Recency (recent purchasers). Likely loyal, high spenders.
	- Acquisition tactic: Not primary — focus on referrals, selective lookalike campaigns to recruit similar customers.
	- Retention tactic: VIP program, early access to new products, personalized premium promotions, exclusive bundles.
	- Suggested KPIs: Average order value, retention rate, CLTV uplift.

- Frequent Low-Value
	- Traits: above-median Frequency but below-median Monetary. Many small purchases or low basket size.
	- Acquisition tactic: Cross-sell bundles and targeted upsell offers (e.g., 10% off when adding accessories).
	- Retention tactic: Loyalty points for hitting spending thresholds, promotions that increase basket size (bundle discounts).
	- Suggested KPIs: Average items per order, basket value, conversion on upsell offers.

- Low-Value / Dormant
	- Traits: low Frequency, low Monetary, high Recency (haven't bought recently).
	- Acquisition tactic: Welcome / reacquisition offers — time-limited discounts, curated welcome bundles using top cross-sell categories (use association rules for combinations).
	- Retention tactic: Re-engagement emails/SMS with personalized recommendations and coupon reminders; win-back campaigns.
	- Suggested KPIs: Reactivation rate, conversion rate of win-back offers, cost-per-acquisition for reactivated users.

Notes on persona interpretation:
- The persona labels are heuristic and based on cluster-level medians (Monetary, Frequency, Recency). Review `cluster_profile.csv` and `cluster_profile_enhanced.csv` to adjust labels or thresholds for your report.
- Use `assoc_rules_top.csv` (if generated) to pick concrete product combinations to show in slide examples for cross-sell bundles.

If you want, I can populate the Members list with your real names and student IDs once you provide them.

## License / Academic Integrity
This work is for academic purposes only. Ensure compliance with course guidelines.
