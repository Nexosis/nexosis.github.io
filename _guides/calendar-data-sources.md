---
title: Calendar Data Sources
description: Learn about using calendars as data sources
copyright: 2017 Nexosis 
layout: default
category: Concepts
tags: [DataSet, Views, Datasources]
use_codestyles: true
---
Many forecasts can be improved by including special dates as features in your time series data. Whether you are including common holidays in a sales foreacast, or sporting events for a traffic forecast, using an existing calendar can simplify your process while improving results.  

-----
## Calendar Data Sources
You can specify the URL of any iCal formatted calendar source as a join target in a [View](/guides/views). If you have a Google calendar simply open the calendar settings and copy the [iCal link](https://support.google.com/calendar/answer/37083#link). Make sure that the calendar is public before attempting to use it as a data source.  Aside from Google calendars, any iCal file which can be accessed by URL is a valid target.

> Note the calendar data will be imported as needed. If you specify a calendar URL and then remove the calendar, the view will fail to build. 

In addition to iCal, the Nexosis API will import any documented named global dataset. [See *Data Sources* for more information](/guides/data-sources).  

### Specifying a Calendar by URL
In order to target an iCal calendar by URL you will modify the join target to use the calendar object as in the example below:

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
In this very simple example we have sourced an existing dataset with the name *MyDataset* as the primary data source and then inside the join we used the keyword *calendar* to specify a calendar as the join source. In this case we have an iCal URL, so we use the *url* property to provide the location of the iCal. The Nexosis API will take care of importing all of the events from the target calendar for the date range within the source dataset's column with the role of 'timestamp'.

> In order to use a well-known global data source as a calendar simply replace *url* with *name*. 

### Calendar Columns
When the Nexosis API imports a calendar it will create additional columns in your view. The number of columns created is dynamic, based on the number of distinct events present in the joined calendar.

The "key" of a joined calendar is always a column named `eventTimestamp`. When joining a calendar, the event timestamp doesn't provide any additional features to the dataset, but it is important when specifying metadata about how the calendar ought to be joined to the parent dataset. See below for an example of specifying a different join interval using the calendar's event timestamp.

Every joined calendar adds a logical column to the view, indicating whether or not the calendar has an event corresponding to the joined date. The name of this column is dynamic, taking the form `calendar:*calendarName*`, where *calendarName* is the name of the calendar from its iCal definition. This column is useful in cases where the names of events aren't important, but you want an indicator variable included in the view for whether or not any event occurred on a given day. For example, say you have a calendar of games for a sports team, where each event is the name of the opponent. You probably don't care about including the specific opponent as a feature in a forecast session, but you may want to include a feature indicating "was there a game today, or not?" In this example, the calendar column will model that feature nicely.

Additionally, every joined calendar adds a logical column to the view for each unique event in the calendar. These column names are also dynamic, taking the form `event:*calendarName*:*eventName*`, where *calendarName* is the name of the calendar from its iCal definition, and *eventName* is the name of the event. (If two events from the same calendar have the same name, then they will be considered to be the same event.) Event columns are most useful for recurring events, such as holidays, weekends, or other regular occurrences. When an event occurs multiple times on a calendar, a machine learning model will be able to ascribe some importance to that event, making predictions more accurate.

### How a Calendar Is Joined
When joining to a calendar, the column being joined from the view's main dataset must be a date. For each row in the parent dataset, any calendar event that overlaps the date will be joined into the row.

Sometimes though, calendar events don't quite line up with dates in the parent dataset, especially if dates in the parent dataset are more coarse-grained than calendar entries. For example, consider the following calendar data:

<table class="table table-striped mb20">
<th>
Event Date
</th>
<th>
Event Name
</th>
<th>
Calendar Name
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

Note that the "Spring Break" sale doesn't kick off until after 2PM. If we were to join this calendar into a dataset of daily sales (where each date has no time component), none of the rows in the daily sales dataset would match the "Spring Break" sale in the calender, because none of the dates exactly match.

To alleviate this issue, you can specify a `joinInterval` property in the `columnOptions` of the join, indicating that you want each event to be "expanded" to fill an entire day, week, month, etc. Specifying a `joinInterval` of `day` would expand the "Spring Break" sale in the calendar to fill the entire day, which will then allow it to map into the parent dataset of daily sales.

If the view definition is specified as follows:

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
          "joinInterval" : "day"
        }
      }
    }
  ]
}
```

Then you will receive a view with the following columns:

<table class="table table-striped mb20">
<th>
timestamp
</th>
<th>
sales
</th>
<th>
calendar:promotions
</th>
<th>
event:promotions:valentinesday
</th>
<th>
event:promotions:startofspringbreak
</th>
<th>
event:promotions:memorialdaysale
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
1
</td>
<td>
0
</td>
<td>
0
</td>
<tr>
<td colspan="6">
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
1
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
<td colspan="6">
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
1
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

If instead you had hourly sales data, then the "Spring Break" sale would be properly mapped to the 2PM hour in that dataset, without you needing to specify a `joinInterval`:

<table class="table table-striped mb20">
<th>
timestamp
</th>
<th>
sales
</th>
<th>
calendar:promotions
</th>
<th>
event:promotions:valentinesday
</th>
<th>
event:promotions:startofspringbreak
</th>
<th>
event:promotions:memorialdaysale
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
When a timezone is provided then the event start time in the source calendar will first be mapped to that timezone before being matched to dates in the parent dataset. If the date values in the parent dataset are in a different timezone than the calendar, you may need to specify a timezone in order to prevent events from overlapping more than one day when at a daily level of granularity.