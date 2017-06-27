---
title: Error Codes
description: Detailed information about error codes
copyright: 2017 Nexosis 
layout: default
category: Getting Started
tags: [Quick Links, Favorite, Reference]
use_codestyles: true
order: 6
---

The following are the different kinds of error messages you could receive when using the Nexosis Api.

## 400

These are generally validation errors on the request being performed.  If you are receiving a 400 error, the property or value which is failing validation will be specified in the `errorDetails` property of the response.  In particular, the following are the validation errors you could see.

### InvalidColumnMetadata

When uploading data, the accompanying column metadata is validated to ensure that we correctly understand how all of the columns in the uploaded dataset should be used.  The column name and the reason that validation failed will be specified.

``` json
{
  "statusCode": 400,
  "message": "One or more columns were supplied with invalid metadata",
  "errorType": "InvalidColumnMetadata",
  "errorDetails": {
    "sales": [
      "Target columns must have data type Numeric"
    ],
  }
}
```

### InvalidDateRange

Any requests that use two date parameters for the `startDate` and `endDate` of a range must both have valid dates, and the `startDate` must be before the `endDate`.  These include any of the endpoints which filter returned results in a GET request, or POSTs which use the dates as parameters to start `Sessions`.  Ensure that the dates are sent in ISO-8601 format, and that the `startDate` is before the `endDate`.

``` json
{
  "statusCode": 400,
  "message": "Start Date must be before End Date",
  "errorType": "InvalidDateRange",
  "errorDetails": {
    "startDate": "2018-01-01T00:00:00+00:00",
    "endDate": "2017-01-05T00:00:00+00:00"
  }
}
```

### NoData

When issuing a PUT to the `/data/{dataSetName}` endpoint, ensure that the intended dataset is being included in the request.  If no data was received, than this error will be returned.

``` json
{
  "statusCode": 400,
  "message": "No data was sent",
  "errorType": "NoData",
  "errorDetails": {
    "dataSetName": "Location-A"
  }
}
```

### SomeParametersRequired

Some requests have all of their parameters marked as optional, but, at least one parameter of the set must be specified.  This indicates that there were no parameter values received when at least one of them are required in order to perform the operation.

``` json
{
  "statusCode": 400,
  "message": "Request is invalid",
  "errorType": "RequestValidation",
  "errorDetails": {
    "requestedAfterDate": [
      "Value for requestedAfterDate must be a valid date"
    ]
  }
}
```

### RequestValidation

This is a general error indicating that we were unable to parse the request that was sent.  Ensure that the request body conforms to the schema provided for that request in the [Api documentation]({{site.api_reference_baseurl}}).

``` json
{
  "statusCode": 400,
  "message": "Request is invalid",
  "errorType": "RequestValidation",
  "errorDetails": {
    "data": [
      "The data field is required."
    ]
  }
}
```

## 401

This indicates that the `api-key` header, which much be present on all requests, was not valid.  Either the header was not included at all, or the provided key was not valid.  Refer to the [Api key documentation](apikeys) for further information.

``` json
{
  "statusCode": 401,
  "message": "Access denied due to invalid subscription key. Make sure to provide a valid key for an active subscription."
}
```

## 404

Returned whenever the specified resource was not found.  This error can also be returned if the endpoint that the request was issued to does not exist.  If a specific resource was requested, the `itemId` and `itemType` will be included in the response.  Ensure that this Id is the one you intended.  Also, check the listing endpoints, such as `/data` or `/sessions` if you are unsure of the id.

``` json
{
  "statusCode": 404,
  "message": "Item of type dataSet with identifier Location-A was not found",
  "errorType": "NotFound",
  "errorDetails": {
    "itemType": "dataSet",
    "itemId": "Location-A"
  }
}
```
