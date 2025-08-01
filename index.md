---
layout: default
---

# Welcome to My Blog


## Categories

<ul>
  <li><a href="/categories/config/">Config</a></li>
  <li><a href="/categories/devtools/">DevTools</a></li>
  <li><a href="/categories/cpp/">C++</a></li>
  <li><a href="/categories/linux/">Linux</a></li>
  <li><a href="/categories/windows/">Windows</a></li>
  <li><a href="/categories/encoder/">Encoder</a></li>
  <li><a href="/categories/network/">Network</a></li>
</ul>

## Latest Posts

<ul>
  {% for post in site.posts limit:50 %}
    <li><a href="{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>
