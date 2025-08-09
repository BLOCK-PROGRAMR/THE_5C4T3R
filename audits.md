---
layout: default
title: Audits
permalink: /audits/
---

<h1>Audit Writeups</h1>

{% if site.audits.size == 0 %}
  <p>No audits available yet. Coming soon!</p>
{% endif %}

<ul>
  {% assign sorted_audits = site.audits | sort: "date" | reverse %}
  {% for post in sorted_audits %}
    <li>
      <a href="{{ post.url | relative_url }}">{{ post.title }}</a> — {{ post.date | date: "%B %d, %Y" }}
    </li>
    <br/>
  {% endfor %}
</ul>
