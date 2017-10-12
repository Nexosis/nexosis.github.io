---
title: Forecasting Quick Start
description: Getting started guide for using the Nexosis API
copyright: 2017 Nexosis 
layout: default
category: Forecasting
tags: [Quick Links, Favorite, REST, QuickStart, TimeSeries]
use_codestyles: true
order: 2
---

This is a quick walkthrough of the basics of the using the Nexosis API.  By following this walkthrough which uses a sample dataset, you will learn all of the steps needed to make forecasts using the Nexosis API.

------

## Step 1: Prepare data for upload

In order to start using the Nexosis API, you'll need to upload some data for the API to process. Data can be uploaded by posting the rows and columns as JSON, or, as a CSV file.  We have a [few datasets available](https://github.com/Nexosis/sampledata) which have examples of sales per day at a store, and sales of a single product per day.  One of the sample datasets look like this:

``` csv
timeStamp,sales,transactions
2012-12-31 00:00:00,2922.13,459
2013-01-01 00:00:00,1500.56,195
2013-01-02 00:00:00,4078.52,696
...
```

Since we are working with time series data, each row must have a timestamp of when the measurement occurred.  The other columns are values which the Nexosis API can run algorithms against.  These are the values which we are interested in forecasting.

------

## Step 2: Start a Session

Now that we have some data, let's upload it and get a forecast of how these values will change over time.  To do this, we need to first send our data to a `Dataset`.  Then we start a `Session`, referencing the `Dataset` we just created and containing parameters needed to determine how the Nexosis machine learning algorithms should work.  Once the `Session` is started, our algorithms will start crunching the numbers to produce a set of forecast results.

For this dataset, we want to forecast the sales for the first quarter of 2017.  All we need to do is specify the `StartDate` and `EndDate` as `2017-01-01` and `2017-04-01`.  The `TargetColumn` parameter also needs to be specified, which is the value which will be forecasted for this date range.  We will set this value to `sales`.

Putting this all together, we will have a two requests that look like the ones below.  Make sure to replace the `{subscription key}` section with your [actual subscription key](https://developers.nexosis.com/developer), and replace the file path with the path to one of the sample files that was downloaded earlier.

### Upload a file

``` bash
curl -v -X POST "https://ml.nexosis.com/v1/data/location-a" \
-H "Content-Type: text/csv" \
-H "api-key: {subscription key}" \
--data-binary "@/path/to/file/Location A.csv"
```

### Start a Session

``` bash
curl -v -X POST "https://ml.nexosis.com/v1/sessions/forecast?dataSetName=location-a&targetColumn=sales&startDate=2017-01-01&endDate=2017-03-31" \
-H "Content-Type: application/json" \
-H "api-key: {subscription key}" \
```

Once the session has been started, you should see a response similar to this:

``` JSON
{
  "sessionId": "{sessionId}",
  "type": "forecast",
  "status": "requested",
  "extraParameters": {},
  "dataSetName": "location-a",
  "targetColumn": "sales",
  "startDate": "2017-01-01T00:00:00+00:00",
  "endDate": "2017-04-01T00:00:00+00:00",
  "columns": 
  {
    "timeStamp": { dataType: "date", role: "timestamp" },
    "sales": { dataType: "numeric", role: "target" },
    "transactions": { dataType: "numeric" }
  }
}
```

Here we can see that we have a `sessionId`, which we will need later on.  Also, the `status` of the session is now `requested`.  The parameters that we sent up before are also echoed back to us.  Now that we have requested a session, we can check the status to see when it completes by sending a GET with the `sessionId` we just got.

### Check status of session

``` bash
curl -v -X GET "https://ml.nexosis.com/v1/sessions/{sessionId}" \
-H "api-key: {subscription key}"
```

Once this request comes back with a `status` of `completed`, the forecast will be available for download.

------

## Step 3: Download results

Results can be downloaded by issuing a GET to the results endpoint.

### Download session results

``` bash
curl -v -X GET "https://ml.nexosis.com/v1/sessions/{sessionId}/results" \
-H "api-key: {subscription key}"
```

The body of this response is the forecasted values over the requested date range.  The results will be formatted like this.

``` JSON
{
  "data": [
    {
      "timestamp": "2017-01-01T00:00:00+00:00",
      "sales": 1911.46871429984
    },
    {
      "timestamp": "2017-01-02T00:00:00+00:00",
      "sales": 4330.20981465731
    },
    {
      "timestamp": "2017-01-03T00:00:00+00:00",
      "sales": 4573.98547777211
    },
...
```

The values object contains a key value pair of the column name, and the prediction value.  You can see what these results look like by plotting both datasets in your favorite charting library.

------

## Next steps

The Nexosis API can also do impact analysis of events.  This can be used to gauge, for example, how impactful promotions or special events are on sales numbers.  These types of sessions can be posted in the same way as forecasts, but to the `sessions/impact` endpoint.


Now that you are familiar with the basics, try running forecasts against new datasets, or, take a look at the [code samples](https://github.com/Nexosis?utf8=✓&q=samples) and [client libraries](/clients), and write an application which integrates with the API.  Show us what you were able to build!
