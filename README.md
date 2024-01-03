# Analysis of promotions based on customer flows in retail
The project is dedicated to customer segmentation and the identification of the most effective types of advertising campaigns (hereafter referred to as 'promos' or 'promotions') based on changes in buyers' purchasing activity (hereafter referred to as 'flows') over time. The data set contains more than **20 million observations**. The analysis was carried out in Python and R.

## Table of Contents
- [Project Overview](#project-overview)
- [Data Description](#data-description)
- [Detailed Description of Analysis](#detailed-description-of-analysis)
- [Results](#results) 

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
![Dataset example](https://github.com/gelya1709/customer_flows/blob/main/Graphs/Dataset%20example.png)

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

#### 2. Clustering Algorithm:

- A subset is selected for each week, and buyer activity is computed with such **metrics as Frequency (number of purchases) and Monetary (total spending)** for each period.
- Both Frequency and Monetary metrics are normalized using the **Standard Scaler**.
- The **K-means++ algorithm** is applied with the selection of 3 clusters, driven by the observation that this configuration yields the highest silhouette score values across the 52 periods.
- **Silhouette score**, computed as an average over all periods, attains a value of 0.59.
- Cluster Labelling with **custom create_names function**: *create_names* is employed to assign labels to clusters, categorizing them into 'sleeping,' 'loyal,' and 'champions' based on the normalized Frequency and Monetary metrics in each period.

**As a result, 3 consumer clusters were received:**

- **sleeping** - average check 2 000 rubles, 1 purchase per week
- **loyal** - average check 5 000 rubles, 3 purchases per week
- **champions** - average check 11 000 rubles, 3 purchases per week

Distribution of mean_monetary values across clusters over 52 weeks:

![Monetary Mean](https://github.com/gelya1709/customer_flows/blob/main/Graphs/Monetary_mean.png)

#### Resultant Datasets:

Two datasets are generated:
- One with client_id and assigned cluster labels for each period.
- Another with client_id, cluster labels, and metrics for each period, with the distribution visualized in Figure 2.

The primary dataset for subsequent analyses is denoted as 'customers_with_metrics(52&2),' encapsulating client_id, cluster labels, and associated metrics over the 52-week duration.

### 3. Calculation of flows.

We will work with the data obtained in the previous step *customers_with_metrics(52&2)*

The purpose of this step is to get the size of each cluster and, accordingly, each flow (for example, sleeping to loyal).
![Flows calculation|320x271](https://github.com/gelya1709/customer_flows/blob/main/Graphs/Methodology%20flows%20calculation.png)

We need to deal with NaN values:
- replace NaN values for columns with cluster labels with churn - this means that the buyer did not make any purchases in this period. This will be the so-called 4th cluster of inactive buyers.
- replace NaN with 0 for such buyers in the Frequency and Monetary metrics

We keep only the buyers and the clusters assigned to them for each period and calculate the cluster size.
![Cluster size](https://github.com/gelya1709/customer_flows/blob/main/Graphs/Size%20of%20clusters.png)

Next, we use a loop to calculate the sizes of flows and save the result in the *count_of_flows* dataset
![Flows size](https://github.com/gelya1709/customer_flows/blob/main/Graphs/Size%20of%20flows.png)

### 4. SSA analysis 

The dataset obtained in the last step is a time series, so we will use analysis for preprocessing and analysis of time series - SSA analysis.

Let's add data on the number of promotions of different types in each period.

The distribution looks like:
![Promo distribution](https://github.com/gelya1709/customer_flows/blob/main/Graphs/Promo%20distributions.png)

The resulting dataset with flows and the number of promotions is a time series, so we will use analysis for preprocessing and analysis of time series - SSA analysis.

We use a function that calculates the SSA decomposition of series into the window_size components
![SSA components](https://github.com/gelya1709/customer_flows/blob/main/Graphs/SSA%20components.png)

In the left column is the original series and its model** (the sum of the components into which the series is divided using the SSA method). Within the accuracy of the drawing, the curves merge.

**In the right column are the components into which the series is divided**. As we can see, in all cases one component SSA1 (blue line) stands out, the values of which are very different from zero. We can say that it describes a trend. The average values of all other components are close to zero, their variation is insignificant. Thus, for further analysis we select only the first component of all series

We build the distributions of values and understood that they are not normal → that’s why we will use first-differences for further analysis
![Distribution of original flows](https://github.com/gelya1709/customer_flows/blob/main/Graphs/Distribution%20of%20original%20flows.png)
![Distribution of first-differences](https://github.com/gelya1709/customer_flows/blob/main/Graphs/Distribution%20of%20first-differences.png)

The next step is to check the correlation coefficient between the size of flows and the number of promotions. We check if there is a correlation between them and calculate p=value to check the significance of the coefficient. (purple values, where p-value<0.05, yellow <

![Corr coefficients](https://github.com/gelya1709/customer_flows/blob/main/Graphs/Correlation%20coefficents.png)

It is clear from the tables that there are statistically significant correlation values, which means we continue our analysis.

### 5. Casual Modelling

As you can see, there is a correlation between promotions and flows, and it depends on the lag. But this does not mean that there is a cause-and-effect relationship.
We will look for it by training the structure of a Gaussian Bayesian dynamic network. The corresponding library is implemented only for R https://github.com/dkesada/dbnR.

The best-implemented method is natPsoho.

This network is a set of linear regression equations that represent cause-and-effect relationships.

Regression output:
![Regression output](https://github.com/gelya1709/customer_flows/blob/main/Graphs/Regression_output.png)

DBN model:
![DBN model](https://github.com/gelya1709/customer_flows/blob/main/Graphs/DBN%20model.png)

### Results

![Results Positive flows](https://github.com/gelya1709/customer_flows/blob/main/Graphs/Results%20Positive%20flows.png)
    
![Results Negative flows](https://github.com/gelya1709/customer_flows/blob/main/Graphs/Results%20Negative%20flows.png)

