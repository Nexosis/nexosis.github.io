---
title: Data Sources
description: Learn about data sources used in forecasts
copyright: 2017 Nexosis 
layout: default
category: Forecasting
tags: [DataSet, Quick Links]
use_codestyles: true
---

The Nexosis API generates predictions in a forecast session based on data sources. Typically these are Data Sets you have provided to the API. As of the August 2017 API update these data sources can also be Views or Global Data Sets.

-----

## Data Sources
When a Session request asks for a data set name it is referencing a unique name that you have assigned to either a [DataSet](/guides/sendingdata) or a [View](/guides/views). These names must be unique within your organization so that a name can be meaingfully used when creating a session. Views are a helper mechanism designed to allow you to mix and match data from different Data Sets you have already loaded to the API. You are encouraged to read more about how they can help you [here](/guides/views).

### Global Data Sets
Another new type of data source is the Global Data Set. Globally available Data Sets may be included by providing the globally unique name. These datasets are not useful for directly forecasting in a session. Instead they are intended as common features one may wish to include with an existing dataset. You might try adding one of these if you suspect the subject matter of the global data set has an influence on your data. In order to add a Global Data Set to your session, include it as a join target in a [view](/guides/views).

#### Available Global Data Sets
- Nexosis.USAHolidays - a time-series based set of day long holidays in the USA
- Nexosis.CANHolidays - a time-series based set of day long holidays in Canada