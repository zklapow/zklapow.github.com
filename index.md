---
layout: page
title: Recent Posts 
tagline: Supporting tagline
---
{% include JB/setup %}
<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <h2><a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></h2></li>
    <p>{{ post.content | replace:'more start -->','' | replace:'<!-- more end','' }}</p>
    <span><a href="{{ BASE_PATH }}{{ post.url }}">Read More</a></span>
  {% endfor %}
</ul>

