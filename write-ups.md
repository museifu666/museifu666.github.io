---
layout: page
title: $ cat write-ups.txt
---

<ul>
{% for post in site.categories.posts %}

{% if post.en %}
<li>{{ post.title }} :: <a href="{{ post.url }}" title="{{ post.description }}">rgbCTF 2020</a>
{% endif %}

{% endfor %}
</ul>
