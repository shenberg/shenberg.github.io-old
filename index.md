---
layout: page
title: Musings
tagline: Things of interest
---
{% include JB/setup %}

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a><div>
    {{ post.excerpt }}<br>
            <a href="{{ post.url }}">Read more...</a></div></li>
  {% endfor %}
</ul>

