# Analysis of promotions based on customer flows in retail
The project is dedicated to customer segmentation and the identification of the most effective types of advertising campaigns (hereafter referred to as 'promos' or 'promotions') based on changes in buyers' purchasing activity (hereafter referred to as 'flows') over time. The data set contains more than **20 million observations**. The analysis was carried out in Python and R.

## Table of Contents
- [Project Overview](#project-overview)
- [Data Description](#data-description)
- [Detailed Description of Analysis](#detailed-description-of-analysis)

### Project Overview

**Motivation:** Big media promotions (billboards, facades, etc.) are widely used for offline marketing
campaigns in retail and present an effective tool for creating a satisfying consumer experience.
However, despite the undeniable effectiveness of such promotions, it is rather difficult to
evaluate their impact. 

**The purpose of this project** is to develop a method for evaluating the effects
of promotions on customer flows and evaluate the effectiveness of various types of Big Media
promotions using it.

**Brief description of analysis:** 
- The first part of the analysis includes data preprocessing, clustering customers using transactional data of a Russian supermarket chain, and calculation of flows between them.
- In the second part such techniques as Singular Spectrum Analysis (SSA), first differences method, and trained Dynamic Bayesian Network (DBN) model are used to find causal relationships between flow coefficients and the number of promotions.

**Result:** The types of promotions that have the greatest impact on positive (when
buyers become more active) and negative (when buying activity decreases) flows were
identified. Also, assumptions have been made about the most sensitive consumer clusters
to promotions. 

**Impact:** The developed method can be used in marketing to evaluate both individual
promotions and their joint effectiveness. The results of the analysis provide managers with a deeper
understanding of changes in customer behavior and the ability to influence these changes

### Data Description
Period: 01.2019 - 12.2020

**Dataset №1 contains transactional data of customers.**

Main variables: 
- client id  
- purchase date
- check id
- number of items
- price

**Dataset №2 contains the information about promotions.**

Main variables: 
- promo type (billboards, facades, seasonal and two-week) 
- promo id
- start date
- end date

**Brief description of analysis:** 
- The first part of the analysis includes data preprocessing, clustering customers using transactional data of a Russian supermarket chain, and calculation of flows between them.
- In the second part such techniques as Singular Spectrum Analysis (SSA), first differences method, and trained Dynamic Bayesian Network (DBN) model are used to find causal relationships between flow coefficients and the number of promotions.

## Detailed Description of Analysis

### 1. Data preprocessing.

In the initial phase of our analysis, we conducted data preprocessing on the primary dataset, *pricing_hackathon_checks_train*, containing information about purchasing transactions.

Due to the real nature of the data, it has been hashed to ensure confidentiality, preventing the release of the original dataset to the public.

The following key steps were taken in the data preprocessing phase:

- **Column Removal:** Unnecessary columns for analysis were eliminated.
- **Date Conversion:** Date entries were converted to the datetime data type.
- **Month Enumeration:** An additional column was added to encode month numbers.
- **Summary Statistics:** Calculated unique buyer counts and plotted revenue distribution across days over two years.
- **Outlier Handling:** To improve data integrity, outliers were identified and removed. Specifically, clients with unusually high purchase frequencies (possibly wholesalers) were identified using a 98% percentile threshold. Figure 1 illustrates the discrete removal of only 13 unique client_ids.

![Revenue Distribution (before&after)](https://github.com/gelya1709/customer_flows/blob/main/Graphs/Revenue_distribution.jpg)

- **Customer Retention Criteria:** Buyers with fewer than 5 purchases in the two-year timeframe were excluded from the dataset. This decision was rooted in the strategic focus on returning customers for subsequent analyses.

In summary, these data preprocessing efforts resulted in the creation of the *data_for_clustering* dataset. This refined dataset comprises 33,918 original buyers who made more than 5 purchases during the two-year analytical window. 

### 2. Clustering

To proceed with the analysis, we first load the preprocessed dataset *data_for_clustering*. 

We choose to shorten the analysis period to a one-year timeframe, spanning from September 2019 to September 2020, thus excluding the period of the COVID-19 pandemic due to its disruptive influence on the stability of time series data.

The dataset is then filtered to encompass weekly periods, with week numbers adjusted using the isocalendar method and subsequently compressed into the 1-50 range for analytical convenience. 

The resulting input data for clustering contains the transactional records of buyers within the specified timeframe with week numbers over 52 full weeks.

The clustering procedure unfolds as follows:

#### 1. Function Definition - create_names: A bespoke function is defined to assign labels to clusters based on the calculated metrics.

#### 2. Clustering Algorithm:

- A subset is selected for each week, and buyer activity is computed with such metrics as Frequency (number of purchases) and Monetary (total spending) for each period.
- Both Frequency and Monetary metrics are normalized using the **Standard Scaler**.
- The **K-means++ algorithm** is applied with the selection of 3 clusters, driven by the observation that this configuration yields the highest silhouette score values across the 52 periods.
- **Silhouette score**, computed as an average over all periods, attains a value of 0.59.

#### Cluster Naming with create_names Function:

The custom function create_names is employed to assign labels to clusters, categorizing them into 'sleeping,' 'loyal,' and 'champions' based on the normalized Frequency and Monetary metrics in each period.

Three distinct clusters emerge: 'sleeping' for less active buyers, 'loyal' for consistently engaged buyers, and 'champions' for top-performing buyers.
The Rusinka depicts the average Monetary distribution across the entire period for the three clusters.

![Monetary Mean](https://github.com/gelya1709/customer_flows/blob/main/Graphs/Monetary_mean.png)

#### Resultant Datasets:

Two datasets are generated:
- One with client_id and assigned cluster labels for each period.
- Another with client_id, cluster labels, and metrics for each period, with the distribution visualized in Figure 2.

The primary dataset for subsequent analyses is denoted as 'customers_with_metrics(52&2),' encapsulating client_id, cluster labels, and associated metrics over the 52-week duration.






    
