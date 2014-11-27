---
layout: article
title: ...что-нибудь про Windows?
date: 
modified:
excerpt:
image:
  feature:
  teaser:
  thumb:
ads: false
---
<ul>
{% for post in site.tags.windows %}
    <li>
        <a href="{{ post.url }}/">{{ post.title }}</a>
        <span>({{ post.date | date:"%Y-%m-%d" }})</span>
    </li>
{% endfor %}
</ul>
