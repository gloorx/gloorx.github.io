---
layout: page
title: Archive
permalink: /archive/
---
{% for post in site.posts %}
  {% capture month %}{{ post.date | date: '%m%Y' }}{% endcapture %}
  {% capture nmonth %}{{ post.next.date | date: '%m%Y' }}{% endcapture %}
    {% if month != nmonth %}
      {% if forloop.index != 1 %}
</ul>
      {% endif %}
<h2>{{ post.date | date: '%Y년 %m월' }}</h2>
<ul>
    {% endif %}
  <li>
    <a href="{{ post.url }}">{{ post.title }}</a>
    <span class="post-date">{{ post.date | date: "%Y년 %m월 %-d일"  }}</span>
  </li>
{% endfor %}