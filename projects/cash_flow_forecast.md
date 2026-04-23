---
title: Cashflow Forecasting

layout: detail

abstract: Cashflow forecasting for corporate customers

# Choose a status:
# coming (planned or in development)
# active (live and ready to be used)
# archived (not longer used, discontinued)
status: active

# Choose a logo (regular version) from https://fontawesome.com/icons?d=gallery
logo: <i class="fa-solid fa-chart-mixed-up-circle-dollar"></i>

subcategory: dsandml-uc

tags:
  - Data Science
  - Machine Learning
  - ML/DS
  - Timeseries Forecasting
  - CashFlow Forecasting
  - Regression

contactpersons:
 - s9338b
 - SEBAnalytics@seb.se

completed: true
---

## Problem Definition & Business objective(s)

This project focuses on the problem of cashflow forecasting for corporate customers in collaboration with Cash Management team in C&PC. 

***CashFlow:*** is the net cash and cash equivalents transferred in and out of a company. Cash received represents inflows, while money spent represents outflows." [Investopedia](https://www.investopedia.com/terms/c/cashflow.asp). 

***CashFlow Forecasting (CFF):*** is the process of forecasting the flow (both in and out flow) of cash in a company's account for an arbitrary number of steps (time-steps) in future. CFF helps companies to evaluate their future cash flow status and better planning their actions to create value for shareholders through their ability to generate positive cash flows and maximize their long term free cash flow. The following image shows a synthetic example of cash flow data for a company.

![cff sample]({{site.baseurl}}/detail_pages/resources/gda/cff_sample_time_series.png)

### Problem definition (theory)
CFF, is often defined as a sequential stochastic process for which time series forecasting is used as a tool for predicting the future cash balances. In particular, we use linear/non-linear single/multi-variate auto-regressive (regression\*) models to predict the future by looking into the past. Similar to other learning methods a common approach is to divide historical data into three parts: ***train***, ***validation*** and ***test***. Training data is used to train the model, validation is used to tune the model parameters during the training phase and the test (also known as unseen or left-out data) is used to evaluate the accuracy of the final trained model. One important difference, compared to common learning methods, is that we can not just use a random distribution of the data to make the train-validation-test split for time-series data. Instead we use a sequential split of the data in chronological order using a rolling window and apply a technique called ***back-testing*** by moving the window forward in time and repeating the training process. Back-testing in the context of time-series forecasting is a technique similar to cross-validation in common machine learning methods that is used to evaluate the models generalization power and to prevent over-fitting and selection-bias problems in the training.

#### \*Regression
The simplest mathematical representation of a regression model is as follows:

<!-- $$y = \beta_0 + \beta x + \epsilon $$ -->
$$y = \alpha + \beta X + \epsilon, \epsilon \sim \mathbf{\mathit{N}}(0,1) $$
<!-- $$ \epsilon \sim \mathbf{\mathit{N}}(0,1) $$ -->

where $y$ is the independent variable, $X$ is a set (vector) of dependent variable(s), $\alpha$ is considered as the intercept, $\beta$ is the regression coefficient, and the $\epsilon $ represents noise. In particular, the goal of this model is to find the equation of a line (hyperplane) that best represents the data given some independent and normally distributed gaussian noise.

### Data
The data containing around 5 years of transactions has been provided by Cash Management team. We use end-of-day balance of the transactions as our time-stamp granularity. End-of-day balance is the account balance as of the end of the previous business day.


### Solution Proposal(s)
To find a proper solution for this application we tried three different approaches:

    (i) single variate linear regression methods: ARMA, ARIMA, SARIMA
    (ii) multivariate linear regression: OLS, GLS, Ridge, and 
    (iii) multivariate non-linear regression methods: XGB_regressor and LGBM_regressor.

In general the first two approaches are faster than the third, but multi-variate non-linear methods exhibit better performance when it comes to non-stationary samples. Non-stationarity, in a simple word, is related to the changes in the statistical properties of the time series (e.g., mean, variance , ...). In other view, one of the main assumptions behind any statistical model (including time-series forecasting) is that there is a random process that is generating the output, and the goal of forecasting is to find the parameters of that random process, so that we can keep generating similar outputs for forecasting. The random process can be very simple like "throwing a fair coin" (getting heads and tails with both 50% chance) but it can also be a very complex random process. Whatever the RP, it's important that it's statistical properties do not change over time when we use linear models. For example the probability of getting heads or tails in a "Throwing Coin" process does not change with time. This is what is known as stationarity.

### Proposed Solution Architecture(s)
Following our development methodology there is going to be a single model for each account and the goal is to make the best predictions every week (5 days) ahead for each account. To deploy this solution two architectures was proposed, ***(1) batch*** or ***(2) online***. The following figures show the visualization of the two approaches. As we can see, both architectures use the same set of components (modules) but in different set-ups. More specifically, in the batch prediction model the training is done during the weekend and the model is stored in a model-registry to be used during the inference time. Wheres, in the online approach both training and predictions are done at the execution time.  In general there are two main differences between the two architectures:
1. the ***training*** component is moved to ***sebshift*** environment in the online processing as opposed to middle ***datalake*** environment in the batch. 
2. the ***model-reg*** component is moved as a table into the oracle database instead of hdfs in datalake.

![cff sample]({{site.baseurl}}/detail_pages/resources/gda/cff_online_processing.png)

![cff sample]({{site.baseurl}}/detail_pages/resources/gda/cff_batch_processing.png)

