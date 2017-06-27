---
title: API Keys
description: API Keys are used to gain access to your Sessions in 
copyright: 2017 Nexosis 
layout: default
category: Security
tags: [API Keys, keys, security, Favorite]
use_codestyles: true
---

Handling API key's securely is an important part of any application design. This article discusses what API keys are and some good Key handling practices.

[You can see and manage your API key's in your profile here.](https://developers.nexosis.com/developer)

------
An API key is a unique security ID that is used to identify who is accessing the API as well as a set of access rights associated with it. You can find your API key under your account Profile. These keys are generated and provided to every subscription and is how billing is accounted for and applied to an account. This means that an API key must be kept a closely guarded secret and care must be taken to protect where they are stored. 

``` bash
PUT https://ml.nexosis.com/v1/data/salesdata HTTP/1.1
Content-Type: application/json
accept: application/json
api-key: {your nexosis apikey here}

{redacted}
```

<p align="center"><em><strong>The API key must be submitted as an HTTP Header named <code>api-key</code> in every request.</strong></em></p>

## Protect Your Key

Each account will always have two concurrent active API keys, designated as <code>Primary</code> and <code>Secondary</code>. In certain situations, it may be necessary to discard one of these keys and create a new one -- for example, if a developer accidently commits their Nexosis API key to a public source code repo for all the world to see. For situations like these, one or both of the API keys can be regenerated.  Activating a new key will immediately supplant the old key, rendering it useless. Any program using the old revoked key will then fail with 401 Unauthorized errors from the API endpoint.

<p align="center">  <img alt="API Keys" src="/assets/img/api_keys.png"/><br/>
<strong><em>Click <code>Regenerate</code> to revoke the old key and generate a new one.</em></strong></p>

### Process for API Key Rotation

In this example, assume the application is using the key named <code>Primary Key</code> and you suspect it has been compromised and it needs regenerated.

1. Regenerate <code>Secondary Key</code> as this ensures it's new and secure.
2. Change all applications using Nexosis API to use newly regenerated <code>Secondary Key</code>.
3. Validate all applications using <code>Secondary Key</code> are functioning correctly. Once tested thoroughly, release to production.
4. Finally, regenerate <code>Primary Key</code> to make sure any unauthorized users cannot use this key to access the application.

### When To Regenerate a Key

There are important thinks to keep a watch out for that should require a key to be regenerated.

<b>REGENERATE YOUR KEY IF:</b>
* API key is exposed publicly, such as a source code repo or in HTML or JavaScript source code.
* A computer with the key stored on it was stolen or comprised.
* You suspect unauthorized key use.
* A developer with access to the Primary Key has left your company.
* Someone gained unauthorized access to your Nexosis Account.
* When a key is old. The older a key is, the higher the probability it could have been compromised.

### Storing and Protecting API Keys

Your Nexosis API keys should be closely guarded and protected from discovery.  Make a plan and automate it. Protecting secrets in an application can be challenging without proper planning. A key change-over should be simple. Here are some guiding principals that will help.

* Do not store keys in code.
* Do not store the API Key in the application source repository like git or svn.
* Encrypt the key when storing it.
* Store it encrypted in a configuration file.
* Store it encrypted in a password manager.
* Store it encrypted using a Hardware Security Module (HSM) or Key Manager.
* Only regenerate one key at a time to prevent downtime. 
* Regenerate keys at regularly scheduled intervals.