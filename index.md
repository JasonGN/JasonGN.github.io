---
layout: page
title: Welcome!
tagline: JasonGN Blog
---

# Blogs
<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

# Documents
<ul class="docs">
  {% for doc in site.docs %}
    <li><a href="{{ BASE_PATH }}{{ doc.url }}">{{ doc.title }}</a></li>
  {% endfor %}
</ul>

