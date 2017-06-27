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

The most basic thing you can do with the API is submit some data and ask for predictions all at once. This can be done if you have a CSV file with the following code:

``` csharp
var client = new NexosisClient("YOUR API KEY HERE");
using (var file = File.OpenText("C:\\path\\to\\file.csv"))
{
    var session = await client.Sessions.CreateForecast(file, "sales", DateTimeOffset.Parse("2017-03-25 -0:00"), DateTimeOffset.Parse("2017-04-25 -0:00"));
    Console.WriteLine($"{session.Id}");
}
```

For this to work, the CSV file must have a header with the names of the columns in the file. One of those must be named "timeStamp", and in this example, there is a second column named "sales".

Once the forecasting is complete, you will receive an email notification. Using the `sessionId` from above, you will want to get results with the following call:

```csharp
    var results = await client.Sessions.GetResults(sessionId);
    // results has a .Data property with the forecast values
```

### Issues
If you run into issues using this client library, create a [new issue](https://github.com/Nexosis/nexosisclient-net/issues/new) in github. Please include code to reproduce the error if possible.

Pull requests are welcome.
