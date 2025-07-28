---
layout: default
title: Audits
permalink: /audits/
---

<h1>Audit Writeups</h1>
<ul>
  {% for post in site.categories.audit %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a> - {{ post.date | date: "%Y-%m-%d" }}
    </li>
  {% endfor %}
</ul>
