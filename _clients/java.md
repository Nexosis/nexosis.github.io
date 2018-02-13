---
title: Java package
description: Use the Nexosis Java Client to communicate with Nexosis API
category: Java
breadcrumb: API Clients
tags: [Java, JAR, Package]
use_codestyles: true
---
> <h5 class="mt0">Supported platforms</h5>
The Nexosis Client for Java supports Java versions 1.7 and 1.8.

## Installing the client

> <p><a href="https://mvnrepository.com/artifact/com.nexosis/nexosisclient-java" class="btn secondary mr10" target="_blank"><i class="fa fa-cube mr5"></i> Package Details</a><a href="https://github.com/Nexosis/nexosisclient-java" class="btn secondary" target="_blank"><i class="fa fa-github mr5"></i> View Source</a></p>

### Maven
Add this dependency to <code>pom.xml</code>:
``` xml
<dependency>
    <groupId>com.nexosis</groupId>
    <artifactId>nexosisclient-java</artifactId>
    <version>1.0.1</version>
</dependency>
```
### Gradle
Add this to the <code>build.gradle</code> file:
``` JSON
dependencies {
    compile 'com.nexosis:nexosisclient-java:1.0.1'
}
```
## Java Quick Start

### Initialize the Client

The <code>NexosisClient</code> has several constructor overloads.

To use the default constructor, create an environment variable called <code>NEXOSIS_API_KEY</code> and set it equal to your API Key. This also uses the default API Endpoint base URI <code>https://ml.nexosis.com/v1</code>.

``` java
NexosisClient client = new NexosisClient(); 
```

Initialize the client with an API key.

``` java 
NexosisClient client = new NexosisClient("api_key_here");
```

Initialize the client with an API key and endpoint.

``` java 
NexosisClient client = new NexosisClient("api_key_here", "https://ml.nexosis.com/v1/");
```

### Creating and Uploading DataSets

Currently, there are two ways to upload data to Nexosis.
1. CSV File
2. Through a Java POJO / Model Object

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
``` java
String dataSetName = "SampleDataSet";
File initialFile = new File("./SampleData.csv");
InputStream inputStream = new FileInputStream(initialFile);

nexosisClient.getDataSets().create(dataSetName, inputStream);
```

Here's an example of creating a DataSet using the <code>DataSetData</code> Model object, using generated random data:
```{:.line-numbers}{:.language-java}
// Generate a dataset
Random rand = new Random();
DateTime startDate = DateTime.parse("2016-08-01T00:00:00Z");
DateTime endDate = DateTime.parse("2017-03-26T00:00:00Z");

List<Map<String, String>> rows = new ArrayList<>();
// Generate a series of containing 
for (DateTime timeStamp = startDate; timeStamp.isBefore(endDate); timeStamp = timeStamp.plusDays(1)) {
    Map<String, String> row = new HashMap<>();
    row.put("timestamp", timeStamp.toDateTimeISO().toString());
    row.put("sales", Double.toString(rand.nextDouble() * 100));
    row.put("promotion", Boolean.toString(false);
    rows.add(row);
}
// Create and add all the rows to the DataSet.
DataSetData dataSet = new DataSetData();
data.setData(rows);

// Setup metadata that describes the dataset columns.
// For time-series, we want to indicate a TIMESTAMP column
// Indicate the sales column is the TARGET to forcast over
// The promotion column is a boolean indicating if the date is a promotional period
Columns cols = new Columns();
cols.setColumnMetadata("timestamp", DataType.DATE, DataRole.TIMESTAMP);
cols.setColumnMetadata("sales", DataType.NUMERIC, DataRole.TARGET); 
cols.setColumnMetadata("promotion", DataType.LOGICAL, DataRole.FEATURE);
// Add the Columns Metadata to the DataSet
dataSet.setColumns(cols);

String dataSetName = "SampleDataSet";
// Send the dataset to the API endpoint
nexosisClient.getDataSets().create(dataSetName, dataSet);
```

### Retrieve a list of DataSets

``` java
 DataSetList list = nexosisClient.getDataSets().list();

 ```

### Create a Forecast Session

To generate forecasts, the platform needs to know the DataSet name to build the forecasting models off of, a target column indicating what columns the model should be predicting, a prediction interval, and the start and end date of the desired prediction dates.

``` java
 nexosisClient.getSessions().createForecast(
                "OtherDataSet",
                "sales",
                DateTime.parse("2017-03-25T0:00:00Z"),
                DateTime.parse("2017-04-24T0:00:00Z"),
                ResultInterval.DAY
        );
```

### Create an Impact Session

To analyze the impact of a past event, the platform needs to know the DataSet name to build the forecasting models off of, a target column indicating what columns the model will analyze, a prediction interval, and the start and end date of the event.

``` java
SessionResponse response = nexosisClient.getSessions().analyzeImpact(
        "websiteTraffic",
        "new-big-announcement",
        "hits",
        DateTime.parse("2016-11-26T00:00:00Z"),
        DateTime.parse("2016-12-25T00:00:00Z"),
        ResultInterval.DAY
);

// Save the SessionID
UUID sessionId = response.getSessionId();
// Use SessionID to check on the status and retrieve results when they are ready
```

### Check On Session Status

``` java
// After starting a Session...

while (results.getStatus() != SessionStatus.COMPLETED) {
        Thread.sleep(4000);
        SessionResult results = nexosisClient.getSessions().getResults(savedSessionId);
}

// Retrieve the Results.
```

### Issues
If you run into issues using this client library, create a [new issue](https://github.com/Nexosis/nexosisclient-java/issues/new){:target="_blank"} in github. Please include code to reproduce the error if possible.
