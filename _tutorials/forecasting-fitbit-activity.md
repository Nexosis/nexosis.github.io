---
title: Forecasting Fitbit Activity
description: This tutorial demonstrates building a web app that forecasts your Fitbit activity using the Nexosis Api
copyright: 2017 Nexosis 
layout: default
category: Forecasting
tags: [Sports & Games, C#, Fitbit]
use_codestyles: true
order: 1
---

Chances are, you or someone you know has some sort of activity tracker, such as a Fitbit or a Garmin. These devices keep track of your activity throughout the day (steps taken, stairs climbed, calories burned, etc).  They also often have a social aspect to them, in that you can compare your activity levels to friends, host friendly challenges, etc. 

As a fun exercise, we built a small web application that uses the Nexosis API and [Fitbit's API](https://dev.fitbit.com/) to forecast activity levels for a user.

------

>You can follow along using the [full source code in github](https://github.com/Nexosis/app-fitbit){:target="_blank"}.


## Getting Started

#### To get ourselves off the ground a bit faster with the APIs we'll be using, we started off by using [Auth0](https://auth0.com/) and their [Asp.Net core quickstart](https://auth0.com/docs/quickstart/webapp/aspnet-core) to handle things like identity management and OAuth with Fitbit, as well as the [Nexosis .Net Client Library](/clients/dotnet){:target="_blank"} to interact with the Nexosis API.  

Lastly, we found a popular [.Net Fitbit Api Client](https://github.com/aarondcoleman/Fitbit.NET) and started using it.  Unfortunately it didn't support .Net Core [yet](https://github.com/aarondcoleman/Fitbit.NET/issues/175), so we [forked](https://github.com/vermeeca/Fitbit.NET) it and made the [bare minimum](https://github.com/vermeeca/Fitbit.NET/commit/2d52c6e3149c08efe4193650cd0eede766515d8f) changes we could make to get it compiling under .Net Standard.

### Fitbit Time Series API

Conveniently for us, Fitbit exposes a [Time Series API](https://dev.fitbit.com/docs/activity/#activity-time-series) that we can use to quickly retrieve many years of history for a given activity.

With an instance of the Fitbit Client all setup, you can retrieve a history of your step count by day with the following call:

``` C#
TimeSeriesDataList timeSeries = await client.GetTimeSeriesAsync(
                                  TimeSeriesResourceType.Steps, 
                                  DateTime.Today, 
                                  DateRangePeriod.SixMonths, "-");
```

So if we want a simple controller action that retrieves six months of data for a given activity and throws it up on a graph like so:

![Steps](/assets/img/tutorials/fitbit-steps-graph.png){:class="img-responsive"}

We can write a [controller](https://github.com/Nexosis/app-fitbit/blob/master/Controllers/ActivityController.cs) action that looks pretty much like this:

``` C#
public async Task<IActionResult> Index(string id)
{
    var client = await fitbit.Connect(User);
    var resourceType = TimeSeriesResourceType.Steps;

    //id, here, is the name of an activity in fitbit, e.g. steps
    if (!Enum.TryParse(id, true, out resourceType))
    {
        return RedirectToAction("Index", new {id = "steps"});                    
    }
      
    var timeSeries = await client.GetTimeSeriesAsync(resourceType, DateTime.Today,
            DateRangePeriod.SixMonths, "-");
    
    return View(new ActivityViewModel(timeSeries.ToPoints(), id));
}
```


## Generating a Forecast

#### Now that we're able to fetch your fitbit activity history, let's write a [controller](https://github.com/Nexosis/app-fitbit/blob/master/Controllers/ActivityController.cs) action to generate a set of predictions for whichever fitbit activity we're looking at.

``` C#
[Authorize]
[HttpPost]
public async Task <IActionResult> Predict(string id)
{
    var client = await fitbit.Connect(User);

    //fetch all of the activities that we care about 
    var stepsSeries = await client.GetTimeSeriesAsync(TimeSeriesResourceType.Steps, DateTime.Today, DateRangePeriod.Max, "-");
    var distanceSeries = await client.GetTimeSeriesAsync(TimeSeriesResourceType.Distance, DateTime.Today, DateRangePeriod.Max, "-");
    var floorsSeries = await client.GetTimeSeriesAsync(TimeSeriesResourceType.Floors, DateTime.Today, DateRangePeriod.Max, "-");
    var caloriesInSeries = await client.GetTimeSeriesAsync(TimeSeriesResourceType.CaloriesIn, DateTime.Today, DateRangePeriod.Max, "-");
    var caloriesOutSeries = await client.GetTimeSeriesAsync(TimeSeriesResourceType.CaloriesOut, DateTime.Today, DateRangePeriod.Max, "-");
    var sleepSeries = await client.GetTimeSeriesAsync(TimeSeriesResourceType.MinutesAsleep, DateTime.Today, DateRangePeriod.Max, "-");
    var fairlyActiveSeries = await client.GetTimeSeriesAsync(TimeSeriesResourceType.MinutesFairlyActive, DateTime.Today, DateRangePeriod.Max, "-");
    var lightlyActiveSeries = await client.GetTimeSeriesAsync(TimeSeriesResourceType.MinutesLightlyActive, DateTime.Today, DateRangePeriod.Max, "-");
    var veryActiveSeries = await client.GetTimeSeriesAsync(TimeSeriesResourceType.MinutesVeryActive, DateTime.Today, DateRangePeriod.Max, "-");
    var waterSeries = await client.GetTimeSeriesAsync(TimeSeriesResourceType.Water, DateTime.Today, DateRangePeriod.Max, "-");
    var weightSeries = await client.GetTimeSeriesAsync(TimeSeriesResourceType.Weight, DateTime.Today, DateRangePeriod.Max, "-");

    //join them all into a single dictionary by date
    var dataSetData = from steps in stepsSeries.DataList
        join distance in distanceSeries.DataList on steps.DateTime equals distance.DateTime
        join floors in floorsSeries.DataList on steps.DateTime equals floors.DateTime
        join caloriesIn in caloriesInSeries.DataList on steps.DateTime equals caloriesIn.DateTime
        join caloriesOut in caloriesOutSeries.DataList on steps.DateTime equals caloriesOut.DateTime
        join minutesAsleep in sleepSeries.DataList on steps.DateTime equals minutesAsleep.DateTime
        join minutesFairlyActive in fairlyActiveSeries.DataList on steps.DateTime equals minutesFairlyActive.DateTime
        join minutesLightlyActive in lightlyActiveSeries.DataList on steps.DateTime equals minutesLightlyActive.DateTime
        join minutesVeryActive in veryActiveSeries.DataList on steps.DateTime equals minutesVeryActive.DateTime
        join water in waterSeries.DataList on steps.DateTime equals water.DateTime
        join weight in weightSeries.DataList on steps.DateTime equals weight.DateTime
        select new Dictionary<string, string>
        {
            ["timeStamp"] = steps.DateTime.ToString("o"),
            [nameof(steps)] = steps.Value,
            [nameof(floors)] = floors.Value,
            [nameof(caloriesIn)] = caloriesIn.Value,
            [nameof(caloriesOut)] = caloriesOut.Value,
            [nameof(minutesAsleep)] = minutesAsleep.Value,
            [nameof(minutesFairlyActive)] = minutesFairlyActive.Value,
            [nameof(minutesLightlyActive)] = minutesLightlyActive.Value,
            [nameof(minutesVeryActive)] = minutesVeryActive.Value,
            [nameof(water)] = water.Value,
            [nameof(weight)] = weight.Value,
        };

    var nexosisClient = nexosis.Connect();
    var fitbitUser = await fitbit.GetFitbitUser(User);

    //send that dictionary to Nexosis as a single DataSet
    var request = new DataSetDetail() {Data = dataSetData.ToList()};
    var dataSetName = $"fitbit.{fitbitUser.UserId}"; 
    await nexosisClient.DataSets.Create(DataSet.From(dataSetName, request));

    //make sure that we've identified which column in the DataSet is our target (the one we want to predict)
    var sessionRequest = Sessions.Forecast(
        dataSetName,
        new DateTimeOffset(DateTime.Today.AddDays(1)),
        new DateTimeOffset(DateTime.Today.AddDays(31)),
        ResultInterval.Day,
        options: new ForecastSessionRequest()
        {
            Columns = new Dictionary<string, ColumnMetadata>()
            {
                [id] = new ColumnMetadata() {Role = ColumnRole.Target, DataType = ColumnType.Numeric}
            }
        });

    await nexosisClient.Sessions.CreateForecast(sessionRequest);
            
    return RedirectToAction("Index", new{id=id});
}
```

This code will upload a DataSet to the Nexosis API that looks something like this:

``` json
{
    "Data": [
        {
            "timeStamp": "2014-08-06T00:00:00.0000000",
            "steps": "11869",
            "floors": "0",
            "caloriesIn": "0",
            "caloriesOut": "3028",
            "minutesAsleep": "0",
            "minutesFairlyActive": "73",
            "minutesLightlyActive": "143",
            "minutesVeryActive": "45",
            "water": "0.0",
            "weight": "87.997"
        },
        {
            "timeStamp": "2014-08-07T00:00:00.0000000",
            "steps": "10945",
            "floors": "0",
            "caloriesIn": "0",
            "caloriesOut": "2860",
            "minutesAsleep": "0",
            "minutesFairlyActive": "110",
            "minutesLightlyActive": "121",
            "minutesVeryActive": "34",
            "water": "0.0",
            "weight": "87.997"
        }
    ]
    **SNIP**
}
        
```

## Getting Forecast Results

#### Now that we've submitted a [Forecast](/guides/forecasting-walkthrough) session, we can go back to our first controller action and update it to fetch the most recent forecast session and return it to the view for graphing.

``` C#
[Authorize]
public async Task<IActionResult> Index(string id)
{

    if (id == null)
    {
        return RedirectToAction("Index", new {id = "steps"});
    }
    
    TimeSeriesDataList timeSeries = null;

    if (!cache.TryGetValue($"{User.Identity.Name}.{id}", out timeSeries))
    {
        var client = await fitbit.Connect(User);

        var resourceType = TimeSeriesResourceType.Steps;
        if (!Enum.TryParse(id, true, out resourceType))
        {
            return RedirectToAction("Index", new {id = "steps"});                    
        }

        timeSeries = await client.GetTimeSeriesAsync(resourceType, DateTime.Today,
            DateRangePeriod.SixMonths, "-");

        cache.Set($"{User.Identity.Name}.{id}", timeSeries);
    }

    var fitbitUser = await fitbit.GetFitbitUser(User);

    var nexosisClient = nexosis.Connect();

    var sessionsForThisActivity =
                (await nexosisClient.Sessions.List(Sessions.Where($"fitbit.{fitbitUser.UserId}")))
                .Items
                .OrderByDescending(o => o.RequestedDate).Where(s => s.TargetColumn == id)
                .ToList();
    
    //look for the most recent completed session targeting the current activity
    var lastSession = sessionsForThisActivity.FirstOrDefault(s => s.Status == Status.Completed)
                      ?? sessionsForThisActivity.FirstOrDefault();

    SessionResult result = null;

    if (lastSession?.Status == Status.Completed)
    {
        //if we have a session, fetch that session's results
        result = await nexosisClient.Sessions.GetResults(lastSession.SessionId);
    }

    var actualPoints = timeSeries.ToPoints().ToList();
    var predictedPoints = result.ToPoints().ToList();

    //make sure the two series have the same number of points, just to satisfy nvd3
    predictedPoints = predictedPoints.AlignWith(actualPoints).ToList();
    actualPoints = actualPoints.AlignWith(predictedPoints).ToList();
    
    return View(new ActivityViewModel(actualPoints, lastSession, predictedPoints, id));
}
```


For reference, the JSON coming back from the Nexosis API will look something like this.
``` json
{
    "Data": [
        {
            "timeStamp": "2017-08-06T00:00:00.0000000Z",
            "steps": "11368.5953994845"
        },
        {
            "timeStamp": "2017-08-07T00:00:00.0000000Z",
            "steps": "7340.19958686538"
        },
        {
            "timeStamp": "2017-08-08T00:00:00.0000000Z",
            "steps": "6881.10072583808"
        },
        *SNIP*
    ],
    "Metrics": {},
    "SessionId": "015dae4d-d393-41de-abfb-fabb6d4d5d13",
    "Type": "forecast",
    "Status": "completed",
    "StatusHistory": [
        {
            "Date": "2017-08-04T17:32:02.323711+00:00",
            "Status": "requested"
        },
        {
            "Date": "2017-08-04T17:32:02.3337913+00:00",
            "Status": "started"
        },
        {
            "Date": "2017-08-04T18:00:42.9391863+00:00",
            "Status": "completed"
        }
    ],
    "ExtraParameters": {},
    "DataSetName": "fitbit.22HCSQ",
    "TargetColumn": "steps",
    "EventName": null,
    "RequestedDate": "2017-08-04T17:32:02.323711+00:00",
    "StartDate": "2017-08-05T00:00:00-04:00",
    "EndDate": "2017-09-04T00:00:00-04:00",
    "Columns": {
        "sleep": {
            "DataType": 1,
            "Role": null
        },
        "steps": {
            "DataType": 1,
            "Role": 2
        },
        "water": {
            "DataType": 1,
            "Role": null
        },
        "floors": {
            "DataType": 1,
            "Role": null
        },
        "weight": {
            "DataType": 1,
            "Role": null
        },
        "timeStamp": {
            "DataType": 3,
            "Role": 1
        },
        "caloriesIn": {
            "DataType": 1,
            "Role": null
        },
        "veryActive": {
            "DataType": 1,
            "Role": null
        },
        "caloriesOut": {
            "DataType": 1,
            "Role": null
        },
        "fairlyActive": {
            "DataType": 1,
            "Role": null
        },
        "lightlyActive": {
            "DataType": 1,
            "Role": null
        },
        "minutesAsleep": {
            "DataType": 1,
            "Role": null
        },
        "minutesVeryActive": {
            "DataType": 1,
            "Role": null
        },
        "minutesFairlyActive": {
            "DataType": 1,
            "Role": null
        },
        "minutesLightlyActive": {
            "DataType": 1,
            "Role": null
        }
    },
    "Links": [
        {
            "Href": "https://ml.nexosis.com.com/v1/sessions/015dae4d-d393-41de-abfb-fabb6d4d5d13/results",
            "Rel": "results"
        },
        {
            "Href": "https://ml.nexosis.com.com/v1/data/fitbit.22HCSQ",
            "Rel": "data"
        }
    ]
}
```

And once that's done, we can show our predicted activity on the same graph;

![Steps (and forecasts)](/assets/img/tutorials/fitbit-steps-forecast-graph.png){:class="img-responsive"}


### Reviewing Results
As you can see, the Fitbit account we've been using for this demonstration has some pretty radical fluctuations in their step count.  However, they definitely have some weekly seasonality that the Nexosis API picks up on.

For example, look at the month of April 2017.  This person most likely is in the habit of going for a run or a hike on the weekend.
![So much for lazy sundays](/assets/img/tutorials/fitbit-steps-april.png){:class="img-responsive"}

However, looking at recent weeks you can see that the weekend spikes remain, but are much more pronounced.
![Long Runs](/assets/img/tutorials/fitbit-steps-long-runs.png){:class="img-responsive"}

This raises more questions.  What happened in recent months that caused the big spikes in step count?  Is there some other data that we could feed into the Nexosis API that will give our algorithms more clues to work from when predicting future steps?  For example, a feature set with a training schedule for an upcoming marathon might yield interesting forecasts.  

Also it might be interesting to use the Nexosis [Impact Analysis](/guides/impact-analysis) API to see what kind of impact vacations have on step counts, such as this week in early June.

![Vacation Steps](/assets/img/tutorials/fitbit-steps-vacation.png){:class="img-responsive"}

## Conclusion
#### As you can see, the Nexosis API can make it easy to take data from other APIs and get a forecast on that data.  Sometimes that forecast leads to other questions, and can lead you toward interesting insights.

> If you have an idea for expanding on this sample application, [let us know](https://github.com/Nexosis/app-fitbit/issues/new)!  If you're so inclined, take a shot at implementing that idea and submit it back to us as a Pull Request.  We'd love to expand on this app and make it more useful to folks.
