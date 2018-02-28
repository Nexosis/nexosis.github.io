---
title: Scala Package
description: Use the Nexosis Java Package to communicate with Nexosis API in Scala
category: Scala
breadcrumb: API Clients
tags: [scala, package]
use_codestyles: true
---
<script src="/assets/js/prism/components/prism-scala.min.js" data-default-language="markup"></script>
> <h5 class="mt0">Supported platforms</h5>
The Scala client uses the Nexosis API Java package - tested with Scala 2.12.x.

## Installing the client

> <p><a href="https://mvnrepository.com/artifact/com.nexosis/nexosisclient-java" class="btn btn-primary mr10" target="_blank"><i class="fa fa-cube mr5"></i> Package Details</a><a href="https://github.com/Nexosis/nexosisclient-java" class="btn btn-primary" target="_blank"><i class="fa fa-github mr5"></i> View Source</a></p>

### SBT
Add this dependency to `build.sbt`:
``` scala
resolvers +=
  Resolver.sonatypeRepo("public")

// https://mvnrepository.com/artifact/com.nexosis/nexosisclient-java
libraryDependencies += "com.nexosis" % "nexosisclient-java" % "1.1.2"
```

## Scala Quick Start

### Initialize the Client

The <code>NexosisClient</code> has several constructor overloads.

To use the default constructor, create an environment variable called <code>NEXOSIS_API_KEY</code> and set it equal to your API Key. This also uses the default API Endpoint base URI <code>https://ml.nexosis.com/v1</code>.



```scala
    val client = new NexosisClient()
```

Initialize the client with an API key.

```scala
    val client = new NexosisClient(
      sys.env("NEXOSIS_API_KEY")
    )
```


Initialize the client with an API key and endpoint.

```scala
    val client = new NexosisClient(
      sys.env("NEXOSIS_API_KEY"),
      sys.env("NEXOSIS_BASE_TEST_URL")
    )
```

### Creating and Uploading DataSets

Currently, there are two ways to upload data to the Nexosis API.
1. A CSV File
2. Through the Model Object `DataSetData`

CVS doesn't yet support the MetaData object so one column must be named timestamp to run a forecasting or impact session. All dates must be in an ISO 8601 format. If no time-zone is specified, it is assumed to be UTC.

``` csv
timeStamp,sales,transactions
2012-12-31 00:00:00,2922.13,459
2013-01-01 00:00:00,1500.56,195
2013-01-02 00:00:00,4078.52,696
2013-01-03 00:00:00,4545.69,743
2013-01-04 00:00:00,4872.63,797
2013-01-05 00:00:00,2420.81,367
2013-01-06 00:00:00,1664.22,241
2013-01-07 00:00:00,3693.01,647
2013-01-08 00:00:00,3874.89,653
2013-01-09 00:00:00,4134.05,700
2013-01-10 00:00:00,4314.94,723
...etc...

```
Given the csv file above, the FileInputStream can be passed to the <code>create()</code> method to upload the data.

``` scala
val csvFile = new FileInputStream("./SampleData.csv")
client.getDataSets.create("SampleDataSet", csvFile)
```

Here's an example of creating a DataSet using the <code>DataSetData</code> Model object, using generated random data:
```{:.line-numbers}{:.language-scala}
// Generate a dataset
val rand = new Random
val startDate = DateTime.parse("2016-08-01T00:00:00Z")
val endDate = DateTime.parse("2017-03-26T00:00:00Z")

val rows: util.List[util.Map[String, String]] =
  new util.ArrayList[util.Map[String, String]]()

val daysCount = Days.daysBetween(startDate, endDate).getDays()

(0 until daysCount).map(startDate.plusDays(_)).foreach { timeStamp =>
  val row: util.Map[String, String] =
    new util.HashMap[String, String]()

  row.put("timestamp", timeStamp.toDateTimeISO().toString())
  row.put("sales", (rand.nextDouble() * 100).toString)
  row.put("promotion", false.toString)

  rows.add(row)
}

// Create and add all the rows to the DataSet.
val dataSet = new DataSetData
dataSet.setData(rows)

// Setup metadata that describes the dataset columns.
// For time-series, we want to indicate a TIMESTAMP column
// Indicate the sales column is the TARGET to forcast over
// The promotion column is a boolean indicating if the date is a promotional period
val cols = new Columns
cols.setColumnMetadata("timestamp", DataType.DATE, DataRole.TIMESTAMP)
cols.setColumnMetadata("sales", DataType.NUMERIC, DataRole.TARGET)
cols.setColumnMetadata("promotion", DataType.LOGICAL, DataRole.FEATURE)
// Add the Columns Metadata to the DataSet
dataSet.setColumns(cols)

// Send the dataset to the API endpoint
client.getDataSets.create("SampleDataSet", dataSet)
```

### Retrieve a list of DataSets

To retrieve all DataSets, just call `list()`
``` scala
val dataSetList = client.getDataSets.list()

dataSetList.getItems.forEach { item =>
  println(item.getDataSetName)
```

To retrieve all DataSets that contains `sales` in the dataset name, call `list(filter)`

``` scala
val dataSetList = client.getDataSets.list("sales")

dataSetList.getItems.forEach { item =>
  println(item.getDataSetName)
```

### Create a Forecast Session

To generate forecasts, the platform needs to know the DataSet name to build the forecasting models off of, a target column indicating what columns the model should be predicting, a prediction interval, and the start and end date of the desired prediction dates.

``` scala
 val session = client.getSessions.createForecast(
  "OtherDataSet",
  "sales",
  DateTime.parse("2017-03-25T0:00:00Z"),
  DateTime.parse("2017-04-24T0:00:00Z"),
  ResultInterval.DAY
)

// Save the session ID's to check status and retireve results when ready
val sessionId = session.getSessionId()
```

### Create an Impact Session
To analyze the impact of a past event, the platform needs to know the DataSet name to build the forecasting models off of, a target column indicating what columns the model will analyze, a prediction interval, and the start and end date of the event.

``` scala
val session = nexosisClient.getSessions().analyzeImpact(
  "websiteTraffic",
  "new-big-announcement",
  "hits",
  DateTime.parse("2016-11-26T00:00:00Z"),
  DateTime.parse("2016-12-25T00:00:00Z"),
  ResultInterval.DAY
)

// Save the session ID's to check status and retireve results when ready
val sessionId = response.getSessionId();
```

### Check On Session Status

``` scala
// After starting a Session...
val status = client.getSessions.getStatus(sessionId)
var results: SessionResult

// Loop until the STARTED status changes
while (status.getStatus == SessionStatus.STARTED ) {
  results = client.getSessions.getResults(sessionId)
  Thread.sleep(5000)
}
// Retrieve the Results.
```

### Issues
If you run into issues using this client library, create a [new issue](https://github.com/Nexosis/nexosisclient-java/issues/new){:target="_blank"} in github. Please include code to reproduce the error if possible.
