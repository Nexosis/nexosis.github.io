---
title: .NET Library
description: Getting started guide for using the Nexosis API
copyright: 2017 Nexosis 
layout: default
category: .NET
tags: [C#, .NET, Nuget]
use_codestyles: true
---

## Installation

You need to be using a project referencing .NET Standard (any version of .NET Core or .NET Framework 4.6.2 or above).

``` 
PM> Install-Package Nexosis.Api.Client 
```

[View Nuget Package details](https://www.nuget.org/packages/Nexosis.Api.Client/)

[View Source in our git repo](https://github.com/Nexosis/nexosisclient-net)

### Usage

The most basic thing you can do with the API is submit some data and then ask for predictions. This can be done if you have a CSV file with the following code:

``` csharp
var client = new NexosisClient();
using (var file = File.OpenText("C:\\path\\to\\sales-file.csv"))
{
    client.DataSets.Create("widget-sales", file);
}
    
// Creating a forecast session

var sessionResponse = client.Sessions.CreateForecast(
    "widget-sales",
    "daily_transaction",
    DateTimeOffset.Parse("2017-12-12 10:11:12 -0:00"), 
    DateTimeOffset.Parse("2017-12-22 22:23:24 -0:00"), 
    ResultInterval.Day
);
```

For this to work, the CSV file must have a header with the names of the columns in the file.

Once the forecasting is complete, you will receive an email notification. Using the `sessionId` from
above, you will want to get results with the following call:

```csharp
// Retrieve forecast results
using (var output = new StreamWriter(File.OpenWrite("results-file.csv")))
{
    client.Sessions.GetResults(sessionResponse.SavedSessionId, output);
}
```

### Issues
If you run into issues using this client library, create a [new issue](https://github.com/Nexosis/nexosisclient-net/issues/new) in github. Please include code to reproduce the error if possible.

Pull requests are welcome.
