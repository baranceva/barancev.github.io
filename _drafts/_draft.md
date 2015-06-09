---
layout: article
title: ...почему ....?
excerpt: Потому что ...
tags: [xunit]
image:
  feature: 2015-06-09-page-loading-complete/feature-wide.png
  credit: Urban Games
  creditlink: http://www.train-fever.com/media/concept-art/
  teaser: 2015-06-09-page-loading-complete/teaser-450x200.jpg
  thumb:
comments: true
toc: true
ads: true
---
## Часть 1

В [предыдущей статье](/test-deps-are-evil/) я пытался объяснить, почему тесты должны быть независимыми. И конечно же спровоцировал ожидаемые возражения -- а что делать со сложными тестами, которые состоят из целой серии шагов?

<iframe width="560" height="315" src="https://www.youtube.com/embed/xDQAonCnjRI" frameborder="0" allowfullscreen></iframe>

## Часть 2

Делаем примерно такой тестовый метод и к нему десяток вспомогательных методов, которые реализуют отдельные шаги:

{% highlight java %}
@Test(dataProvider = "tripDataProvider")
public void testCanBuyTicket(TripInfo tripInfo) {
  selectDepartureStation(tripInfo.from);
  selectArrivalStation(tripInfo.to);
  submitDepartureArrivalSelection();
  selectTrain(tripInfo.train);
  selectSeats(tripInfo.travelers);
  enterTravelersInfo(tripInfo.travelers);
  submitOrder();
  performPayment(tripInfo);
  List<Ticket> tickets = verifyTickets(tripInfo.travelers);
  verifySeatsAreBuzy(tickets);
}
{% endhighlight %}

## Резюме

Шаги зависят друг от друга. Тесты -- нет.