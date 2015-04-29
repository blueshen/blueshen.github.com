---
layout: page
title: 博客
permalink: /blog/
---

<div class="home">

{% for post in site.posts %}
  <h1><a href="{{ post.url }}">{{ post.title }}</a></h1>
  <p class="author">
    <span class="date">{{ post.date | date: "%Y-%m-%d %H:%m" }}</span>
  </p>
{% endfor %}