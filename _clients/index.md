---
title: API Clients
description: 
breadcrumb: API Clients
layout: fullwidth
exclude_navbreadcrumb: true
exclude_navsidebar: true
exclude_navfooter: true
exclude_from_search: true
---

{% assign rawcats = "" %}
{% for post in site.clients %}
  {% assign tcats = post.category | join:'|' | append:'|' %}
  {% assign rawcats = rawcats | append:tcats %}
{% endfor %}

{% assign rawcats = rawcats | split:'|' | sort %}

{% assign cats = "" %}

{% for cat in rawcats %}
  {% if cat != "" %}

    {% if cats == "" %}
      {% assign cats = cat | split:'|' %}
    {% endif %}

    {% unless cats contains cat %}
      {% assign cats = cats | join:'|' | append:'|' | append:cat | split:'|' %}
    {% endunless %}
  {% endif %}
{% endfor %}

<div class="row">
  <div class="col-sm-12 col-md-12 col-lg-12 col-xl-12">
    <h1>{{ page.title }}</h1>
    <hr>
  </div>
</div>

<div class="row">
  {% for ct in cats %}
    {% for post in site.clients %}
      {% if post.category contains ct %}
        {% case post.category %}
          {% when ".NET" %}
            {% assign category = "dotnet" %}
          {% when "Node.js" %}
            {% assign category = "nodejs" %}
          {% when "Python" %}
            {% assign category = "python" %}
          {% when "Ruby" %}
            {% assign category = "ruby" %}
          {% when "Java" %}
            {% assign category = "java" %}
          {% when "Scala" %}
            {% assign category = "scala" %}
          {% when "Javascript" %}
            {% assign category = "javascript" %}
          {% when "curl" %}
            {% assign category = "curl" %}
          {% when "php" %}
            {% assign category = "php" %}
          {% when "ObjC" %}
            {% assign category = "objc" %}
          {% when "PowerShell" %}
            {% assign category = "powershell" %}
        {% endcase %}
        <div class="col-sm-3">
          <div class="panel bg-color-lightGray">
            <div class="panel-body center pt20">
                <img src="/assets/img/{{ category }}.png">
                <h3 id="{{ ct | slugify }}" class="jumptarget center mt20">{{ ct }}</h3>
                <hr>
                <p><strong><a href="{{ site.url }}{{ post.url }}">{{ post.title }} <i class="fa fa-angle-right ml5"></i></a></strong></p>
            </div>
          </div>
        </div>
      {% endif %}
    {% endfor %}
  {% endfor %}
</div>