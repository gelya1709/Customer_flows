# Analysis of promotions based on customer flows in retail
The project is dedicated to customer segmentation and the identification of the most effective types of promotions based on changes in buyers' purchasing activity over time. The data set contains more than 20 mln records. The analysis was carried out in Python and R.

## Table of Contents
- [Project Overview](#project-overview)
- [Data Source](#data-source)
- [Steps of Analysis](#detailed-description-of-analysis)
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

**Result:** We identified types of promotions that have the greatest impact on positive (when
buyers become more active) and negative (when buying activity decreases) flows were
identified. 

**Impact:** The developed method can be used in marketing to evaluate both individual
promotions and their joint effectiveness. The results of the analysis provide managers with a deeper
understanding of changes in customer behavior and the ability to influence these changes

### Data Source

Data was used at the Hackathon in 2021. The data provider is one of the largest supermarket chains. 

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

## Detailed Description of Analysis

### 1. Data preprocessing.

In the initial phase of our analysis, we conducted data preprocessing on the primary dataset, *pricing_hackathon_checks_train*, containing information about purchasing transactions. We calculated revenue and excluded unnecessary columns.

Dataset:
![Dataset example](https://github.com/gelya1709/customer_flows/blob/main/Graphs/Dataset%20example.png)

Due to the real nature of the data, it has been hashed to ensure confidentiality, preventing the release of the original dataset to the public.

<br>
The following key steps were taken in the data preprocessing phase:

- **Date Conversion:** Date entries were converted to the datetime data type.
- **Month Enumeration:** An additional column was added to encode month numbers.
- **Summary Statistics:** Calculated unique buyer counts and plotted revenue distribution across days over two years.
- **Outlier Handling:** Clients with unusually high purchase frequencies (possibly wholesalers) were identified using a 98% percentile threshold. 

Revenue distribution before and after Outlier Removal.
![Revenue Distribution (before&after)](https://github.com/gelya1709/customer_flows/blob/main/Graphs/Revenue_distribution.jpg)

- **Customer Retention Criteria:** As soon as analysis is focused on returning customers, buyers with less than 5 purchases in the 2-year timeframe were excluded from the dataset.

The resulting dataset contains 33,918 original buyers who made more than 5 purchases during the 2-year analytical window. <br />


### 2. Clustering

We chose to shorten the analysis period to a 1-year timeframe, from September 2019 to September 2020, as we wanted to exclude the period of the COVID-19 pandemic.

Also, we added a new column with weeks in the range of 1-52. 

#### Clustering Algorithm:

- Computation of buyer activity with such **metrics as Frequency (number of purchases) and Monetary (total spending)** for each period (one period = one week).
- Both Frequency and Monetary metrics are normalized using the **Standard Scaler**.
- The **K-means++ algorithm** is applied with the selection of 3 clusters.
- **Silhouette score**, computed as an average over all periods, equals to 0.59.
- Cluster Labelling with **custom create_names function**: *create_names* is employed to assign labels to clusters, categorizing them into 'sleeping,' 'loyal,' and 'champions' based on Frequency and Monetary metrics.

**As a result, 3 consumer clusters were received:**

- **sleeping** - average check 2 000 rubles, 1 purchase per week
- **loyal** - average check 5 000 rubles, 3 purchases per week
- **champions** - average check 11 000 rubles, 3 purchases per week

<br>
Distribution of mean_monetary values across clusters over 52 weeks:

![Monetary Mean](https://github.com/gelya1709/customer_flows/blob/main/Graphs/Monetary_mean.png)

As the result, two datasets are generated:
- 1st one with client IDs and assigned cluster labels.
- 2nd one with client IDs, cluster labels, and Frequency and Monetary metrics.

<br>
### 3. Calculation of flows.

The purpose of this step is to get the size of each cluster and of each flow (for example, sleeping to loyal).
![Flows calculation](https://github.com/gelya1709/customer_flows/blob/main/Graphs/Flows%20calculation.png)

We need to deal with NaN values:
1. Replace NaN with 'churn' values for columns with cluster labels. This means that the buyer did not make any purchases in this period. "Churn" is the 4th cluster with inactive buyers.
2. Replace NaN with 0 for inactive buyers in the Frequency and Monetary metrics.<br />

Now we can calculate the sizes of flows and save the result in the *count_of_flows* dataset.
![Flows size](https://github.com/gelya1709/customer_flows/blob/main/Graphs/Size%20of%20flows.png)


### 4. SSA analysis 

We need to add data on the number of promotions of different types in each period.

The distributions of clusters and promotions:
![Promo distribution](https://github.com/gelya1709/customer_flows/blob/main/Graphs/Promo%20distributions.png)

The resulting dataset with flows and the number of promotions is a time series, so we will use **SSA analysis** to work with this data.

We use a function that calculates the **SSA decomposition of series into the window_size components**.
<br>
![SSA components](https://github.com/gelya1709/customer_flows/blob/main/Graphs/SSA%20components.png)

**In the left column** is the original series and its model (the sum of the components into which the series is divided using the SSA method). Within the accuracy of the drawing, the curves merge.

**In the right column** are the components into which the series is divided. 

As we can see, in all cases one component SSA1 (blue line) stands out, the values of which are very different from zero. We can say that it describes a trend. The average values of all other components are close to zero, their variation is insignificant. Thus, for further analysis we select only the first component of all series.

We built the distributions of values and understood that they are not normal → that’s why we will use **first-differences**.

Distribution of original flows:
![Distribution of original flows](https://github.com/gelya1709/customer_flows/blob/main/Graphs/Distribution%20of%20original%20flows.png)

Distribution of first differences:
![Distribution of first-differences](https://github.com/gelya1709/customer_flows/blob/main/Graphs/Distribution%20of%20first-differences.png)

The next step is to check the **correlation coefficient between the size of flows and the number of promotions**. We check if there is a correlation between them and calculate p-value to estimate the significance of the coefficient (purple color indicates p-value < 0.01, yellow one < 0.001)

Correlation coefficients between the size of flows and the number of promotions:
![Corr coefficients](https://github.com/gelya1709/customer_flows/blob/main/Graphs/Correlation%20coefficents.png)

### 5. Casual Modelling

As we can see, there is a correlation between promotions and flows. But this does not mean that there is a cause-and-effect relationship.
We will look for it by **training the structure of a Gaussian Bayesian dynamic network**. The corresponding library is implemented only for R https://github.com/dkesada/dbnR.

The best-implemented method is natPsoho.

This network is a set of linear regression equations that represent cause-and-effect relationships.

Regression output of DBN model:
![Regression output](https://github.com/gelya1709/customer_flows/blob/main/Graphs/Regression_output.png)

Visualization of DBN model:
![DBN model](https://github.com/gelya1709/customer_flows/blob/main/Graphs/DBN%20model.png)

### Results

Positive flows analysis:
![Results Positive flows](https://github.com/gelya1709/customer_flows/blob/main/Graphs/Results%20Positive%20flows.png)

Negative flows analysis:
![Results Negative flows](https://github.com/gelya1709/customer_flows/blob/main/Graphs/Results%20Negative%20flows.png)

