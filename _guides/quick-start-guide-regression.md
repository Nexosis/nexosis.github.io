---
title: Regression Quick Start
description: Getting started guide for using the Nexosis API
category: Regression
tags: [REST, Quick Start, Regression]
use_codestyles: true
order: 1
---

This is a quick walkthrough of the basics of the using the Nexosis API.  By following this walkthrough which uses a sample dataset, you will learn all of the steps needed to make predictions using the Nexosis API.

------

## Step 1: Prepare data for upload

#### In order to start using the Nexosis API, you'll need to upload some data for the API to process. 

Data can be uploaded by posting the rows and columns as JSON, or, as a CSV file.  We have [several DataSets available](https://github.com/Nexosis/sampledata){:target="_blank"} which includes a [file named auto-mpg.data.json](https://raw.githubusercontent.com/Nexosis/sampledata/master/auto-mpg.data.json){:target="_blank"} which contains various car engine specifications which help determine MPG.  That DataSet includes data like this:

``` json
 "data": [
    {
      "MPG": 18,
      "Cylinders": 8,
      "Displacement": 307,
      "Horsepower": 130,
      "Weight": 3504,
      "Acceleration": 12,
      "ModelYear": 70,
      "Origin": 1,
      "Name": "chevrolet chevelle malibu",
      "Make": "chevrolet"
    },
...
```

The MPG field is our target value.  The other columns are values which the Nexosis API can run algorithms against. Download this file to a local drive and make note of where you saved it; you'll need to put this file path in the request outlined in step 2.

------

## Step 2: Start a Session

#### Now that we have some data, let's upload it and get a model which we can use to predict MPG from previously unseen values.  

To do this, we need to first send our data to a `DataSet`.  Then we start a `Session`, referencing the `DataSet` we just created and containing parameters needed to determine how the Nexosis machine learning algorithms should work.  Once the `Session` is started, our algorithms will start crunching the numbers to produce a model.

For this DataSet, we want to predict the MPG given the engine specifications.  The `TargetColumn` parameter will then be specified as *MPG*.

Putting this all together, we will have a two requests that look like the ones below.  Make sure to replace the `{subscription key}` section with your [actual subscription key](https://developers.nexosis.com/developer), and replace the file path with the path to one of the sample files that was downloaded earlier.

### Upload a file

``` bash
curl -s -X PUT "https://ml.nexosis.com/v1/data/auto-mpg" \
            -H "Content-Type: application/json" \
            -H "api-key: {subscription key}" \
 --data-binary "@/path/to/file/auto-mpg.data.json"
```

### Start a Session

``` bash
curl -s -X POST "https://ml.nexosis.com/v1/sessions/model" \
             -H "Content-Type: application/json" \
             -H "api-key: {subscription key}" \
             -d '{"dataSourceName": "auto-mpg", "predictionDomain": "regression", "targetColumn": "mpg"}'
```

Once the session has been started, you should see a response similar to this:

``` JSON
{
    "columns": {
        ... columns elided...
    },
    "sessionId": "015f129a-4671-4863-8a16-9c267c9fcc8f",
    "type": "model",
    "status": "requested",
    "predictionDomain": "regression",
    "requestedDate": "2017-10-12T22:00:21.361294+00:00",
    "statusHistory": [
        {
            "date": "2017-10-12T22:00:21.361294+00:00",
            "status": "requested"
        }
    ],
    "extraParameters": {},
    "messages": [],
    "dataSourceName": "mpg-dataset",
    "dataSetName": "mpg-dataset",
    "targetColumn": "mpg",
    ... other information elided...
}

```

Here we can see that we have a `sessionId`, which we will need later on.  Also, the `status` of the session is now `requested`.  The parameters that we sent up before are also echoed back to us.  Now that we have requested a session, we can check the status to see when it completes by sending a GET with the `sessionId` we just got.

### Check status of session

``` bash
curl -s -X GET "https://ml.nexosis.com/v1/sessions/{sessionId}" \
            -H "api-key: {subscription key}"
```

Once this request comes back with a `status` of `completed`, the model will be available for us to use in prediction request.

------

## Step 3: Predict

#### Predictions can be made by issuing a POST to the model/predict endpoint. When you request a prediction you send in a JSON body which contains the feature values on which the prediction should be made.

### Get the model id
In order to predict you first need the modelId for the model trained by the session in step 2. This comes back in the session results

``` bash
curl -s -X GET "https://ml.nexosis.com/v1/sessions/{sessionId}/results" \
            -H "api-key: {subscription key}"
```

This response will look a lot like the other above but importantly has the field *modelId*.

``` json
{
  "sessionId": "015f129a-4671-4863-8a16-9c267c9fcc8f",
  "type": "model",
  "status": "completed",
  "predictionDomain": "regression",
  "modelId": "48c66472-03d6-4eb9-a15f-b2b2d7d60148",
  "requestedDate": "2017-10-12T22:00:21.361294+00:00"
}
```

### Predict using model
With the model id and a set of new values for the features you're ready to request a prediction.

``` bash
curl -s -X POST "https://ml.nexosis.com/v1/models/{modelId}/predict" \
            -H "api-key: {subscription key}" \
            -H "Content-Type: application/json" \
            -d '{"data":[{ "Make": "plymouth", "Origin": "1", "Weight": "3430", "Cylinders": "6", "ModelYear": "78", "Horsepower": "100", "Acceleration": "17.2", "Displacement": "225"}] }'
```

The body of the response will include your data field echoed back to you but this time with your predicted field

``` json
{
    "data": [
        {
            "Make": "plymouth",
            "Origin": "1",
            "Weight": "3430",
            "Cylinders": "6",
            "ModelYear": "78",
            "Horsepower": "100",
            "Acceleration": "17.2",
            "Displacement": "225",
            "MPG": "23.7877100428104"
        }
    ],
...
}
```

------

## Next steps

#### The Nexosis API can also do timeseries forecasting and impact analysis of events.  You can read more about forecasting in [that quick start guide](http://docs.nexosis.com/guides/quick-start-guide-forecast).

Now that you are familiar with the basics, try getting predictions from new datasets, or, take a look at the [code samples](https://github.com/Nexosis?utf8=✓&q=samples) and [client libraries](/clients), and write an application which integrates with the API.  Show us what you were able to build!
