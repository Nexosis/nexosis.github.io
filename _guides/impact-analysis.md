---
title: Impact Analysis 
description: Using the Nexosis API to determine the Impact of an event on your data
copyright: 2017 Nexosis 
layout: default
category: Forecasting
tags: [Causal Impact, Impact, Quick Links, Favorite]
use_codestyles: true
order: 3
---

Impact analysis is when we analyze your historical time series data and determine the impact of internal and external factors. You can use the predictive power of impact analysis to evaluate the impact of an event, promotion, limited time offer, etc. on your bottom line, determine impact of weather or illness on productivity, or understand how holidays will impact sales.

-----

### Creating an Impact Session

#### Impact sessions determine the impact of a particular event on a dataset. To create an impact session, specify the dataset for which to determine impact, as well as the start and end dates of the impactful event. The Nexosis API will execute a series of machine learning algorithms to determine the impact of the event on the dataset.

> Both the start and end dates for the impact session must always be on or before the specified timestamp of the last record in your dataset.

_To understand how to submit data, you can read more in the [Sending Data](/guides/sending-data) article._

**Here are the parameters you can include in either the query string or the body of a POST request to create an Impact Analysis Session:**
* `dataSourceName` - Name of the data source (dataset or view) for which to determine impact
* `targetColumn`- Column in the specified dataset for which to determine impact
* `eventName` - Name of the event for which to determine impact
* `resultInterval` - The interval at which predictions should be generated. Possible values are `Hour`, `Day`, `Week`, `Month`, and `Year`. Defaults to `Day`
* `startDate` - Format date-time (as date-time in ISO8601). First date of the event
* `endDate` - Format date-time (as date-time in ISO8601). Last date of the event
* `callbackUrl` - The Webhook URL that will receive updates when the Session status changes

### Starting an Impact Session

#### Impact sessions are essentially identical to forecast sessions, so make sure to read and understand how to [create predictions](forecasting-walkthrough). All of that information will apply with a couple of additional points that will be outlined in the rest of this section.

**Here are the differences:**

1. Parameters include an additional value called `eventName` so you can create a friendly name for the intervention period.
2. Impact session results also include metrics that describe the overall impact of the event on the dataset. These metrics are:
    * `pValue` - Statistical value used to determine the significance of the impact. A small p-value indicates strong evidence of impact, whereas a p-value approaching 0.5 indicates weak evidence of impact.
    * `absoluteEffect` - Total absolute effect of the event on the dataset. Answers the question, "How much did this event affect my dataset?"
    * `relativeEffect` - Percentage effect of the event on the dataset. Answers the question, "By what percentage did this event affect my dataset?"

><strong>IMPORTANT:</strong> If you want to find the impact a particular feature had on the target value, you must set that feature's value off during the dates of the intervention period otherwise it will not measure the impact correctly. Impact session results consist of the predictions of what would have happened over the specified date range, had the impactful event not occurred, which is why that feature should be removed from the intervention period.
