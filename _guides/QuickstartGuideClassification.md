---
title: Classification Quick Start
description: Getting started guide for using the Nexosis API
copyright: 2017 Nexosis
layout: default 
category: Classification
tags: [Quick Links, Favorite, REST, QuickStart, Classification]
use_codestyles: true
order: 2
---

This is a quick walkthrough of the basics of the using the Nexosis API.  By following this walkthrough which uses a sample dataset, you will learn all of the steps needed to predict classes using the Nexosis API.

------

## Step 1: Prepare data for upload

In order to start using the Nexosis API, you'll need to upload some data for the API to process. Data can be uploaded by posting the rows and columns as JSON, or, as a CSV file.  We have [several DataSets available](https://github.com/Nexosis/sampledata){:target="_blank"} which includes the famous iris data with some identifiable features of different species of Iris flowers.  The [Iris JSON DataSet](https://github.com/Nexosis/sampledata/blob/master/iris.json){:target="_blank"} includes data like this (json shown):

``` json
[
  {
    "sepal_len": 5.1,
    "sepal_width": 3.5,
    "petal_len": 1.4,
    "petal_width": 0.2,
    "iris": "setosa"
  },
... more instances of iris features
]
```

The *iris* field is our target value.  The other columns will be treated as features which the Nexosis API can run algorithms against. Note that string valued targets are preserved when building a classification model though that wouldn't make sense with other model types.

------

## Step 2: Start a Session

Now that we have some data, let's upload it and get a model which we can use to classify irises from previously unseen values.  To do this, we need to first send our data to a `DataSet`.  Then we start a `Session`, referencing the `DataSet` we just created and containing parameters needed to determine how the Nexosis machine learning algorithms should work.  Once the `Session` is started, our algorithms will start crunching the numbers to produce a model.

For this DataSet, we want to classify iris type given some measurements of the flower in centimeters.  The `TargetColumn` parameter will then be specified as *iris*.

Putting this all together, we will have a two requests that look like the ones below.  Make sure to replace the `{subscription key}` section with your [actual subscription key](https://developers.nexosis.com/developer), and replace the file path with the path to one of the sample files that was downloaded earlier.

### Upload a file

#### JSON File 
If you're using the `iris.json` file from our sampleData repo: [Iris JSON DataSet](https://github.com/Nexosis/sampledata/blob/master/iris.json){:target="_blank"}. Make sure to submit the correct `Content-Type` header.

``` bash
curl -v -X PUT "https://ml.nexosis.com/v1/data/iris" \
            -H "Content-Type: application/json" \
            -H "api-key: {subscription key}" \
 --data-binary "@/path/to/file/iris.json"
```

#### CSV File
If you're using the `iris.csv` file from our sampleData repo: [Iris CSV DataSet](https://github.com/Nexosis/sampledata/blob/master/iris.csv){:target="_blank"}. Make sure to submit the correct `Content-Type` header.

``` bash
curl -v -X PUT "https://ml.nexosis.com/v1/data/iris" \
            -H "Content-Type: text/csv" \
            -H "api-key: {subscription key}" \
 --data-binary "@/path/to/file/iris.csv"
 ```

### Start a Session

``` bash
curl -v -X POST "https://ml.nexosis.com/v1/sessions/model \
             -H "Content-Type: application/json" \
             -H "api-key: {subscription key}" \
             -d '{"dataSourceName": "iris", "predictionDomain": "classification", "targetColumn": "iris"}'
```

Once the session has been started, you should see a response similar to this:

``` JSON
{
    "columns": {
        ... columns elided...
    },
    "sessionId": "015fdb7d-7161-49a1-9ac7-6cee6818842d",
    "type": "model",
    "status": "requested",
    "predictionDomain": "classification",
    "balance": true,
    "requestedDate": "2017-11-20T22:00:21.361294+00:00",
    "statusHistory": [
        {
            "date": "2017-11-20T22:00:21.361294+00:00",
            "status": "requested"
        }
    ],
    "extraParameters": {},
    "messages": [],
    "dataSourceName": "iris",
    "dataSetName": "iris",
    "targetColumn": "iris",
    "isEstimate": false,
    ... other information elided...
}

```

Here we can see that we have a `sessionId`, which we will need later on.  Also, the `status` of the session is now `requested`.  The parameters that we sent up before are also echoed back to us.  Now that we have requested a session, we can check the status to see when it completes by sending a GET with the `sessionId` we just got.

### Check status of session

``` bash
curl -v -X GET "https://ml.nexosis.com/v1/sessions/{sessionId}" \
            -H "api-key: {subscription key}"
```

Once this request comes back with a `status` of `completed`, the model will be available for us to use in prediction request.

------

## Step 3: Predict Class

Classification predictions can be made by issuing a POST to the model/predict endpoint. When you request a prediction you send in a JSON body which contains the feature values on which the prediction should be made.

### Get the model id
In order to predict you first need the modelId for the model trained by the session in step 2. This comes back in the session results

``` bash
curl -v -X GET "https://ml.nexosis.com/v1/sessions/{sessionId}/results" \
            -H "api-key: {subscription key}"
```

This response will look a lot like the other above but importantly has the field *modelId*.

``` json
{
  "sessionId": "015fdb7d-7161-49a1-9ac7-6cee6818842d",
  "type": "model",
  "status": "completed",
  "predictionDomain": "regression",
  "modelId": "8faca9ec-1e43-4400-a85c-8c2af5186dd5",
  "requestedDate": "2017-11-20T22:00:21.361294+00:00"
}
```

### Predict using model
With the model id and a set of new values for the features you're ready to request a prediction.

``` bash
curl -v -X POST "https://ml.nexosis.com/v1/models/{modelId}/predict" \
            -H "api-key: {subscription key}" \
            -H "Content-Type: application/json" \
            -d '{"data":[{ "sepal_len": "5.1", "sepal_width": "3.5", "petal_len": "1.4", "petal_width": "0.2"}] }'
```

The body of the response will include your data field echoed back to you but this time with the predicted class filled in.

``` json
{
    "data": [
        {
            "sepal_len": "5.1",
            "sepal_width": "3.5",
            "petal_len": "1.4",
            "petal_width": "0.2",
            "iris": "setosa"
        }
    ],
...
}
```

------

## Next steps

The Nexosis API can also do [timeseries forecasting](http://docs.nexosis.com/guides/quickstartguideforecast) and impact analysis of events, and [regression analysis](http://docs.nexosis.com/guides/quickstartguidepredict) to create predictions of continuous variables.

Now that you are familiar with the basics, try getting predictions from new datasets, or, take a look at the [code samples](https://github.com/Nexosis?utf8=✓&q=samples) and [client libraries](/clients), and write an application which integrates with the API.  Show us what you were able to build!
