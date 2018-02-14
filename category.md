---
layout: fullwidth
title: Content by Category
exclude_from_search: true
---

{% include category-finder.html %}

<style>
  .title {font-size: 1.15em; font-weight: 700; margin: 5px 0; padding: 5px 10px 0;}
  .description {color: #afb0b4; font-size: .85em; padding: 0 10px 5px 10px; margin-bottom: 10px;}
  h5.section {font-size: 1.5em;font-weight: 600;}
</style>

<div class="row">
  <div class="col-sm-4 col-md-3 col-lg-3 col-xl-3 mt15">
    <!-- Nav -->
    <div class="panel-group nav-group" id="accordion" role="tablist" aria-multiselectable="true">
      <div class="panel active">
        <div id="categories-heading" role="tab">
          <h5 class="category-listing" role="button" data-toggle="collapse" data-parent="#accordion" href="#categories" aria-expanded="false" aria-controls="categories">
              Categories <i class="fa fa-angle-down pull-right"></i>
          </h5>
        </div>
        <div id="categories" class="panel-collapse collapse in" role="tabpanel" aria-labelledby="categories-heading">
          <div class="panel-body">
            <!-- Start Post Loop -->
            {% for ct in cats %}
              <p class="post-listing"><i class="fa fa-file-text-o"></i> <a href="#{{ ct | slugify }}">{{ ct }}</a></p>
            {% endfor %}
            <!-- End Post Loop -->
          </div>
        </div>
      </div>
    </div>
    <!-- End Nav -->
  </div>
  <div class="col-sm-8 col-md-9 col-lg-9 col-xl-9">
    <h1>{{ page.title }}</h1>
    <hr>
    <div class="docs-content">
    <!-- Category Listig -->
    {% for ct in cats %}
      <div class="panel">
        <div class="panel-body">
          <h5 id="{{ ct | slugify }}" class="jumptarget section">{{ ct }} <span class="color-mediumGray pull-right"><i class="fa fa-file-text-o"></i></span></h5>
          <div class="row">
            <!-- Guides -->
            {% for post in site.guides %}
              {% if post.category contains ct %}
                <div class="col-sm-12 col-md-12 col-lg-12 col-xl-12">
                  <p class="title">
                    <a href="{{ site.url }}{{ post.url }}">{{ post.title }}</a>
                  </p>
                  <p class="description">
                    {{ post.description }}           
                  </p>
                </div>
              {% endif %}
            {% endfor %}
            <!-- Tutorials -->
            {% for post in site.tutorials %}
              {% if post.category contains ct %}
                <div class="col-sm-12 col-md-12 col-lg-12 col-xl-12">
                  <p class="title">
                    <a href="{{ site.url }}{{ post.url }}">{{ post.title }}</a>
                  </p>
                  <p class="description">
                    {{ post.description }}           
                  </p>
                </div>
              {% endif %}
            {% endfor %}
            <!-- API Clients -->
            {% for post in site.clients %}
              {% if post.category contains ct %}
                <div class="col-sm-12 col-md-12 col-lg-12 col-xl-12">
                  <p class="title">
                    <a href="{{ site.url }}{{ post.url }}">{{ post.title }}</a>
                  </p>
                  <p class="description">
                    {{ post.description }}           
                  </p>
                </div>
              {% endif %}
            {% endfor %}
          </div>
        </div>
      </div>
      <hr>
      {% endfor %}
      <!-- End Category Listing -->
    </div>
  </div>
</div>