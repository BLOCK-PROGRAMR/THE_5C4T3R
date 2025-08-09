---
layout: default
title: Analysis
permalink: /analysis/
---

<h1>Analysis</h1>

{% if site.analysis.size == 0 %}
  <p>No analysis posts yet. Coming soon!</p>
{% endif %}

<ul>
  {% assign sorted_analysis = site.analysis | sort: "date" | reverse %}
  {% for post in sorted_analysis %}
    <li>
      <a href="{{ post.url | relative_url }}">{{ post.title }}</a> — {{ post.date | date: "%B %d, %Y" }}
    </li>
    <br/>
  {% endfor %}
</ul>

