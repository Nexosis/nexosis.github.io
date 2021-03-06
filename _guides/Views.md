---
title: Views
description: Learn about Views and how they help with forecasting
category: Nexosis Concepts
subcategory: Upload your data
tags: [dataset, views]
use_codestyles: true
order: 4
---

Views are data sources created by mixing two other datasets. Using a view you can create a session based on a dataset of observations mixed with a different dataset which contains the features.

-----

## Views

#### API Views are conceptually similar to database views. 

To stretch the analogy a little, if a Dataset is a table, then an API View creates an ephemeral dataset (table) based on a definition which mixes data from other Datasets (tables). The analogy is best ended there as these Views are really quite simple in their construction. The key point is that if you have a Dataset which represents some observations and want to use another for features which you don't want to mix directly - allow Views to do it for you.

### View Definition

In order to define a View you need to give the view a name and then provide the names of two Datasets. The most basic View definition would then be:

``` json
{
   "dataSetName" : "MyObservationSet",
   "joins" : [
     {
       "dataSet" : {
          "name" : "MyFeatureSet"
       }
     }
   ]
}
```

In order to create this View you would PUT to the views endpoint along with a name for your view:

```
https://ml.nexosis.com/v1/views/{viewName}
```

Of course the easiest way to build a join is through one of our existing [API Clients](http://docs.nexosis.com/clients/).

### How Joins Join

There are some things to think about when joining two different datasets. First, how should any one row in the right side of the join be understood to match a row in the left side? Going back to our database analogy, this depends on the key field of the right-side dataset, and the field being joined to in the left-side dataset. In all cases, right-side rows which are matched will add values to the left-side row. 

When the join columns are simple keys, rows from the right side of the join with a key value exactly matching the join column value in the left side are matched. In database terminology, this is an equi-join, and it's the most common type of join to use in views.

When joining to a date field in the left-side dataset, joining gets a bit more complicated. More often than not timestamps won't match exactly, but we'll cover that detail below. 

### Viewing Join Results

Before submitting a session based on a View it's best to have a look at the join results by viewing the View data. By examining View data you can ensure that joins are being interpreted the way that you intended.

##### Example

It will probably help at this point to just look at an example.

Let's say we have a set of observations like the following as 'DailySales':

<table class="table table-bordered mb20">
    <thead>
        <tr>
            <th>timestamp</th>
            <th>sales</th>
        </tr>
    </thead>
    <tbody>
        <tr class="success">
            <td>2010-02-11T00:00Z</td>
            <td class="right">24924.5</td>
        </tr>
        <tr class="success">
            <td>2010-02-12T00:00Z</td>
            <td class="right">46039.49</td>
        </tr>
        <tr class="success">
            <td>2010-02-13T00:00Z</td>
            <td class="right">41595.55</td>
        </tr>
        <tr class="success">
            <td>2010-02-14T00:00Z</td>
            <td class="right">19403.54</td>
        </tr>
        <tr class="info">
            <td>2010-02-15T00:00Z</td>
            <td class="right">18399.22</td>
        </tr>               
    </tbody>
</table>

And you have another dataset containing information about which dates are holidays or weekends called 'HolidayCalendar':

<table class="table table-bordered mb20">
    <thead>
        <tr>
            <th>timestamp</th>
            <th>IsWeekend</th>
            <th>IsHoliday</th>
        </tr>
    </thead>
    <tbody>
        <tr class="success">
            <td>2010-02-11T00:00Z</td>
            <td class="right">false</td>
            <td class="right">false</td>
        </tr>
        <tr class="success">
            <td>2010-02-12T00:00Z</td>
            <td class="right">false</td>
            <td class="right">false</td>
        </tr>
        <tr class="success">
            <td>2010-02-13T00:00Z</td>
            <td class="right">true</td>
            <td class="right">false</td>
        </tr>
        <tr class="success">
            <td>2010-02-14T00:00Z</td>
            <td class="right">true</td>
            <td class="right">false</td>
        </tr>
        <tr class="info">
            <td>2010-02-15T00:00Z</td>
            <td class="right">false</td>
            <td class="right">true</td>
        </tr>               
    </tbody>
</table>

We can define a join of these two datasets with the following View Definition:

``` json
{
	"dataSetName": "DailySales",
	"columns": {
		"timestamp": {
			"dataType": "date",
			"role": "timestamp",
			"imputation": null,
			"aggregation": null
		},
		"sales": {
			"dataType": "numeric",
			"role": "target",
			"imputation": null,
			"aggregation": null
		},
		"IsWeekend": {
			"dataType": "logical",
			"role": "feature"
		},
		"IsHoliday": {
			"dataType": "logical",
			"role": "feature"
		}
	},
	"joins": [
		{
			"dataSet": {
				"name": "HolidayCalendar"
			},
			"columnOptions": {
				"timestamp": { 
					"joinInterval": "Day"
				}
			}
		}
	]
}
```

In the above definition we have not only specified which datasets will provide the values, but have given roles to the newly joined columns; identifying them as features.

The *joinInterval* option simply states that we should try to match the holiday dataset to the daily sales dataset based on the day level of granularity. If used, it should be specified on the timestamp column of the joined dataset.

### Join Results

Having created the join above we should expect the following output:

<table class="table table-bordered mb20">
    <thead>
        <tr>
            <th>timestamp</th>
            <th>sales</th>
            <th>IsWeekend</th>
            <th>IsHoliday</th>
        </tr>
    </thead>
    <tbody>
        <tr class="success">
            <td>2010-02-11T00:00Z</td>
            <td class="right">24924.5</td>
            <td class="right">false</td>
            <td class="right">false</td>
        </tr>
        <tr class="success">
            <td>2010-02-12T00:00Z</td>
            <td class="right">46039.49</td>
            <td class="right">false</td>
            <td class="right">false</td>
        </tr>
        <tr class="success">
            <td>2010-02-13T00:00Z</td>
            <td class="right">41595.55</td>
            <td class="right">true</td>
            <td class="right">false</td>
        </tr>
        <tr class="success">
            <td>2010-02-14T00:00Z</td>
            <td class="right">19403.54</td>
            <td class="right">true</td>
            <td class="right">false</td>
        </tr>
        <tr class="info">
            <td>2010-02-15T00:00Z</td>
             <td class="right">18399.22</td>
            <td class="right">false</td>
            <td class="right">true</td>
        </tr>               
    </tbody>
</table>

### Using Joins

#### Once you have a View defined it can be used in a Session as the data source. 

If you have used previous versions of the API you'll notice that we have changed the parameter 'dataSetName' in the request for a Session to 'dataSourceName'. This change indicates that you can use a data source - either a view or a dataset. If we had named our view above "SalesWithCalendar" then we could create a Session by POSTing to

```
https://ml.nexosis.com/v1/sessions/forecast?dataSourceName=SalesWithCalendar&startDate=2010-02-16&endDate=2010-02-20
``` 
