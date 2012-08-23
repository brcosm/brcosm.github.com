---
layout: page
title: Developer Notes
tagline: Recent posts and notes

---
{% include JB/setup %}

  {% for post in site.posts %}

### [{{ post.title }}]({{ BASE_PATH }}{{ post.url }})

{{ post.excerpt }}

  {% endfor %}
