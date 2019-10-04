---
permalink: /archive/
layout: page
title: Archive
---

## Blog Posts

{% for post in site.posts %}
  * {{ post.date | date_to_string }} &nbsp; [ {{ post.title }} ]({{ post.url }})
{% endfor %}

<!-- &raquo; -->