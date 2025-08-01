---
layout: page-no-title
category: Encoder
permalink: /categories/encoder/
---

<h1>Encoder</h1>

<ul>
  {% for post in site.posts %}
    {% if post.categories contains page.category %}
      <li><a href="{{ post.url }}">{{ post.title }}</a> - {{ post.date | date: "%Y-%m-%d" }}</li>
    {% endif %}
  {% endfor %}
</ul>