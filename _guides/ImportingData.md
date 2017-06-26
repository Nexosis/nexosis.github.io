---
title: Sending Data
description: In order to forecast or measure Impact, we need your data in JSON or CSV format.
copyright: 2017 Nexosis 
layout: default
category: Getting Started
tags: [Data, CSV, JSON, Quick Links]
use_codestyles: true
order: 2
---

Before the Nexosis API can do anything really useful, you're going to need to send it some data that it can use for [Forecasts](forecast) or [Impact Analysis](impactanalysis).

------

## Named vs Session-scoped Data

There are two ways that you can send data for the Nexosis API to use: as a Named DataSet or as a DataSet attached to a specific [Session](session).

When you'll use each of these depends on how you intend to use the Nexosis API.

### Named DataSets

If you have a set of data that will need updated over time, or on which you'll want run many different kinds of analyses, then you'll want to send that data as a 'Named' DataSet.

Named DataSets are created and updated through the [Data]({{ site.api_reference_baseurl }}/operations/5919ef80a730020dd851f233) endpoint.

For example, you'll want to use this type of DataSet if you're forecasting sales data for the next month for a product.  For that case you'd want to create a named DataSet called 'sales', initially populate it with your historical sales data for that product, and then periodically send updated sales data to that same DataSet and generate a new set of forecasts.

### Session-scoped DataSets

Session-scoped DataSets are more for single-use scenarios.  For example, if you have a static DataSet for which you want to perform [Impact Analysis](ImpactAnalysis) a single time, then you'll want to send that data as session-scoped.

Session-scoped DataSets are created by sending the data in the body of a [Session](session) request.

------

## Examples

In this example we'll create a Named DataSet called *sales* by sending some data to it.

Data can be sent to the Nexosis API as either `JSON` or `CSV`.

<ul id="profileTabs" class="nav nav-tabs">
    <li class="active"><a href="#json" data-toggle="tab">JSON</a></li>
    <li><a href="#csv" data-toggle="tab">CSV</a></li>
</ul>
<div class="tab-content">
    <div role="tabpanel" class="tab-pane active" id="json">
      <p>`timestamp` is the date and time being observed and `values` are a dictionary of values observed at that time.</p>
      <pre class="language-bash">
        <code class="language-bash code-toolbar">
          curl -v -X PUT "https://ml.nexosis.com/api/data/sales" \
          -H "Content-Type: application/json" \
          -H "api-key: {subscription key}" \
          --data-binary "@/path/to/file/data.json"
        </code>
      </pre>
      
      <pre class="language-javascript">
          <code class="language-json code-toolbar">
          {
            "data": [
              {
                "timestamp": "2017-05-25T00:00:00+00:00",
                "values": {
                  "sales": 1000.0,
                  "orders": 100
                }
              },
              {
                "timestamp": "2017-05-26T00:00:00+00:00",
                "values": {
                  "sales": 980.0,
                  "orders": 98
                }
              },
              {
                "timestamp": "2017-05-27T00:00:00+00:00",
                "values": {
                  "sales": 1100.0,
                  "orders": 110
                }
              },
              {
                "timestamp": "2017-05-28T00:00:00+00:00",
                "values": {
                  "sales": 1080.0,
                  "orders": 108
                }
              },
              {
                "timestamp": "2017-05-29T00:00:00+00:00",
                "values": {
                  "sales": 1110.0,
                  "orders": 111
                }
              }
            ]
          }
          </code>
      </pre>
    </div>
    <div role="tabpanel" class="tab-pane" id="csv">
      <div>
      <h3>TimeStamps</h3>
      <p>
      By default, the API will assume that the timestamp in your CSV DataSet is in a column named *timestamp*.
      </p>
      <p>
      If your csv data has a timestamp with a different column name, you can provide that name with a parameter in the query string named *timestampColumn*
      </p>
      </div>
      <pre class="language-bash">
        <code class="language-bash code-toolbar">
          curl -v -X PUT "https://ml.nexosis.com/api/data/sales?timestampColumn=date" \
          -H "Content-Type: text/csv" \
          -H "api-key: {subscription key}" \
          --data-binary "@/path/to/file/data.csv"
        </code>
      </pre>
      <pre class="language-csv">
          <code class="language-csv code-toolbar">
          date,sales,orders
          2017-05-25,1000,100
          2017-05-26,980,98
          2017-05-27,1100,110
          2017-05-28,1080,108
          2017-05-29,1110,111
          </code>
      </pre>
    </div>
</div>

### Response

The response to the `PUT` will be an `HTTP 200` with a response body that is a summary of the DataSet.

``` json
{
  dataSetName: "sales"
}
```

------

## Updating Data

With a Named DataSet you are able to modify the DataSet with additional and/or updated data by issuing a `PUT` to the same DataSet name.

So, issuing a `PUT` to the same *sales* DataSet above like so

``` bash
curl -v -X PUT "https://ml.nexosis.com/api/data/sales?timestampColumn=date" \
-H "Content-Type: text/csv" \
-H "api-key: {subscription key}" \
--data-binary "@/path/to/file/data.csv"
```

``` csv
date,sales,orders
2017-05-29,1120,112
2017-05-30,1234,123
2017-05-31,1235,123
```

Will result in the sales DataSet containing the following records:

``` csv
2017-05-25,1000,100
2017-05-26,980,98
2017-05-27,1100,110
2017-05-28,1080,108
2017-05-29,1120,112
2017-05-30,1234,123
2017-05-31,1235,123
```

Where the timeStamp that overlapped between the first `PUT` and the second `PUT` overwrote the value from the first, and the non-overlapping timeStamps were appended.