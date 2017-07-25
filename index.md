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

<!-- Guides -->
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
              <div class="col-xs-5 col-sm-4 col-md-4 col-lg-4 col-xl-4">
                {% case ct%}
                {% when "Forecasting" %}
                  <img src="/assets/img/forecasting.png">
                {% when "Impact Analysis" %}
                  <img src="/assets/img/impact-analysis.png">
                {% when "Getting Started" %}
                  <img src="/assets/img/getting-started.png">
                {% when "Security" %}
                  <img src="/assets/img/security.png">
                {% endcase %}
              </div>
              <div class="col-xs-7 col-sm-8 col-md-8 col-lg-8 col-xl-8">
                <h5 id="{{ ct | slugify }}" style="margin-top:20px;">{{ ct }}</h5>
                {% assign guides = site.guides | sort: "order" %}
                {% for post in guides %}
                  {% if post.category contains ct and post.tags contains "Favorite" %}
                    <p class="post-listing"><i class="fa {{list-icon}}"></i> <a href="{{ site.url }}{{ post.url }}">{{ post.title }}</a></p>
                  {% endif %}
                {% endfor %}
                <p class="post-listing"><i class="fa {{list-icon}}"></i> <a href="/guides#{{ ct }}">See more…</a></p>
              </div>
            </div>
          </div>
        </div>
      </div>
  {% endfor %}
</div>

<!-- Tutorials -->
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

<div class="row">
  {% for ct in cats %}
    <div class="col-sm-12 col-md-6 col-lg-4 col-xl-4">
        <div class="panel bg-color-lightGray">
          <div class="panel-body">
            <div class="row">
              <div class="col-xs-4 col-sm-4 col-md-4 col-lg-4 col-xl-4">
                {% case ct %}
                  {% when "Customer Service" %}
                    <img src="http://nexosis.com/assets/img/use-case/customer-service.png" style="width:100px;">
                  {% when "Distribution & Logistics" %}
                    <img src="http://nexosis.com/assets/img/use-case/distribution-logistics.png" style="width:100px;">
                  {% when "Energy" %}
                    <img src="http://nexosis.com/assets/img/use-case/energy.png" style="width:100px;">
                  {% when "Greater Good" %}
                    <img src="http://nexosis.com/assets/img/use-case/greater-good.png" style="width:100px;">
                  {% when "Human Resources" %}
                    <img src="http://nexosis.com/assets/img/use-case/human-resources.png" style="width:100px;">
                  {% when "IoT" %}
                    <img src="http://nexosis.com/assets/img/use-case/IoT.png" style="width:100px;">
                  {% when "Manufacturing & Operations" %}
                    <img src="http://nexosis.com/assets/img/use-case/manufacturing-operations.png" style="width:100px;">
                  {% when "Sales & Marketing" %}
                    <img src="http://nexosis.com/assets/img/use-case/sales-marketing.png" style="width:100px;">
                  {% when "Sports & Games" %}
                    <img src="http://docs.nexosis.com/assets/img/sports-games.png" style="width:100px;">
                {% endcase %}
              </div>
              <div class="col-xs-8 col-sm-8 col-md-8 col-lg-8 col-xl-8">
                <h5 id="{{ ct | slugify }}" style="margin-top: 20px;">{{ ct }}</h5>
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

<!-- API Clients -->
<div class="row">
  <div class="col-sm-12 col-md-12 col-lg-12 col-xl-12">
    <h3 class="badge badge-warning">API Clients</h3>
  </div>
</div>
<div id="api-clients" class="row">
  <div class="col-sm-12 col-md-12 col-lg-12 col-xl-12">
    <div class="panel bg-color-lightGray">
      <div class="panel-body">
          <div class="row">
            <div class="col-xs-6 col-sm-2 col-md-1 col-lg-1 col-xl-1">
              <p class="center">
                <a href="/clients/dotnet">
                  <img src="/assets/img/dotnet.png" style="width: 50px;"> <br />
                  <span class="small">.NET</span>
                </a>
              </p>
            </div>
            <div class="col-xs-6 col-sm-2 col-md-1 col-lg-1 col-xl-1">
              <p class="center">
                <a href="/clients/nodejs">
                  <img src="/assets/img/nodejs.png" style="width: 50px;"> <br />
                  <span class="small">node.js</span>
                </a>
              </p>
            </div>
            <div class="col-xs-6 col-sm-2 col-md-1 col-lg-1 col-xl-1">
              <p class="center">
                <a href="/clients/python">
                  <img src="/assets/img/python.png" style="width: 50px;"> <br />
                  <span class="small">Python</span>
                </a>
              </p>
            </div>
            <div class="col-xs-6 col-sm-2 col-md-1 col-lg-1 col-xl-1">
              <p class="center">
                <a href="/clients/ruby">
                  <img src="/assets/img/ruby.png" style="width: 50px;"> <br />
                  <span class="small">Ruby</span>
                </a>
              </p>
            </div>
             <div class="col-xs-6 col-sm-2 col-md-1 col-lg-1 col-xl-1">
              <p class="center">
                <a href="/clients/java">
                  <img src="/assets/img/java.png" style="width: 50px;"> <br />
                  <span class="small">Java</span>
                </a>
              </p>
            </div>
             <div class="col-xs-6 col-sm-2 col-md-1 col-lg-1 col-xl-1">
              <p class="center">
                <a href="/clients/scala">
                  <img src="/assets/img/scala.png" style="width: 50px;"> <br />
                  <span class="small">Scala</span>
                </a>
              </p>
            </div>
          </div>
      </div>
    </div>
  </div>
</div>