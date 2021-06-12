---
title: "Cohort analysis with Pandas and Seaborn"
author: ''
date: '2021-06-12'
slug: [cohort]
categories: [analytics]
tags: [analytics]
subtitle: 'A step-by-step tutorial'
summary: ''
authors: []
lastmod: '2021-06-12T11:00:01+02:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---



This article was originally shared on Medium: https://joaomacalos.medium.com/cohort-analysis-with-pandas-and-seaborn-dd90c8ba5cdc


## Introduction

The objective of this notebook is to show, step by step, how to implement a cohort analysis with Pandas and Seaborn.

Coming from an R background, I like working with pipes and to chain commands to modify a DataFrame. The idea of writing this article came when I was trying to implement this analysis with Python and struggled with the existing tutorials. Therefore, the main objective of this post is to present the code — if you are curious about cohort analysis, there are plenty of articles available online.

## Cohort analysis

- What is a cohort analysis?

Cohort analysis is a type of analysis that follows a group of customers over time. For example, instead of analyzing our customers individually, we group them by date of sign-up in our app. In this way, we can analyze whether a marketing campaing implemented in a given month was effective or not.

- Why does it matter?

Cohort analyses matter because churn is one of the main problems faced by companies nowadays, and every information that can lead to less churning is a valuable business information.

## Olist data

To show how to implement a cohort analysis with `Pandas` and `Seaborn`, I will work with the Olist dataset. It is available on [Kaggle](https://www.kaggle.com/olistbr/brazilian-ecommerce) and provide a good example of an e-commerce dataset. The schema of the data is available in the link above.

The Olist plataform connects small business owners (the sellers) with customers from all over Brazil. Understanding whether sellers come back to the platform after their first sale is a crucial business question for Olist.

Therefore, I will make the cohorts based on the month of the first sale of each seller. With this cohorts, I will track the proportion of sellers from each cohort that came back to the platform in the subsequent months.

Hence, we must connect the `olist_sellers_dataset` where the `id` of each seller is located with the `olist_orders` dataset where we can find the date of each sale (`order_purchase_timestamp`) in the platform. These two tables are connected through the `olist_order_items_dataset`.

### 1. Import the required packages


```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
```

### 2. Load the data


```python
df_orders = pd.read_csv('../../../Datasets/olist_orders_dataset.csv')
df_order_items = pd.read_csv('../../../Datasets/olist_order_items_dataset.csv')
df_sellers = pd.read_csv('../../../Datasets/olist_sellers_dataset.csv')
```


### 3. Join the data

Join the datasets, convert `date` to correct type, remove observations outside the analysis boundaries, and select relevant columns:


```python
df = pd.merge(df_order_items, df_orders, on='order_id')
df = pd.merge(df, df_sellers, on='seller_id')
df = (df
     .assign(date = lambda x: pd.to_datetime(x.order_purchase_timestamp))
     .query('date >= "2017-01-01" & date < "2018-09-01"')
     .loc[:, ['seller_id', 'date']]
     )
df.head(3)
```

```
##                           seller_id                date
## 0  48436dade18ac8b2bce089ec2a041202 2017-09-13 08:59:02
## 1  48436dade18ac8b2bce089ec2a041202 2017-07-23 16:13:37
## 2  48436dade18ac8b2bce089ec2a041202 2017-08-10 12:17:35
```


### 4. Create the cohorts

To find the cohorts, we must find the first appearance (`order_purchase_timestamp`) of each individual seller (`seller_id`). To do that, I'll order the dataset by date of sale and drop the duplicates in the `seller_id` column.


```python
seller_cohort = (df[['seller_id', 'date']]
                 .assign(cohort = lambda x: x.date.dt.to_period('M'))
                 .sort_values('date')
                 .drop_duplicates('seller_id')
                 .reset_index()
                 .loc[:, ['seller_id', 'cohort']]
                )
seller_cohort.head(3)
```

```
##                           seller_id   cohort
## 0  48efc9d94a9834137efd9ea76b065a38  2017-01
## 1  8f119a0aee85c0c8fc534629734e94fd  2017-01
## 2  b14db04aa7881970e83ffa9426897925  2017-01
```


### 5. Merge data and calculate delta

In the next step, we need to:

- Merge the `seller_cohort` DataFrame with df on the `seller_id` column;
- Assign a column with the months of each sale;
- Assign a column that calculates the difference in months between the date of sale and the cohort (`delta`).


```python
from operator import attrgetter

cohort_df = (
  pd.merge(df, seller_cohort, on='seller_id')
  .assign(date_of_sale = lambda x: x.date.dt.to_period('M'),
          delta = lambda x: (x.date_of_sale - x.cohort).apply(attrgetter('n')))
  .loc[:, ['seller_id', 'date', 'cohort', 'date_of_sale', 'delta']]
  )

cohort_df.head(3)
```

```
##                           seller_id                date  ... date_of_sale delta
## 0  48436dade18ac8b2bce089ec2a041202 2017-09-13 08:59:02  ...      2017-09     6
## 1  48436dade18ac8b2bce089ec2a041202 2017-07-23 16:13:37  ...      2017-07     4
## 2  48436dade18ac8b2bce089ec2a041202 2017-08-10 12:17:35  ...      2017-08     5
## 
## [3 rows x 5 columns]
```


Before proceeding with the analysis, it’s important to visualize the size of each cohort we just created to see if there are no clear problems in the data:


```python
size_cohorts = (cohort_df
                .groupby(['cohort', 'delta'])[['seller_id']]
                .nunique()
                .query('delta == 0')
                .reset_index()
                )

# Plot:
sns.set(style="whitegrid", color_codes=True)
pal = sns.color_palette("Greens_d", len(size_cohorts))
rank = size_cohorts.seller_id.argsort().argsort()   
# plt.figure(figsize=(8, 6))
ax = sns.barplot(
  data=size_cohorts, 
  y='seller_id', 
  x='cohort', 
  palette=np.array(pal)[rank]
  )
ax.set(xlabel='Cohort', ylabel='# of sellers');
plt.xticks(rotation=45);
plt.tight_layout();
plt.show()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-6-1.png" width="672" />


As we can see, there were some variability in the number of new sellers joining Olist in the period under analysis, but nothing really outstanding that invalidate our exercise in this post.

### 6. Count sellers in each cohort per delta and calculate the proportion

The next step consist in:

- Counting the unique sellers in each delta by each cohort;
- Counting the total sellers in each cohort (the number of sellers in the first month, i.e., when delta is equal to zero);
- Calculating the proportion of sellers present in each cohort afterwards.


```python
cohort_df = (cohort_df
             .groupby(['cohort', 'delta'])[['seller_id']]
             .nunique()
             .assign(total = lambda x: x.groupby('cohort')[['seller_id']]
                                       .transform(lambda x: x[0]),
                     proportion = lambda x: 100 * x.seller_id / x.total)
             .reset_index()
             .loc[:, ['cohort', 'delta', 'proportion']]
            )

cohort_df.head(3)
```

```
##     cohort  delta  proportion
## 0  2017-01      0  100.000000
## 1  2017-01      1   75.770925
## 2  2017-01      2   73.568282
```

### 7. Pivot the DataFrame

Now we have all the information we need to calculate make the cohort analysis, but it is in the long format. The following step consists in applying a `pivot()` method to reshape it:


```python
cohort_pivot = (
  cohort_df[['cohort', 'delta', 'proportion']]
  .pivot(columns='delta', index='cohort')['proportion']
  )
cohort_pivot
```

```
## delta       0          1          2   ...         17         18         19
## cohort                                ...                                 
## 2017-01  100.0  75.770925  73.568282  ...  38.766520  35.682819  39.207048
## 2017-02  100.0  61.176471  54.509804  ...  28.627451  25.882353        NaN
## 2017-03  100.0  57.954545  54.545455  ...  27.840909        NaN        NaN
## 2017-04  100.0  55.371901  47.107438  ...        NaN        NaN        NaN
## 2017-05  100.0  50.000000  50.000000  ...        NaN        NaN        NaN
## 2017-06  100.0  42.105263  40.789474  ...        NaN        NaN        NaN
## 2017-07  100.0  67.521368  60.683761  ...        NaN        NaN        NaN
## 2017-08  100.0  54.263566  46.511628  ...        NaN        NaN        NaN
## 2017-09  100.0  53.125000  54.687500  ...        NaN        NaN        NaN
## 2017-10  100.0  66.216216  55.405405  ...        NaN        NaN        NaN
## 2017-11  100.0  51.832461  54.973822  ...        NaN        NaN        NaN
## 2017-12  100.0  57.777778  52.222222  ...        NaN        NaN        NaN
## 2018-01  100.0  57.446809  50.354610  ...        NaN        NaN        NaN
## 2018-02  100.0  52.500000  52.500000  ...        NaN        NaN        NaN
## 2018-03  100.0  60.176991  53.097345  ...        NaN        NaN        NaN
## 2018-04  100.0  59.900990  51.980198  ...        NaN        NaN        NaN
## 2018-05  100.0  56.395349  56.976744  ...        NaN        NaN        NaN
## 2018-06  100.0  56.020942  59.685864  ...        NaN        NaN        NaN
## 2018-07  100.0  56.315789        NaN  ...        NaN        NaN        NaN
## 2018-08  100.0        NaN        NaN  ...        NaN        NaN        NaN
## 
## [20 rows x 20 columns]
```


### 8. Plot with Seaborn

With the pivoted column, we can simply use the `sns.heatmap()` function to visualize the cohorts:


```python
sns.heatmap(
  cohort_pivot, 
  cmap='crest', 
  annot=True,
  annot_kws={"fontsize":7},
  fmt='.0f'
  ).set_title("Olist Cohort Analysis")

plt.show()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-9-1.png" width="672" />


### Conclusion

It’s interesting to note that the first cohort was relatively more loyal to Olist than the cohorts that followed, with a participation rate 6 months after their first sale in the platform that was similar to the participation rate of the other cohorts. It’s also noteworthy that the cohort “2017–07” was also loyal to the brand. Now it is up to the marketing team of Olist to investigate further why some of the cohorts presented churn rates much lower than the others and act upon this information.

### Full code

The notebook with the full code of this analysis can be found here: https://github.com/joaomacalos/Medium/blob/main/cohort-pandas-seaborn.ipynb


