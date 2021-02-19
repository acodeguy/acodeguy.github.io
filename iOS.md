---
layout: page
permalink: /ios/
title: iOS Programming
---
{% for post in site.categories.ios %}
 <li><span>{{ post.date | date_to_string }}</span> &nbsp; <a href="{{ post.url }}">{{ post.title }}</a></li>
{% endfor %}