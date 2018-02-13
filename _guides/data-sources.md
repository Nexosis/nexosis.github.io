---
title: Data Sources
description: Learn about data sources used in forecasts
category: Nexosis Concepts
subcategory: Upload your data
tags: [DataSet, Quick Links, Favorite]
use_codestyles: true
order: 5
---

The Nexosis API generates predictions in a forecast session based on data sources. Typically these are Data Sets you have provided to the API. As of the August 2017 API update these data sources can also be Views or Global Data Sets.

-----

## Overview
When a Session request asks for a dataset name it is referencing a unique name that you have assigned to either a [DataSet](/guides/sending-data) or a [View](/guides/views). These names must be unique within your organization so that a name can be meaningfully used when creating a session. Views are a helper mechanism designed to allow you to mix and match data from different Data Sets you have already loaded to the API. You are encouraged to read more about how they can help you [here](/guides/views).

## Global Data Sets
Another new type of data source is the Global Data Set. Globally available Data Sets may be included by providing the globally unique name. These datasets are not useful for directly forecasting in a session. Instead they are intended as common features one may wish to include with an existing dataset. You might try adding one of these if you suspect the subject matter of the global dataset has an influence on your data. In order to add a Global Data Set to your session, include it as a join target in a [view](/guides/views).

### Available Global Data Sets
The Nexosis API currently supports holiday calendars as global data sets. In order to reference a holiday calendar you use the prefix *Nexosis.Holidays-* followed by the [two character country code as defined by ISO-3166](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2). For example:
 
- Nexosis.Holidays-US - a time-series based set of day long holidays in the USA
- Nexosis.Holidays-CA - a time-series based set of day long holidays in Canada

> See our [calendar listing in GitHub](https://github.com/Nexosis/holiday-calendars) for which country codes we support.

In order to define a named calendar as a data source in a join you would include the following join syntax in your view definition:

``` 
{
  "dataSetName": "MyDataset",
  "joins": [
    {
      "calendar": {
        "name": "Nexosis.Holidays-US"
      }
    }
  ]
}

```
> See [our guide to calendars](/guides/calendar-data-sources) for more details about how calendars can be used to create features in your view.