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

### Installed packages (core) — generated from current environment
Below is a small fragment of the packages currently installed in the environment used to develop and run the notebook (only core packages used by the notebook). You can regenerate this fragment on your machine with the command shown after the block.

```
matplotlib==3.9.2
mlxtend==0.23.4
numpy==1.26.4
pandas==2.2.2
scikit-learn==1.5.1
seaborn @ file:///... (see `pip freeze` for exact spec)
```

To regenerate the same snippet on Windows PowerShell (recommended), run:

```powershell
pip freeze | findstr /R "pandas numpy scikit-learn matplotlib seaborn python-pptx mlxtend" > requirements_core.txt
cat requirements_core.txt
```

If you want to pin exact versions for reproducibility, copy the lines from `requirements_core.txt` into `requirements.txt` (or append them), then run `pip install -r requirements.txt` in a clean venv.

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

## Cleaning decisions (documented)
This section records the concrete cleaning and fallback decisions implemented in `Group_6.ipynb`. These are the rules applied so reviewers can reproduce and (if desired) change them.

- Missing `LineAmount` / missing price: the notebook falls back to a unit proxy value of `1.0` per line when `LineAmount` or `Price` is missing. This keeps invoice-level aggregation working, but note this bias may understate monetary value for affected invoices. Code reference: if `LineAmount` missing -> `merged['LineAmount'] = 1.0`.

- `ProductList` / multi-product splitting: when a transaction row contains a comma-separated product list (column name `Product id list` or similar), we split by comma, `explode()` into line-level rows and `str.strip()` to remove whitespace. After exploding, we `merge` with the `products` table on product id to get product attributes.

- Missing invoice totals (`InvoiceAmount`): if missing, `InvoiceAmount` is recomputed as grouped sum over `LineAmount` by `Invoice no` to guarantee a consistent invoice-level amount.

- Missing behavioral features (e.g., `AvgInterpurchaseDays`, `MallDiversity`, `PaymentDiversity`, `CategoryConcentration`): these are filled with the median of the feature across customers (a conservative choice). Code reference: `rfm_enhanced[c] = rfm_enhanced[c].fillna(rfm_enhanced[c].median())`.

- Outlier handling and transforms:
	- Monetary and invoice-skew are handled with log transforms (e.g., `MonetaryLog = np.log1p(Monetary)` and `AvgInvoiceLog = np.log1p(AvgInvoice)`) instead of hard truncation. This preserves relative ordering while reducing skew influence on clustering.
	- If you prefer hard capping, consider applying a 99th-percentile cap on `InvoiceAmount` / `Monetary` before computing logs: `cap = df['Monetary'].quantile(0.99); df['Monetary'] = df['Monetary'].clip(upper=cap)`.

- Date parsing: `Invoice date` is parsed with `format='%d/%m/%Y'` and `errors='coerce'` (so malformed dates become `NaT`). Review rows with `NaT` dates before running RFM; they are excluded when computing recency/frequency by default.

These rules are intentionally conservative (median fills, log1p transforms). If you expect many missing prices or large outliers, adjust the rules and re-run the pipeline; we recommend documenting any change in the notebook cell near the cleaning code.

---

## Run the notebook in a clean environment (recommended commands)
Below are recommended PowerShell commands to create a fresh virtual environment, install dependencies, and execute the notebook end-to-end non-interactively. Adjust timeout if your machine is slow or dataset is large.

```powershell
# Create & activate venv (PowerShell)
python -m venv .venv
.\.venv\Scripts\Activate.ps1

# Upgrade pip and install dependencies
python -m pip install --upgrade pip
pip install -r requirements.txt

# Execute the notebook (run top-to-bottom). Increase --ExecutePreprocessor.timeout seconds if required.
jupyter nbconvert --to notebook --execute Group_6.ipynb --output executed_Group_6.ipynb --ExecutePreprocessor.timeout=600
```

Expected runtime & memory (approximate):
- For the sample dataset used during development (a few thousand transactions): expect 1–5 minutes on a modern laptop (4 CPU cores, 8 GB RAM). Peak memory usage is typically under 2–4 GB.
- If your dataset is much larger (tens or hundreds of thousands of transactions), runtime may increase to 10–30+ minutes and memory requirements may exceed 8 GB. In that case either increase memory or run the notebook in chunks / sample mode.

If `nbconvert` times out on long-running steps, re-run interactively in Jupyter (Restart & Run All) or increase the `--ExecutePreprocessor.timeout` value.

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
