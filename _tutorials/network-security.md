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

**Time:** 20 minutes<br/>
**Level:** Intermediate / 201

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

This tutorial will walk through a Data Science competition called the [KDD Cup](http://www.kdd.org/kdd-cup){:target="_blank"} from [1999](http://www.kdd.org/kdd-cup/view/kdd-cup-1999){:target="_blank"}.

> "This is the data set used for The Third International Knowledge Discovery and Data Mining Tools Competition, which was held in conjunction with KDD-99 The Fifth International Conference on Knowledge Discovery and Data Mining."

The abstract describes the goal of the competition as follows:

>"The competition task was to build a network intrusion detector, a predictive model capable of distinguishing between 'bad' connections, called intrusions or attacks, and 'good' normal connections. This database contains a standard set of data to be audited, which includes a wide variety of intrusions simulated in a military network environment."

*See [KDD Cup 1999 Data](http://kdd.ics.uci.edu/databases/kddcup99/kddcup99.html){:target="_blank"} and [Intrustion Detection Learning](http://kdd.ics.uci.edu/databases/kddcup99/task.html){:target="_blank"} for more details, including downloads of the prepared data, task description and data dictionary.*

The initial program used to create the data for this competition was prepared and managed by MIT Lincoln Labs. They setup an environment that captured raw network traffic (TCP dump data) for a network simulating a typical U.S. Air Force LAN for 9 weeks. While capturing data, they hit it with multiple attacks. They collected about 4 GB of the network data, compressed and then transformed it into almost 5 million connection records and did some post processing to make sure the records contained some information that would be useful to detecting attackers.

<h3 id="datasets" class="jumptarget">DataSets</h3>

The data gathered fell into three different categories:
1. Basic features of individual TCP connections 
2. Content features within a connection suggested by domain knowledge.
3. Traffic features computed using a two-second time window.

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

***Table 1:** Basic features of individual TCP connections.*

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

The last bit of information required to build a model for classification is labled data. Labeled data is simpley tagging each row of data as `normal` or assigning it the name of malicious attack it is (e.g. `nmap`, `neptune`, `warezclient`, `teardrop`, etc). Below are the different labels provided in the datasets from the KDD Cup 1999 dataset. The DataSets contain the specific attack, and each specific attack can be grouped into a more general category.

There are four general attack categories, and one to represent normal traffic:
* Denial of Service (`dos`)
* User-to-Root (`u2r`)
* Remote-to-Local (`r2l`)
* Probe (`probe`)
* Normal (`normal`)

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

There are many files provided by the competition which can lead to some confusion about which ones to use. There are full datasets with and without labels, 10% dataset with and without labels, as well as test data. Additionally there were some mistakes in the dataset which were subsequently fixed in the original files, which are also available.

Since we're building a Classification model, we'll want to use a labeled dataset so we can eliminate any dataset with unlabeled data. Additionaly, we can ignore any test data since that will be used to check the model after it's built. This leaves the following two files to choose from:

* [kddcup.data.gz](http://kdd.ics.uci.edu/databases/kddcup99/kddcup.data.gz){:target="_blank"}. - The full data set, with labels (18M; 743M Uncompressed)
* [kddcup-data_10_percent.gz](http://kdd.ics.uci.edu/databases/kddcup99/kddcup.data_10_percent.gz){:target="_blank"}. - A 10% subset, with labels. (2.1M; 75M Uncompressed)

> The Nexosis API will automatically split the training data into internal train and test datasets (80% to train, and 20% test cases) when building and tuning its models. Subsequently, it's common to hold back even more test data to validate that the model works with other data it's never seen before.

The full dataset is 743MB which is rather large. It's better to start with a smaller set of data considering more data is more computationally expensive and the accuracy gains deminish after a certain point. We'll choose the 10% dataset to start out and see how well it performs. 75MB is still potentially larger than it needs to be but we'll start there, knowing we can futher reduce it later if desired.

> To read more about finding the idea training dataset size, read [Data Education: The Value of Data Experimentation](https://content.nexosis.com/blog/the-value-of-data-experimentation){:target="_blank"} blog post.

One last catch. The file does not have a CSV header. A CSV header should be added to the top of the csv file. The names of the features, or column headers, are provided in [kddcup.names](http://kdd.ics.uci.edu/databases/kddcup99/kddcup.names){:target="_blank"} file.

#### Prepare Training Data File

Here are the specific steps used to prepare the file to send to the Nexosis API to train a model:

<ol>
    <li> Download the 10% training file <a href="http://kdd.ics.uci.edu/databases/kddcup99/kddcup.data_10_percent.gz" target="_blank">kddcup-data_10_percent.gz</a>. </li>

    <li> Decompress the file using a decompression tool, such as <a href="http://www.7-zip.org/" target="_blank">7-ZIP</a>. The extracted file will be named <code>kddcup.data_10_percent_corrected</code>.</li>

    <li> Create a new file called <code>kddheader.csv</code> and paste the following in it and save it in the same folder with the uncompressed kddcup data file:

    <pre class="language-txt"><code class="language-txt code-toolbar">duration,protocol_type,service,flag,src_bytes,dst_bytes,land,wrong_fragment,urgent,hot,num_failed_logins,logged_in,num_compromised,root_shell,su_attempted,num_root,num_file_creations,num_shells,num_access_files,num_outbound_cmds,is_host_login,is_guest_login,count,srv_count,serror_rate,srv_serror_rate,rerror_rate,srv_rerror_rate,same_srv_rate,diff_srv_rate,srv_diff_host_rate,dst_host_count,dst_host_srv_count,dst_host_same_srv_rate,dst_host_diff_srv_rate,dst_host_same_src_port_rate,dst_host_srv_diff_host_rate,dst_host_serror_rate,dst_host_srv_serror_rate,dst_host_rerror_rate,dst_host_srv_rerror_rate,type
    </code></pre>
    </li>

    <blockquote><b>WARNING:</b> Make sure there is exactly ONE empty line in the <code>kddheader.csv</code> file after the header row or you will end up with the first row on the same line as the headers or some blank rows between the header and the data when the files are combined in the next step.</blockquote>

    <li>Merge the <code>kddheader.csv</code> with the <code>kddcup.data_10_percent</code> file.<br/>
    <br/>
    On Windows, from a command prompt, use the <code>copy</code> command to merge the files: 
    <pre class="language-batch"><code class="language-batch code-toolbar">copy kddheader.csv+kddcup.data_10_percent_corrected kddcup-training.csv</code></pre>
    On Linux or Mac you can use the <code>cat</code> command to merge the files:
    <pre class="language-bash"><code class="language-bash code-toolbar">cat kddheader.csv kddcup.data_10_percent_corrected >  kddcup-training.csv</code></pre>
    </li>

    <li>You should now have a file named <code>kddcup-training.csv</code>. Here's a small random selection of data from the file including the csv header:
<pre class="language-batch"><code class="language-batch code-toolbar">
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
0,tcp,other,REJ,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,510,1,0.22,0.00,0.78,1.00,0.00,1.00,0.00,255,1,0.00,1.00,0.00,0.00,0.25,0.00,0.75,1.00,satan</code></pre>
    </li>
    <li> <b>(Optional)</b> - In the Original CSV, every line in the CSV is terminated with a period. Because of that, all the items in the <code>type</code> column will contain a period (e.g. <code>noraml.</code>, <code>satan.</code>, <code>buffer_overflow.</code>, etc.) If you are on linux or mac, or if you have unix tools installed on windows (i.e. gitbash, cygwin, etc.) you can quickly clean this up using the <code>sed</code> command with a regular expression, like so:
     <pre class="language-bash"><code class="language-bash code-toolbar">sed -i -e 's/\.$//' kddcup-training.csv</code></pre>
    </li>
    <li><b>IMPORTANT CRITICAL FINAL STEP BEFORE SUBMITTING TO NEXOSIS API: Validate the CSV file.</b><br/>
      Notice the odd square on line <code>494023</code> below. 
<img src="/assets/img/tutorials/kddcup-csv-bad-char.png" alt="Bad character in CSV file." class="img-responsive">
This will need removed or the import will fail. An easy way to fix this in bash is with the <code>head</code> command, which removes the last line:
<pre class="language-batch"><code class="language-batch code-toolbar">
head -n -1  kddcup-training.csv > kddcup-training.tmp ; mv kddcup-training.tmp kddcup-training.csv
</code></pre>

    If there are any illeagal characters, jagged rows, unpaired quotations around strings, extra commas, etc. the Import will fail. The Nexosis API will attempt to provide details in Response body as to where the error was encountered. In this case, the file we downloaded has an illegal byte at the end of the file. By opening reviewing the file in Visual Studio Code, Notepad++, or another text editor, it can help identify issues. Excel can help validate there are no jagged rows, but it can also mask other issues as well.

    <blockquote>Avoid using <code>notepad.exe</code> to review CSV files unless the file is very small - small in both number of rows as well as the length of each line. Notepad has proven time and again to be increadibly unreliable in many ways.</blockquote>

    Alternativly, if the file is not too large you can use Notepad++, Visual Studio Code, Atom, etc. to review and make manual tweaks to the file.

    <blockquote>For more information on data munging, read <a href="https://content.nexosis.com/blog/5-text-transforming-techniques-for-total-techies" target="_blank">5 Text Transforming Techniques for Total Techies</a>.</blockquote>
</li>
</ol>

<h4 id="test-data" class="jumptarget">Test Data</h4>

Now that the training dataset has been prepared, an additional test dataset can used to further validate if the model is working as it should. A test dataset must also be labeled so the predicted value and the actual value can be compared. For the competition, teams are given an unlabeled test dataset to perform predictions on and the judges would then compare those predictions to the actuals they kept hidden (just like Kaggle competitions work). This prevents teams from cheating by having the answer by biasing the model towards the the test set used for judging. Since the competition is over, a labeled test set was released so anyone can validate their test set against the answer key. 

This labeled test dataset can be used to measure how our model performs:

* [corrected.gz](http://kdd.ics.uci.edu/databases/kddcup99/corrected.gz){:target="_blank"}. - Corrected Test data, with labels. (1.4M Compressed, 46M Uncompressed)

After the model is built, we can use this dataset to measure our model performance against data it's never seen before. So for now, there's nothing to do with this file other than save it for later. It will be used later to calculate metrics to get a more realistic sense of the accuracy.

<h3 id="submitting-the-data" class="jumptarget">Submitting the Data to Nexosis</h3>

Since the dataset is larger than the maximum POST size allowed by the API, the file will need to be uploaded in batches (1MB at a time), or hosted at a URL.

In this tutorial, [Digital Ocean Spaces](https://www.digitalocean.com/products/spaces/){:target="_blank"} is used for file storage to host the training dataset file. Nexosis API also supports files stored in AWS S3, Azure Blob, or any URL that contains a path to a CSV file.

> For more information on Importing CSV Files, see the documentation on [Importing Data](/guides/importing-data){:target="_blank"} in the documentation to import data from other sources.

1. To create a Space in Digital Ocean, create and account and login. Click `Spaces` link in the nav bar at the top.![Digital Ocean Spaces](/assets/img/tutorials/do-spaces.png){:.img-responsive}

2. Once you are in the Spaces area, click the green `Create` button in the uppper right. Choose type a Space Name, datacenter region, and file listing permission:![DO Spaces: Create a Space](/assets/img/tutorials/do-create-a-space.png){:.img-responsive}

3. Once the space is created, click the blue `Upload Files` button. Upload the `kddcup-training.csv` file. Since there's nothing sensitive in this file, make sure to make it `Publically Readable`.![DO Spaces: Upload File](/assets/img/tutorials/do-spaces-uploadfile.png){:.img-responsive}

4. Wait for the file to finish uploading.![DO Spaces: File Uploading](/assets/img/tutorials/do-uploadfile.png){:.img-responsive}

5. Once the upload has completed, hover over the file and wait for the pop-up. Click the `Copy URL` button to get a public link to the training file.![DO Spaces: Get File URL](/assets/img/tutorials/do-get-url.png){:.img-responsive}

6. The URL should look something like this:<br/> `https://jm0nty-public.nyc3.digitaloceanspaces.com/kddcup-training.csv`

Now that the the large training dataset is in a place where it can be retrieved, the Nexosis API needs instructions to import it. To import a file from Digital Ocean Spaces, we need to send an HTTP `POST` request to `https://ml.nexosis.com/v1/imports/Url`. In the HTTP request body, the following JSON will tell the Nexosis API what URL to retrieve the data from and the name to give the dataset.
```json
{
	"dataSetName": "kddcup-training",
	"url": "https://jm0nty-public.nyc3.digitaloceanspaces.com/kddcup-training.csv"
}
```
Here are the steps to accomplish this using the JSON above:
1. Open Postman, and in a New Request set the HTTP Verb to `POST`, and set the URL to `https://ml.nexosis.com/v1/imports/Url`. Set the `Accept`, `Content-Type`, and `api-key` headers.
![Postman: Set Headers](/assets/img/tutorials/kddcup-import-headers.png){:.img-responsive}
> For information on finding your API key, read [this support article](https://support.nexosis.com/hc/en-us/articles/115002132274-Where-do-I-find-my-API-Key){:target="_blank"}.
2. Click on the request `Body` tab in Postmane and include the `JSON` from above: 
![Postman: Set POST Body](/assets/img/tutorials/kddcup-import-post.png){:.img-responsive}
3. Click `Send` in Postman and receive something similar to the following HTTP response from the API:
![Postman: Response from POST](/assets/img/tutorials/kddcup-import-response.png){:.img-responsive}
> **Important:** Copy the `importId` out of the response body as you will need this in the next step to check on the Import progress.
4. In a new Postman request, send a `GET` request to `https://ml.nexosis.com/v1/imports/{importId}` and replace `{importid}` with the ID  saved from the previous step. 
![Postman: Import GET Request](/assets/img/tutorials/kddcup-get-import.png){:.img-responsive}
5. Keep checking back every few minutes until `"status": "completed"` is reported in the `statusHistory` array in the JSON response.
```JSON
...data elided...
 "statusHistory": [
        {
            "date": "2018-02-16T22:33:56.297532+00:00",
            "status": "requested"
        },
        {
            "date": "2018-02-16T22:33:56.8138766+00:00",
            "status": "started"
        },
        {
            "date": "2018-02-16T22:42:56.3124271+00:00",
            "status": "completed"
        }
    ],
...data elided...
```

Now that the training data has been submitted to the Nexosis API, experimentation with building a model can commence.

<h3 id="building-the-model" class="jumptarget">Building the Model</h3>

1. From the Nexosis API Collections, open the Sessions folder and click `POST /sessions/model`.
 <img src="/assets/img/tutorials/kddcup-postman5.png" alt="Click Send" style="padding: 10px;" class="img-responsive"><br>
 If you don’t have the Nexosis API collections, you can simply select `POST` from the dropdown and type `https://ml.nexosis.com/v1/sessions/model` in the text bar.
 <img src="/assets/img/tutorials/kddcup-postman6.png" alt="Choose the Session Endpoint" style="padding: 10px;" class="img-responsive">
2. Click `Body` and paste the following code. - **Note:** `type` was the column in our dataset that had the network attack type in it (e.g. ). By setting `type` as the target column, we are telling the Nexosis API  
 <br>
 ```json
{
      "dataSourceName": "kddcup-training",
      "targetColumn": "type",
      "predictionDomain": "classification"
}
 ```
 <br>
 You should have a POST request that looks like this (don't forget to set the `api-key` header): 
 <br>
<img src="/assets/img/tutorials/kddcup-postman7.png" alt="Set the JSON Body" style="padding: 5px;" class="img-responsive">
3. Click the blue `Send` button to the Right of the URL in Postman.<br>
 <img src="/assets/img/tutorials/postman-send-black.png" alt="Click Send" style="padding: 10px;" class="img-responsive"><br>
 When you scroll to the bottom of the body, you will see your unique session ID and the response `status` should say `requested` or `started`. 

TODO - INSERT PIC HERE

> **Important:** Copy your `SessionID` out of the response body as you will need this in the next steps.
