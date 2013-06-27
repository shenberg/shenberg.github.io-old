---
layout: page
title: Musings
tagline: Things of interest
---
{% include JB/setup %} 

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a><div>
        <br>
        {# work around liquid template issue with comment hack #}
        {{ post.excerpt | prepend: '<!--' }}
    <br>
            <a href="{{ post.url }}">Read more...</a></div></li>
  {% endfor %}
</ul>

