---
title: Retrieving Data
description: How do I get my results and my original data back out?
copyright: 2017 Nexosis 
layout: default
category: Getting Started
tags: [Data, CSV, JSON, Quick Links]
use_codestyles: true
order: 5
---

Once you've [sent data](importingdata) to the Nexosis API and perhaps run a [Forecast](forecast) session to generate some forecasts, how do you get those results back out?

------

## Listing DataSets

You can retrieve a listing of all of the DataSets you've uploaded by issuing a `GET` to [/data]({{site.api_reference_baseurl}}/operations/5919ef80a730020dd851f231).

Optionally, you can provide a `partialName` parameter in the query string that filters the DataSets you get back to only those whose name contain that string.

The response is a summary listing of your DataSets

``` json
{
    "dataSets": [
       { "dataSetName": "Sales" },
       { "dataSetName": "test1234" }
   ]
}
```

**Note** this listing only includes [Named DataSets.](importingdata)  It does not include Session-Scoped DataSets.

------

## Retrieving Data

To retrieve data from a DataSet, issue a `GET` request to [/data/\{dataSetName\}]({{site.api_reference_baseurl}}/operations/5919ef80a730020dd851f232), where `dataSetName` is the name you provided when uploading the DataSet, or in the case of a Session-Scoped DataSet the generated name.  You can get that name from the Links section of the [Session](session).

You can also pass one of the following optional parameters in the query string.

* `startDate` - Limits results to those on or after the specified date
* `endDate` - Limits results to those on or before the specified date
* `page` - Zero-based page number of results to retrieve
* `pageSize` - Count of results to retrieve in each page (max 1000)
* `include` - Limits results to the specified columns

The response from this request will look like the following:

``` json
{
  "data": [
    {
      "timestamp": "2013-01-01T00:00:00+00:00",
      "sales": 1500.56,
      "orders": 150
    },
    {
      "timestamp": "2013-01-02T00:00:00+00:00",
      "sales": 4078.52,
      "orders": 407
    },
    ...
  ]
}
```

Where `timestamp` is the date and time being observed and `values` are a dictionary of values observed at that time.

------

<!--
## Retrieving Forecasts

The results of [Forecast](forecast) sessions on a DataSet are accumulated under that DataSet, and you can get to them as a single forecast under [data/\{dataSetName\}/forecast]({{site.api_reference_baseurl}}/operations/591c8ff5a730020dd851f24d), where `dataSetName` is typically a Named Dataset.

You can also pass one of the following optional parameters in the query string.

* `startDate` - Limits results to those on or after the specified date
* `endDate` - Limits results to those on or before the specified date
* `page` - Zero-based page number of results to retrieve
* `pageSize` - Count of results to retrieve in each page (max 1000)
* `include` - Limits results to forecasts on the specified columns

### Forecasts example

Why is this feature useful?

Say you have your Sales DataSet as referenced above, and you want to build an application that generates a forecast line for those sales.

![Like this]({{site.url}}/assets/img/forecastline.png)

You want that forecast line more or less continually updated with new forecasts, letting you always see what the next few weeks will bring as well as how accurate your forecasts were in the past.

Additionally, you want to forecast both `sales` and `orders`from your DataSet and show both of those lines on your graph.

The Forecast endpoint in our API makes this trivial.

You simply run two separate Forecast sessions on your Sales DataSet, one for each `sales` and `orders`

``` text
GET /session/forecast?dataSetName=sales&targetColumn=sales&startDate=2016-12-01&endDate=2017-01-01
GET /session/forecast?dataSetName=orders&targetColumn=sales&startDate=2016-12-01&endDate=2017-01-01
```

Once those two sessions are complete, you can retrieve the forecast lines for *both* orders and sales from the same endpoint.

``` text
GET /data/sales/forecast
```

``` json
{ "data":
   [ { "timestamp": "2016-12-01T00:00:00+00:00",
       "values": { "orders": "791.31258260961", "sales": "5697.50107076731" } },
     { "timestamp": "2016-12-02T00:00:00+00:00",
       "values": { "sales": "5871.03094336378", "orders": "867.180972995588" } },
     { "timestamp": "2016-12-03T00:00:00+00:00",
       "values": { "orders": "605.690300736409", "sales": "3618.07590625415" } },
     { "timestamp": "2016-12-04T00:00:00+00:00",
       "values": { "orders": "412.105861305029", "sales": "2549.36159545999" } },
     { "timestamp": "2016-12-05T00:00:00+00:00",
       "values": { "sales": "4850.19476590616", "orders": "760.441346135503" } },
     { "timestamp": "2016-12-06T00:00:00+00:00",
       "values": { "orders": "782.855764849173", "sales": "4469.20081190654" } },
    ...
   ]
}
```

-->