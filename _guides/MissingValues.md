---
title: Missing Values
description: What to do with missing values
copyright: 2017 Nexosis 
layout: default
category: Concepts
tags: [Imputation]
use_codestyles: true
---

### Working with Missing Values
When working with time-series data you may have gaps in the target interval where no data was collected.  This can cause some issues for our time series prediction algorithms because it is not clear how to interpret that data and causes most algorithms to perform poorly.

If you are cleaning your data before submitting it to Nexosis then it's best to figure out what a missing value means to you in your domain of expertise and fill in the missing values at the interval you intend to predict.

For example, given a daily dataset:

<table>
<th>Timestamp</th>
<th>Value</th>
<tr>
    <td>2017-08-13 08:12:00</td>
    <td>685.22</td>
</tr>
<tr>
    <td>2017-08-14 09:10:00</td>
    <td>871.29</td>
</tr>
<tr>
    <td>2017-08-15 08:12:00</td>
    <td>358.11</td>
</tr>
<tr>
    <td>2017-08-17 08:12:00</td>
    <td>62.58</td>
</tr>
<tr>
    <td colspan="2">...</td>
</tr>
</table>

We're missing the data for the 16th. This missing data could be an important indicator of the trend from that point. Of course, the more values which are missing, the greater the effect.

The Nexosis API makes an inference when there is a gap between observed values, and imputes (creates values) zeroes for the gap at the appropriate interval. In other words, the modified dataset on which we would run predictions would be:
<table>
<th>Timestamp</th>
<th>Value</th>
<tr>
    <td>2017-08-13 08:12:00</td>
    <td>685.22</td>
</tr>
<tr>
    <td>2017-08-14 09:10:00</td>
    <td>871.29</td>
</tr>
<tr>
    <td>2017-08-15 08:12:00</td>
    <td>358.11</td>
</tr>
<tr>
    <td>2017-08-16 00:00:00</td>
    <td>0.00</td>
</tr>
<tr>
    <td>2017-08-17 08:12:00</td>
    <td>62.58</td>
</tr>
<tr>
    <td colspan="2">...</td>
</tr>
</table>
