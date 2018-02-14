---
title: PowerShell Module
description: PowerShell module for using the Nexosis API
category: PowerShell
breadcrumb: API Clients
tags: [powershell, module]
use_codestyles: true
---

## Installing the client

#### The client has been tested against Powershell 5.0 on Windows and OS X.

To install it from the Powershell Gallery (make sure you have an elevated client and have trusted the PS Gallery store - [more information on that topic here](https://blogs.technet.microsoft.com/poshchap/2015/08/07/getting-started-with-the-powershell-gallery/){:target="_blank"}.


```powershell
PS> Install-Module -Name PSNexosisClient
```

Or to choose a path to save the module to:
```powershell
PS> Save-Module -Name PSNexosisClient -Path <path>
```

> <p><a href="https://www.powershellgallery.com/packages/PSNexosisClient/" class="btn secondary mr10" target="_blank"><i class="fa fa-cube mr5"></i> Package Details</a><a href="https://github.com/Nexosis/nexosisclient-ps" class="btn secondary" target="_blank"><i class="fa fa-github mr5"></i> View Source</a></p>

## Basic Usage

Once you've installed the module it needs to be imported. The API key can be set via Environment Variable or through the `Set-NexosisConfig` function.

```powershell
PS> $env:NEXOSIS_API_KEY = 'a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6'
PS> Import-Module PSNexosisClient
PS> Get-NexosisConfig

DefaultPageSize ApiKey                           ApiBaseUrl                       
--------------- ------                           ----------                       
            100 a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6 https://ml.nexosis.com/v1
```

or to set it temporarily using `Set-NexosisConfig`:

```powershell
PS> Import-Module PSNexosisClient
PS> Set-NexosisConfig -ApiKey 'a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6'
PS> Get-NexosisConfig

DefaultPageSize ApiKey                           ApiBaseUrl                       
--------------- ------                           ----------                       
            100 a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6 https://ml.nexosis.com/v1
```

Once it's imported, all of the functions will be available. To discover all the commands available, you can run `Get-Command`

```powershell
PS> Get-Command -Module PSNexosisClient

CommandType     Name                                               Version    Source                                  
-----------     ----                                               -------    ------                                   
Function        Get-NexosisAccountQuotas                           2.2.0      PSNexosisClient
Function        Get-NexosisConfig                                  2.2.0      PSNexosisClient
Function        Get-NexosisContest                                 2.2.0      PSNexosisClient
Function        Get-NexosisContestant                              2.2.0      PSNexosisClient
Function        Get-NexosisContestChampion                         2.2.0      PSNexosisClient
Function        Get-NexosisContestSelection                        2.2.0      PSNexosisClient
Function        Get-NexosisDataSet                                 2.2.0      PSNexosisClient
Function        Get-NexosisDataSetData                             2.2.0      PSNexosisClient
Function        Get-NexosisImport                                  2.2.0      PSNexosisClient
Function        Get-NexosisModel                                   2.2.0      PSNexosisClient
Function        Get-NexosisModelDetail                             2.2.0      PSNexosisClient
Function        Get-NexosisSession                                 2.2.0      PSNexosisClient
Function        Get-NexosisSessionAnomalyScore                     2.2.0      PSNexosisClient
Function        Get-NexosisSessionClassScore                       2.2.0      PSNexosisClient
Function        Get-NexosisSessionConfusionMatrix                  2.2.0      PSNexosisClient
Function        Get-NexosisSessionResult                           2.2.0      PSNexosisClient
Function        Get-NexosisSessionStatus                           2.2.0      PSNexosisClient
Function        Get-NexosisSessionStatusDetail                     2.2.0      PSNexosisClient
Function        Get-NexosisView                                    2.2.0      PSNexosisClient
Function        Get-NexosisViewData                                2.2.0      PSNexosisClient
Function        Get-NexosisVocabulary                              2.2.0      PSNexosisClient
Function        Get-NexosisVocabularySummary                       2.2.0      PSNexosisClient
Function        Import-NexosisDataSetFromAzure                     2.2.0      PSNexosisClient
Function        Import-NexosisDataSetFromCsv                       2.2.0      PSNexosisClient
Function        Import-NexosisDataSetFromJson                      2.2.0      PSNexosisClient
Function        Import-NexosisDataSetFromS3                        2.2.0      PSNexosisClient
Function        Import-NexosisDataSetFromUrl                       2.2.0      PSNexosisClient
Function        Invoke-NexosisPredictTarget                        2.2.0      PSNexosisClient
Function        New-NexosisDataSet                                 2.2.0      PSNexosisClient                             
Function        New-NexosisView                                    2.2.0      PSNexosisClient
Function        Remove-NexosisDataSet                              2.2.0      PSNexosisClient
Function        Remove-NexosisModel                                2.2.0      PSNexosisClient
Function        Remove-NexosisSession                              2.2.0      PSNexosisClient
Function        Remove-NexosisView                                 2.2.0      PSNexosisClient
Function        Set-NexosisConfig                                  2.2.0      PSNexosisClient
Function        Start-NexosisForecastSession                       2.2.0      PSNexosisClient
Function        Start-NexosisImpactSession                         2.2.0      PSNexosisClient
Function        Start-NexosisModelSession                          2.2.0      PSNexosisClient
```

To get detailed help on any of the Functions above, type `Get-Help`

```powershell

PS> Get-Help Get-NexosisAccountQuotas

NAME
    Get-NexosisAccountQuotas
    
SYNOPSIS
    Retrieves the Account Usage Status of your Nexosis API Account.
    
    
SYNTAX
    Get-NexosisAccountQuotas [<CommonParameters>]
    
    
DESCRIPTION
    Given the current API Key, Get-NexosisAccountQuotas returns the Account Usage Stats tracked for current pricing tier.
    

RELATED LINKS
    http://docs.nexosis.com/clients/powershell

REMARKS
    To see the examples, type: "get-help Get-NexosisAccountQuotas -examples".
    For more information, type: "get-help Get-NexosisAccountQuotas -detailed".
    For technical information, type: "get-help Get-NexosisAccountQuotas -full".
    For online help, type: "get-help Get-NexosisAccountQuotas -online"
```

Here's an example of a quick program that submits a dataset from a CSV file to the Nexosis API, creates a forecast session, and retrieves the results.

```powershell
PS> Import-NexosisDataSetFromCsv `
    -dataSetName 'widget-sales' `
    -csvFilePath 'sales-file.csv'

PS> $session = Start-NexosisForecastSession `
        -dataSetName 'widget-sales' `
        -targetColumn 'daily_transaction' `
        -startDate 2017-12-12 `
        -endDate 2017-12-22 `
        -resultInterval Day 

PS> $results = Get-NexosisSessionResults -sessionId $session.SessionID
```

## Detailed Usage

To get started, first read [How It Works](http://docs.nexosis.com/guides/how-it-works). That will help you understand the basic concept of submitting data, running sessions, and retrieving results. 


### DataSets
A DataSet in the Nexosis API contains data that is fed into the machine learning algorithms to create predictions. 

Creating a dataset is easy. Simply create the proper object in Powershell, shown below, containing at LEAST two columns - a `timestamp` and a target to be predicted. In this example I have two data points - daily sales totals and daily number of transactions.

```powershell
 $data = @(
    @{
        timestamp = "2013-01-01T00:00:00+00:00"
        sales = "1500.56"
        transactions = "195.0"
    },
    @{
        timestamp = "2013-01-02T00:00:00+00:00"
        sales = "4078.52"
        transactions = "696.0"
    },
    @{
        timestamp = "2013-01-03T00:00:00+00:00"
        sales = "4545.69"
        transactions = "743.0"
    },
    @{
        timestamp = "2013-01-04T00:00:00+00:00"
        sales = "4872.63"
        transactions = "797.0"
    },
    @{
        timestamp = "2013-01-05T00:00:00+00:00"
        sales = "2420.81"
        transactions = "367.0"
    }
    #... etc
) 
```
Once the DataSet object is created, a call to `New-NexosisDataSet` passing in a name and the dataset object like so:

```powershell
PS> New-NexosisDataSet -dataSetName 'myNewDataSetv1' -data $data

dataSetName    columns                             
-----------    -------                             
myNewDataSetv1 @{sales=; timestamp=; transactions=}
```

This will return an object with the DataSetName as well as the Columns Metadata, created automatically.

Columns MetaData can be used to help the platform make smart decisions about your data. You can indicate data types, imputation, and aggregation strategies.

[You can read more about Columns MetaData in the Nexosis API Documentation here.](http://docs.nexosis.com/guides/column-metadata)

Here's an example of how Columns Metadata should be structured when submitted a new DataSet or Session (see more on sessions below):

```powershell
 $columns = @{
    timestamp = @{
        dataType = "date"
        role = "timestamp"
        imputation = "zeroes"
        aggregation = "sum"
    }
    sales = @{
        dataType = "numeric"
        role = "target"
        imputation = $null
        aggregation = $null
    }
    transactions = @{
        dataType = "numeric"
        role = "none"
        imputation = "zeroes"
        aggregation = "sum"
    }
}
```
### Imports
Since DataSets can be large, it might be more efficient to store it in S3 and request the Nexosis API platform import directly, like so:

```powershell
 PS> Import-NexosisDataSetFromS3 `
        -dataSetName 'salesdata' `
        -S3BucketName 'nexosis-sample-data' `
        -S3BucketPath 'LocationA.csv' `
        -S3Region 'us-east-1'
```

### Views

Views are data sources created by mixing two or more other datasets, analogous to a Database View combining multiple tables together into a single set. For example, by using a view, you can create a session based on a dataset of observations mixed with another dataset containing the features.

[For a more detailed explanation of the Views concept, read the Nexosis API documentation on Views](http://docs.nexosis.com/guides/views)

To construct a view in PowerShell, use the `New-NexosisView` command and specify a View Definition, like so:

```powershell
PS> $joins = @(
        @{
            dataSetName="promoData"
            columnOptions = @{
                timestamp=@{
                    joinInterval="Day"
                    alias="promoDate"
                }
                isPromo=@{
                    alias="promo"
                }
            }
        }
    )

PS> New-NexosisView -viewName 'SalesWithPromosView' -dataSetName 'salesData' -joins $joins
```

The Primary dataset `salesData` is specified by the `-dataSetName` parameter above. The joining table(s) are specified in the `$joins` definition.  Notice how columns can be aliased - but they do not have to be. Additionally for `timestamp` data types, you can specify the `joinInterval` - like `Day` in this example. This is also optional.

Once you create a view, you can use the `Get-NexosisView` command to show the view definition. TO see what the data looks like, you can run the `Get-NexosisViewData` to see how the joined datasets look.

### Sessions
Once datasets or views have been created, a Session can be created to perform a predictive task on that data using `Start-NexosisForecastSession` or `Start-NexosisImpactSession`.

[For language agnostic details on creating Forecast sessions, read more in the Nexosis API documentation](http://docs.nexosis.com/guides/forecasting-walkthrough).

To create a Forecast on the DataSource (a dataSet or View) named 'salesData', targeting Daily forecast on column `sales` starting on `01-06-2013` through `01-13-2013`, you would type the following command to start the session.

```powershell
PS> Start-NexosisForecastSession `
        -dataSourceName 'salesdata' `
        -targetColumn 'sales' `
        -startDate 2013-01-06 `
        -endDate 2013-01-13 `
        `resultInterval Day

sessionId       : 015e2995-a5f5-485d-b614-56294aa4abd8
type            : forecast
status          : started
requestedDate   : 2017-08-28T16:03:46.805531+00:00
statusHistory   : @{date=2017-08-25T22:19:39.025329+00:00; status=requested}, @{date=2017-08-25T22:19:42.8003049+00:00; status=started}}
extraParameters : 
messages        : 
columns         : @{sessions=; timestamp=}
dataSourceName  : salesdata
targetColumn    : sales
startDate       : 2013-01-06T00:00:00+00:00
endDate         : 2017-01-13T00:00:00+00:00
callbackUrl     : 
resultInterval  : day
links           :
```

To measure daily impact of a promotion on Sales between `01-03-2013` and `01-10-2013` you would type, to start the session:

```powershell
Start-NexosisImpactSession `
            -dataSourceName 'salesdata' `
            -eventName 'promo-impact' `
            -targetColumn 'sales' `
            -startDate 2013-01-03 `
            -endDate 2013-01-10 `
            -resultInterval Day

sessionId       : 015e1b7a-aa3d-4c8b-9f15-90bded3308ac
type            : impact
status          : started
requestedDate   : 2017-08-28T16:03:46.805531+00:00
statusHistory   : @{date=2017-08-25T22:19:39.025329+00:00; status=requested}, @{date=2017-08-25T22:19:42.8003049+00:00; status=started}}
extraParameters : 
messages        : 
columns         : @{sessions=; timestamp=}
dataSourceName  : salesdata
targetColumn    : sales
startDate       : 2013-01-03T00:00:00+00:00
endDate         : 2013-01-10T00:00:00+00:00
callbackUrl     : 
resultInterval  : day
links           :
```

An object containing information about the session is returned, such as the `sessionID` which can be used to check the status and return the Session Results when completed.

To check on the status of these jobs, you would use the `Get-NexosisSessionStatus` function which will return one of the follow status messages `Started`, `Requested`, `Completed`, `Cancelled`, or `Failed`.

```powershell
PS> Get-NexosisSessionStatus -sessionID '015e2995-a5f5-485d-b614-56294aa4abd8'
Started
```

To get more detailed information on a Session, you can call `Get-NexosisSessionStatusDetail`:

```powershell
Get-NexosisSessionStatusdetail -sessionId '015e2995-a5f5-485d-b614-56294aa4abd8'

sessionId       : 015e2995-a5f5-485d-b614-56294aa4abd8
type            : forecast
status          : started
requestedDate   : 2017-08-28T16:03:46.805531+00:00
statusHistory   : {@{date=2017-08-28T16:03:46.805531+00:00; status=requested}, @{date=2017-08-28T16:03:48.3227156+00:00; status=started}}
extraParameters : 
messages        : 
columns         : @{sessions=; timestamp=}
dataSourceName  : websiteTraffic
dataSetName     : websiteTraffic
targetColumn    : sessions
startDate       : 2017-08-25T00:00:00+00:00
endDate         : 2017-09-10T00:00:00+00:00
callbackUrl     : 
resultInterval  : hour
links           : 

```

Messages will contain information about the session run, including any failure messages if `status` is `failure`.

Finally once the Status is `completed` the Session results can be retrieved using `Get-NexosisSessionResult` which will contain the details of the job as well as the forecasted data:

```powershell
PS> $sessionResults = Get-NexosisSessionResult -SessionId '015e1b6d-9d37-4111-8568-8fc0f19992ac'
PS> $sessionResults.data

timeStamp                    sales           
---------                    -----           
2017-03-25T00:00:00.0000000Z 73
2017-03-25T00:00:00.0000000Z 71
2017-03-25T00:00:00.0000000Z 75
2017-03-25T00:00:00.0000000Z 79
2017-03-25T00:00:00.0000000Z 81
...
```
### Issues

If you run into issues using this client library, create a [new issue](https://github.com/Nexosis/nexosisclient-ps/issues/new){:target="_blank"} in GitHub. Please include code to reproduce the error if possible.

Pull requests are welcome.
