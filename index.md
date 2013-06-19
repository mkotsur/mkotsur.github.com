---
layout: page
title: Cheers!
tagline: Supporting tagline
---
{% include JB/setup %}

Hello. My name is Mike Kotsur, here I write stuff.

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

