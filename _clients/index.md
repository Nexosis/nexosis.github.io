---
title: API Clients
description: 
copyright: 2017 Nexosis 
layout: default
breadcrumb: API Clients
layout: fullwidth
exclude_navbreadcrumb: true
exclude_navsidebar: true
exclude_navfooter: true
exclude_from_search: true
---

<style>
  img.api-client {
    float: right;
    height: 30px;
    margin: -5px 10px 10px;
  }
</style>

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
      {% endcase %}
      <div class="col-sm-6 col-md-6 col-lg-6 col-xl-6">
        <div class="panel bg-color-lightGray">
          <div class="panel-body">
            <h5 id="{{ ct | slugify }}">
              {{ ct }}
              <img src="/assets/img/{{ category }}.png" class="api-client">
            </h5>
            <hr>
            <div class="row">
              <div class="col-sm-12 col-md-12 col-lg-12 col-xl-12">
                <p>
                  <strong><a href="{{ site.url }}{{ post.url }}">{{ post.title }}</a></strong> 
                  {% for tag in post.tags %}
                      {% if tag != "Quick Links" %}
                          <a class="label label-info pull-right" style="margin-right: 10px;" href="/tags#{{ tag | slugify }}">{{ tag }}</a>
                      {% endif %}
                  {% endfor %}
                </p>
                <p class="color-mediumGray" style="font-size: 85%; border-bottom: 1px dotted #afb0b4; padding-bottom: 10px;">
                  {{ post.description }}           
                </p>
              </div>
            {% endif %}
          {% endfor %}
        </div>
      </div>
    </div>
  </div>
  {% endfor %}
</div>