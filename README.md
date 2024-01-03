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

## Brief description of analysis 
- The first part of the analysis includes data preprocessing, clustering customers using transactional data of a Russian supermarket chain, and calculation of flows between them.
- In the second part such techniques as Singular Spectrum Analysis (SSA), first differences method, and trained Dynamic Bayesian Network (DBN) model are used to find causal relationships between flow coefficients and the number of promotions.

### Detailed Description of Analysis










    
