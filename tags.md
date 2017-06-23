---
layout:   fullwidth
title:    Content by Tag
exclude_from_search: true
---

{% include tag-finder.html %}

<div class="row">
  <div class="col-sm-12 col-md-12 col-lg-12 col-xl-12">
    <h1>{{ page.title }}</h1>
    <hr>
    {% for tag in tags %}
      <a class="badge badge-info" style="margin-left: 10px;" href="#{{ tag | slugify }}"> {{ tag }} </a>
    {% endfor %}
    <hr>
  </div>
</div>

<div class="row">
  {% for tag in tags %}
    {% if tag != "Quick Links" and tag != "Favorite" %}
    <div class="col-sm-6 col-md-6 col-lg-6 col-xl-6">
      <div class="panel bg-color-lightGray">
        <div class="panel-body">
          <h5 id="{{ tag | slugify }}">{{ tag }} <span class="color-mediumGray pull-right"><i class="fa fa-tags"></i></span></h5>
          <hr>
          <div class="row">
            <!-- Guides -->
            {% for post in site.guides %}
              {% if post.tags contains tag %}
                <div class="col-sm-12 col-md-12 col-lg-12 col-xl-12">
                  <p>
                    <strong><a href="{{ site.url }}{{ post.url }}">{{ post.title }}</a></strong> 
                    <a class="label label-success pull-right" style="margin-right: 10px;" href="/category#{{ post.category }}">{{ post.category }}</a>
                  </p>
                  <p class="color-mediumGray" style="font-size: 85%; border-bottom: 1px dotted #afb0b4; padding-bottom: 10px;">
                    {{ post.description }}           
                  </p>
                </div>
              {% endif %}
            {% endfor %}
            <!-- Tutorials -->
            {% for post in site.tutorials %}
              {% if post.tags contains tag %}
                <div class="col-sm-12 col-md-12 col-lg-12 col-xl-12">
                  <p>
                    <strong><a href="{{ site.url }}{{ post.url }}">{{ post.title }}</a></strong> 
                    <a class="label label-success pull-right" style="margin-right: 10px;" href="/category#{{ post.category }}">{{ post.category }}</a>
                  </p>
                  <p class="color-mediumGray" style="font-size: 85%; border-bottom: 1px dotted #afb0b4; padding-bottom: 10px;">
                    {{ post.description }}           
                  </p>
                </div>
              {% endif %}
            {% endfor %}
            <!-- API Clients -->
            {% for post in site.clients %}
              {% if post.tags contains tag %}
                <div class="col-sm-12 col-md-12 col-lg-12 col-xl-12">
                  <p>
                    <strong><a href="{{ site.url }}{{ post.url }}">{{ post.title }}</a></strong> 
                    <a class="label label-success pull-right" style="margin-right: 10px;" href="/category#{{ post.category }}">{{ post.category }}</a>
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
    {% endif %}
  {% endfor %}
</div>