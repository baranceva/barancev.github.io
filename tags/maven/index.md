---
layout: article
title: ...что-нибудь про Maven?
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
{% for post in site.tags.maven %}
    <li>
        <a href="{{ post.url }}/">{{ post.title }}</a>
        <span>({{ post.date | date:"%Y-%m-%d" }})</span>
    </li>
{% endfor %}
</ul>
