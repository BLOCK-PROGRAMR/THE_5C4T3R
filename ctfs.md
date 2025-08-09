---
layout: default
title: Writeups
permalink: /writeups/
---

<h1>Writeups</h1>
<ul >
  {% for post in site.categories.writeups %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a> - {{ post.date | date: "%Y-%m-%d" }}
    </li>
    <br/>
  {% endfor %}
</ul>


