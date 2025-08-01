---
layout: page-no-title
category: C++
permalink: /categories/c++/
---

<h1>C++</h1>

<ul>
  {% for post in site.posts %}
    {% if post.categories contains page.category %}
      <li><a href="{{ post.url }}">{{ post.title }}</a> - {{ post.date | date: "%Y-%m-%d" }}</li>
    {% endif %}
  {% endfor %}
</ul>