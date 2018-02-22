---
title: Python Module
description: Python module for using the Nexosis API
category: Python
breadcrumb: API Clients
tags: [python, module]
use_codestyles: true
---

## Installing the client

#### The client has been tested against Python 2.7, and 3.3-3.6. Install with `pip`.

```bash
$ pip install nexosisapi
```

> <p><a href="https://pypi.python.org/pypi/nexosisapi" class="btn btn-primary mr10" target="_blank"><i class="fa fa-cube mr5"></i> PyPI Package Details</a><a href="https://github.com/Nexosis/nexosisclient-py" class="btn btn-primary" target="_blank"><i class="fa fa-github mr5"></i> View Source</a></p>

## Basic Usage

In the following example we'll assume you have downloaded our sample transactions [file called 'LocationA.csv'](https://raw.githubusercontent.com/Nexosis/sampledata/master/LocationA.csv){:target="_blank"} to a local drive. You can load data from that CSV file and create forecasts for the 'transactions' data. You should be able to
get the results with just a few lines of code.

```python
import nexosisapi
import dateutil.parser as date_parser

client = nexosisapi.Client('your api key here')

with open('yourlocaldrive/LocationA.csv') as f:
    result = client.datasets.create_csv('widget-sales', f)

session = client.sessions.create_forecast('widget-sales', 'transactions',
        date_parser.parse('2017-01-22 00:00:00 -0:00'), date_parser.parse('2017-02-22 00:00:00 -0:00'))
# after some time passes you can get results...
results = client.sessions.get_results(session.session_id)
```

Your `results` variable is an object with a `data` property which is a `list` of `dict` of the
timestamped forecast values.

## Detailed Usage

The `Client` object has three properties on it which you will use to do the majority of your work
with the Nexosis API. These properties are `datasets`, `sessions`, and `imports`. You will need to
use the first two to get much of anything done, and will probably want to use the third once you
have a system in place to run regular analysis. Each of these properties will be covered in the
following sections.

### Datasets

A dataset in the Nexosis API represents the input to the algorithms. To create a dataset, you can
send a `list` of `dict` containing the data, or read from a CSV file.

```python
# `data` should be a list of dict with the values to store
client.datasets.create('coffee-consupmption', data)

# alternatively, a CSV file can be used
with open('cups_of_coffee.csv') as f:
    client.datasets.create('coffee-consumption', f)
```

When creating a dataset, you can specify `ColumnMetadata` that tells the system how to interpret the
data. See the [Column Metadata guide](/guides/column-metadata) for more information about what that
means to the API.

Once data is saved, you can list all the datasets stored by the API:

```python
datasets = client.datasets.list()
```

Or you can delete a dataset when it is no longer needed.

```python
# example cleaning all datasets, using `datasets` from the call to `list()` above:
[client.datasets.remove(d.name) for d in datasets]
```

### Sessions

A session is used to run an analysis in the Nexosis API. You will need to have previously saved data
in a dataset to create a session. Conceptually, a session is the handle that is used to reference
the results the analysis you created with the API. There are two types of sessions: forecast and
impact. 

Start a forecast session with the `create_forecast` method:

```python
# predict the number of cups of coffee per day in December 2017 (assuming the 'coffee-consumption`
#  dataset already exists)
start_date = datetime.datetime.date(2017, 12, 1)
end_date = datetime.datetime.date(2017, 12, 31)

client.sessions.create_forecast('coffee-consumption', 'cups', start_date, end_date,
    result_interval=TimeInterval.Day)
```

Or look at the impact of a past event with the `analyze_impact` method:

```python
# look at the change in cups of coffee consumed during a week of vacation 
start_date = datetime.datetime.date(2017, 7, 1)
end_date = datetime.datetime.date(2017, 7, 7)

client.sessions.analyze_impact('coffee-consumption', 'cups', 'vacation', start_date, end_date,
    result_interval=TimeInterval.Week)
```

### Imports

You may run into issues when uploading data to the Nexosis API as there are limits on request sizes.
Additionally, you may already have data stored in other places like Amazon's S3. If that is the
case, then the import portion of the service will work well for you. Importing data from a remove
server on the internet is as simple as a single call to the API.

```python
import_response = client.imports.import_from_s3('test-python-import', 'sample-data', 'some-file.csv', 'us-east-1')
```

This call starts the import of the file from the given S3 bucket ('us-east-1' region, 'sample-data'
bucket) into the 'test-python-import' dataset in the Nexosis API. 


### Issues
If you run into issues using this client library, create a [new issue](https://github.com/Nexosis/nexosisclient-py/issues/new){:target="_blank"} in GitHub. Please include code to reproduce the error if possible.

#### Pull requests are welcome.

> <strong>See Also:</strong> [A video tutorial on using the python client](https://content.nexosis.com/blog/exploring-the-nexosis-api-with-ipython){:target="_blank"}
