# COMP3134 BI & CRM Project – Group 6

RetailX Customer Analytics and Promotion Strategy – Project Documentation

This document serves as the project specification for COMP3134 "Business Intelligence & Customer Relationship Management" (Group 6). It describes the project background, data and methods, key deliverables, and how to reproduce all analysis and figures in `Group_6.ipynb`.

---

## 1. Project Overview

**Business Scenario**  
RetailX is a retail group operating multiple shopping malls. It has transactional invoice records, customer profiles, and product information, and aims to leverage data analytics to:

- Understand customer shopping behaviour and value (e.g., purchase frequency, spending level, category preferences)
- Segment customers and build personas to support targeted marketing and CRM decisions
- Discover product affinity patterns to design promotion bundles and cross-selling strategies

**Project Objectives**  
This project focuses on:

- Cleaning and integrating RetailX transaction, customer, and product data
- Applying RFM metrics and KMeans clustering to segment customers and derive representative personas
- Mining association rules between products to support promotion and retention strategies
- Producing analysis outputs (tables, figures, and exported files) that can be directly used in the presentation

**Main Deliverables**

- `Group_6.ipynb`: main analysis notebook covering the full data pipeline
- Customer segmentation results and persona tables (multiple CSV files)
- High-value customer list and boundary customer list
- Product association rules table (`association_rules.csv`)
- All charts (PNG) used in the project presentation

---

## 2. Data & Methods Overview

**Data Sources**

- `data/sales_6.csv`: transaction-level data, including invoice ID, customer ID, mall, invoice date, payment method, and product ID list
- `data/customers_6.csv`: customer-level data, including customer ID, gender, age, and other basic attributes
- `data/products_6.csv`: product-level data, including product ID, category, price, and related attributes

**Key Processing Steps**

1. **Data cleaning and validation**
   - Check missing values and potential outliers
   - Validate invoice, customer, and product ID formats
   - Parse date fields and standardize date formats
2. **Data integration**
   - Split the comma-separated `Product id list` in each invoice into individual line items
   - Join with the product and customer tables to obtain a complete line-level dataset
3. **Descriptive analytics**
   - Analyse revenue and transaction volume by mall and product category
   - Examine payment method distribution, temporal patterns (monthly/daily), and customer age/gender structure
4. **RFM and customer clustering**
   - Compute Recency, Frequency, and Monetary value for each customer
   - Log-transform and scale Monetary, then perform KMeans clustering
   - Use PCA for visualisation and cluster quality assessment (silhouette score)
5. **Personas and high-value customers**
   - Build persona descriptions based on cluster characteristics and behavioural metrics
   - Identify high-value and boundary customers and provide CRM recommendations
6. **Association rule mining**
   - Convert invoices into basket-format transaction data
   - Use Apriori to mine product association rules and select rules with strong lift and confidence

---

## 3. Project Structure

The main files and directories related to this analysis are:

- `Group_6.ipynb`: main analysis notebook for this project
- `requirements.txt`: Python dependency list

Data and output directories:

- `data/`: raw input data
  - `sales_6.csv`
  - `customers_6.csv`
  - `products_6.csv`
- `output/csv/`: all CSV outputs from the analysis, for example:
  - `merged_data.csv`: transaction × product × customer merged line-level data
  - `cluster_profile.csv`: R/F/M characteristics and size of each cluster
  - `customers_with_cluster.csv`: cluster labels for each customer
  - `customers_with_cluster_enriched.csv`: customer file enriched with demographics and spending indicators
  - `personas_table.csv`: persona summary table
  - `high_value_customers.csv`: high-value customer list
  - `boundary_customers.csv`: customers located near cluster boundaries
  - `association_rules.csv`: product association rules
- `output/image/`: all PNG figures generated from the notebook, ready to be used in slides
- `output/model/`: saved models and preprocessing objects, e.g.:
  - `kmeans_model.joblib`
  - `scaler.joblib`
  - `pca.joblib`

---

## 4. Environment Setup

**Python version**  
- Recommended: Python ≥ 3.9

**Install dependencies (PowerShell)**

```powershell
pip install -r requirements.txt
```

**Key package versions used during development**

```text
matplotlib==3.9.2
mlxtend==0.23.4
numpy==1.26.4
pandas==2.2.2
scikit-learn==1.5.1
seaborn==0.13.2
```

---

## 5. How to Reproduce the Analysis

1. **Prepare data**  
   Place the following files inside the `data/` directory:
   - `sales_6.csv`
   - `customers_6.csv`
   - `products_6.csv`

2. **Install dependencies**  
   From the project root directory (where `requirements.txt` is located), run:

   ```powershell
   pip install -r requirements.txt
   ```

3. **Run the notebook**  
   Option 1 – Jupyter interface: open `Group_6.ipynb` in Jupyter Notebook or JupyterLab, then choose "Restart & Run All" to execute all cells from top to bottom.

   Option 2 – Command line execution:

   ```powershell
   jupyter nbconvert --to notebook --execute Group_6.ipynb --output executed_Group_6.ipynb --ExecutePreprocessor.timeout=600
   ```

4. **Verify outputs**  
   After execution, verify that:
   - `output/csv/` contains the various CSV outputs listed above
   - `output/image/` contains the generated PNG figures
   - `output/model/` contains the saved model and preprocessing objects

   Check that the file timestamps reflect your run time to ensure the results are up to date.

---

## 6. Notebook Implementation Flow (Technical Appendix)

The following summarises the main implementation steps in `Group_6.ipynb` to help readers navigate the code.

1. **Setup paths & style**
   - Ensure the `data/`, `output/csv`, `output/image`, and `output/model` directories exist (`Path(...).mkdir(exist_ok=True)`).
   - Set a consistent visual style using `sns.set(style="whitegrid")`.

2. **Load raw CSVs**
   - Use `pd.read_csv` to load `sales_6.csv`, `customers_6.csv`, and `products_6.csv`.
   - Inspect `.shape` and `.head()` to confirm columns and row counts.

3. **Data validation**
   - Compute missing values with `df.isna().sum()`.
   - Validate ID formats using regular expressions (e.g. `INV\d{5}`, `C\d{5}`, `P\d{5}`).
   - Parse invoice dates with `pd.to_datetime(..., errors="coerce")` and flag invalid dates.

4. **Product explode & merge**
   - Split `Product id list` on commas, strip whitespace, and `explode` into line items.
   - Join with product and customer tables and export the merged result to `output/csv/merged_data.csv`.

5. **Exploratory analysis**
   - Create bar charts for revenue by mall and category, payment method distributions, invoice date histograms, age-by-gender boxplots, and monthly trend lines.
   - Save all Matplotlib/Seaborn figures to `output/image/` for direct use in slides.

6. **RFM calculation**
   - Aggregate at customer level to compute Monetary, Frequency, and last purchase date.
   - Compute Recency from a snapshot date; apply `np.log1p` to Monetary; use `pd.qcut` to assign 1–5 scores for each of R, F, and M.

7. **KMeans clustering**
   - Scale R, F, and log(M) using `RobustScaler`.
   - Sweep k from 2 to 8, compute silhouette scores, and select the best k.
   - Train `KMeans(n_clusters=best_k, n_init=20)` and output:
     - `cluster_profile.csv`
     - `customers_with_cluster.csv`
     - PCA visualisations and model-related objects saved under `output/model/`

8. **Cluster diagnostics**
   - Plot standardised R/F/M means per cluster (radar or line charts) to highlight behavioural differences.
   - Use distances in PCA space to identify boundary customers and export `output/csv/boundary_customers.csv`.

9. **Persona & high-value view**
   - Merge cluster labels with demographic attributes and compute annualised spend and other indicators.
   - Draft persona descriptions for each cluster and derive corresponding acquisition/retention tactics.
   - Main outputs include: `personas_table.csv`, `customers_with_cluster_enriched.csv`, `high_value_customers.csv`, and top-category figures/CSVs per cluster.

10. **Association rules**
   - Use `TransactionEncoder` to transform invoices into basket format.
   - Run `apriori(min_support=0.01)` to find frequent itemsets, then `association_rules(..., metric="confidence", min_threshold=0.3)` to generate rules.
   - Filter rules with `lift > 1.2` and export them to `association_rules.csv`.

---

## 7. Cleaning & Modeling Notes

- **ID validation**: Invoice, customer, and product IDs are expected to follow the patterns `INV\d{5}`, `C\d{5}`, and `P\d{5}`; invalid IDs are flagged early in the pipeline.
- **Date parsing**: Dates are parsed using `pd.to_datetime(..., format="%d/%m/%Y", errors="coerce")`; rows with `NaT` are handled or removed before Recency calculation.
- **Product split**: `.str.split(',')` combined with `.apply(lambda lst: [pid.strip() for pid in lst])` ensures that product IDs align correctly with the product master table.
- **Monetary skew**: Applying `np.log1p` to Monetary and using `RobustScaler` mitigates skewness before clustering.
- **KMeans config**: `n_init=20`, with k selected from 2–8 based on silhouette score; PCA plots (e.g. `customer_clusters_pca.png`) are used to visually assess cluster separation.

---

## 8. Results Usage in Slides

- `personas_table.csv` and related visualisations support slides on target segments, positioning, and value propositions.
- `top_categories_per_persona` outputs together with `association_rules.csv` provide concrete bundle ideas, coupon target categories, and cross-selling strategies.
- `high_value_customers.csv` is used to discuss VIP identification and loyalty programme proposals.
- `cluster_profile.csv` underpins quantitative explanations of segmentation results and behavioural differences.

---

## 9. Troubleshooting Guide

| Symptom                         | Possible Cause                            | Suggested Fix                                                                 |
| --------------------------------| ------------------------------------------| ------------------------------------------------------------------------------|
| All parsed dates become `NaT`   | CSV date format is not `DD/MM/YYYY`       | Adjust the `format` argument or convert dates before reading the CSV          |
| Silhouette very low             | k not suitable or features not informative| Extend the k range or add more behavioural features (e.g. average basket size)|
| Association rules empty         | Support/confidence thresholds too strict  | Lower `min_support` or `confidence` thresholds and rerun                      |
| Notebook crashes from memory    | Data volume >> sample / low RAM           | Use chunked reading for `sales` or sample invoices for prototyping            |

---

## 10. Use of AI Tools

The code and analytical workflow for this project were designed and implemented by Group 6 members. Generative AI tools (e.g. GitHub Copilot) were used as an assistant during documentation and code review, specifically to:

- Suggest code syntax and minor refactoring options
- Help refine English wording and structure in this README

All final code, figures, and business interpretations were reviewed and validated by the team, who take full responsibility for their correctness and academic integrity in accordance with course requirements.


