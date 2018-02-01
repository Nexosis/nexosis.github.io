---
title: Sending & Updating Data
description: In order to forecast or measure Impact, we need your data in JSON or CSV format.
copyright: 2017 Nexosis 
layout: default
category: Concepts
tags: [Data, CSV, JSON, Quick Links]
use_codestyles: true
order: 4
---

Before the Nexosis API can do anything really useful, you're going to need to send it some data that it can use for [Forecasts](forecasting-walkthrough) or [Impact Analysis](impact-analysis).

------

## Datasets

#### You can send data to the Nexosis API through the [Data]({{ site.api_reference_baseurl }}/operations/5919ef80a730020dd851f233) endpoint.

Data can be sent to the Nexosis API as either `JSON` or `CSV`, with optional [metadata](column-metadata)

------

## Examples

#### In this example we'll create a Named DataSet called *sales* by sending some data to it.


<ul id="profileTabs" class="nav nav-tabs">
    <li class="active"><a href="#json" data-toggle="tab">JSON</a></li>
    <li><a href="#csv" data-toggle="tab">CSV</a></li>
</ul>
<div class="tab-content">
    <div role="tabpanel" class="tab-pane active" id="json">
      <p>`timestamp` is the date and time being observed and `values` are a dictionary of values observed at that time.</p>
      <pre class="language-bash">
        <code class="language-bash code-toolbar">
          curl -v -X PUT "https://ml.nexosis.com/v1/data/sales" \
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
                "sales": 1000.0,
                "orders": 100
              },
              {
                "timestamp": "2017-05-26T00:00:00+00:00",
                "sales": 980.0,
                "orders": 98
              },
              {
                "timestamp": "2017-05-27T00:00:00+00:00",
                "sales": 1100.0,
                "orders": 110
              },
              {
                "timestamp": "2017-05-28T00:00:00+00:00",
                "sales": 1080.0,
                "orders": 108
              },
              {
                "timestamp": "2017-05-29T00:00:00+00:00",
                "sales": 1110.0,
                "orders": 111
              }
            ]
          }
          </code>
      </pre>
    </div>
    <div role="tabpanel" class="tab-pane" id="csv">
      <div>
      <h3>TimeStamps</h3>
      <p>The API will assume a <em>timestamp</em> role column if your data contains a single date type column with unique values and you have not otherwise specified a key or timestamp role column in metadata.</p>
      <p>
      If your csv data has a timestamp with a different column name, you can provide that name with a parameter in the query string named <em>timestampColumn</em>.  For JSON data, simply specify the correct role in the <a href="/guides/column-metadata">column metadata</a>.
      </p>
      </div>
      <pre class="language-bash">
        <code class="language-bash code-toolbar">
          curl -v -X PUT "https://ml.nexosis.com/v1/data/sales?timestampColumn=date" \
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
  "dataSetName": "sales"
}
```

------

## Updating Data

#### You can modify a keyed DataSet or a time-series DataSet with additional and/or updated data by issuing a `PUT` to the same DataSet name. If your data has no user provided key, nor a timestamp role column then new data will simply be appended to the DataSet.

So, issuing a `PUT` to the same *sales* DataSet above like so

``` bash
curl -v -X PUT "https://ml.nexosis.com/v1/data/sales?timestampColumn=date" \
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