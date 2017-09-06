---
title: Calendar Data Sources
description: Learn about using calendars as data sources
copyright: 2017 Nexosis 
layout: default
category: Forecasting
tags: [DataSet, Quick Links, Views, Datasources]
use_codestyles: true
---
Many forecasts can be improved by including special dates as features in your time series data. Whether you are including common holidays in a sales foreacast, or sporting events for a traffic forecast, using an existing calendar can simplify your process while improving results.  

-----
## Calendar Data Sources
You can specify the url of any iCal formatted calendar source as a join target in a [View](/guides/views). If you have a Google calendar simply open the calendar settings and copy the [iCal link](https://support.google.com/calendar/answer/37083#link). Make sure that the calendar is public before attempting to use it as a data source.  Aside from Google calendars, any iCal file which can be accessed by url is a valid target.

> Note the calendar data will be imported as needed. If you specify a calendar url and then remove the calendar, the view will fail to build. 

In addition to iCal, the Nexosis API will import any documented named global data set. [See *Data Sources* for more information](/guides/datasources).  

### Specifying a Calendar by URL
In order to target an iCal calendar by url you will modify the join target to use the calendar object as in the example below:

``` json
{
  "dataSetName": "MyDataset",
  "joins": [
    {
      "calendar": {
        "url": "https://calendar.google.com/calendar/ical/en.usa%23holiday%40group.v.calendar.google.com/public/basic.ics"
      }
    }
  ]
}
```
In this very simple example we have sourced an existing DataSet with the name *MyDataset* as the primary data source and then inside the join we used the keyword *calendar* to specify a calendar as the join source. In this case we have an iCal url, so we use the *url* property to provide the location of the iCal. The Nexosis API will take care of importing all of the events from the target calendar for the date range within the source data set's column with the role of 'timestamp'.

> In order to use a well-known global data source as a calendar simply replace *url* with *name*. 

### How a Calendar Is Joined
When the Nexosis API imports a calendar it will create the following additional columns in your view:

- eventTimestamp: the date and time of the event start taken from the calendar's list of events
- eventName: the title of the event given in the calendar. Matching event names will be treated as a single column when determining date overlap (see below for more information).
- calendarName: the name of the calendar joined to - useful if more than one calendar has been joined in the same View.

The time stamp of each event is then overlapped with the timestamp column in the primary dataset dependent on the granularity of the join specified in the columnOptions object. The default granularity for all forecasts is 'day' and this is retained in calendar joins. If you specify nothing then each event which occurs on a day will be treated as having happened with the day of the source data set's daily observations.  

For example consider the following imported calendar data:

<table class="table table-striped mb20">
<th>
eventTimestamp
</th>
<th>
eventName
</th>
<th>
calendarName
</th>
<tr class="bg-faded">
<td>
2014-02-14T00:00:00.0000000Z
</td>
<td>
Valentines Day Sale
</td>
<td>
Promotions
</td>
</tr>
<tr>
<td>
2014-04-20T14:00:00.0000000Z
</td>
<td>
Start of Spring Break
</td>
<td>
Promotions
</td>
</tr>
<tr class="bg-faded">
<td>
2014-05-26T00:00:00.0000000Z
</td>
<td>
Memorial Day Sale
</td>
<td>
Promotions
</td>
</tr>
</table>

Consider that while the Spring Break sale doesn't kick off until after 2PM - if we are mapping it to a daily sales DataSet then we're simply going to join to the entire day of the 20th. In the following example the events have been "one hot encoded" such that each event is a feature column which is either "happening" denoted by a *1* value or "not happening" denoted by a *0* value.

<table class="table table-striped mb20">
<th>
timestamp
</th>
<th>
sales
</th>
<th>
Promotions.ValentinesDay
</th>
<th>
Promotions.StartofSpringBreak
</th>
<th>
Promotions.MemorialDaySale
</th>
<tr class="bg-faded">
<td>
2014-02-14T00:00:00.0000000Z
</td>
<td>
6585.25
</td>
<td>
1
</td>
<td>
0
</td>
<td>
0
</td>
<tr>
<td colspan="5">
several other days elided...
</td>
</tr>
</tr>
<tr>
<td>
2014-04-20T00:00:00.0000000Z
</td>
<td>
8690.44
</td>
<td>
0
</td>
<td>
1
</td>
<td>
0
</td>
</tr>
<tr>
<td colspan="5">
several other days elided...
</td>
</tr>
<tr class="bg-faded">
<td>
2014-05-26T00:00:00.0000000Z
</td>
<td>
2358.75
</td>
<td>
0
</td>
<td>
0
</td>
<td>
1
</td>
</tr>
</table>

If instead you had hourly sales data, then the valentine's related sale could be properly mapped to the 2PM hour in that dataset...

<table class="table table-striped mb20">
<th>
timestamp
</th>
<th>
sales
</th>
<th>
Promotions.ValentinesDay
</th>
<th>
Promotions.StartofSpringBreak
</th>
<th>
Promotions.MemorialDaySale
</th>
<tr class="bg-faded">
<td>
2014-02-14T13:00:00.0000000Z
</td>
<td>
685.75
</td>
<td>
0
</td>
<td>
0
</td>
<td>
0
</td>
</tr>
<tr>
<td>
2014-02-14T14:00:00.0000000Z
</td>
<td>
890.94
</td>
<td>
1
</td>
<td>
0
</td>
<td>
0
</td>
</tr>
<tr class="bg-faded">
<td>
2014-02-14T15:00:00.0000000Z
</td>
<td>
258.85
</td>
<td>
1
</td>
<td>
0
</td>
<td>
0
</td>
</tr>
</table>
In order to map a join at the hourly level, include a columnOption for the calendar against the eventTimestamp column:

``` json
{
  "dataSetName": "MyDataset",
  "joins": [
    {
      "calendar": {
        "url": "https://calendar.google.com/calendar/ical/en.usa%23holiday%40group.v.calendar.google.com/public/basic.ics"
      },
      "columnOptions": {
        "eventTimestamp" : {
          "joinInterval" : "hour"
        }
      }
    }
  ]
}
```

### Timezones
Finally, if you have a specific time zone for the calendar then you can specify this within the calendar object definition. Use the [TZ string value](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones) for the timezone value:

``` json
{
  "dataSetName": "MyDataset",
  "joins": [
    {
      "calendar": {
        "url": "https://calendar.google.com/calendar/ical/en.usa%23holiday%40group.v.calendar.google.com/public/basic.ics"
        "timezone": "America/Barbados"
      }
    }
  ]
}
``` 
When a timezone is provided then the event start time in the source calendar will first be mapped to that timezone before being overlayed with the target dataset timestamps. If the target dataset timestamp is in a different timezone then you may expect certain events to overlap more than one day when at a daily level of granularity.