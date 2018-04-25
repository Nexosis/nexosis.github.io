---
title: Understanding Model Metrics
description: explaining each metric used for various model comparisons
category: Nexosis Concepts
subcategory: Build a model
tags: [reference, metrics, results]
quick_link: true
use_codestyles: true
order: 9
---

## Understanding Model Metrics
It is important to have objective measures to evaluate how accurate the model is whenever a new model is created. It turns out that even the word 'accurate' can be controversial - but what we mean here is some way to tell how close your predictive model's forecasts are to reality.

Each model type may utilize different metrics and different metrics can tell you slightly different things about the same model. The Nexosis API returns several metrics for most models so that you can evaluate your model more fully. Let's look at each metric grouped by the model type.

### <a name="regressionmetrics" class="jumptarget"></a> Forecast & Regression

#### <a name="mae" class="jumptarget"></a>Mean Absolute Error
Describes in absolute units the difference between the predicted and actual target values, on average for a regression or time series model. This metric does not penalize large prediction errors as heavily as Root Mean Squared Error.

#### <a name="mape" class="jumptarget"></a>Mean Absolute Percent Error
Describes in percentage how far off the forecasts are from actuals, on average, for a time series model.

#### <a name="mase" class="jumptarget"></a>Mean Absolute Scaled Error
This scale-free error metric can be used to compare forecast methods on a single time series and also to compare forecast accuracy between time series. This metric is well suited to intermittent-demand series because it never gives infinite or undefined values except in the irrelevant case where all historical data are equal.

#### <a name="rsquared" class="jumptarget"></a>R-Squared
Measures how close the predicted target values are to the actual target values from a regression model, with a value typically between 0 and 1. (A very bad model can report negative values.) Values closer to 1 indicate that the model better explains the variability in the target values.

#### <a name="rmse" class="jumptarget"></a>Root Mean Squared Error
Describes in absolute units how far off the forecasts are from actuals, on average. This metric penalizes large prediction errors more heavily than Mean Absolute Error.

### <a name="classifiermetrics" class="jumptarget"></a> Classification

#### <a name="accuracy" class="jumptarget"></a>Accuracy
Reports the percentage of correctly classified observations out of all observations tested against a classification model.

#### <a name="maf1" class="jumptarget"></a>Macro Average F1 Score
Reports the harmonic mean of Macro Average Precision and Macro Average Recall, with a value between 0 and 1. This metric quantifies both the quality and the completeness of the positive predictions from a classification model.

#### <a name="map" class="jumptarget"></a>Macro Average Precision
Reports the precision for each class, averaged over all classes, with a value between 0 and 1. Precision is the percentage of true positives out of predicted positives. This metric quantifies quality of the positive predictions from a classification model. Because it is an unweighted average over classes, poor performance on minority classes can negatively impact this metric. The confusion matrix is a more detailed assessment of performance for unbalanced classification problems.

#### <a name="mar" class="jumptarget"></a>Macro Average Recall
Reports the recall for each class, averaged over all classes, with a value between 0 and 1. Recall is the percentage of correctly classified positives out of actual number of positives. It quantifies the completeness of the positive predictions from a classification model. Because it is an unweighted average over classes, poor performance on minority classes can negatively impact this metric. The confusion matrix is a more detailed assessment of performance for unbalanced classification problems.

#### <a name="mcc" class="jumptarget"></a>Matthews Correlation Coefficient
Reports the correlation coefficient between the predicted and actual classes, with a value between -1 and 1. It is a measures of the quality of a classification model, generally regarded as a balanced measure. To achieve a high score on this metric a model must do well at all aspects of a classification problem.

#### <a name="rocauc" class="jumptarget"></a>Area Under ROC Curve
Describes the diagnostic ability of a binary (two-class) classification model, with a value between 0 and 1. A value of 1 indicates a perfect model. A value of .5 can be achieved by randomly guessing the class. A value less than 0.5 indicates a model that does worse than randomly guessing.

### <a name="impactmetrics" class="jumptarget"></a> Impact Analysis
 
#### <a name="abseff" class="jumptarget"></a>Absolute Effect
Total absolute effect of the event on the data source. Answers the question, "How much did this event affect my data source?"

#### <a name="releff" class="jumptarget"></a>Relative Effect
Percentage effect of the event on the data source. Answers the question, "By what percentage did this event affect my data source?"

#### <a name="pvalue" class="jumptarget"></a> P Value
Determines the statistical significance of an impact, with a value between 0 and 1. A small value indicates strong evidence of the impact, whereas a large value indicates weak evidence of impact.

### <a name="anomalymetrics" class="jumptarget"></a> Anomalies
Because we're usually measuring model output against reality - we don't have an error metric for anomaly detection. Anomaly detection is a type of model built using *unsupervised* learning. By definition we don't know what *actual* should be. 

#### <a name="pa" class="jumptarget"></a>Percent Anomalies
Percent of the dataset that was determined to be anomalous.  This value is actually set during the model building process and does not represent "how well" the model did. A proper analysis of anomalies found should include your own understanding of the data domain and how the anomaly score matches your expectations for outliers.