---
layout: page
title: 
tagline: 
comments: false
---
{% include JB/setup %}
<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date: "%Y-%m-%d"}}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

