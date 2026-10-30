# Superstore Sales: Data Analysis & Visualization

End-to-end analysis of the Superstore retail dataset (9,994 order lines, Jan 2014 to Dec 2017):
data loading, cleaning, outlier handling, feature engineering, encoding/scaling, EDA with
Matplotlib/Seaborn, and business insights.

## Project structure

```
superstore-analysis/
├── data/
│   ├── raw/superstore_raw.csv               # original data (never modified)
│   └── processed/
│       ├── superstore_cleaned.csv           # cleaned + engineered features (readable)
│       └── superstore_model_ready.csv       # encoded + scaled, all-numeric
├── src/analysis.py                          # full pipeline (script, commented cell by cell)
├── notebooks/superstore_analysis.ipynb      # same pipeline, executed, with outputs
├── reports/
│   ├── insights.md                          # key findings (auto-generated from the data)
│   └── figures/                             # 13 PNG charts
├── build_notebook.py                        # rebuilds the notebook from src/analysis.py
├── requirements.txt
└── README.md
```

## Setup

```bash
git clone <your-repo-url>
cd superstore-analysis
python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

## Run

```bash
python src/analysis.py                 # regenerates cleaned data, figures and insights.md
jupyter notebook notebooks/superstore_analysis.ipynb   # or open the notebook interactively
python build_notebook.py               # optional: rebuild + re-execute the notebook
```

## Dataset overview

21 columns: order/ship dates, ship mode, customer & segment, location (city, state, postal code, region),
product (category, sub-category, name) and the measures Sales, Quantity, Discount, Profit.
The file is Windows-1252 encoded, so it is read with `encoding="cp1252"`.

## Methodology

**1. Exploration**: shape, dtypes, summary statistics, missing values, duplicates, whitespace, impossible
values, date consistency, skewness.

**2. Cleaning & transformation**

| Issue / step | Treatment |
|---|---|
| Missing values | None found. Pipeline still includes median (numeric) / mode (categorical) imputation and dropping rows lacking key IDs |
| Duplicates | None found; `drop_duplicates` kept for safety |
| Dates stored as text | Parsed to datetime (`%m/%d/%Y`) |
| Postal codes lost leading zeros | Read as text, zero-padded to 5 digits |
| Stray whitespace in text | Stripped |
| Outliers (Sales, Profit extremely skewed) | Detected with IQR rule, z-scores and box plots. They are real large transactions, so they are **flagged** (`*_Outlier`), **winsorized** at 1st/99th percentile (`*_Winsor`) and `Sales` is log-transformed. Original values are preserved |
| Categorical encoding | One-hot: Ship Mode, Segment, Region, Category. Label encoding: Sub-Category, State. Ordinal: Discount Band |
| Scaling | `StandardScaler` on continuous features (`*_Scaled` columns) |
| New features | Order year/month/quarter/day of week, Shipping Days, Profit Margin, Unit Price, Discount Band, Is Loss, Order Size, Customer Orders |

**3. EDA**: descriptive statistics, histograms/KDE, scatter plots, pair plot, Pearson and Spearman
correlations, yearly/monthly trends, and breakdowns by category, sub-category, region, segment,
ship mode and state.

## Key findings

Full list with numbers in [`reports/insights.md`](reports/insights.md).

- Sales $2.30M, profit $286K, **12.5% margin**; sales grew 29.5% in 2016 and 20.4% in 2017.
- **Discounts drive losses**: undiscounted lines have a 29.5% margin, discounts above 20% are almost always loss-making (97% of lines).
- **Furniture earns only 2.5% margin** vs ~17% for Office Supplies and Technology. Tables, Bookcases and Supplies lose money overall.
- The **Central region** has the lowest margin (7.9%); Texas, Ohio and Pennsylvania are the least profitable states.
- Strong late-year seasonality (Sep, Nov, Dec). The top 10% of customers make ~31% of sales.

## Feature selection notes

Keep `Discount`, `Sales_Winsor`/`Log_Sales`, `Quantity`, `Unit Price`, category/region/segment encodings.
`Profit Margin` and `Is Loss` are derived from Profit, so they would leak the target in a Profit model.
Shipping Days, Order Size and Customer Orders have almost no correlation with profit.

## Tools

Python 3, pandas, NumPy, Matplotlib, Seaborn, scikit-learn, Jupyter.
