---
title: Importing Data
description: Import data into the Nexosis platform
copyright: 2017 Nexosis 
layout: default
category: Concepts
tags: [Data, CSV, JSON, Quick Links]
use_codestyles: true
order: 4
---

As an alternative to [sending](sendingdata) your data directly to the Nexosis API, you can import your data into the Nexosis platform from external sources.

 ---

## Importing from AWS S3
If you have a `CSV` or `JSON` file hosted out on [AWS S3](https://aws.amazon.com/s3), you can tell the Nexosis Platform to import that data into a [DataSet](howitworks#dataset).

To tell Nexosis to import data from S3, you'll issue a `POST` request to [/imports/s3]({{site.api_reference_baseurl}}/operations/595ce629e0ef6e0c98d37f2f). The body of this request should have the following information.

* `dataSetName` : The name of the Dataset into which the data from the S3 file should be imported.
* `bucket` : The S3 [bucket](http://docs.aws.amazon.com/AmazonS3/latest/dev/UsingBucket.html) containing the file to be imported.
* `path` : The path to the file inside of the bucket.  The data inside this file will be imported.
* `region` (optional) : The AWS [region](https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/) in which your bucket is hosted. This field is optional, if your bucket is in the default AWS region ( `us-east-1` ).
* `columns` (optional) : The [Column Metadata](columnmetadata) to be applied to the data being imported.

``` json
{
  "dataSetName": "Location-A",
  "bucket": "nexosis-sample-data",
  "path": "LocationA.csv",
  "region": "us-east-1",
  "columns": {}
}
```

The response from this POST will be some details about the import.

``` json
{
  "importId": "015d1d17-2a45-4f61-aa97-f21cc4fd656c",
  "type": "s3",
  "status": "requested",
  "dataSetName": "Location-A",
  "parameters": {
    "bucket": "nexosis-sample-data",
    "path": "LocationA.csv",
    "region": "us-east-1"
  },
  "requestedDate": "2017-07-07T12:47:23.682205+00:00",
  "statusHistory": [
    {
      "date": "2017-07-07T12:47:23.682205+00:00",
      "status": "requested"
    }
  ],
  "messages": [],
  "columns": {},
  "links": [
    {
      "rel": "data",
      "href": "https://ml.nexosis.com.com/v1/data/Location-A"
    }
  ]
}
```

The response will also contain a `Nexosis-Import-Status` header with the status of the import.

### Supported File Extensions
At this time we can support the following file extensions and formats.
* `CSV`
* `JSON` : If you're importing a JSON file, the contents of the JSON should be a simple array of JSON objects without any additional metadata:

``` json
[
  {
    "col1": "value11",
    "col2": "value21"
  },
  {
    "col1": "value12",
    "col2": "value22"
  }
]
```

* `gz` : You can optionally gzip a file in one of the above formats.

See the guide on [Sending Data](/guides/sendingdata) for more examples.

## Checking the status of an import
Once you've submitted an import, you may want to check back to see when it completes.  To see the status of an import you make a `GET` request to [/imports/{importId}]({{site.api_reference_baseurl}}/operations/595ce629e0ef6e0c98d37f30).  

The response from this endpoint will be in the same format as when you made the initial `POST` to start the import.

``` json
{
  "importId": "015d1d17-2a45-4f61-aa97-f21cc4fd656c",
  "type": "s3",
  "status": "completed",
  "dataSetName": "Location-A",
  "parameters": {
    "bucket": "nexosis-sample-data",
    "path": "LocationA.csv",
    "region": "us-east-1"
  },
  "requestedDate": "2017-07-07T12:47:23.682205+00:00",
  "statusHistory": [
    {
      "date": "2017-07-07T12:47:23.682205+00:00",
      "status": "requested"
    },
    {
      "date": "2017-07-07T12:47:24.8153474+00:00",
      "status": "started"
    },
    {
      "date": "2017-07-07T12:47:27.5910919+00:00",
      "status": "completed"
    }
  ],
  "messages": [],
  "columns": {},
  "links": [
    {
      "rel": "data",
      "href": "https://ml.nexosis.com/v1/data/Location-A"
    }
  ]
}
```

## Listing imports
You can also query the imports you've run previously by issuing a `GET` to [/imports]({{site.api_reference_baseurl}}/operations/595ce629e0ef6e0c98d37f31). 

You can also provide the following parameters in the query string to filter the imports you've run.

* `dataSetName` : Limits imports to those for a particular dataset

* `requestedAfterDate` : Limits imports to those requested on or after the specified date

* `requestedBeforeDate` : Limits imports to those requested on or before the specified date

* `page` : Zero-based page number of imports to retrieve

* `pageSize` : Count of imports to retrieve in each page (max 1000)

The response from this endpoint will be an object containing an array of import records

``` json
{
  "items": [
    {
      "importId": "015d1836-ab9e-4839-be01-6dda3d710d06",
      "type": "s3",
      "status": "completed",
      "dataSetName": "s3-import-locationa",
      "parameters": {
        "bucket": "nexosis-sample-data",
        "path": "LocationA.csv",
        "region": "us-east-1"
      },
      "requestedDate": "2017-07-06T14:03:42.326925+00:00",
      "statusHistory": [
        {
          "date": "2017-07-06T14:03:42.326925+00:00",
          "status": "requested"
        },
        {
          "date": "2017-07-06T14:03:43.3510578+00:00",
          "status": "started"
        },
        {
          "date": "2017-07-06T14:03:46.5554317+00:00",
          "status": "completed"
        }
      ],
      "messages": [],
      "columns": null,
      "links": [
        {
          "rel": "data",
          "href": "https://ml.nexosis.com.com/v1/data/s3-import-locationa"
        }
      ]
    }
  ]
}
```