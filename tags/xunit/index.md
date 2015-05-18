---
layout: article
title: ...что-нибудь про тестовые фреймворки?
date: 
modified:
excerpt:
image:
  feature:
  teaser:
  thumb:
ads: true
---
<ul>
{% for post in site.tags.xunit %}
    <li>
        <a href="{{ post.url }}/">{{ post.title }}</a>
        <span>({{ post.date | date:"%Y-%m-%d" }})</span>
    </li>
{% endfor %}
</ul>
