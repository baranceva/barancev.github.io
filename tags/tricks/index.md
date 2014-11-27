---
layout: article
title: ...что-нибудь достаточно сложное?
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
{% for post in site.tags.tricks %}
    <li>
        <a href="{{ post.url }}/">{{ post.title }}</a>
        <span>({{ post.date | date:"%Y-%m-%d" }})</span>
    </li>
{% endfor %}
</ul>
