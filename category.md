---
layout: fullwidth
title: Content by Category
exclude_from_search: true
---

{% include category-finder.html %}

<div class="row">
  <div class="col-sm-12 col-md-12 col-lg-12 col-xl-12">
    <h1>{{ page.title }}</h1>
    <hr>
    {% for ct in cats %}
      <a class="badge badge-success" style="margin-left: 10px;" href="#{{ ct | slugify }}"> {{ ct }} </a>
    {% endfor %}
    <hr>
  </div>
</div>

<!-- Category Groups -->
<div class="row">
  {% for ct in cats %}
  <div class="col-sm-6 col-md-6 col-lg-6 col-xl-6">
    <div class="panel bg-color-lightGray">
      <div class="panel-body">
        <h5 id="{{ ct | slugify }}" class="jumptarget">{{ ct }} <span class="color-mediumGray pull-right"><i class="fa fa-file-text-o"></i></span></h5>
        <hr>
        <div class="row">
          <!-- Guides -->
          {% for post in site.guides %}
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
          <!-- Tutorials -->
          {% for post in site.tutorials %}
            {% if post.category contains ct %}
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
          <!-- API Clients -->
          {% for post in site.clients %}
            {% if post.category contains ct %}
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