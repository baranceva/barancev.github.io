---
layout: article
title: ...что-нибудь про .NET?
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
{% for post in site.tags.dotnet %}
    <li>
        <a href="{{ post.url }}/">{{ post.title }}</a>
        <span>({{ post.date | date:"%Y-%m-%d" }})</span>
    </li>
{% endfor %}
</ul>
