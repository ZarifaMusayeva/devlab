# Customer Segmentation with RFM and Clustering

This project groups retail customers into segments based on their purchase behavior, using RFM (Recency, Frequency, Monetary) analysis and unsupervised clustering.

## Dataset

[Online Retail Dataset (UCI)](https://archive.ics.uci.edu/dataset/352/online+retail) — around 541,909 transactions from a UK-based online store, Dec 2010 to Dec 2011.

## How to Run

1. Install dependencies:
   ```
   pip install -r requirements.txt
   ```
2. Place `Online Retail.xlsx` inside `data/raw/`.
3. Open and run `notebooks/customer_segmentation_rfm.ipynb` from top to bottom.

Outputs (figures and the processed dataset with cluster labels) will be saved automatically to `outputs/` and `data/processed/`.

## What This Project Does

1. Cleans the raw transaction data (missing CustomerID, cancelled orders, zero prices).
2. Builds Recency, Frequency, and Monetary features per customer.
3. Reduces skew with log transformation and scales the features.
4. Compares three clustering methods: K-Means, Hierarchical (Agglomerative), and DBSCAN.
5. Selects the number of clusters using the silhouette score.
6. Profiles and visualizes the final segments with PCA.

## Results

Three customer segments were found:

| Segment | Customers | Description |
|---|---|---|
| VIP / Champions | 770 | Buy often, buy recently, spend the most |
| Regular / Loyal | 1,696 | Buy occasionally, spend a medium amount |
| At Risk / Hibernating | 1,872 | Haven't bought in a long time, low spend |

See [`note.md`](./note.md) for methodology details and [`cluster_profile_report.md`](./cluster_profile_report.md) for the business-facing summary.

## Tech Stack

pandas, numpy, scikit-learn, scipy, seaborn, matplotlib