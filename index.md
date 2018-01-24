---
title: Documentation
description: Learn how to use the Nexosis API
copyright: 2017 Nexosis 
layout: fullwidth
exclude_navbreadcrumb: true
exclude_navsidebar: true
exclude_navfooter: true
exclude_from_search: true
---

{% assign list-icon = "fa-file-text-o" %}
<hr>
<h1 class="center">Nexosis API Documentation</h1>
<hr>

<div class="row">
  <div class="col-md-3">
    <div class="panel bg-color-lightGray">
      <div class="panel-body center pt20">
          <img src="/assets/img/guides.png">
          <h3 id="guides" class="jumptarget center mt20">Guides</h3>
          <hr>
          <p><strong><a href="/guides">View now <i class="fa fa-angle-right ml5"></i></a></strong></p>
      </div>
    </div>
  </div>
  <div class="col-md-3">
    <div class="panel bg-color-lightGray">
      <div class="panel-body center pt20">
          <img src="/assets/img/tutorials.png">
          <h3 id="tutorials" class="jumptarget center mt20">Tutorials</h3>
          <hr>
          <p><strong><a href="/tutorials">View now <i class="fa fa-angle-right ml5"></i></a></strong></p>
      </div>
    </div>
  </div>
  <div class="col-md-3">
    <div class="panel bg-color-lightGray">
      <div class="panel-body center pt20">
          <img src="/assets/img/example-applications.png">
          <h3 id="example-applications" class="jumptarget center mt20">Example Apps</h3>
          <hr>
          <p><strong><a href="/#">View now <i class="fa fa-angle-right ml5"></i></a></strong></p>
      </div>
    </div>
  </div>
  <div class="col-md-3">
    <div class="panel bg-color-lightGray">
      <div class="panel-body center pt20">
          <img src="/assets/img/api-clients.png">
          <h3 id="api-clients" class="jumptarget center mt20">API Clients</h3>
          <hr>
          <p><strong><a href="/clients">View now <i class="fa fa-angle-right ml5"></i></a></strong></p>
      </div>
    </div>
  </div>
</div>

<hr>
<h1 class="center">Quick Start Guides</h1>
<hr>

<div class="row">
  <div class="col-md-3">
    <div class="panel bg-color-lightGray">
      <div class="panel-body center pt20">
          <img src="/assets/img/classification.png">
          <h3 id="guides" class="jumptarget center mt20">Classification</h3>
          <hr>
          <p><strong><a href="/guides/quickstartguideclassification">View now <i class="fa fa-angle-right ml5"></i></a></strong></p>
      </div>
    </div>
  </div>
  <div class="col-md-3">
    <div class="panel bg-color-lightGray">
      <div class="panel-body center pt20">
          <img src="/assets/img/regression.png">
          <h3 id="tutorials" class="jumptarget center mt20">Regression</h3>
          <hr>
          <p><strong><a href="/guides/quickstartguidepredict">View now <i class="fa fa-angle-right ml5"></i></a></strong></p>
      </div>
    </div>
  </div>
  <div class="col-md-3">
    <div class="panel bg-color-lightGray">
      <div class="panel-body center pt20">
          <img src="/assets/img/forecasting.png">
          <h3 id="example-applications" class="jumptarget center mt20">Forecasting</h3>
          <hr>
          <p><strong><a href="/guides/quickstartguideforecast">View now <i class="fa fa-angle-right ml5"></i></a></strong></p>
      </div>
    </div>
  </div>
  <div class="col-md-3">
    <div class="panel bg-color-lightGray">
      <div class="panel-body center pt20">
          <img src="/assets/img/anomaly-detection.png">
          <h3 id="api-clients" class="jumptarget center mt20">Anomaly Detection</h3>
          <hr>
          <p><strong><a href="/guides/quickstartguideanomaly">View now <i class="fa fa-angle-right ml5"></i></a></strong></p>
      </div>
    </div>
  </div>
</div>

<hr>

<!-- Guides OLD 
<div class="row">
  <div class="col-sm-12 col-md-12 col-lg-12 col-xl-12">
    <h3 class="badge badge-info">Guides</h3>
  </div>
</div>

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
  {% for ct in site.guides-category-order %}
    <div class="col-sm-12 col-md-6 col-lg-6 col-xl-6">
        <div class="panel guides bg-color-lightGray">
          <div class="panel-body">
            <div class="row">
              <div class="col-xs-5 col-sm-3 col-md-3 col-lg-3 col-xl-3">
                {% case ct%}
                {% when "Forecasting" %}
                  <img src="/assets/img/forecasting.png">
                {% when "Regression" %}
                  <img src="/assets/img/regression.png">
                {% when "Classification" %}
                  <img src="/assets/img/classification.png">
                {% when "Anomaly Detection" %}
                  <img src="/assets/img/anomaly-detection.png">
                {% when "Impact Analysis" %}
                  <img src="/assets/img/impact-analysis.png">
                {% when "Getting Started" %}
                  <img src="/assets/img/getting-started.png">
                {% when "Concepts" %}
                  <img src="/assets/img/concepts.png">
                {% when "Security" %}
                  <img src="/assets/img/security.png">
                  {% when "Troubleshooting" %}
                  <img src="/assets/img/troubleshooting.png">
                {% endcase %}
              </div>
              <div class="col-xs-7 col-sm-9 col-md-9 col-lg-9 col-xl-9">
                <h5 id="{{ ct | slugify }}" style="margin-top:15px;">{{ ct }}</h5>
                {% assign guides = site.guides | sort: "order" %}
                {% for post in guides %}
                  {% if post.category contains ct and post.tags contains "Favorite" %}
                    <p class="post-listing"><i class="fa {{list-icon}}"></i> <a href="{{ site.url }}{{ post.url }}">{{ post.title }}</a></p>
                  {% endif %}
                {% endfor %}
                <p class="post-listing"><i class="fa {{list-icon}}"></i> <a href="/guides#{{ ct | slugify }}">See more…</a></p>
              </div>
            </div>
          </div>
        </div>
      </div>
  {% endfor %}
</div>
-->

<!-- Tutorials
<div class="row">
  <div class="col-sm-12 col-md-12 col-lg-12 col-xl-12">
    <h3 class="badge badge-success">Tutorials</h3>
  </div>
</div>

{% assign rawcats = "" %}
{% for post in site.tutorials %}
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

<style>.panel.tutorial{min-height: 100px;} </style>

<div class="row">
  {% for ct in cats %}
    <div class="col-sm-12 col-md-6 col-lg-4 col-xl-4">
        <div class="panel tutorial bg-color-lightGray">
          <div class="panel-body">
            <div class="row">
              <div class="col-xs-2 col-sm-2 col-md-2 col-lg-2 col-xl-2">
                {% case ct %}
                  {% when "Customer Service" %}
                    <img src="https://nexosis.com/assets/img/use-case/customer-service.png" style="width:60px;">
                  {% when "Distribution & Logistics" %}
                    <img src="https://nexosis.com/assets/img/use-case/distribution-logistics.png" style="width:60px;">
                  {% when "Energy" %}
                    <img src="https://nexosis.com/assets/img/use-case/energy.png" style="width:60px;">
                  {% when "Greater Good" %}
                    <img src="https://nexosis.com/assets/img/use-case/greater-good.png" style="width:60px;">
                  {% when "Human Resources" %}
                    <img src="https://nexosis.com/assets/img/use-case/human-resources.png" style="width:60px;">
                  {% when "IoT" %}
                    <img src="https://nexosis.com/assets/img/use-case/IoT.png" style="width:60px;">
                  {% when "Manufacturing & Operations" %}
                    <img src="https://nexosis.com/assets/img/use-case/manufacturing-operations.png" style="width:60px;">
                  {% when "Sales & Marketing" %}
                    <img src="https://nexosis.com/assets/img/use-case/sales-marketing.png" style="width:60px;">
                  {% when "Sports & Games" %}
                    <img src="http://docs.nexosis.com/assets/img/sports-games.png" style="width:60px;">
                  {% when "Fun!" %}
                    <img src="http://docs.nexosis.com/assets/img/fun.png" style="width:60px;">
                {% endcase %}
              </div>
              <div class="col-xs-10 col-sm-10 col-md-10 col-lg-10 col-xl-10">
                <h5 id="{{ ct | slugify }}" style="margin-top: 10px;">{{ ct }}</h5>
                {% for post in site.tutorials %}
                  {% if post.category contains ct %}
                    <p class="post-listing"><i class="fa {{list-icon}}"></i> <a href="{{ site.url }}{{ post.url }}">{{ post.title }}</a></p>  
                  {% endif %}
                {% endfor %}
              </div>
            </div>
          </div>
        </div>
      </div>
  {% endfor %}
</div>
-->

{% include apiClients.html %}