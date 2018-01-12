---
title: Anomaly Quick Start
description: Getting started guide for using the Nexosis API
copyright: 2018 Nexosis
layout: default 
category: Anomaly Detection
tags: [Quick Links, Favorite, REST, QuickStart, Anomaly Detection]
use_codestyles: true
order: 4
---

This is a quick walkthrough of the basics of the using the Nexosis API.  By following this walkthrough which uses a sample dataset, you will learn all of the steps needed to find anomalies using the Nexosis API.

------

## Step 1: Prepare data for upload

In order to start using the Nexosis API, you'll need to upload some data for the API to process. Data can be uploaded by posting the rows and columns as JSON, or, as a CSV file.  We have [several DataSets available](https://github.com/Nexosis/sampledata){:target="_blank"} which includes a [file named cardio.json](https://raw.githubusercontent.com/Nexosis/sampledata/master/cardio.json){:target="_blank"} which contains some heartbeat data that help identify abnormal signals. That DataSet includes data like this:

``` json
 "data": [
    {
      "X.1": 0.00491231466176838,
      "X.2": 0.693190774919386,
      "X.3": -0.203640485666361,
      "X.4": 0.595322119977384,
      "X.5": 0.353189609125384,
      "X.6": -0.0614006444954018,
      "X.7": -0.278294946006147,
      "X.8": -1.65044441802655,
      "X.9": 0.759072460103087,
      "X.10": -0.420487347915062,
      "X.11": 0.37214915553696,
      "X.12": 1.48597291930713,
      "X.13": -0.798376447055488,
      "X.14": 1.85472761207619,
      "X.15": 0.622631032861473,
      "X.16": 0.963082541396573,
      "X.17": 0.301464348018426,
      "X.18": 0.193113439105929,
      "X.19": 0.231497949713972,
      "X.20": -0.28978574486636,
      "X.21": -0.493293969440142,
      "target": 0,
      "key": 1
    },
    ...
```

There are 3 types of columns here. All of the *X.#* columns are the measurements which make up the features of the dataset. The *target* column identifies whether or not the record is an anomaly. The [metadata included in the json file](http://docs.nexosis.com/guides/columnmetadata){:target="_blank"} is going to mark this column to be ignored. If you include this column as a target then you can create a classification model. In this case we're going to try and detect anomalies without it. Finally, there is a key column which we're going to identify as the key in the metadata so we can match up anomalous rows with the original dataset if desired.

> You can execute all of the following code in a terminal window based on the [anomalies gist from the Nexosis GitHub account](https://gist.github.com/nexosisops/f328d588b0238b50c54260392d1a82b5){:target="_blank"}. Be sure and add your own api key and to check your version of Python as the gist adds some JSON results parsing not represented below.

------

## Step 2: Start a Session

Now that we have some data, let's upload it and get a model which we can use to find anomalies.  To do this, we need to first upload the data so it becomes a `DataSet`.  Then we start a `Session`, referencing the `DataSet` we just created and containing parameters needed to determine how the Nexosis machine learning algorithms should work.  Once the `Session` is started, our algorithms will start crunching the numbers to produce a model.

Putting this all together, we will have a two requests that look like the ones below.  Make sure to replace the `{subscription key}` section with your [actual subscription key](https://developers.nexosis.com/developer).

### Import the JSON file

``` bash
curl -s -X POST "https://ml.nexosis.com/v1/imports/url" \
			-H "Content-Type: application/json" \
			-H "api-key: {subscription key}" \
			-d '{"dataSetName":"cardio","url":"https://raw.githubusercontent.com/Nexosis/sampledata/master/cardio.json"}'
```

### Start a Session

``` bash
curl -s -X POST "https://ml.nexosis.com/v1/sessions/model" \
             -H "Content-Type: application/json" \
             -H "api-key: {subscription key}" \
             -d '{"dataSourceName": "cardio", "predictionDomain": "anomalies"}'
```

Once the session has been started, you should see a response similar to this:

``` JSON
{
    "columns": {
        ... columns elided...
    },
    "sessionId": "0160ea7b-3306-4a58-9655-8075306c7d96",
    "type": "model",
    "status": "requested",
    "predictionDomain": "anomalies",
    "availablePredictionIntervals": [],
    "requestedDate": "2018-01-12T13:07:10.72398+00:00",
    "statusHistory": [
        {
            "date": "2018-01-12T13:07:10.72398+00:00",
            "status": "requested"
        }
    ],
    "extraParameters": {
        "containsAnomalies": true
    },
    "messages": [],
    "dataSourceName": "cardio",
    "dataSetName": "cardio",
    "targetColumn": "anomaly",
    ... other information elided...
}

```

Here we can see that we have a `sessionId`, which we will need later on.  Also, the `status` of the session is now `requested`.  The parameters that we sent up before are also echoed back to us.  Now that we have requested a session, we can check the status to see when it completes by sending a GET with the `sessionId` we just got.

### Check status of session

``` bash
# replace {sessionId} from the response to your session request
curl -v -X GET "https://ml.nexosis.com/v1/sessions/{sessionId}" \
            -H "api-key: $apiKey"
```

Once this request comes back with a `status` of `completed` two things will be available: first, we will be able to get the anomalies found by requesting the results; second, we will now have a model that can identify anomaly or not on future data points.

------

## Step 3: Retrieve Anomalies
The Nexosis API identified anomalous observations in the DataSet as part of the model building process. These anomalies will be returned as part of the "data" array property on the results response. We can get that by making a GET request to the results endpoint:

``` bash
# replace {sessionId} from the response to your session request
curl -s -H "api-key: {subscription key}" https://ml.nexosis.com/v1/sessions/{sessionId}/results
            
```

The results JSON contains a data array with each anomalous observation containing a new field named 'anomaly'. This field contains a score of the observation record as anomalous. When this result is a negative number, the record is anomalous. In this case all scores are negative because we have returned only anomalous entries. The score is a relative rank of how anomalous, such that a more negative number is more anomalous than another record with a score closer to zero.
 
``` json
{
	"metrics": {
        "percentAnomalies": 0.10049153468050245
    },
	"data": [
        {
            "key": "141",
            "anomaly": "-0.0452571326093725",
            "X.1": "0.00491231466176838",
            "X.2": "2.25671813545889",
            "X.3": "-0.203640485666361",
            "X.4": "-0.757128570390479",
            ... additional values and entries ...
        }
    ],
    ... other information elided...
}
```

## Step 4: Predict Anomalies
Predictions can be made by issuing a POST to the model/predict endpoint. When you request a prediction you send in a JSON body which contains the feature values on which the prediction should be made.

### Get the model id
In order to predict you first need the modelId for the model trained by the session in step 2. This comes back in the session results

``` bash
curl -s -X GET "https://ml.nexosis.com/v1/sessions/{sessionId}/results" \
            -H "api-key: {subscription key}"
```

This response will look a lot like the other above but importantly has the field *modelId*.

``` json
{
  	"sessionId": "0160ea7b-3306-4a58-9655-8075306c7d96",
    "type": "model",
    "status": "completed",
    "predictionDomain": "anomalies",
    "availablePredictionIntervals": [],
    "modelId": "10777ba7-d3f4-4da7-9405-21e53b758832",
    "requestedDate": "2018-01-12T13:07:10.72398+00:00",
    ...
}
```

### Predict using model
With the model id and a set of new values for the features you're ready to request a prediction.

``` bash
curl -s -H "api-key: $apiKey" \
		 -X POST $baseUrl/models/10777ba7-d3f4-4da7-9405-21e53b758832/predict \
		 -H "Content-Type: application/json" 
		 -d '{ "data": [{"X.1": -1.15907509242824,"X.2": -0.91998843744384,"X.3": -0.178808273342188,"X.4": 0.0119005653086114,"X.5": 3.14482390535516,"X.6": 17.314053637174,"X.7": -0.278294946006147,"X.8": 1.28100338235996,"X.9": 0.759072460103087,"X.10": -0.420487347915062,"X.11": -1.39808792464813,"X.12": 0.744634905731968,"X.13": -1.26165666571131,"X.14": -0.400329665815939,"X.15": 0.278624569020388,"X.16": 0.963082541396573,"X.17": -4.14708829524105,"X.18": -3.19083518334924,"X.19": -3.21686477340904,"X.20": 2.58264294083969,"X.21": -2.12660546680886,"key": 1780}]}'
```

Just as with the data in the session result, the body of the response will include your data echoed back to you but this time with the *anomaly* field filled in:

``` json
{
    "data": [
        {
            "X.1": "-1.15907509242824",
            "X.2": "-0.91998843744384",
            "X.3": "-0.178808273342188",
            "X.4": "0.0119005653086114",
            "X.5": "3.14482390535516",
            "X.6": "17.314053637174",
            "X.7": "-0.278294946006147",
            "X.8": "1.28100338235996",
            "X.9": "0.759072460103087",
            "X.10": "-0.420487347915062",
            "X.11": "-1.39808792464813",
            "X.12": "0.744634905731968",
            "X.13": "-1.26165666571131",
            "X.14": "-0.400329665815939",
            "X.15": "0.278624569020388",
            "X.16": "0.963082541396573",
            "X.17": "-4.14708829524105",
            "X.18": "-3.19083518334924",
            "X.19": "-3.21686477340904",
            "X.20": "2.58264294083969",
            "X.21": "-2.12660546680886",
            "key": "1780",
            "anomaly": "-0.101920780676883"
        }
    ],
		...
}
```

------

## Next steps
Check out [the gist](https://gist.github.com/nexosisops/f328d588b0238b50c54260392d1a82b5){:target="_blank"} which encapsulates all of these steps and evaluate results against the live API, test your own values, etc. It's free to play around with the API and start learning.

Now that you are familiar with the basics, try getting predictions from new datasets, or, take a look at the [code samples](https://github.com/Nexosis?utf8=✓&q=samples) and [client libraries](/clients), and write an application which integrates with the API.  Show us what you were able to build!

Check out other quick starts for different types of predictions:

* [Timeseries Forecasts](http://docs.nexosis.com/guides/quickstartguideforecast)
* [Regression](http://docs.nexosis.com/guides/quickstartguidepredict)
* [Classification](http://docs.nexosis.com/guides/quickstartguideclassification)