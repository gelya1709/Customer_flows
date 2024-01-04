# Analysis of promotions based on customer flows in retail
The project is dedicated to customer segmentation and the identification of the most effective types of promotions based on changes in buyers' purchasing activity over time. The data set contains more than 20 mln records. The analysis was carried out in Python and R.

## Table of Contents
- [Project Overview](#project-overview)
- [Data Source](#data-source)
- [Steps of Analysis](#steps-of-analysis)
- [Results](#results) 

## Project Overview

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

## Data Source

Data was used at the Hackathon in 2021. The data provider is one of the largest supermarket chains. Due to the real nature of the data, it has been hashed to ensure confidentiality, preventing the release of the original dataset to the public.

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

## Steps of Analysis

### 1. Data preprocessing. [Link to code](https://github.com/gelya1709/Customer_flows/blob/main/Step_1_Data_preprocessing.ipynb)

In the initial phase of our analysis, we conducted data preprocessing on the primary dataset, *pricing_hackathon_checks_train*, containing information about purchasing transactions. We calculated revenue and excluded unnecessary columns.

Dataset:
![Dataset example](https://github.com/gelya1709/customer_flows/blob/main/Graphs/Dataset%20example.png)


The following key steps were taken in the data preprocessing phase:

- **Date Conversion:** Date entries were converted to the datetime data type.
- **Month Enumeration:** An additional column was added to encode month numbers.
- **Summary Statistics:** Calculated unique buyer counts and plotted revenue distribution across days over two years.
- **Outlier Handling:** Clients with unusually high purchase frequencies (possibly wholesalers) were identified using a 98% percentile threshold. 

Revenue distribution before and after Outlier Removal.
![Revenue Distribution (before&after)](https://github.com/gelya1709/customer_flows/blob/main/Graphs/Revenue_distribution.jpg)

- **Customer Retention Criteria:** As soon as analysis is focused on returning customers, buyers with less than 5 purchases in the 2-year timeframe were excluded from the dataset.

The resulting dataset contains 33,918 original buyers who made more than 5 purchases during the 2-year analytical window. <br />

___
### 2. Clustering. [Link to code](https://github.com/gelya1709/Customer_flows/blob/main/Step_2_Clustering.ipynb)

We chose to shorten the analysis period to a 1-year timeframe, from September 2019 to September 2020, as we wanted to exclude the period of the COVID-19 pandemic.

Also, we added a new column with weeks in the range of 1-52. 

#### Clustering Algorithm:

- Computation of buyer activity with such **metrics as Frequency (number of purchases) and Monetary (total spending)** for each period (one period = one week).
- Both Frequency and Monetary metrics are normalized using the **Standard Scaler**.
- The **K-means++ algorithm** is applied with the selection of 3 clusters.
- **Silhouette score**, computed as an average over all periods, equals 0.59.
- Cluster Labelling with **custom create_names function**: *create_names* is employed to assign labels to clusters, categorizing them into 'sleeping,' 'loyal,' and 'champions' based on Frequency and Monetary metrics.

**As a result, 3 consumer clusters were received:**

- **sleeping** - average check 2 000 rubles, 1 purchase per week
- **loyal** - average check 5 000 rubles, 3 purchases per week
- **champions** - average check 11 000 rubles, 3 purchases per week

<br>
Distribution of mean_monetary values across clusters over 52 weeks:

![Monetary Mean](https://github.com/gelya1709/customer_flows/blob/main/Graphs/Monetary_mean.png)

As a result, two datasets are generated:
- 1st one with client IDs and assigned cluster labels.
- 2nd one with client IDs, cluster labels, and Frequency and Monetary metrics.


___
### 3. Calculation of flows. [Link to code](https://github.com/gelya1709/Customer_flows/blob/main/Step_3_Calculation_of_flows.ipynb)

The purpose of this step is to get the size of each cluster and of each flow (for example, sleeping to loyal).
![Flows calculation](https://github.com/gelya1709/customer_flows/blob/main/Graphs/Flows%20calculation.png)

We need to deal with NaN values:
1. Replace NaN with 'churn' values for columns with cluster labels. This means that the buyer did not make any purchases in this period. "Churn" is the 4th cluster with inactive buyers.
2. Replace NaN with 0 for inactive buyers in the Frequency and Monetary metrics.<br />

Now we can calculate the sizes of flows and save the result in the *count_of_flows* dataset.
![Flows size](https://github.com/gelya1709/customer_flows/blob/main/Graphs/Size%20of%20flows.png)


___
### 4. Time-series analysis. [Link to code](https://github.com/gelya1709/Customer_flows/blob/main/Step_4_SSA_analysis.ipynb) 

We need to add data on the number of promotions of different types in each period.

The distributions of clusters and promotions:
![Promo distribution](https://github.com/gelya1709/customer_flows/blob/main/Graphs/Promo%20distributions.png)

#### -- SSA analysis --

The resulting dataset with flows and the number of promotions is a time series, so we will use a function that calculates the **SSA decomposition of series into the window_size components**.

Output of decomposition to 4 components:
![SSA components](https://github.com/gelya1709/customer_flows/blob/main/Graphs/SSA%20components.png)

- In the left column is the original series and its model (the sum of the components into which the series is divided using the SSA method). Within the accuracy of the drawing, the curves merge.
- In the right column are the components into which the series is divided. 

As we can see, in all cases one component SSA1 (blue line) stands out, the values of which are very different from zero. We can say that it describes a trend. The average values of all other components are close to zero, their variation is insignificant. Thus, for further analysis we select only the first component of all series.

#### -- First-differences transformation --

We built the distributions of values and understood that they are not normal → that’s why we will use **first-differences**.

Distribution of original flows:
![Distribution of original flows](https://github.com/gelya1709/customer_flows/blob/main/Graphs/Distribution%20of%20original%20flows.png)

Distribution of first differences:
![Distribution of first-differences](https://github.com/gelya1709/customer_flows/blob/main/Graphs/Distribution%20of%20first-differences.png)

#### -- Correlation --

The next step is to check the **correlation coefficient between the size of flows and the number of promotions**. We check if there is a correlation between them and calculate p-value to estimate the significance of the coefficient (purple color indicates p-value < 0.01, yellow one < 0.001)

Correlation coefficients between the size of flows and the number of promotions:
![Corr coefficients](https://github.com/gelya1709/customer_flows/blob/main/Graphs/Correlation%20coefficents.png)


___
### 5. Casual Modelling. [Link to code](https://github.com/gelya1709/Customer_flows/blob/main/Step_5_Casual_modelling.ipynb)

As we can see, there is a correlation between promotions and flows. But this does not mean that there is a cause-and-effect relationship.
We will look for it by **training the structure of a Gaussian Bayesian dynamic network**. The corresponding library is implemented only for R https://github.com/dkesada/dbnR.

The best-implemented method is natPsoho.

This network is a set of linear regression equations that represent cause-and-effect relationships.

Regression equation:

**y =** 𝒂𝟎 + 𝒂𝟏\*𝒙𝟏 + 𝒂𝟐\*𝒙𝟐 + 𝒂𝟑\*𝒙𝟑 + 𝒂𝟒\*𝒙𝟒,
where **y** − first difference of flows that presents the intensity of customer flows,
𝒙𝐢 − first difference of promotions that presents the intensity of promotions.

Regression output of DBN model:
![Regression output](https://github.com/gelya1709/customer_flows/blob/main/Graphs/Regression_output.png)

Visualization of DBN model:
![DBN model](https://github.com/gelya1709/customer_flows/blob/main/Graphs/DBN%20model.png)


## Results

For analyzing coefficients, we divided the consumer flows into 3 groups: 
- **positive flows** (sleeping to loyal, sleeping to champions, loyal to champions, churn to sleeping, churn to loyal, churn to champions),
- **negative flows** (sleeping to churn, loyal to sleeping, loyal to churn, champions to loyal, champions to sleeping, champions to loyal),
- **neutral flows** (sleeping to sleeping, loyal to loyal, champions to champions and churn to churn accordingly).

For positive flows, the higher the coefficients for the promo, the more effective these promotions are for the business. Positive values indicate that the positive flow has increased due to, for example, placed billboards. For negative flows, on the contrary, the lower the coefficient for the promotion, the more it reduces the flow which is unfavorable for business.
___
#### -- Positive flows analysis --
![Results Positive flows](https://github.com/gelya1709/customer_flows/blob/main/Graphs/Results%20Positive%20flows.png)
___
#### -- Negative flows analysis --
![Results Negative flows](https://github.com/gelya1709/customer_flows/blob/main/Graphs/Results%20Negative%20flows.png)
___
#### Conclusions and recommendations

- The segment of buyers with low activity (Sleeping) requires constant stimulation, otherwise, these buyers flow **should be on an ongoing basis to run two-week promotions designed for a buyer with a low average check**
- Billboards and facades affect customers the fastest (within 1 week).
They have the greatest effect on buyers with medium and high activity (Loyal and Champions segments) -> **billboards and facades should constantly be present to maintain customer activity, but their number should be optimized**
- Seasonal and two-week promotions have a defined effect and begin to work after 2 weeks from the date of launch -> **these types of promotions should be launched 2 weeks before the expected fall in demand**

The proposed method can be used to estimate the effect of specific promotions without considering other factors. Marketers will be able to track the effect that a specific launched promotion gives, as well as determine its starting point and expected duration. 

In practice, this also will help to track not just an increase in the activity of the buyer, but to assess the increase in activity relative to other buyers. This opportunity is especially relevant for highly seasonal businesses: in their case, the characteristics of all customer segments change over the period and an increase in the activity of one segment can only be assessed relative to others.
