# COMP3134 BI & CRM Project – Group 6

## Project Structure

- `Group_6.ipynb`: Main analysis notebook (data loading → validation → transformation → EDA → feature engineering → modeling → insights → export)
- `data/`: Contains the three source CSVs `sales_6.csv`, `customers_6.csv`, `products_6.csv`
- `output/csv/`: Notebook exports, including `merged_data.csv`, `cluster_profile.csv`, `customers_with_cluster.csv`, `customers_with_cluster_enriched.csv`, `cluster_top_categories.csv`, `top_categories_per_persona.csv`, `boundary_customers.csv`, `personas_table.csv`, `high_value_customers.csv`, and `association_rules.csv`
- `output/image/`: Saved charts (mall/category sales, payment mix, invoice distribution, seasonality, cluster PCA plot, persona plots, etc.)
- `output/model/`: Serialized `kmeans_model.joblib`, `scaler.joblib`, `pca.joblib`

## Environment Setup

### Recommended Versions

- Python >= 3.9

### Install Dependencies

Using pip (Windows PowerShell):

```powershell
pip install -r requirements.txt
```

`requirements.txt` already includes every package needed by the notebook (NumPy/Pandas/Matplotlib/Seaborn/Scikit-learn/mlxtend). No optional installs are required.

## Reproducibility Steps

1. Place all three source CSV files in the same folder as the notebook.
2. Open and run `Group_6.ipynb` top to bottom (Restart & Run All recommended).
3. Outputs (see the `output/csv/` list above) will be refreshed near the end.

### Installed packages (core) — generated from current environment

Below is a small fragment of the packages currently installed in the environment used to develop and run the notebook (only core packages used by the notebook). You can regenerate this fragment on your machine with the command shown after the block.

```
matplotlib==3.9.2
mlxtend==0.23.4
numpy==1.26.4
pandas==2.2.2
scikit-learn==1.5.1
seaborn==0.13.2
```

To regenerate the same snippet on Windows PowerShell (recommended), run:

```powershell
pip freeze | findstr /R "pandas numpy scikit-learn matplotlib seaborn mlxtend" > requirements_core.txt
cat requirements_core.txt
```

If you want to pin exact versions for reproducibility, copy the lines from `requirements_core.txt` into `requirements.txt` (or append them), then run `pip install -r requirements.txt` in a clean venv.

## Data Processing Overview

| Phase               | Key Actions                                                                 |
| ------------------- | --------------------------------------------------------------------------- |
| Validation          | Missing-value scan, ID format regex checks, invoice-date parsing            |
| Transformation      | Split comma-separated product lists, trim IDs, explode to line level, join product & customer tables |
| Feature Engineering | Compute RFM metrics (Recency, Frequency, Monetary + log transform)          |
| EDA                 | Mall/category revenue, payment mix, invoice distributions, demographics, seasonality |
| Modeling            | KMeans on robust-scaled RFM features, silhouette-based k selection, PCA viz |
| Persona & Rules     | Map clusters to personas + tactics, perform Apriori association rules       |
| Export              | Persist merged data, cluster assignments, persona summaries, association rules, and model artifacts |

## Modeling Details

- Scaling: `RobustScaler` (reduces influence of monetary outliers)
- Algorithm: `KMeans` (`n_init=20` for stability)
- k selection: silhouette score across range 2–8 (best k picked automatically)
- Profiling: mean feature values per cluster

## Interpreting Clusters (Example Template)

| Cluster | Traits                                     | Possible Strategy                          |
| ------- | ------------------------------------------ | ------------------------------------------ |
| 0       | High Monetary, Low Recency (recent buyers) | Early access / premium loyalty program     |
| 1       | High Frequency, Moderate Monetary          | Bundle offers to lift order value          |
| 2       | Low Frequency, Low Monetary                | Acquisition push: first-purchase incentive |

Adjust after reviewing actual profile numbers.

## Marketing Strategy Draft

- Acquisition Tactic: Target low-frequency segment with welcome bundles combining top 2 cross-sell categories.
- Retention Tactic: Offer tiered loyalty perks to high-Monetary cluster (exclusive previews, upsell electronics accessories).

## Extending the Analysis

- Association rules (Apriori/FP-Growth) for product bundling.
- Time-series forecasting (monthly revenue) using SARIMA or Prophet.
- Classification model predicting future high-value customers.

## Cleaning decisions (documented)

Concrete rules implemented in `Group_6.ipynb`:

- ID validation: invoices, customers, and products must match `INV\d{5}`, `C\d{5}`, `P\d{5}`. Rows violating the pattern are flagged in the QA cell.
- Date parsing: `Invoice date` is parsed with `format='%d/%m/%Y'` and `errors='coerce'`; rows with `NaT` dates are excluded automatically from Recency calculations.
- Product list handling: comma-separated product IDs are split, whitespace-trimmed, and exploded before joining with the product master to avoid mismatched keys.
- Monetary skew: `log1p` is applied to Monetary before clustering; no synthetic values are injected for missing prices (the provided dataset already contains price per product).
- Scaling: `RobustScaler` is used so extreme spenders do not dominate cluster boundaries.

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

| Issue                             | Cause                  | Fix                                     |
| --------------------------------- | ---------------------- | --------------------------------------- |
| Date column all NaT               | Locale/format mismatch | Verify format `%d/%m/%Y`              |
| Empty clusters or poor silhouette | k too high/low         | Rerun k selection range or add features |
| Memory issues                     | Large dataset          | Sample or chunk read                    |

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

- The persona labels are heuristic and based on cluster-level medians (Monetary, Frequency, Recency). Review `cluster_profile.csv` to adjust labels or thresholds for your report.
- Use `association_rules.csv` to pick concrete product combinations to show in slide examples for cross-sell bundles.

If you want, I can populate the Members list with your real names and student IDs once you provide them.

## License / Academic Integrity

This work is for academic purposes only. Ensure compliance with course guidelines.
