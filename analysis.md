---
layout: default
title: Analysis
permalink: /analysis/
---

<h1>Analysis</h1>
<ul>
  {% for post in site.categories.analysis %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a> - {{ post.date | date: "%Y-%m-%d" }}
    </li>
  {% endfor %}
</ul>
