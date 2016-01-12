---
layout: page
title: Category
permalink: /category/
---
You can search a specific post. Use brower's search feature by pressing **<code>Ctrl + F</code>**

{% for tag in site.tags %}
    {% assign t = tag | first %}
    {% assign posts = tag | last %}
<h2>{{ t }}</h2>
<ul>
{% for post in posts %}
    {% if post.tags contains t %}
   <li>
    <a href="{{ post.url }}">{{ post.title }}</a>
    <span class="post-date">{{ post.date | date: "%Y년 %m월 %-d일"  }}</span>
  </li>
  {% endif %}
{% endfor %}
</ul>
{% endfor %}