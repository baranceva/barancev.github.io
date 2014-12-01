---
layout: article
title: ...как запустить тесты в NUnit в заданном порядке?
excerpt: Упорядочить классы легко, нужно просто сделать из них Suite. Упорядочить методы сложнее, придётся использовать самодельное расширение класса TestSuite.
tags: [c#, nunit, tricks]
image:
  feature: how-to-order-tests-in-nunit/domino.jpg
  credit: 
  creditlink: http://invisibledisabilities.org/invisible-no-more-campaign/
  teaser: how-to-order-tests-in-nunit/order-chaos.jpg
  thumb:
comments: true
---
Вопрос "как выполнить тесты в заданном порядке" входит в десятку вопросов про тестовые фреймворки, на которые мне приходится отвечать в своих тренингах по автоматизации.

Вообще-то я обычно отвечаю, что 