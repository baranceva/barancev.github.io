---
layout: article
title: ...что-нибудь несложное?
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
{% for post in site.tags.howto %}
    <li>
        <a href="{{ post.url }}/">{{ post.title }}</a>
        <span>({{ post.date | date:"%Y-%m-%d" }})</span>
    </li>
{% endfor %}
</ul>
