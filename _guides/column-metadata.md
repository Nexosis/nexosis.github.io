---
title: Column Metadata
description: How to define column metadata on a dataset
category: Nexosis Concepts
subcategory: Upload your data
tags: [reference, data]
quick_link: true
use_codestyles: true
order: 7
---

When [uploading data](sending-data), [importing data](importing-data), or [starting a session](sessions), you can specify metadata about the columns in your dataset.  This allows the Nexosis API to better understand how your data should be used and interpreted when running machine learning algorithms.

> Note: Setting column metadata is totally optional. For simple datasets with a couple of columns, the contents of the columns are inferred, and providing metadata may not make a difference in how the algorithms execute.

----

## <span name="dataTypes">Column DataTypes</span>

#### Setting the column `dataType` property is defining the kind of data that the column contains. The supported data types are:

- Numeric - Numbers, which can be integers or floats.
- NumericMeasure - Exactly like a Numeric column, except that Imputation and Aggregation are handled differently.
- Logical - Boolean values, which can be one of the following pairs of values:
    - `True` - `False`
    - `1` - `0`
    - `On` - `Off`
    - `Yes` - `No`
- Date - Dates or dates and times which should be in [ISO-8601 format](working-with-dates).
- String - Any other data which does not fit into the above categories.  These values can the thought of as labels on a row of data. The data science technique we use is called [One Hot Encoding](https://www.quora.com/What-is-one-hot-encoding-and-when-is-it-used-in-data-science).
- Text - Free-form text. See [Using Text Based Features](text-based-features) for more information.

Our handling of Imputation and Aggregation are discussed in more detail in the sections below.

----

## Imputation and Aggregation

#### Whenever you request a [Forecast session](forecasting-walkthrough) or an [Impact Analysis session](impact-analysis) from the Nexosis API, you ask for that forecast to come back in a certain `resultInterval`.  That interval is something like `hour`, `day`, `week`, etc.

When you request a one of these sessions, the Nexosis API goes through the following process against your data to prepare it for the Session:

- Loads your data from its source DataSet
- Orders the data according to the column with the role `timestamp`
- Processes the data into numeric values, using techniques such as [One Hot Encoding](https://www.quora.com/What-is-one-hot-encoding-and-when-is-it-used-in-data-science)
- Aggregates that data by the `resultInterval` requested in the session

> The Nexosis API doesn't care whether your data is a daily/hourly/weekly rollup, or whether it's a raw feed of sensor data.

- Once the data is aggregated by `resultInterval`, it looks for gaps in the aggregated data and Imputes missing values.

Depending on the nature of the data you're working with, you may need the Nexosis API to act differently with regard to the handling of missing values [(Imputation)](https://en.wikipedia.org/wiki/Imputation_%28statistics%29) and the aggregation (e.g. if your data is hourly data but you're forecasting by day) of that data.

For example, if a column of data is a record of the number of sales, then the defaults (Imputing with a zero and summing up data when aggregating) make sense.  However, if a column is a temperature reading then those defaults don't make sense.  In that case you'd probably want a missing value to be filled in with the average of the adjacent values, and to use the mean temperature when aggregating.

Each `dataType` available for a column of data in the Nexosis API comes with a set of default Imputation and Aggregation strategies that we feel are sensible choices for those data types.

If you wish to manually override the strategy used by the API for a column of data, you can do so through the `imputation` and `aggregation` fields.

### Imputation and Aggregation Option Definitions

- **Mean** -- The mean is the **average** of all the values. To calculate this value, we add up all the values in the column and divide by the rows. *(Example: 1, 2, 3, 5, 8 = mean of 3.8)*
- **Median** -- The median is the **middle** value. To calculate this value, we take all the values in sequential order and find the one that is in the middle. *(Example: 1, 2, 3, 5, 8 = median of 3. If there are two values in the middle as you would find in a dataset with an even number of rows, the lower of those two values is returned as the median.)*
- **Mode** -- The mode is the value that appears **most often**. To calculate this value, we take all of the values and determine which we see the most. *(Example: 1, 2, 2, 3, 5, 8 = mode of 2)*
- **Max** -- The max is simply the **highest value**.
- **Min** -- The min is the **lowest value**.

### Imputation

#### The Nexosis API offers the following options for Imputation.

- `zeroes` -- A missing value is filled in with 0
- `mean` -- A missing value is filled in with the **average** of the nearest values we can find on either side of that gap.
- `median` -- A missing value is filled in with the **median** value of the rest of the values in that column
- `mode` -- A missing value is filled in with the **mode** of the rest of the values in that column
- `max` -- A missing value is filled in with the **maximum** of the rest of the values in that column
- `min` -- A missing value is filled in with the **minimum** of the rest of the values in that column

### Aggregation Options

#### The Nexosis API offers the following options when aggregating to a given `resultInterval`

- `sum` -- The resulting value is the **sum** of all column values that fall within the `resultInterval`
- `mean` -- The resulting value is the **average** of all column values that fall within the `resultInterval`
- `median` -- The resulting value is the **median** of all column values that fall within the `resultInterval`
- `mode` -- The resulting value is the **mode** of all column values that fall within the `resultInterval`
- `max` -- The resulting value is the **maximum** of all column values that fall within the `resultInterval`
- `min` -- The resulting value is the **minimum** of all column values that fall within the `resultInterval`

### Defaults per dataType

#### Nexosis tries to assign sensible defaults for each `dataType` available.  Those defaults are below.

| DataType | Typical Usage | Imputation Default | Aggregation Default |
| -------- | ------------- | ------------------ | ------------------- |
| Numeric | Number of sales, etc. | `zeroes` | `sum` |
| NumericMeasure | Temperature reading, or a value that's an interval itself (transactions/sec, etc) | `mean` | `mean` |
| Logical | Values that are a simple yes/no | `mode` | `mode` |
| String | Categorical data as described [in DataTypes](#dataTypes) | `mode` | `mode` |

Date values are generally used for timestamps, and so you cannot explicitly set imputation or aggregation strategy for these.

Text columns are not currently used by time-series sessions, and so missing values are never aggregated. Missing text values are treated as empty.

----

## Column Roles

#### While the data type of a column defines what kind of data is in a column, the role of a column defines how that column is used by the Nexosis API.  These roles can actually be changed depending on what you are trying to learn from your dataset.  The available roles are:

- None - Setting a column role to None means that this column will not be used when generating results.
- Timestamp - Defines the column used for ordering/aggregation when performing time-series analysis.
- Target - Specifies that this is the column which should be the target of the session.  This is the column that will have results generated by sessions.  Only one target column may be specified at a time.
- Feature - A feature column contains other supporting data that may be related to the target column.  These values provide more information that the Nexosis API is able to use when generating results.
- Key - An optional unique key for each record to aid in data set operations or matching up session results with originals. Nexosis will create an internal key if no key column is specified, but it will not be returned in DataSet queries.

----

## Overriding Roles

#### Column roles can be specified on both a dataset, and on a session that uses the same dataset.  In this situation, the column roles specified on the session will be used only for that session.  This can be useful when you want to predict the values of different columns in a dataset and use the other columns as features.

> Impact sessions should be started with an override of column roles if the value being tested for impact is defined as a feature in the dataset.  This ensures that the algorithms process the dataset as if the impact feature was not present.  The column which specifies the impact event should be set to a role of `None`.

Refer to the [Specifying Features](specifying-features) tutorial for a more in-depth look at overriding column roles.

----

## Example

#### An example showing how you can specify some of the options available in Column Metadata

``` json
{
    "columns" : {
        "timeStamp" : {
            "dataType" : "date",
            "role" : "timestamp"
        },
        "sales" : {
            "dataType" : "numeric",
            "role" : "target"
        },
        "temperature" : {
            "dataType" : "numericMeasure",
            "role" : "feature"
        },
        "promotion" : {
            "dataType" : "logical",
            "role" : "feature"
        },
        "peakCustomersPerHour" : {
            "dataType" : "numeric",
            "role" : "feature",
            "imputation" : "zeroes",
            "aggregation" : "mean"
        }
    }
}
```

## Validations and restrictions

#### Column datatypes and roles must meet some restrictions and validations.  If any of these requirements are not met, you will receive a [400 error response](error-codes) which details which column has the validation issue, and what that validation restriction is.  These validations ensure that the Nexosis API is able to understand how to use the values in your dataset, and is able to generate accurate results.
