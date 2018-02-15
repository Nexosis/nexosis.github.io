---
title: Detecting Hackers Like It's 1999
description: Using network data to classify network attacks 
category: Classification
tags: [classification, security, reference]
use_codestyles: true
order: 4
---

<style>
table{
    border-collapse: collapse;
    border-spacing: 0px;
    border:2px solid #000000;
}

th{
    border:2px solid #000000;
    padding: 4px;
}

td{
    border:1px solid #000000;
    padding: 4px;
}
</style>

This tutorial will  using the Nexosis API to classify malicious network traffic using prepared network data captured from a Defense Department Network while it was under attack. Given this data, we'll show how Machine Learning can be used to build a model that will classify network data as malicious or normal based on it's characteristics.

**Time:** 10 minutes<br/>
**Level:** Introductory / 101

#### **Prerequisites:**
* [Postman](https://www.getpostman.com){:target="_blank"}
* [Nexosis API key](https://developers.nexosis.com/developer){:target="_blank"}

------
![hackers-movie-poster.jpeg](/assets/img/tutorials/hackers-movie-poster.jpeg){:.img-responsive}

Obligatory `Hackers` movie poster - &copy;1995 United Artists

## Outline

Understanding the problem:
1. [Initial understanding](#initial-understanding)
2. [Data Sets](#datasets)
   * [Training Data](#training-data)
   * [Test Data](#test-data)
3. [Submitting the Data to Nexosis](#submitting-the-data)
4. [Building the Model](#building-the-model)
5. [Results](#results)
    * [Understanding the Result](#understanding-results)
    * [How to use the results](#how-to-use-results)
    * [Batch Predictions](#batch-predictions)
6. [Next Steps](#next-steps)

## Problem Definition

<h3 id="initial-understanding" class="jumptarget">Initial Understanding</h3>

> "This is the data set used for The Third International Knowledge Discovery and Data Mining Tools Competition, which was held in conjunction with KDD-99 The Fifth International Conference on Knowledge Discovery and Data Mining."

The abstract describes the goal of the competition as follows:

>"The competition task was to build a network intrusion detector, a predictive model capable of distinguishing between 'bad' connections, called intrusions or attacks, and 'good' normal connections. This database contains a standard set of data to be audited, which includes a wide variety of intrusions simulated in a military network environment."

*See [KDD Cup 1999 Data](http://kdd.ics.uci.edu/databases/kddcup99/kddcup99.html){:target="_blank"} and [Intrustion Detection Learning](http://kdd.ics.uci.edu/databases/kddcup99/task.html){:target="_blank"} for more details, including downloads of the prepared data, task description and data dictionary.*

The initial program used to create the data for this competition was prepared and managed by MIT Lincoln Labs. They setup an environment that captured raw network traffic (TCP dump data) for a network simulating a typical U.S. Air Force LAN for 9 weeks. While capturing data, they hit it with multiple attacks. They collected about 4 GB of the network data, compressed and then transformed it into almost 5 million connection records and did some post processing to make sure the records contained some information that would be useful to the model.

<h3 id="datasets" class="jumptarget">DataSets</h3>

The data gathered fell into three different categories:
1. Basic features of individual TCP connections 
2. Content features within a connection suggested by domain knowledge.
3. Traffic features computed using a two-second time window.

Here's a small selection of some of the data in the CSV file:
```csv
duration,protocol_type,service,flag,src_bytes,dst_bytes,land,wrong_fragment,urgent,hot,num_failed_logins,logged_in,num_compromised,root_shell,su_attempted,num_root,num_file_creations,num_shells,num_access_files,num_outbound_cmds,is_host_login,is_guest_login,count,srv_count,serror_rate,srv_serror_rate,rerror_rate,srv_rerror_rate,same_srv_rate,diff_srv_rate,srv_diff_host_rate,dst_host_count,dst_host_srv_count,dst_host_same_srv_rate,dst_host_diff_srv_rate,dst_host_same_src_port_rate,dst_host_srv_diff_host_rate,dst_host_serror_rate,dst_host_srv_serror_rate,dst_host_rerror_rate,dst_host_srv_rerror_rate,type
0,tcp,http,SF,215,45076,0,0,0,0,0,1,0,0,0,0,0,0,0,0,0,0,1,1,0.00,0.00,0.00,0.00,1.00,0.00,0.00,0,0,0.00,0.00,0.00,0.00,0.00,0.00,0.00,0.00,normal
0,tcp,http,SF,162,4528,0,0,0,0,0,1,0,0,0,0,0,0,0,0,0,0,2,2,0.00,0.00,0.00,0.00,1.00,0.00,0.00,1,1,1.00,0.00,1.00,0.00,0.00,0.00,0.00,0.00,normal
0,tcp,http,SF,236,1228,0,0,0,0,0,1,0,0,0,0,0,0,0,0,0,0,1,1,0.00,0.00,0.00,0.00,1.00,0.00,0.00,2,2,1.00,0.00,0.50,0.00,0.00,0.00,0.00,0.00,normal
21,tcp,ftp,SF,89,345,0,0,0,1,0,1,0,0,0,0,1,0,0,0,0,0,1,1,0.00,0.00,0.00,0.00,1.00,0.00,0.00,255,1,0.00,0.02,0.00,0.00,0.00,0.00,0.00,0.00,rootkit
98,tcp,telnet,SF,621,8356,0,0,1,1,0,1,5,1,0,14,1,0,0,0,0,0,1,1,0.00,0.00,0.00,0.00,1.00,0.00,0.00,255,4,0.02,0.02,0.00,0.00,0.00,0.00,0.00,0.00,rootkit
0,tcp,ftp_data,SF,0,5636,0,0,0,0,0,1,2,0,0,2,0,0,0,0,0,0,1,1,0.00,0.00,0.00,0.00,1.00,0.00,0.00,1,41,1.00,0.00,1.00,0.10,0.00,0.00,0.00,0.00,rootkit
61,tcp,telnet,SF,294,3929,0,0,0,0,0,1,0,1,0,4,1,0,0,0,0,0,1,1,0.00,0.00,0.00,0.00,1.00,0.00,0.00,255,4,0.02,0.02,0.00,0.00,0.00,0.25,0.73,0.25,rootkit
0,tcp,imap4,REJ,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,239,15,0.00,0.00,1.00,1.00,0.06,0.06,0.00,255,15,0.06,0.08,0.00,0.00,0.00,0.00,1.00,1.00,neptune
0,tcp,imap4,REJ,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,240,16,0.00,0.00,1.00,1.00,0.07,0.06,0.00,255,16,0.06,0.08,0.00,0.00,0.00,0.00,1.00,1.00,neptune
0,tcp,imap4,REJ,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,243,17,0.00,0.00,1.00,1.00,0.07,0.06,0.00,255,17,0.07,0.07,0.00,0.00,0.00,0.00,1.00,1.00,neptune
0,icmp,eco_i,SF,8,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,10,0.00,0.00,0.00,0.00,1.00,0.00,1.00,4,204,1.00,0.00,1.00,0.25,0.00,0.00,0.00,0.00,nmap
0,icmp,eco_i,SF,8,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,6,0.00,0.00,0.00,0.00,1.00,0.00,1.00,1,205,1.00,0.00,1.00,0.25,0.00,0.00,0.00,0.00,nmap
0,icmp,eco_i,SF,8,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,12,0.00,0.00,0.00,0.00,1.00,0.00,1.00,2,206,1.00,0.00,1.00,0.25,0.00,0.00,0.00,0.00,nmap
0,tcp,other,REJ,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,510,1,0.22,0.00,0.78,1.00,0.00,1.00,0.00,255,1,0.00,1.00,0.00,0.00,0.25,0.00,0.75,1.00,satan
0,tcp,other,REJ,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,510,1,0.22,0.00,0.78,1.00,0.00,1.00,0.00,255,1,0.00,1.00,0.00,0.00,0.25,0.00,0.75,1.00,satan
0,tcp,other,REJ,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,510,1,0.22,0.00,0.78,1.00,0.00,1.00,0.00,255,1,0.00,1.00,0.00,0.00,0.25,0.00,0.75,1.00,satan
```

Here is the data dictionary of each type of features in the dataset:

**feature name** | **description** | **type**
--- | --- | ---
duration | length (number of seconds) of the connection | continuous
protocol_type | type of the protocol, e.g. tcp, udp, etc. | discrete
service | network service on the destination, e.g., http, telnet, etc. | discrete
src_bytes | number of data bytes from source to destination | continuous
dst_bytes | number of data bytes from destination to source | continuous
flag | normal or error status of the connection | discrete 
land | 1 if connection is from/to the same host/port; 0 otherwise | discrete
wrong_fragment | number of ``wrong'' fragments | continuous
urgent | number of urgent packets | continuous

* **Table 1:** Basic features of individual TCP connections.*

**feature name** | **description** | **type**
--- | --- | ---
hot | number of 'hot' indicators | continuous
num_failed_logins | number of failed login attempts | continuous
logged_in | 1 if successfully logged in; 0 otherwise | discrete
num_compromised | number of 'compromised' conditions | continuous
root_shell | 1 if root shell is obtained; 0 otherwise | discrete
su_attempted | 1 if 'su root' command attempted; 0 otherwise | discrete
num_root | number of 'root' accesses | continuous
num_file_creations | number of file creation operations | continuous
num_shells | number of shell prompts | continuous
num_access_files | number of operations on access control files | continuous
num_outbound_cmds | number of outbound commands in an ftp session | continuous
is_hot_login | 1 if the login belongs to the ``hot'' list; 0 otherwise | discrete
is_guest_login | 1 if the login is a ``guest''login; 0 otherwise | discrete

***Table 2:** Content features within a connection suggested by domain knowledge.* 

**feature name** | **description** | **type**
--- | --- | ---
count | number of connections to the same host as the current connection in the past two seconds | continuous
 | **Note:** The following  features refer to these same-host connections.	| 
serror_rate | % of connections that have ``SYN'' errors | continuous
rerror_rate | % of connections that have ``REJ'' errors | continuous
same_srv_rate | % of connections to the same service | continuous
diff_srv_rate | % of connections to different services | continuous
srv_count | number of connections to the same service as the current connection in the past two seconds | continuous
 | **Note:** The following features refer to these same-service connections. | 
srv_serror_rate | % of connections that have ``SYN'' errors | continuous
srv_rerror_rate | % of connections that have ``REJ'' errors | continuous
srv_diff_host_rate | % of connections to different hosts | continuous 

***Table 3:** Traffic features computed using a two-second time window.*

The above data dictionaries are defined here: [Intrustion Detection Learning](http://kdd.ics.uci.edu/databases/kddcup99/task.html)

The last bit of information required to build a model for classification is labled data. Labeled data is simpley tagging each feature as `normal` or assigned the type of malicious attack it is. Here are the different labels that are provided in the datasets from the KDD Cup 1999 dataset. The DataSets contain the specific attack, and each specific attack can be grouped into a more general category.

There are four general attack categories, and one to represent normal traffic:
* Denial of Service (dos)
* User-to-Root (u2r)
* Remote-to-Local (r2l)
* Probe
* Normal

Here are all the specific attack types and what general category they fall under.

Specific Attack Type | General Attack Category
--- | ---
back | dos
buffer_overflow | u2r
ftp_write | r2l
guess_passwd | r2l
imap | r2l
ipsweep | probe 
land | dos
loadmodule | u2r
multihop | r2l
neptune | dos
nmap | probe
perl | u2r
phf | r2l
pod | dos
portsweep | probe
rootkit | u2r
satan | probe
smurf | dos
spy | r2l
teardrop | dos
warezclient | r2l
warezmaster | r2l
normal | normal (normal traffic, no attack)

***Table 3:** Of Attack Types / Classes.*

The labeled dataset provided has each row labeled with the specific attack type, but not the General attack cateogry meaning we'll need to append that column to each row.

<h4 id="training-data" class="jumptarget">Training Data</h4>

There are many files with data provided by the competition which can lead to some confusion around which ones to use. There is full datasets with and withou labels, 10% of the dataset with and without labels, as well as test data. Additionally there were some mistakes in the dataset which were subsequently fixed in the original files, which are also available.

Since we're building a model, we'll want to use the labeled datasets so we can eliminate using anything unlabeled. Additionaly, we can ignore any test data since that will be used to check the model after it's built, leaving the following two files to choose from:

* [kddcup.data.gz](http://kdd.ics.uci.edu/databases/kddcup99/kddcup.data.gz){:target="_blank"}. - The full data set, with labels (18M; 743M Uncompressed)
* [kddcup-data_10_percent.gz](http://kdd.ics.uci.edu/databases/kddcup99/kddcup.data_10_percent.gz){:target="_blank"}. - A 10% subset, with labels. (2.1M; 75M Uncompressed)

> The Nexosis API will automatically split the training data into internal train and test datasets (80% to train, and 20% test cases) when building and tuning its models. Subsequently, it's common to hold back even more test data to validate that the model works with other data it's never seen before.

<h4 id="test-data" class="jumptarget">Test Data</h4>

Now that the training dataset has been selected, an additional test dataset can used to further validate if the model is working as it should. A test dataset must also be labeled so the predicted value and the actual value can be compared. For the competition, teams would use an unlabeled test datasets and submit the predictions and the judges would then compare the predictions to the actuals (just like Kaggle competitions work) so teams cannot cheat by having the answers by biasing the model towards the test set. Since the competition is over, a labeled test set has been released so anyone can validate their test set against the answer key. This dataset is the one we can use to measure how our model performs:

* [corrected.gz](http://kdd.ics.uci.edu/databases/kddcup99/corrected.gz){:target="_blank"}. - Corrected Test data, with labels. (1.4M Compressed, 46M Uncompressed)

