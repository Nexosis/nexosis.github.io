---
title: Algorithm Contest Results
description: Dig into the performance of various algorithms run during a session.
category: Nexosis Concepts
subcategory: Run a session
tags: [Quick Links, Predict, Data Science]
use_codestyles: true
order: 12
---
Every session involves a process we call "Reigning Champion" where several candidate algorithms compete. We expose contest endoints to allow you insight into what happened during the selection process.

------

First off it is important to note that contest endpoints are available only in the paid tiers. [Consider upgrading](https://www.nexosis.com/pricing) if you have a Community license but would like to examine your sessions in detail.

## To Choose a Champion
#### The contest endpoints will make a little more sense when you understand the terms we use for the participants in the process. 

* `Contest` - The process of choosing a champion within a single session.
* `Champion` - because we call the process "Reigning Champion" we consider the winning algorithm to be the *champion*. Think of it like Thunderdome - many algorithms enter, one algorithm leaves.
* `Contestant` - of course in order to choose a champion you have to have a competition and every algorithm selected to compete is a *contestant* in the scope of a single session.
* `Champion Metric` - Each class of algorithms has a particular metric by which the champion is chosen. While many metrics may be calculated, the *championMetric* is the one used to pick a winner. 

## Contest Endpoints
#### We have provided 5 endpoints to retrieve information about the contest:

* /sessions/{sessionId}/contest - Overview of champion and all contestant algorithm's performance
* /sessions/{sessionId}/contest/champion - Champion metrics and test data
* /sessions/{sessionId}/contest/contestants - Listing of every metric without additional session data. Note that the champion is also a member of the contestants list.
* /sessions/{sessionId}/contest/contestants/{contestantId} - Specific algorithm with test data results
* /sessions/{sessionId}/contest/selection - Information about the dataset we use to filter algorithms

While there are several endpoints, there are two important concepts and data structures.

### Algorithm
Each champion or contestant algorithm is returned with the same json schema. Contestants are returned within an array, but otherwise all algorithms will come back with the follow data structure:

``` json
 {
    "id": "01606a66-616e-44e3-be1d-76ea8834d508",
    "algorithm": {
        "name": "Elastic Net Regularization",
        "description": "Elastic Net Regularization",
        "key": ""
    },
    "dataSourceProperties": [
        "Imputed",
        "Scaled"
    ],
    "metrics": {
        "rootMeanSquareError": 0.035829456097769073,
        "rSquared": 0.99999993889778993
    },
    "links": [
        {
            "rel": "self",
            "href": "https://ml.nexosis.com/v1/sessions/01606a66-11e6-4263-a739-f98af5d5e110/contest/contestants/01606a66-616e-44e3-be1d-76ea8834d508"
        }
    ]
}
``` 

##### The properties and objects included are as follows:

* `id` - A unique identifier for this algorithm within this session. This is used primarily as a way to pull back the test data from the contestants endpoint.
* `algorithm` - contains the name and description for this particular algorithm.
* `dataSourceProperties` - these are general identifiers of actions the API took on the data source before performing the calculations. 
* `metrics` - a collection of name value pairs where the names are a varied set of metrics for the given algorithm run. Each class of algorithm will tend to have the same set, and the *championMetric* identified by the contest endpoint should always be an available metric key.
* `links` - hypermedia support for related endpoints

### Test Data
The second important data structure is the test data which lead to the performance metric used. The data for the champion is available at the *contest/champion* endpoint while each contestant's data is available when using an individual contestant id at the *contest/contestants/{contestantId}* endpoint. In both cases the data will be returned in an array named *data*. 

``` json
{
    "data": [
        {
            "profit": "42012.84375",
            "ny": "0",
            "cali": "1",
            "florida": "0",
            "R.D.Spend": "0",
            "profit:actual": "42559.73",
            "Administration": "135426.92",
            "Marketing.Spend": "0"
        },
      ... additional data points elided
    ]
}
```
Each data object in the array contains all of the features used in the model build along with the target and a column suffixed with *:actual*. The calculated value for the particular contestant is the in the target column without suffix. By default the data is paged and has a page size of 50. If you want all of the data you may have to 