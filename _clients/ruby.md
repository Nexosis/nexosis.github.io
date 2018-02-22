---
title: Ruby GEM
description: Ruby GEM for using the Nexosis API
category: Ruby
tags: [ruby, gem]
use_codestyles: true
---

## Installing the client

#### Include the gem in your gemfile and use bundler to install your dependencies.

``` Ruby
gem 'nexosis_api', '>= 2.0.0'
```
> <p><a href="https://rubygems.org/gems/nexosis_api" class="btn btn-primary mr10" target="_blank"><i class="fa fa-cube mr5"></i> Gem Details</a><a href="https://github.com/Nexosis/nexosisclient-rb" class="btn btn-primary" target="_blank"><i class="fa fa-github mr5"></i> View Source</a></p>

## Ruby Quick Start

### Initialize the Client

The <code>NexosisApi</code> class exposes the client as a class method. The client will be cached across uses so you can initalize it once with the API Key:

``` Ruby
require 'nexosis_api'
NexosisApi.client(:api_key => 'your key here') 
```

In most cases you'll want to load the key as a secret or from the environment so as not to expose it in your source code.

### Repeated Calls
Once you have the client initialized you can call again without specifying the key

``` Ruby
balance = NexosisApi.client.get_account_balance
```

### Issues
If you run into issues using this client library, create a [new issue](https://github.com/Nexosis/nexosisclient-rb/issues/new){:target="_blank"} in github. Please include code to reproduce the error if possible.