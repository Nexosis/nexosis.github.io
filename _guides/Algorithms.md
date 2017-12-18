---
title: Algorithms Employed
description: A list of algorithms the Nexosis Machine Learning API employs 
copyright: 2017 Nexosis 
layout: default
category: Concepts
tags: [forecasting, regression]
use_codestyles: true
---

Our machine learning platform has a growing suite of about 30 core algorithms with a total of over 300 permutations. Given your data, our suite of algorithms will tune and select a model that best fits your data and the problem you're trying to solve.

## Forecasting a future quantity

Solves the question, how much of x can I expect in the future?

Algorithms support hourly, daily, weekly, monthly, and annual seasonalities

All support Anomaly Smoothing and Model Ensembling

* ARIMA
  * Various combinations of 
  * AutoRegressive component with p parameters
  * Differencing component with d parameters
  * Moving Average component with q parameters
  * with external regressors
* Exponential Smoothing
  * Simple
  * Double
  * Triple
  * with Box-Cox Transformation
* Autoregressive Neural Network
  * with or without external regressors
* Multiple Linear Regression
  * with or without external regressors
* Spline
* Seasonal and Trend Decomposition using Loess
  * with ARIMA
  * with or without external regressors
  * with Exponential Smoothing
* Bayesian Time Series Regression
  * with or without external regressors
* Additive Model
* Home-grown Nexosis Algorithms

## Predicting a variable

Solves the question, what can I expect x to be?

* Least Squares
  * Linear
  * Polynomial
* Elastic Net
* Lasso
* Ridge
* Support Vector Regression
  * Linear Kernel
  * Polynomial Kernel
  * Radial Basis Function kernel
  * Sigmoid Kernel
* Multi-Layer Perceptron (Neural Network)
  * with 1, 2, or 3 hidden layers
  * Rectified Linear Unit Function
  * Hyperbolic Tan Function
  * Sigmoid Function
* Random Forest
* K-Nearest Neighbor
* Logistic Regression
* Naive Bayes
* XGBoost