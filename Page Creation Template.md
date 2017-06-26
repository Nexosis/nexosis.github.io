---
title: Title Here
description: Description here
copyright: 2017 Nexosis 
layout: default
category: guides or tutorials or clients
tags: [tag 1, tag 2]
exclude_from_search: true
use_codestyles: true
---

# How to build a page (Do not include page title or H1 or # on document)

------

### {{page.description}}

------

## H2 Element - Section Headers 

(use six dashes above an h2 element)

### H3 Element

#### H4 Element

------

## Where to save

All articles should go in the "_articles" directory.

------

## Formatting Frontmatter (crap at the top)

### Title and Description

Choose a title that is clear, concise, and meaningful. Your description should also be short like a blurb or excerpt.

### Copyright

This should be the current year.

### Layout

For now we have one layout template. It is "default". If the layout is insufficient to what you need, please make a UX request through JIRA to have a template created.

### use_codestyles: true

Set this to true if you are using code snippets in your article and reference below for how to use it. If not, you can delete this line.


### Choose a category:

* Guides
* Tutorials
* SDK

Categories are used for post grouping as well as navigation. 

An article should have one category but are allowed multiple tags.

**If you need to create a category, like when we have SDKs, update the following files:**

* _inclues/nav.html
* _includes/nav-breadcrumb-nav.html

### Choose your tags:

* General (for general information)
* Getting Started (for anything that helps a dev… get started)
* Quick Links (if post will be viewed often or quick access is desirable)
* programming languages
  * Curl
  * C#
  * Java
  * JavaScript
  * ObjC
  * PHP
  * Python
  * Ruby
* Add others to this list as needed

-----

## Formatting Source Code

### Note: use_codestyles: true in frontmatter

Here's an example of formatting source code in Markdown:

```{:.line-numbers}{:.language-csharp}
public class Bob() {
    public static string foo() {

    }
}
```


```{:.line-numbers}{:.language-javascript}
function foo() {
    doIt();
}
```