---
title: Search Results
copyright: 2017 Nexosis 
layout: fullwidth
exclude_navbreadcrumb: true
exclude_navfooter: true
tipue_search_active: true
---

<h1>Search Results</h1>
{% comment%}
<!--
<form action="{{ page.url | relative_url }}">
    <div class="input-group">
    <span class="input-group-addon"><i class="fa fa-search fa-fw"></i></span>
    <input class="form-control" name="q" type="text" placeholder="Search" id="tipue_search_input" pattern=".{3,}" title="At least 3 characters" required>
    </div>
</form>
-->
{% endcomment%}
<div id="tipue_search_content"></div>

<script>
$(document).ready(function() {
  $('#tipue_search_input').tipuesearch({
      'mode': 'static',
      'show': 10,
      'minimumLength': 3,
      'debug': false,
      'highlightTerms': true,
      'showURL': false,
      'highlightTerms': true
    });
});
</script>