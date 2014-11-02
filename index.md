---
layout: page
title:
---

{% for post in site.posts %}
- ### [{{ post.title }}]({{ post.url }}) <time >{{ post.date | date: '%Y-%m-%d' }}</time>

  {% if post.summary %}{{ post.summary }}{% else %}{{ post.content | strip_html | strip_newlines | truncate: 120 }}{% endif %}

  [Read more &raquo;]({{ site.url}}{{post.url }})
{% endfor %}
