---
title: Working With Dates
description: Formatting dates properly can be tricky.
copyright: 2017 Nexosis 
layout: default
category: Getting Started
tags: [Dates, Formatting, Data]
use_codestyles: true
order: 4
---

When it comes to forecasting, getting the dates correct is critical. When dealing with dates as strings the date formatting has to be correct. We use libraries in all languages that support [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) formats for interchanging dates.

-----
> If a Date/Time formatted string submitted does not contain a Time Zone indicator, Nexosis API will assume it to be UTC. If you request your DataSet back, it will be returned without the Time Zone indicator since we make an effort to not modify original submitted data.

### Use a robust DateTime library

All DateTimes, when converted to strings should be converted to ISO 8601 format and include the TimeZone / Offset

* .NET - [Noda Time](http://nodatime.org/)
* Java - [Joda Time](http://www.joda.org/joda-time/)
    * <code>DateTime.now().toDateTimeISO().toString()</code>
* Ruby - [Time.rb](https://ruby-doc.org/stdlib-2.1.1/libdoc/time/rdoc/Time.html)
    * <code>Time.now.iso8601</code>
* NodeJS - [Moment.js](https://momentjs.com/)
    * <code>moment().toISOString()</code>
* Python - [datetime module](https://docs.python.org/2/library/datetime.html) 
    * <code>date.isoformat()</code>
* Python - [iso8601 module](https://pypi.python.org/pypi/iso8601)
    * <code>iso8601.parse_date("2007-01-25T12:00:00Z")</code>
