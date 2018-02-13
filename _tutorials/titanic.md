---
title: The Titanic - ML Beginner Course (101)
description: This course will introduce some basic concepts of Machine Learning using the Nexosis API and some simple data about each passenger aboard the Titanic.
category: Classification
tags: [Postman, '101']
use_codestyles: true
order: 1
---

Understanding the survivability on the Titanic is a common way to get introduced to Machine Learning. 

This beginner course will introduce the basic concepts of Machine Learning using the Nexosis API and some simple data about each passenger aboard the Titanic. Given this data, we'll show how Machine Learning can be used to build a model to predict a passenger's survivability based on factors such as age, gender, as well as a measure of socio-economic status.

**Time:** 10 minutes<br/>
**Level:** Introductory / 101

#### **Prerequisites:**
* [Postman](https://www.getpostman.com){:target="_blank"}
* [Nexosis API key](https://developers.nexosis.com/developer){:target="_blank"}

------

![The Titanic](/assets/img/tutorials/titanic.png){:.img-responsive}

## Outline

Understanding the problem:
1. [Initial understanding](#initial-understanding)
   * [Programming Approach ](#programming-approach)
   * [Machine Learning Approach](#machine-learning-approach)
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

First step for any model building process is making sure you have a basic understanding of the problem.  In this scenario, we’re going to try and predict if a passenger would survive if they’re on the Titanic.  With this in mind, here are some of the initial factors we believe have an impact on survival rates:

* **Gender and age** - “women and children first”
* **Wealth** - first class fared better than third class

The Titanic had 20 lifeboats which could accommodate a total of 1,178 people.  However, there were 2,224 people onboard (**Note:** this is disputed and we’ll talk about that later).  Even more interesting is that only 728 people survived, well below the capacity that the lifeboats could accommodate.

Some immediate questions you should think about:

How many women, children, and men were on board?  What class were they in?  What about the crew? Why were there only 728 survivors -- only ~62% of capacity of the lifeboats. 

<h4 id="programming-approach" class="jumptarget">Programming Approach</h4>
We’re going to take an agile approach to machine learning during this course.  This means we’ll be starting with what we believe is the minimum and adding to it over time.  We’ll be trying things, seeing what happens, and adjusting from there.

<h4 id="machine-learning-approach" class="jumptarget">Machine Learning Approach</h4>

With machine learning the primary focus is on the data being used to create our model.  Upfront we believe the following data points or features support our stated problem above:

1. Gender
2. Class (first class, second class, third class)
3. Age
4. Spouse / Children 
5. Price of Ticket
6. Port of Embarking

<h3 id="datasets" class="jumptarget">Datasets</h3>

We’re going to start off by using a slightly modified version of the Kaggle Titanic training and testing datasets.  You can grab the training dataset [here](https://raw.githubusercontent.com/Nexosis/sampledata/master/titanic-train.csv){:target="_blank"}.

The training dataset is what we’ll use to build our model.  The test dataset is how we’ll evaluate the model accuracy.

<h4 id="training-data" class="jumptarget">Training Data</h4>

Open up [`titanic-train.csv`](https://raw.githubusercontent.com/Nexosis/sampledata/master/titanic-train.csv){:target="_blank"} in Google Sheets or Microsoft Excel.  You should see columns titled 

<table border="1">
<tr>
<td align="center">Survived</td>
<td align="center">Pclass</td>
<td align="center">Embarked</td>
<td align="center">Sex</td>
<td align="center">Age</td>
<td align="center">SibSp</td>
<td align="center">Parch</td>
<td align="center">Fare</td>
</tr>
</table>

<br clear="all">
You’ll notice that some columns are missing data.  Don’t worry about this for now.  

Install Postman and the Nexosis API Collection (if you haven’t already done so). For instructions on how to install the Nexosis Postman collection, read our Support Document 
[How to Install Postman and the Nexosis API Collection](https://support.nexosis.com/hc/en-us/articles/115004559894-How-to-Install-Postman-and-the-Nexosis-API-Collection){:target="_blank"}

### DIRECTIONS

<h3 id="submitting-the-data" class="jumptarget">Submitting the Data to Nexosis</h3>

We’re going to use Postman to submit our data via the Nexosis API to create a model.

1. The Nexosis API Collections should be listed in postman on the left-hand side. Open the Data folder and click `PUT /data/:dataSetName`
 ![Postman: Using PUT /data/:dataSetName](/assets/img/tutorials/titanic-postman1.png){:.img-responsive}
2. Click on Headers and add keys and values. 
    <table border="1">
    <tr>
    <th style="text-align: center;">Key</th>
    <th style="text-align: center;">Value</th>
    </tr>
    <tr>
    <td style="text-align: center;">api-key</td>
    <td style="text-align: center;">your unique API key</td>
    </tr>
    <tr>
    <td style="text-align: center;">Content-Type</td>
    <td style="text-align: center;">text/csv </td>
    </tr>
    </table>
    <br clear="all">
 For directions to find your API key, click [HERE](https://support.nexosis.com/hc/en-us/articles/115002132274-Where-do-I-find-my-API-Key-){:target="_blank"}.<br>
 <img src="/assets/img/tutorials/titanic-postman3.png" alt="Setting :dataSetName param" style="padding: 10px;" class="img-responsive">
3. Click the `Params` button just to the right the URL in Postman. In the `Key` column, find the parameter that is associated with the URL parameter `:dataSetName` in the list and type in your dataset name in the field in the column named `Value`.<br>
 <img src="/assets/img/tutorials/titanic-postman2.png" alt="Setting :dataSetName param" style="padding: 10px;" class="img-responsive">
4. Click `Body`, and select `raw`, and paste the contents of the CSV data in the box.<br>
 <img src="/assets/img/tutorials/titanic-postman4.png" alt="Setting :dataSetName param" style="padding: 10px;" class="img-responsive">
5. Click the blue `Send` button to the Right of the URL in Postman.<br>
 <img src="/assets/img/tutorials/postman-send.png" alt="Click Send" style="padding: 10px;" class="img-responsive">
6. If all went well, you should be able to scroll down under the request and see the response with a status code of `201 Created` with a `Body` that looks something like this:
 <img src="/assets/img/tutorials/titanic-postman4a.png" alt="Click Send" style="padding: 10px;" class="img-responsive">

<h3 id="building-the-model" class="jumptarget">Building the Model</h3>

1. From the Nexosis API Collections, open the Sessions folder and click `POST /sessions/model`.
 <img src="/assets/img/tutorials/titanic-postman5.png" alt="Click Send" style="padding: 10px;" class="img-responsive"><br>
 If you don’t have the Nexosis API collections, you can simply select `POST` from the dropdown and type `https://ml.nexosis.com/v1/sessions/model` in the text bar.
 <img src="/assets/img/tutorials/titanic-postman6.png" alt="Choose the Session Endpoint" style="padding: 10px;" class="img-responsive">
2. Click `Body` and paste the following code. - **Note:** `dataSourceName` must match whatever you named your dataset at the beginning. 
 <br>
 ```json
{
      "dataSourceName": "titanic",
      "targetColumn": "Survived",
      "predictionDomain": "classification"
}
 ```
 <br>
 You should have a POST request that looks like this: 
 <br>
 <img src="/assets/img/tutorials/titanic-postman7.png" alt="Set the JSON Body" style="padding: 5px;" class="img-responsive">
3. Click the blue `Send` button to the Right of the URL in Postman.<br>
 <img src="/assets/img/tutorials/postman-send.png" alt="Click Send" style="padding: 10px;" class="img-responsive"><br>
 When you scroll to the bottom of the body, you will see your unique session ID and the response `status` should say `requested` or `started`. 
  <img src="/assets/img/tutorials/titanic-postman8.png" alt="Session Response" style="padding: 10px;" class="img-responsive"><br>
 > **Important:** Copy your `SessionID` out of the response body as you will need this in the subsequent steps.

### Check if Session is complete:

When the model is finished, you will receive an email with your session ID. You can also check the status of your session in Postman. To do this, you will need the session ID you copied from the previous step. 

1. Open the Sessions folder and click `GET /sessions/:sessionId`
   <img src="/assets/img/tutorials/titanic-postman9.png" alt="Session Response" style="padding: 10px;" class="img-responsive"><br>
  
2. Click `Params` and paste the unique session ID for `sessionId` Value.
 > **Note:** Your session ID can be found on the previous Postman tab. For this example, the `sessionId` is `0161301c-4d30-422d-9ebb-e4a60671714e`.
  <img src="/assets/img/tutorials/titanic-postman10.png" alt="Session Response" style="padding: 10px;" class="img-responsive"><br>
3. Click the blue `Send` button to the Right of the URL in Postman.<br>
 <img src="/assets/img/tutorials/postman-send.png" alt="Click Send" style="padding: 10px;" class="img-responsive">
 4. Scroll to the bottom to check if the status has completed. If the status has completed, your model ID will be provided.
 <img src="/assets/img/tutorials/titanic-postman11.png" alt="Session Status" style="padding: 10px;" class="img-responsive">
 > **Important:** Copy your unique model Id. For this example the `modelID` is `f57a19be-a464-4c46-a1aa-0911452e6e57`
 
 <h3 id="results" class="jumptarget">Results</h3>
 Now that the model has been created, is it any good? The Session results will contain metrics and the results of an internal test set. 

 1. Open the Sessions folder and click `GET /sessions/:sessionId/results`
 <img src="/assets/img/tutorials/titanic-postman12.png" alt="Session Results" style="padding: 10px;" class="img-responsive">
2. Click Params and paste the unique session ID for `sessionId` Value.
> **Note:** Your session ID can be found on the previous Postman tab. For this example, the `sessionId` is `0161301c-4d30-422d-9ebb-e4a60671714e`.
 <img src="/assets/img/tutorials/titanic-postman13.png" alt="Session Results Params" style="padding: 10px;" class="img-responsive">
3. Click the blue `Send` button to the Right of the URL in Postman.<br>
 <img src="/assets/img/tutorials/postman-send.png" alt="Click Send" style="padding: 10px;" class="img-responsive">
4. The Session Result response body contains both metrics of how accurate the model's predictive capabilities, as well as the test data used to calculate the model's accuracy.
 <img src="/assets/img/tutorials/titanic-postman14.png" alt="Prediction Results" style="padding: 10px;" class="img-responsive">

<h4 id="understanding-results" class="jumptarget">Understanding the Results</h4>

The metrics returned in the session results as follows:

```json
{
    "metrics": {
        "macroAverageF1Score": 0.81294661622530473,
        "rocAreaUnderCurve": 0.83836898395721926,
        "accuracy": 0.8314606741573034,
        "macroAveragePrecision": 0.83721624850657106,
        "macroAverageRecall": 0.8018716577540107,
        "matthewsCorrelationCoefficient": 0.63810979606417906
    },
    ...
```

For this introduction, we'll only focus on the `accuracy` metric. The `accuracy` metric reads `0.8314606741573034` - which can be converted to a percent by multiplying by 100. The accuracy of this model is ~83%, meaning that when Nexosis analyzed the test set against the model, 83% of the predictions made by the model were correct.

<h4 id="test-data" class="jumptarget">Test Data</h4>

The `data` section in the session results contains the internal test set used to calculate the accuracy metrics. Notice there is two `Survived` fields - `Survived:actual` and `Survived`. `Survived:actual` is the original field used to train the model, and `Survived` is the value predicted by the model. In the two example cases below, the actual matches the predicted.

```json
...
 "data": [
        {
            "Survived": "1",
            "Age": "42",
            "Sex": "female",
            "Fare": "227.525",
            "Cabin": "",
            "Parch": "0",
            "SibSp": "0",
            "Pclass": "1",
            "Embarked": "C",
            "Survived:actual": "1"
        },
        {
            "Survived": "1",
            "Age": "28",
            "Sex": "male",
            "Fare": "26.55",
            "Cabin": "C52",
            "Parch": "0",
            "SibSp": "0",
            "Pclass": "1",
            "Embarked": "S",
            "Survived:actual": "1"
        },
    ...
```
<h4 id="how-to-use-results" class="jumptarget">How to Use the Results</h4>
Now that the model exists, a prediction endpoint is available that can be used to provide predictions. 

1. Open the `models` folder and click `POST /models/:modelId/predict`
 <img src="/assets/img/tutorials/titanic-postman15.png" alt="Prediction Results" style="padding: 10px;" class="img-responsive">
2. Click Params and paste the model ID for `modelId` Value.
 <img src="/assets/img/tutorials/titanic-postman16.png" alt="Prediction Results" style="padding: 10px;" class="img-responsive">
3. Next, click Body and paste the following code:
 ```json
 {
	"data": [
	       {
	        "Pclass":  "3",
	        "Sex":  "male",
	        "Age":  "34.5",
	        "SibSp":  "0",
	        "Parch":  "0",
	        "Fare":  "7.8292",
	        "Cabin":  "",
	        "Embarked":  "Q"
	    },
	    {
	        "Pclass":  "3",
	        "Sex":  "female",
	        "Age":  "47",
	        "SibSp":  "1",
	        "Parch":  "0",
	        "Fare":  "7",
	        "Cabin":  "",
	        "Embarked":  "S"
	    }
    ]
}
 ```
 The above JSON represents the data of two different Titanic passengers. By sending these features to the predict endpoint in the request body, the model's prediction of their survivability will be returned.
 <img src="/assets/img/tutorials/titanic-postman17.png" alt="Prediction Results" style="padding: 10px;" class="img-responsive">
4. Click the blue `Send` button to the Right of the URL in Postman.<br>
 <img src="/assets/img/tutorials/postman-send.png" alt="Click Send" style="padding: 10px;" class="img-responsive">
5. Scroll down to see the Response. The predicted `Survived` field will be included along with the features submitted.
 <img src="/assets/img/tutorials/titanic-postman18.png" alt="Prediction Results" style="padding: 10px;" class="img-responsive">

<h4 id="batch-predictions" class="jumptarget">Batch Predictions</h4>
As shown in the previous example, more than one set of features can be submitted at once as a batch. Currently the prediction endpoint only allows `JSON` formatted input but will soon allow `CSV`. 

Here's the slightly modified version of the Kaggle Titanic **test** dataset formatted in `JSON`. You can grab this dataset [here](https://raw.githubusercontent.com/Nexosis/sampledata/master/titanic-test.json){:target="_blank"}.

By repeating the previous prediction 5 steps above (don't forget to set your ModelID), you can submit a much larger set of predictions like so:

 <img src="/assets/img/tutorials/titanic-postman19.png" alt="Prediction Results" style="padding: 10px;" class="img-responsive">

> **Note**: It's important not to submit too many predictions. If the JSON body is too large, it will time-out and you will receive an error. 

<h3 id="next-steps" class="jumptarget">Next Steps</h3>
I started writing this tutorial on the belief that the Titanic is the ‘hello world’ of machine learning.  However, after building this model I’m starting to re-think this, as should you.

Here are some questions we’ll be looking into in part 2 of this series:
* What about the crew?
* Why were the lifeboats only at 62% capacity? 
* How much do we really trust this dataset?
