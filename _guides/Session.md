---
title: Sessions 
description: What is a Session in the Nexosis API?
copyright: 2017 Nexosis 
layout: default
category: Concepts
tags: [Causal Impact, Predict, Quick Links, Favorite]
use_codestyles: true
order: 6
---

A Session is a request for the Nexosis API to perform some calculations on data, resulting in either time-series results, or a model that you can use to perform predictions.

------

## Types of Sessions

#### A Session can be one of the following types:
* A model-building session, used to build a [Regression](regression), [Classification](quickstartguideclassification), or [Anomaly Detection](quickstartguideanomaly) model.
* A [Forecast](forecast) session, used to predict future values based on past values in a time-series dataset.
* An [Impact Analysis](impactanalysis) session, used to analyze impact of some event during a certain window of a time-series dataset.

------

## Starting a Session

#### To start a session, you issue a `POST` to one of the following URLs, based on the type of session you're trying to start:
* [/sessions/model]({{site.api_reference_baseurl}}/operations/59d79fa1adf47c0d60484fe9)
* [/sessions/forecast]({{site.api_reference_baseurl}}/operations/59149d7da730020f20dd41ab)
* [/sessions/impact]({{site.api_reference_baseurl}}/operations/59149d7da730020f20dd41aa)

### Session Requests

#### A request to start a session takes the following parameters in the json payload:

##### Model Building
* `dataSourceName` - The name of the data source (dataset or view) on whose data the session performs its calculations.
* `targetColumn` - The column in the data source that is the target of the calculation being performed.
* `predictionDomain` - The type of model to build: 'regression', 'classification', or 'anomalies'
* `columns` - A columns metadata object of the form 

``` json
"columns": {
	"columnName":{
		"property" : "value"
	}
}
```

Where property and value are [available metadata options](/guides/columnmetadata).
 

##### Forecast

* `dataSourceName` - The name of the data source on whose data the session performs its calculations.
* `targetColumn` - The column in the data source that is the target of the calculation being performed.  In the case of [Forecasting](forecast), this is column for which we want to generate predictions. 
* `startDate` - The start date of the session.  In Forecast sessions, this is the start of the forecast period.  In Impact Analysis sessions, this is the start of the even whose impact is being calculated.
* `endDate` - The end date of the session.
* `resultInterval` *(optional)* - The date/time interval (e.g. Day, Hour) at which predictions should be generated.  So, if `Hour` is specified for this parameter you will get a Result record for each hour between `startDate` and `endDate`.  If unspecified, we'll generate predictions at a `Day` interval.
* `callbackUrl` *(optional)* - The Webhook url that will receive updates when the Session status changes.  Those updates will come in the form of an HTTP `POST` with a `JSON` body that's the same as the response shown in [Retrieving a Session](#retrievingSession).
If you provide a callback url, your response will contain a header named Nexosis-Webhook-Token. You will receive this same header in the request message to your Webhook, which you can use to validate that the message came from Nexosis.
* `columns` - A columns metadata object. If your data source doesn't already have a column with the role of "timestamp", make sure to indicate which field should be used as the timestamp for this session. Data from your data source will be ordered and aggregated based on whatever column you specify as the timestamp for the session.

##### Impact Analysis
The same values as a forecast, with the addition of `eventName` - A label to put on the the thing whose impact we are trying to calculate.

------

## <span name="retrievingSession" class="jumptarget">Retrieving a Session</span>

#### You can retrieve an individual session by issuing a `GET` request to [/sessions/\{SessionId\}]({{site.api_reference_baseurl}}/operations/59149d7da730020f20dd41a8/)

The session response will look like the following:

``` json
{ "sessionId": "015c5f15-cbc6-43ee-a7a9-f7af8a456bd1",
  "type": "forecast",
  "status": "completed",
  "statusHistory":
   [ { "date": "2017-05-31T15:18:02.950271+00:00",
       "status": "requested" },
     { "date": "2017-05-31T15:18:03.0877088+00:00", "status": "started" },
     { "date": "2017-05-31T15:23:01.2470011+00:00", "status": "completed" } ],
  "extraParameters": {},
  "dataSourceName": "my-data-set",
  "targetColumn": "foo",
  "startDate": "2017-05-12T00:00:00+00:00",
  "endDate": "2017-07-01T00:00:00+00:00",
  "callbackUrl": "https://d8fe87ba.ngrok.io/payload",
  "resultInterval": "day",
  "links":
   [ { "rel": "results",
       "href": "https://ml.nexosis.com/v1/sessions/015c5f15-cbc6-43ee-a7a9-f7af8a456bd1/results" },
     { "rel": "data",
       "href": "https://ml.nexosis.com/v1/data/my-data-set" } ]
}
```

* `sessionId` - The unique identifier of the session
* `type` - The type of the session.  Either ["forecast"](forecast), ["impact"](impactanalysis), or "model".
* `status` - The status of the session.  Can be one of the following:
  * `requested` - The session has been submitted but not picked up yet.
  * `started` - Calculations have started on the session.
  * `completed` - Calculations have completed and the results are ready.
  <!--`cancelled` (leaving this out because we don't have cancel capability yet)-->
  * `failed` - There was a failure when trying to run the session.
* `statusHistory` - The history of status changes on this session.
* `extraParameters` - In the case of Impact sessions, this property will contain the event name, e.g. `{eventName: "MyEvent"}`.
* `dataSourceName` - The `dataSourceName` provided in the request to start the session. 
* `targetColumn` - The `targetColumn` from the request to start the session.
* `startDate` - The `startDate` from the request to start the session.
* `endDate` - The `endDate` from the request to start the session.
* `resultInterval` - The `resultInterval` from the request to start the session.
* `callbackUrl` - The `callbackUrl` from the request to start the session.


### Status Header

An individual session request will also respond with an HTTP Response header named `Nexosis-Session-Status`, containing the same status as in the body.  So, you can use the HTTP `HEAD` verb to only get this status without returning the session body.

### Session Links

The Session object returned also contains some links with urls pointing to resources related to that session:

* `results` - The results of the session.  Either the forecasted values, the Impact Analysis results, or the results of testing the generated model.
* `data` - Data source used for the session.

------

## Querying Sessions

#### You can list your sessions by issuing a `GET` request to [/sessions]({{site.api_reference_baseurl}}/operations/59149d7da730020f20dd41a6/)

A sessions query takes the following optional parameters in the query string

* `dataSourceName` - Limits sessions to those for a particular data source
* `eventName` - Limits impact sessions to those for a particular event
* `startDate` - Limits sessions to those created on or after the specified date
* `endDate` - Limits sessions to those created on or before the specified date

The response from this request will be an object with a Results property, containing an array of Session objects.

``` json
{
    "results": [
        { "sessionId": "015c5f15-cbc6-43ee-a7a9-f7af8a456bd1",
        "type": "forecast",
        "status": "completed",
        "statusHistory":
        [ { "date": "2017-05-31T15:18:02.950271+00:00",
            "status": "requested" },
            { "date": "2017-05-31T15:18:03.0877088+00:00", "status": "started" },
            { "date": "2017-05-31T15:23:01.2470011+00:00", "status": "completed" } ],
        "extraParameters": {},
        "dataSourceName": "my-data-set",
        "targetColumn": "foo",
        "startDate": "2017-05-12T00:00:00+00:00",
        "endDate": "2017-07-01T00:00:00+00:00",
        "callbackUrl": "https://d8fe87ba.ngrok.io/payload",
        "resultInterval": "day",
        "links":
        [ { "rel": "results",
            "href": "https://ml.nexosis.com/v1/sessions/015c5f15-cbc6-43ee-a7a9-f7af8a456bd1/results" },
            { "rel": "data",
            "href": "https://ml.nexosis.com/v1/data/my-data-set" } ]
        }
    ]
  }
```

------

## Retrieving Session Results

#### You can retrieve the results from an individual session by issuing a `GET` request to [/sessions/\{sessionId\}/results]({{site.api_reference_baseurl}}/operations/59149d7da730020f20dd41a7), where the `sessionId` is the unique Id for a session.

Session Results, in general, come back in the following form:

``` json
{
  "metrics": {},
  "session": {
    "sessionId": "273bd1d0-db27-4655-a68d-1774d2373e6b",
    "type": "forecast",
    "status": "completed",
    "statusHistory": [],
    "extraParameters": {},
    "dataSourceName": "Location-A",
    "targetColumn": "sales",
    "startDate": "2017-01-01T00:00:00+00:00",
    "endDate": "2017-12-31T00:00:00+00:00",
    "callbackUrl": "",
    "links": []
  },
  "data": [
    {
      "timestamp": "2017-01-01T00:00:00+00:00",
      "sales": 2948.74805590679
    },
    {
      "timestamp": "2017-01-02T00:00:00+00:00",
      "sales": 1906.35816791768
    },
    ...
  ]
}
```

* `metrics` - Metrics about the results.
* `session` - The session that generated these results
* `data` - The predicted values that the Nexosis API generated.

Interpretation of the values in `metrics` and `data` is different between the different types of sessions, so you'll want to read more about those session types in order to know how to interpret results.

### Model Session Results
Model building sessions will additionally return the `modelId` once the model has been successfully built.
