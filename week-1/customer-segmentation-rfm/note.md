This file explains my steps and my choices in this project.

## 1. Data Cleaning

The raw data has 541,909 rows. I found three problems:

- **Missing CustomerID.** 135,080 rows have no CustomerID. I cannot use these rows for RFM, because RFM needs a customer. I removed these rows.
- **Cancelled orders.** Some `InvoiceNo` values start with the letter "C". These are cancelled orders. They have negative `Quantity`. I removed these rows. After that, I checked again, and there were 0 rows with negative quantity left.
- **Zero price.** Some rows have `UnitPrice` equal to 0 or less. I looked at these rows, but I did not see any special "test" or "manual" label. Still, a price of 0 is not a real sale, so I removed these rows too.

After cleaning, the data has about 4,338 unique customers.

## 2. Building RFM Features

I used `groupby('CustomerID')` to change the data from transaction-level to customer-level. I built three features:

- **Recency**: number of days since the customer's last purchase. I used the last date in the dataset, plus one day, as the reference date.
- **Frequency**: number of unique invoices (`InvoiceNo`) per customer.
- **Monetary**: total money spent per customer. First I made a new column `TotalPrice = Quantity * UnitPrice`, then I summed it by customer.

## 3. Feature Preparation

I checked the skewness of the RFM values:

| Feature | Skewness |
|---|---|
| Recency | 1.25 |
| Frequency | 12.07 |
| Monetary | 19.32 |

Frequency and Monetary are very skewed. This means a few customers have very high values (for example, one customer spent £77,183 in only one order). This is a problem for clustering, because distance-based models like K-Means are sensitive to big values.

To fix this, I did two steps:
1. `log1p` transformation on all three features, to reduce the skew.
2. `StandardScaler`, to put all features on the same scale.

## 4. Choosing the Number of Clusters

I tested K-Means with `k` from 2 to 10, and calculated the silhouette score for each `k`:

| k | Silhouette Score |
|---|---|
| 2 | 0.4329 |
| 3 | 0.3365 |
| 4 | 0.3371 |
| 5 | 0.3161 |
| 6 | 0.3133 |
| 7 | 0.3100 |
| 8 | 0.3008 |
| 9 | 0.2817 |
| 10 | 0.2787 |

The score is highest at `k=2`, but I chose **k=3**, because it gives more useful business groups. With `k=2`, I only get two groups, and it is harder to separate "VIP" customers from "regular" customers. With `k=3`, the three groups are clear and each one has a different meaning for marketing.

I also compared three clustering methods at `k=3`:

| Model | Silhouette Score |
|---|---|
| K-Means | 0.3365 |
| Hierarchical (Agglomerative) | 0.3526 |
| DBSCAN (eps=0.45, min_samples=6) | 0.2946 |

Hierarchical clustering got a slightly higher score than K-Means, but the difference is small. DBSCAN found only 2 clusters and 81 noise points (outliers). It could not find 3 clear groups, so it is not the best choice for this task.

**Final choice: K-Means, k=3.** The score is close to Hierarchical's score, but K-Means is faster, simpler, and easier to use again when new data comes in the future. It also gives 3 clear, balanced groups that are easy to explain to a business team.

## 5. What Each Cluster Means

| Cluster | Recency (days) | Frequency | Monetary (£) | Customers | Segment name |
|---|---|---|---|---|---|
| 0 | 17.1 | 13.3 | 7,905 | 770 | VIP / Champions |
| 1 | 167.4 | 1.4 | 363 | 1,872 | At Risk / Hibernating |
| 2 | 44.2 | 3.4 | 1,265 | 1,696 | Regular / Loyal |

- **VIP / Champions**: buy often, buy recently, spend the most money. This is the most important group.
- **Regular / Loyal**: buy sometimes, spend a medium amount. This is the biggest and most stable group.
- **At Risk / Hibernating**: did not buy for a long time, buy rarely, spend little money. This group may stop being a customer soon.

## 6. Limitations and Next Steps

- A few "whale" customers (very high spending) can still affect the clustering. An outlier check with Isolation Forest could help.
- I could add more features, like how many product categories a customer buys, or the return rate.
- t-SNE or UMAP could give a clearer visualization than PCA.
