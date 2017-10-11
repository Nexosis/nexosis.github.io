---
title: Column Metadata
description: How to define column metadata on a dataset
copyright: 2017 Nexosis 
layout: default
category: Concepts
tags: [Reference, Quick Links, Data, Favorite]
use_codestyles: true
order: 7
---

When [uploading data](sendingdata), [importing data](importingdata), or [starting a session](session), you can specify metadata about the columns in your dataset.  This allows the Nexosis API to better understand how your data should be used and interpreted when running machine learning algorithms.

> Note: Setting column metadata is totally optional.  For simple datasets with a couple of columns, the contents of the columns are inferred, and providing metadata may not make a difference in how the algorithms execute.

----

## <a name="dataTypes">Column DataTypes</a>

Setting the column `dataType` property is defining the kind of data that the column contains.  The supported data types are:

- Numeric - Numbers, which can be integers or floats.
- Logical - Boolean values, which can be one of the following pairs of values:
    - `True` - `False`
    - `1` - `0`
    - `On` - `Off`
    - `Yes` - `No`
- Date - Dates or dates and times which should be in [ISO-8601 format](workingwithdates).
- String - Any other data which does not fit into the above categories.  These values can the thought of as labels on a row of data. The data science technique we use is called [One Hot Encoding](https://www.quora.com/What-is-one-hot-encoding-and-when-is-it-used-in-data-science).
- NumericMeasure - Exactly like a Numeric column, except that Imputation and Aggregation are handled differently.

Our handling of Imputation and Aggregation are discussed in more detail in the sections below.

----

## Imputation and Aggregation

Whenever you request a [Forecast session](forecast) or an [Impact Analysis session](impactanalysis) from the Nexosis API, you ask for that forecast to come back in a certain `resultInterval`.  That interval is something like `hour`, `day`, `week`, etc.

When you request a session, the Nexosis API goes through the following process against your data to prepare it for the Session:

- Loads your data from its source DataSet
- Aggregates that data by the `resultInterval` requested in the session.

> Nexosis itself doesn't care one bit whether your data is a daily/hourly/weekly rollup, or whether it's a raw feed of sensor data.

- Once the data is aggregated by `resultInterval`, it looks for gaps in the aggregated data and Imputes missing values.

Depending on the nature of the data you're working with, you may need the Nexosis API to act differently with regard to the handling of missing values [(Imputation)](https://en.wikipedia.org/wiki/Imputation_%28statistics%29) and the aggregation (e.g. if your data is hourly data but you're forecasting by day) of that data.

For example, if a column of data is a record of the number of sales, then the defaults (Imputing with a zero and summing up data when aggregating) make sense.  However, if a column is a temperature reading then those defaults don't make sense.  In that case you'd probably want a missing value to be filled in with the average of the adjacent values, and to use the mean temperature when aggregating.

Each `dataType` available for a column of data in the Nexosis API comes with a set of default Imputation and Aggregation strategies that we feel are sensible choices for those data types.

If you wish to manually override the strategy used by the API for a column of data, you can do so through the `imputation` and `aggregation` fields.

### Imputation

The Nexosis API offers the following options for Imputation.

- `zeroes` -- A missing value is filled in with 0
- `mean` -- A missing value is filled in with the **average** of the nearest values we can find on either side of that gap.
- `median` -- A missing value is filled in with the **median** value of the rest of the values in that column
- `mode` -- A missing value is filled in with the **mode** of the rest of the values in that column

### Aggregation Options

The Nexosis API offers the following options when aggregating to a given `resultInterval`

- `sum` -- The resulting value is the **sum** of all column values that fall within the `resultInterval`
- `mean` -- The resulting value is the **average** of all column values that fall within the `resultInterval`
- `median` -- The resulting value is the **median** of all column values that fall within the `resultInterval`
- `mode` -- The resulting value is the **mode** of all column values that fall within the `resultInterval`

### Defaults per dataType

Nexosis tries to assign sensible defaults for each `dataType` available.  Those defaults are below.

| DataType | Typical Usage | Imputation Default | Aggregation Default |
| -------- | ------------- | ------------------ | ------------------- |
| Date | Timestamp values.  You can't explicitly set an imputation or aggregation strategy for these | | |
| Numeric | Number of sales, etc. | `zeroes` | `sum` |
| Logical | Values that are a simple yes/no | `mode` | `mode` |
| String | Categorical data as described [in DataTypes](#dataTypes) | `mode` | `mode` |
| NumericMeasure | Temperature reading, or a value that's an interval itself (transactions/sec, etc) | `mean` | `mean` |

----

## Column Roles

While the data type of a column defines what kind of data is in a column, the role of a column defines how that column is used by the Nexosis API.  These roles can actually be changed depending on what you are trying to learn from your dataset.  The available roles are:

- None - Setting a column role to None means that this column will not be used when generating results.
- Timestamp - Defines the column used for time series algorithms.
- Target - Specifies that this is the column which should be the target of the session.  This is the column that will have results generated by sessions.  Only one target column may be specified at a time.
- Feature - A feature column contains other supporting data that may be related to the target column.  These values provide more information that the Nexosis API is able to use when generating results.

----

## Overriding Roles

Column roles can be specified on both a dataset, and on a session that uses the same dataset.  In this situation, the column roles specified on the session will be used only for that session.  This can be useful when you want to predict the values of different columns in a dataset and use the other columns as features.

> Impact sessions should be started with an override of column roles if the value being tested for impact is defined as a feature in the dataset.  This ensures that the algorithms process the dataset as if the impact feature was not present.  The column which specifies the impact event should be set to a role of `None`.

Refer to the [Specifying Features](specifyingfeatures) tutorial for a more in-depth look at overriding column roles.

----

## Example

An example showing how you can specify some of the options available in Column Metadata

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

Column datatypes and roles must meet some restrictions and validations.  If any of these requirements are not met, you will receive a [400 error response](errorcodes) which details which column has the validation issue, and what that validation restriction is.  These validations ensure that the Nexosis API is able to understand how to use the values in your dataset, and is able to generate accurate results.
