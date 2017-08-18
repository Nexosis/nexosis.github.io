---
title: Generating Predictions
description: Lean how to genreate forecasts using the Nexosis API
copyright: 2017 Nexosis 
layout: default
category: Forecasting
tags: [Predict, Quick Links, Favorite]
use_codestyles: true
---

A Forecast is when we take your historical time series data and predict what will happen in the future. You can forecast product demand at a sku level, predict call volume in a call center, forecast machine usage to get a better understanding of when equipment might need maintenance, anticipate wait times, or foresee turnover rates.

-----

### Creating a Forecast Session
Forecast sessions is the mechanism in the API to generate these predictions. To create a forecast session, sspecify the datasource for which to forecast from, as well as the start and end dates of the forecast period. If the saved dataset includes `feature` columns, it must have values for those features over the prediction period.

To understand how to submit data, you can read more in the [Sending Data](/guides/sendingdata) article.

*Here are some important points to consider:*
> The forecast start date should be on the same day as (or before) the last date in the dataset. If there is a gap between your forecast start date and the date of the last record in your data set, the Nexosis API will behave as if there is no gap. Read our [Missing Values](/guides/missingvalues) article for more information about how we handle gaps.


Here are the Query String Parameters you can pass along when creating a Forecast Session:
* `dataSetName` (required) - Name of the dataset to forecast. 
* `targetColumn` - Column in the specified dataset to forecast
* `startDate` - Format - date-time (as date-time in ISO8601). First date to forecast
* `endDate` - Format - date-time (as date-time in ISO8601). Last date to forecast
* `resultInterval` - The interval at which predictions should be generated. Possible values are `Hour`, `Day`, `Week`, `Month`, and `Year`. Defaults to `Day`
* `callbackUrl` - The Webhook url that will receive updates when the Session status changes. If you provide a callback url, your response will contain a header named Nexosis-Webhook-Token. You will receive this same header in the request message to your Webhook, which you can use to validate that the message came from Nexosis.
* `isEstimate` - If specified, the submitted data will not be saved, and the session will not be processed. 

> To get a Cost Estimate, include and set the `isEstimate` query string parameter to `true`. The returned `Nexosis-Request-Cost` header will be populated with the estimated cost that the request would have incurred.

### Column Metadata

Columns Metadata is optional, but can be included to fix or change the DataSet metadata. If the Columns Metadata is not compatible with the DataSet, it will throw a 400 Error code. 

> <b>NOTE:</b> If you are submitting `feature` columns in the columns metadata you MUST include values for those features in the named DataSet for the prediction period.

For example, if you are uploading the data in green below, you should provide data for features over the prediction period indicated by the `blue` columns, and the platform can then more accurately predict the values for yellow. For example 2/15/2010 (President's day) is in the prediction range and set to true since it's a holiday.

<table class="table table-bordered mb20">
    <thead>
        <tr>
            <th>timestamp</th>
            <th>sales</th>
            <th>holiday</th>
            <th>promo1</th>
        </tr>
    </thead>
    <tbody>
        <tr class="success">
            <td>2010-02-11T00:00Z</td>
            <td class="right">24924.5</td>
            <td>false</td>
            <td>true</td>
        </tr>
        <tr class="success">
            <td>2010-02-12T00:00Z</td>
            <td class="right">46039.49</td>
            <td>false</td>
            <td>false</td>
        </tr>
        <tr class="success">
            <td>2010-02-13T00:00Z</td>
            <td class="right">41595.55</td>
            <td>false</td>
            <td>false</td>
        </tr>
        <tr class="success">
            <td>2010-02-14T00:00Z</td>
            <td class="right">19403.54</td>
            <td>false</td>
            <td>false</td>
        </tr>
        <tr class="info">
            <td>2010-02-15T00:00Z</td>
            <td class="warning"></td>
            <td>true</td>
            <td>false</td>
        </tr>
        <tr class="info">
            <td>2010-02-16T00:00Z</td>
            <td class="warning"></td>
            <td>false</td>
            <td>false</td>
        </tr>
        <tr class="info">
            <td>2010-02-17T00:00Z</td>
            <td class="warning"></td>
            <td>false</td>
            <td>false</td>
        </tr>                
    </tbody>
</table>

<ul id="profileTabs" class="nav nav-tabs">
    <li class="active"><a href="#json" data-toggle="tab">JSON</a></li>
</ul>
<div class="tab-content">
    <div role="tabpanel" class="tab-pane active" id="json">
      <p>`timestamp` is the date and time being observed and `values` are a dictionary of values observed at that time.</p>
      <pre class="language-bash">
        <code class="language-bash code-toolbar">
curl -v -X POST "https://ml.nexosis.com/v1/sessions/forecast?dataSetName=Location-A&targetColumn=sales&startDate=01/01/2017T00:00:00Z&endDate=01/05/2017T00:00:00Z&resultInterval=day"
-H "Content-Type: application/json"
-H "api-key: {subscription key}"

--data-ascii "{body}"
        </code>
      </pre>
       <pre class="language-javascript">
          <code class="language-json code-toolbar">
{
  "columns": {
    "timestamp": {
      "dataType": "date",
      "role": "timestamp"
    },
    "sales": {
      "dataType": "numeric",
      "role": "target"
    },
    "orders": {
      "dataType": "numeric",
      "role": "feature"
    }
  }
}
        </code>
      </pre>
    </div>
</div>

### Request Response

An `200 OK` response will include the following JSON output, confirming the Columns metadata, target column, and the parameters of the forecast.

``` json
{
  "sessionId": "bb55b431-cfce-4dc7-bec4-24ee3b1d0d56",
  "type": "forecast",
  "status": "requested",
  "requestedDate": "0001-01-01T00:00:00+00:00",
  "statusHistory": [],
  "extraParameters": {},
  "columns": {
    "timestamp": {
      "dataType": "date",
      "role": "timestamp"
    },
    "sales": {
      "dataType": "numeric",
      "role": "target"
    },
    {
      "sales": {
      "dataType": "numeric",
      "role": "target"
    }
  },
  "dataSetName": "Location-A",
  "targetColumn": "sales",
  "startDate": "2017-01-01T00:00:00+00:00",
  "endDate": "2017-01-05T00:00:00+00:00",
  "callbackUrl": "",
  "isEstimate": false,
  "resultInterval": "Day",
  "links": []
}
```


If the metadata was incompatible with the DataSet, a `400 Error` will be returned along with some error details.

``` json
{
  "statusCode": 400,
  "message": "Request is invalid",
  "errorType": "RequestValidation",
  "errorDetails": {
    "startDate": [
      "Value for startDate must be a valid date"
    ]
  }
}
```

To learn about other response codes and what they mean, read more on the [Error Codes](/guides/errorcodes) reference article.

### Retrieving Forecast Predictions

Forecast session results consist of the predictions for the dates specified when the session was created.

<ul id="profileTabs" class="nav nav-tabs">
    <li class="active"><a href="#json" data-toggle="tab">JSON</a></li>
</ul>
<div class="tab-content">
    <div role="tabpanel" class="tab-pane active" id="json">
      <p>`timestamp` is the date and time being observed and `values` are a dictionary of values observed at that time.</p>
      <pre class="language-bash">
        <code class="language-bash code-toolbar">
curl -v -X GET "https://ml.nexosis.com/v1/sessions/{sessionId}/results"
-H "api-key: {subscription key}"

--data-ascii "{body}"
        </code>
        </pre>
       <pre class="language-javascript">
          <code class="language-json code-toolbar">
{
  "metrics": {},
  "data": [
    {
      "timestamp": "2017-01-01T00:00:00+00:00",
      "sales": "2948.74805590679"
    },
    {
      "timestamp": "2017-01-02T00:00:00+00:00",
      "sales": "1906.35816791768"
    },
    {
      "timestamp": "2017-01-03T00:00:00+00:00",
      "sales": "4523.42595831904"
    },
    {
      "timestamp": "2017-01-04T00:00:00+00:00",
      "sales": "4586.85369155064"
    },
    {
      "timestamp": "2017-01-05T00:00:00+00:00",
      "sales": "4538.04628241634"
    }
  ],
  "sessionId": "f673f41f-ac5c-4b09-9c92-e71662a00c9e",
  "type": "forecast",
  "status": "completed",
  "requestedDate": "0001-01-01T00:00:00+00:00",
  "statusHistory": [],
  "extraParameters": {},
  "columns": {
    "timestamp": {
      "dataType": "date",
      "role": "timestamp"
    },
    "sales": {
      "dataType": "numeric",
      "role": "target"
    }
  },
  "dataSetName": "Location-A",
  "targetColumn": "sales",
  "startDate": "2017-01-01T00:00:00+00:00",
  "endDate": "2017-12-31T00:00:00+00:00",
  "callbackUrl": "",
  "isEstimate": false,
  "resultInterval": null,
  "links": []
}
        </code>
      </pre>
    </div>
</div>
