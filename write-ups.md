---
layout: page
title: $ cat write_ups.txt
---

<ul>
{% for post in site.categories.posts %}

<li><a href="{{ post.url }}" title="{{ post.description }}">rgbCTF 2020</a>
</ul>