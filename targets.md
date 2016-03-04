---
layout: page
head: "标签"
permalink: /targets/
---

<ul class="tags-box">
    {% if site.posts != empty %}
    {% for tag in site.targets %}
    <a href="#{{ tag[0] }}" title="{{ tag[0] }}" rel="{{ tag[1].size }}">{{ tag[0] }}<span class="tags-size"> ({{ tag[1].size }})</span></a>
    {% endfor %}
</ul>

<ul class="tags-box">
    {% for tag in site.targets %}
    <li  id="{{ tag[0] }}"><h3>{{ tag[0] }}</h3></li>
    {% for post in tag[1] %}
    <time datetime="{{ post.date | date:"%Y-%m-%d" }}">{{ post.date | date:"%Y-%m-%d" }}</time> &raquo;
    <a href="{{ site.url }}{{ post.url }}" title="{{ post.title }}">{{ post.title }}</a><br />
    {% endfor %}
    {% endfor %}
    {% else %}
    <span>No posts</span>
    {% endif %}
</ul>
