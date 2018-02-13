---
title: Node.JS Module
description: Using NodeJS with the Nexosis API.
category: Node.js
breadcrumb: API Clients
tags: [node, nodejs, npm]
use_codestyles: true
---
> <h5 class="mt0">Supported platforms</h5>
NodeJs.  This package will not yet work in a web browser.

## Installing the client

> <p><a href="https://www.npmjs.com/package/nexosis-api-client" class="btn secondary mr10" target="_blank"><i class="fa fa-cube mr5"></i> Package Details</a><a href="https://github.com/Nexosis/nexosisclient-js" class="btn secondary" target="_blank"><i class="fa fa-github mr5"></i> View Source</a></p>


Either install directly into your application at the command line:

``` bash
npm install nexosis-api-client
```

or add to your package dependencies and then run npm install:

``` javascript
"nexosis-api-client": ">=1.0.0"
```

## Node Quick Start

### Initialize the Client
You'll need your api key to pass to the constructor. If you follow the sample below, it has been added as an environment variable on the host first.

``` javascript
var client = require('nexosis-api-client').default;
var nexosisClient = new client({ key: process.env.NEXOSIS_API_KEY });
```
Once you have the client, you can interact with Sessions, DataSets, and Imports by accessing a property on the client...
``` javascript
nexosisClient.DataSets.get("mydataset");
nexosisClient.Sessions.get("b19d17da-ec5c-4de8-81af-53b4bf486bae");
nexosisClient.Imports.list();
```

In practice, each of the operations returns a Promise. To access the session data returned from a get for example you would write:
``` javascript
nexosisClient.Sessions
.get("b19d17da-ec5c-4de8-81af-53b4bf486bae")
.then(
    (data) => { console.log(data); }
);
```

#### To get started with NodeJS and the Nexosis API, [view the source code](https://github.com/Nexosis/nexosisclient-js){:target="_blank"}

Pull requests are welcome.

### Issues
If you run into issues using this client library, create a [new issue](https://github.com/Nexosis/nexosisclient-js/issues/new){:target="_blank"} in github. Please include code to reproduce the error if possible.