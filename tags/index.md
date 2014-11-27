---
layout: article
title: ...как структурированы статьи в моём блоге?
date: 
modified:
excerpt:
image:
  feature:
  teaser:
  thumb:
ads: false
---
Вообще-то они почти никак не структурированы, на то он и блог.

Тем не менее, я помечаю статьи тегами. Все используемые теги перечислены ниже. Чем более тёмный фон или крупнее шрифт -- тем больше статей, помеченных этим тегом. Кликнув на тег, можно найти полный список статей, помеченных этим тегом.

<div id="tagcloud" class="tagcloud">
<ul class="alt">
{% for tag in site.tags %}
    <li class="tag{{ tag | first | size | times: 5 | divided_by: site.tags.size }}">
        <a href="{{ site.url }}/tags/{{ tag | first | slugize }}/">
            {{ tag | first }}
        </a>
    </li>
{% endfor %}
</ul>

<script type="text/javascript">
  var switcher = $('<a href="javascript:void(0)" class="toggle">Облако/таблица</a>').click(
    function(){
      if ($("#tagcloud ul").hasClass("alt")) {
        $("#tagcloud ul").hide().removeClass("alt").fadeIn("fast");
      } else {
        $("#tagcloud ul").hide().addClass("alt").fadeIn("fast");
      }
    }
  );
  $('#tagcloud').append(switcher);
  // create sort buttons
  var sortabc = $('<a href="javascript:void(0)" class="toggle">Сортировать по алфавиту</a>').click(
    function(){
      if ($("#tagcloud ul").hasClass("abc_desc")) {
        $("#tagcloud ul li").tsort({order:"asc"});
        $("#tagcloud ul").removeClass("abc_desc");
        $("#tagcloud ul").addClass("abc_asc");
      } else {
        $("#tagcloud ul li").tsort({order:"desc"});
        $("#tagcloud ul").removeClass("abc_asc");
        $("#tagcloud ul").addClass("abc_desc");
      }
    }
  );
  $('#tagcloud').append(sortabc);
  var sortstrength = $('<a href="javascript:void(0)" class="toggle">Сортировать по частоте</a>').click(
    function(){
      if ($("#tagcloud ul").hasClass("strength_desc")) {
        $("#tagcloud ul li").tsort({order:"asc",attr:"class"});
        $("#tagcloud ul").removeClass("strength_desc");
        $("#tagcloud ul").addClass("strength_asc");
      } else {
        $("#tagcloud ul li").tsort({order:"desc",attr:"class"});
        $("#tagcloud ul").removeClass("strength_asc");
        $("#tagcloud ul").addClass("strength_desc");
      }
    }
  );
  $('#tagcloud').append(sortstrength);
</script>