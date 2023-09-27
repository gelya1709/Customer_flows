# Analysis of the effectiveness of promotions based on customer reaches
The project is devoted to the segmentation of customers and identifying the most effective types of advertising shares based on buyers between clusters in time. The data set contains more than **20 million observations**. The analysis was carried out in Python and R.

## Motivation
Large retail companies launch many promotions to maintain customer activity.
The task of marketers is to plan promoters in such a way that promotions bring maximum benefit to the business.

**The task of this project** is to develop a new analysis method to make the following decisions:

- What consumer segments can be distinguished by their value for business?
- What types of promotions are the most effective and what segments of consumers?
- What should be the duration of the promotions and when should they be launched?

## Data
Period: 09.2019 - 09.2020

**Dataset №1 contains transactional data of customers.**

Main variables: 
- customer id  
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

## Segmentation 

1. Dataset division into 52 periods (1 week = 1 period)
2. The allocation of consumer clusters with the K-Means algorithm **in each period**. **Frequency** and **monetary** metrics from RFM models were selected as metrics for segmentation (more details see [here](https://www.investopedia.com/terms/r/rfm-recency-frequency-monetary-value.asp))

**As a result, 4 consumer clusters were received:** 

- **churn** - are not active in the period under consideration
- **sleeping** - average check 2 000 rubles, 1 purchase per week
- **loyal** - average check 5 000 rubles, 3 purchases per week
- **champions** - average check 11 000 rubles, 3 purchases per week

### Calculation of flows
Of the 4 clusters obtained in each of the 52 weeks, **16 flows** (for example, from_churn_to_sleeping for 51 periods) are calculated

### Analysis Promo Dataset
The average number of promos of each type for each period:

- billboards - 50
- facades - 70
- seasonal promo - 150
- two-week promo - 130

## Definition of the influence of promo on the flows (in R)

1. SSA Analysis to detect a trend component and get rid of noise
2. The method of the first difference to bring the distribution to normal
3. Training of the DBN model to identify causal relationships

## Conclusions and recommendations


1. The segment of buyers with low activity (Sleeping) requires constant stimulation, otherwise, these buyers flow
: arrow_right: **should be on an ongoing basis to run two-week promotions designed for a buyer with a low average check**

2. Billboards and facades affect customers the fastest (within 1 week).
They have the greatest effect on buyers with medium and high activity (Loyal and Champions segments)
: arrow_right: **billboards and facades should constantly be present to maintain customer activity, but their number should be optimized**

3. Seasonal and two-week promotions have a defined effect and begin to work after 2 weeks from the date of launch
: arrow_right: **these types of promotions should be launched 2 weeks before the expected fall in demand**







    
