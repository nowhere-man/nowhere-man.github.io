---
layout: category
category: Windows
permalink: /categories/windows/
---

<h1>Windows</h1>

<ul>
  {% for post in site.posts %}
    {% if post.categories contains page.category %}
      <li><a href="{{ post.url }}">{{ post.title }}</a> - {{ post.date | date: "%Y-%m-%d" }}</li>
    {% endif %}
  {% endfor %}
</ul>