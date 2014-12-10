---
layout: page
title: Category
permalink: /category/
---
찾으시는 글이 있다면 <code>Ctrl + F</code>를 눌러 브라우저의 검색기능을 이용하세요.

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