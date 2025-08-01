---
layout: default
---

# Welcome to My Blog


## Categories

<ul>
  {% for category in site.categories %}
    <li><a href="/categories/{{ category[0] | downcase }}/">{{ category[0] }}</a></li>
  {% endfor %}
</ul>

## Latest Posts

<ul>
  {% for post in site.posts limit:5 %}
    <li><a href="{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>
