---
title: Using Text Based Features
description: Working with free-form entry text fields such as user feedback, tweets, or maintenance logs
copyright: 2018 Nexosis 
layout: default
category: Concepts
tags: [Text, Sentiment, NLP, Reference, Quick Links, Favorite]
use_codestyles: true
order: 5
---
## {{page.title}}

Sometimes you'll have free-form text associated with your data that can hold the key to a good classification of the records. Multiple common examples exist such as

- [Customer satisfaction based on reviews](https://archive.ics.uci.edu/ml/datasets/Amazon+Commerce+reviews+set){:target="_blank"}<sup name="a1">[1](#f1)</sup>
- [Positive or negative sentiment of a tweet](https://www.kaggle.com/c/twitter-airlines-sentiment-analysis){:target="_blank"}
- [Separating Spam from Ham in SMS messages](https://www.kaggle.com/uciml/sms-spam-collection-dataset){:target="_blank"}

You may also use text to provide a continuous value such as in the case of [scoring short answer essays](https://www.kaggle.com/c/asap-aes){:target="_blank"}.

[Many example text datasets are available from sites like the UCI Machine Learning Repository](https://archive.ics.uci.edu/ml/datasets.html?area=&att=&format=&numAtt=&numIns=&sort=nameUp&task=&type=text&view=table){:target="_blank"}.

While the full scale of natural language processing that can be done is [quite broad](https://en.wikipedia.org/wiki/Natural-language_processing){:target="_blank"}, the Nexosis API can automatically help you use text without digging deep into the specifics.

### How to Provide your Text Data
Getting text into features is as easy as identifying a *Text* data type in your metadata for one or more columns in your dataset. We attempt to identify text automatically just like other data types but if you need to set it yourself see our [metadata instructions](http://docs.nexosis.com/guides/columnmetadata) for more information. Each text column will have a vocabulary built and features will be added to the dataset when we run algorithms - that's it, you don't have to do any additional work to use the text. 

### What the API Does with Text
The Nexosis API goes through a multi-stage process to add additional columns for each word found throughout the examples provided in your dataset. Ultimately we aim to add a word importance score based on something called [term frequency–inverse document frequency](https://en.wikipedia.org/wiki/Tf%E2%80%93idf){:target="_blank"}. You don't have to concern yourself with the details of the score but you can think of it as a weighting for each word roughly based on how often it occurs. Let's look at a few of the steps in more detail.

- *Building the Vocabulary* - You might find this called the *corpus* or *dictionary* in some contexts. This is the total collection of all unique words found in every example. If your data set has multiple text columns then we will build one vocabulary for each column. The vocabulary also contains the count for how many times the word appeared overall. When we build a vocabulary we don't include every word as a potential feature. Some words such as 'a', 'and', 'of' are too common and lack predictive power. We label these *stop words* within the vocabulary so that they are not used in subsequent processing. Finally, the vocabulary includes both 1-gram and 2-gram entries - which just means single words and words which have been found together in pairs. 
- *Creating a Score* - with a vocabulary we can go about creating feature columns by running our version of the *tf-idf* algorithm mentioned above. We will create feature columns based on vocabulary words. These can be single words or pairs of words. Each score is specific to a single example so these are very sparse columns as it's easy to get a vocabulary of 10s of thousands of words where each example only has a few hundred.
- *Down-select Features* - Related to the thousands of sparse columns created above we next run a feature selection algorithm to remove those columns which are least likely to provide additional predictive power to our model. Having too many columns can both increase computation time and in many cases reduce effectiveness of the model.

A small example might help you to visualize what will happen to your data. Let's say we have the following dataset:

| product_key | description | product_class |
|:----|:----:|:----:|
| 1 | "The best at cleaning" | "cleaning" |
| 2 | "increases your digging power" | "garden" |

From this dataset we would start with this set of words in our vocabulary:

|word|type|occurrences|
|:---:|:---:|---:|
| best | word | 1
| cleaning | word | 1
| best cleaning | word | 1
| increases | word | 1
| digging | word | 1
| power |  word | 1
| increases digging |  word | 1
| digging power |  word | 1
| the | stop_word | 1
| at | stop_word | 1
| your | stop_word | 1

After generating scores we can add the appropriate feature columns (down-selected heavily for brevity of the example):

|product_class|best cleaning|cleaning|digging|
|:---:|:---:|:---:|:---:|
|cleaning|0.707106781|1||
|garden|||0.577350269|

A few additional notes are in order given the need for a simplistic example. Every word occurring once in the vocabulary would be abnormal for words with actual predictive power. We dropped the key column because we don't actually use identified keys in model building. Classes will be processed as numbers in the final preparation before model building.
### Viewing Vocabularies via the API
If you want to see the vocabulary for a particular column you can see a [list of available vocabularies](https://developers.nexosis.com/docs/services/98847a3fbbe64f73aa959d3cededb3af/operations/5a67315badf47c095051206d) through the API. Each vocabulary has a unique id and identifies the dataset and column for which it was created. 

``` json
{
    "id": "6e0d9884-a5a5-4a30-b9f1-fea8380be51e",
    "dataSourceName": "AirlineTweets",
    "columnName": "text",
    "dataSourceType": "dataSet",
    "createdOnDate": "2018-01-22T19:25:18.6662961+00:00",
    "createdBySessionId": "01611f54-7568-4a52-8693-a6c3d77b3964"
}
```
This id can be used to actually pull back all of the word instances along with whether they are a stop word, and the word rank if not a stop word.

``` json
{
  "id": "6e0d9884-a5a5-4a30-b9f1-fea8380be51e",
  "items": [{
    "text": "united",
    "type": "word",
    "rank": 0
  },
  {
    "text": "totes",
    "type": "stopWord"
  }]
}
```
The word rank is the relative importance of the word when it was used in the model. The lowest number is the most important and the vocabulary will be returned in rank order by default.

<span style="font-size:.8em"><b id="f1">1</b> Lichman, M. (2013). UCI Machine Learning Repository [http://archive.ics.uci.edu/ml]. Irvine, CA: University of California, School of Information and Computer Science.</span> [↩](#a1)