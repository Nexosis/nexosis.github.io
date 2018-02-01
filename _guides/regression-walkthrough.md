---
title: Regression Walkthrough
description: Lean how to generate regression models using the Nexosis API
copyright: 2017 Nexosis 
layout: default
category: Regression
tags: [Predict, Quick Links, Favorite, Regression, Walkthrough]
order: 2
use_codestyles: true
---

Regression models can be created for all types of DataSets. The Nexosis API helps you build the best model to fit your DataSet and then you can repeatedly use that model in real time.

-----
## Creating Regression Models

#### Creating a regression model starts with the data just like any other prediction capability of our API. To understand how to submit data, you can read more in the [Sending Data](/guides/sending-data) article.

Once you have a DataSet you are ready to get going. If you have used the API to do timeseries forecasting, some of this will be familiar but there is at least one new step. When building a regression model you don't get your prediction results right away. In this case we're going to build a model with your DataSet, but you'll submit more feature data to get predictions only after the model is built.

Let's back up and go through the steps in detail.

### Starting A Model Building Session
Building a model means training an algorithm so that we can use it over and over again with data that it hasn't seen before to get your predictions. Whereas in a timeseries scenario the most recent data in time can be the most important, a regression model can continue to be useful without taking new observations into account. Like any model you will want to refresh it occasionally but in general you'll keep using one regression model for a longer period than for timeseries. That said, model building is our first step and we do that by running a model building session by posting to the following API endpoint...

```url
https://ml.nexosis.com/v1/sessions/model
```
This endpoint requires a small json data body with the following parameters defined:

- dataSourceName: the data source is the name of either a DataSet or a View. This named data source will provide the observations on which to train the model.
- predictiondomain: while required, when running regression this is always just 'regression'.
- targetColumn: this optional field identifies exactly one column in the data source which will be predicted. All other columns in your data source will be treated as 'features' by default. If you have already provided a target in your column definitions you don't need to pass this here.
- columns: You can specify column metadata to indicate some advanced options for how we should treat the columns in your named data source. You can [learn more about metadata in our guide](/guides/column-metadata).

Let's say you have a data source called HousingData and the target column is *SalePrice*; then you would send

``` json
{
  "dataSourceName": "HousingData",
  "targetColumn": "SalePrice",
  "predictionDomain": "regression"
}
```
The Nexosis API will then go and figure out the best model to build for the data source and target you have identified. We'll build several different models with different algorithms and compare the results for you in order to pick the best one.

### Model Session Results
A model training session can last for a while so we don't return results right away. The initial call does however return a SessionId which you can use to check the status of your session by occasionally making a call to the 'get session' endpoint
```url
https://ml.nexosis.com/v1/sessions/{your session id}
```
When your model has been built the status will be 'completed' and the modelId property of the session response will be populated. This is a GUID/UUID similar to the sessionId itself, but is unique to the model. Once your session is complete you can check out the model test results through the session results endpoint
```url
https://ml.nexosis.com/v1/sessions/{your session id}/results
```
The session results will include a *data* property which is populated with the observations taken from your DataSet as the 'test' data. When we train a model we use a large portion of the data to train - but hold back a part to test the results in order to make sure we created the best possible model. This test data is now returned along with the original values you sent marked with a *:actual* suffix.

``` json
{
  ...session info elided
  "data": [
    {
      "MyFeatureColumn": 2.9,
      "MyTargetColumn": 3.234,
      "MyTargetColumn:actual:" 3.175
    },
    {
      "MyFeatureColumn": 1.7,
      "MyTargetColumn": 2.925,
      "MyTargetColumn:actual:" 2.842
    }
   	 ... additional test data
  ] 
}
```
On the session results for your model you will also find metrics we calculated on your model's performance

``` json
  "metrics": {
    "meanAbsoluteError": 15990.948459514153,
    "meanAbsolutePercentError": 0.092227013821557124,
    "rootMeanSquareError": 29872.102288912662
  },
```
While multiple metrics are used to test the accuracy of our models, the *meanAbsolutePercentError* metric is good one to look at as it is straightforward to understand. Taken as a percent this simply shows the average error of each tested prediction against the actual values. A lower percentage is better.
#### Using Your Trained Model
Once a trained model exists we expose it as an endpoint on our API using the unique modelId value which was returned by the completed session.

```url
https://ml.nexosis.com/v1/models/{the model id returned}/predict
```

In order to call this new endpoint you will need to have a set of values on which to predict. This set of values must match the features you submitted with the DataSet that trained the model.  For example if you trained on

``` json
{
  "data": [
    "feature1": 8489.22,
    "feature2": 33.4,
    "target": 23
  ]
}
``` 
Then you need to submit a set of values for both 'feature1' and 'feature2' in order to predict on 'target'.

The prediction endpoint will take an array of feature value hashes and return a prediction for each one. So, if we want just one prediction we will post the following to the endpoint:

``` json
{
  "data": [
    "feature1": 8158.86,
    "feature2": 29.5
  ]
}
``` 

You will get a response back which also has a *data* array - this time including your target with a predicted value.

``` json
{
  "data": [
    "feature1": 8158.86,
    "feature2": 29.5,
    "target": 21.2
  ]
}
```