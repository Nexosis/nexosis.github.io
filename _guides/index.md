---
title: Guides
description: Guides that explain how to use the Nexosis API
copyright: 2017 Nexosis 
layout: fullwidth
breadcrumb: Guides
exclude_navbreadcrumb: true
exclude_navsidebar: true
exclude_navfooter: true
exclude_from_search: true
---

{% assign rawcats = "" %}
{% for post in site.guides %}
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
    <!-- 
    {% for ct in cats %}
      <a class="badge badge-success" style="margin-left: 10px;" href="#{{ ct | slugify }}"> {{ ct }} </a>
    {% endfor %}
    <hr>
    -->
  </div>
</div>

<div class="row">
  {% for ct in site.guides-category-order %}
  <div class="col-sm-6 col-md-6 col-lg-6 col-xl-6">
    <div class="panel bg-color-lightGray">
      <div class="panel-body">
      <h5 id="{{ ct | slugify }}">{{ ct }}</h5>
      <hr>
        <div class="row">
          {% assign guides = site.guides | sort: "order" %}
          {% for post in guides %}
            {% if post.category contains ct %}
              <div class="col-sm-12 col-md-12 col-lg-12 col-xl-12">
                <p>
                  <strong><a href="{{ site.url }}{{ post.url }}">{{ post.title }}</a></strong> 
                  {% for tag in post.tags %}
                      {% if tag != "Quick Links" and tag != "Favorite" %}
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