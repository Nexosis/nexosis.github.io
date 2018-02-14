---
title: Retrieving Data
description: How do I get my results and my original data back out?
category: Nexosis Concepts
subcategory: Upload your data
tags: [data, csv, json]
quick_link: true
use_codestyles: true
order: 2
---

Once you've [sent data](sending-data) to the Nexosis API and perhaps run a [Forecast](forecasting-walkthrough) session to generate some forecasts, how do you get those results back out?

------

## Listing DataSets

#### You can retrieve a listing of all of the DataSets you've uploaded by issuing a `GET` to [/data]({{site.api_reference_baseurl}}/operations/5919ef80a730020dd851f231).

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
------

## Retrieving Data

#### To retrieve data from a DataSet, issue a `GET` request to [/data/\{dataSetName\}]({{site.api_reference_baseurl}}/operations/5919ef80a730020dd851f232), where `dataSetName` is the name you provided when uploading the DataSet, or in the case of a Session-Scoped DataSet the generated name.  You can get that name from the Links section of the [Session](sessions).

You can also pass one or more of the following optional parameters in the query string.

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