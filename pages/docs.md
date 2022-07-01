---
layout: page
title: Documentation
permalink: /docs/
---

# Documentation

Welcome to the {{ site.title }} Documentation pages! Here you can quickly jump to a 
particular page.

<div class="section-index">
    <hr class="panel-line">
    {% assign page_docs = site.docs | sort: 'order' %}
    {% for post in page_docs  %}        
    <div class="entry">
    <h5><a href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a></h5>
    <p>{{ post.description }}</p>
    </div>{% endfor %}
</div>
