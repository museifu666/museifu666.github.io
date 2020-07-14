---
layout: page
title: $ cat posts.txt
---

<ul>
{% for post in site.categories.posts %}

{% if post.en %}
<li>{{ post.title }} :: <a href="{{ post.url }}" title="{{ post.description }}">rgbCTF 2020 Writeup</a>
{% endif %}

{% endfor %}
</ul>
