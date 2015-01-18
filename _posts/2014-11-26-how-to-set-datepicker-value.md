---
layout: article
title: ...как в Selenium выбрать дату в jQuery Datepicker?
date: 2014-11-26T13:10:24+03:00
excerpt: Самый надёжный и быстрый способ -- обратиться к datepicker'у через jQuery API при помощи JavascriptExecutor
tags: [selenium, tricks, ajax]
image:
  feature: 2014-11-26-how-to-set-datepicker-value/feature.png
  credit: Artan Sinani
  creditlink: http://rtsinani.github.io/jquery-datepicker-skins/
  teaser: 2014-11-26-how-to-set-datepicker-value/datepicker.jpg
  thumb:
comments: true
ads: true
---
Datepicker -- это поле ввода, предназначенное для ввода даты. Но это не простое текстовое поле ввода. При нажатии на него появляется календарик, в котором можно выбрать нужную дату. Демонстрацию различных вариантов представления этого элемента можно найти [на сайте jQuery](http://jqueryui.com/datepicker/).

Не знаю, насколько удобно человеку работать с этим календариком по сравнению с обычным текстовым полем. Но для автоматизатора это кромешный ужас. Для того, чтобы просто ввести дату, надо сделать множество действий -- кликнуть в поле ввода, чтобы появился календарик, многочисленными кликами по стрелочкам влево и вправо выбрать нужный месяц и год, а потом выбрать нужный день месяца из таблицы. Столько нажатий кнопки мыши лишь для того, чтобы заполнить одно поле ввода! Да ещё и проверки, чтобы понять, нужный месяц и год находится правее или левее уже выбранного. Но и это ещё не всё! Календарик анимированный, поэтому нужно ещё добавить ожидание появления элементов и ожидание завершения анимации перед каждым кликом. В общем, можно потратить не один час на написание функции для заполнения этого поля.

Есть ли альтернатива?

Первая мысль, которая приходит в голову, совершенно очевидна -- а давайте просто не будем обращать внимания на календарик. Это же обычное текстовое поле, можно просто взять и ввести в него текст!

{% highlight c# %}
public void SetDatepicker(IWebDriver driver, string cssSelector, string date)
{
    new WebDriverWait(driver, TimeSpan.FromSeconds(30)).Until<bool>(
        d => driver.FindElement(By.CssSelector(cssSelector)).Displayed);
    driver.FindElement(By.CssSelector(cssSelector)).SendKeys(date);
}
{% endhighlight %}

_(примеры кода здесь и ниже на языке C#)_

Проблема в том, что в тот момент, когда мы начинаем вводить текст -- автоматически появляется календарик, и он не закрывается после того, как ввод текста завершён.

Для того, чтобы он закрылся, надо фокус ввода переместить в какое-нибудь другое место -- например, на другое поле ввода, или можно кликнуть "в любое свободное место на странице":

{% highlight c# %}
public void SetDatepicker(IWebDriver driver, string cssSelector, string date)
{
    new WebDriverWait(driver, TimeSpan.FromSeconds(30)).Until<bool>(
        d => driver.FindElement(By.CssSelector(cssSelector)).Displayed);
    driver.FindElement(By.CssSelector(cssSelector)).SendKeys(date);
    driver.FindElement(By.CssSelector("body")).Click();
}
{% endhighlight %}

Однако кликнуть по другому полю может быть сложно, потому что календарик его закрывает, а если кликнуть в произвольное место страницы, оно может оказаться не "свободным", и мы нечаянно попадём по какой-нибудь ссылке или кнопке, что вызовет нежелательный эффект.

Но главная проблема подстерегает нас в другом месте. Иногда текстовое поле ввода нельзя редактировать, а бывает так, что его вообще нет! [Календарик есть, а текстового поля ввода нет](http://jqueryui.com/datepicker/#inline)!

Что же делать?

Есть другой путь -- использовать jQuery API для взаимодействия с этим элементом. У календарика есть метод setDate, и обратиться к нему можно вот так:

{% highlight c# %}
public void SetDatepicker(IWebDriver driver, string cssSelector, string date)
{
    new WebDriverWait(driver, TimeSpan.FromSeconds(30)).Until<bool>(
        d => driver.FindElement(By.CssSelector(cssSelector)).Displayed);
    (driver as IJavaScriptExecutor).ExecuteScript(
        String.Format("$('{0}').datepicker('setDate', '{1}')", cssSelector, date));
}
{% endhighlight %}

Ну и пример использования:

{% highlight c# %}
IWebDriver driver = new FirefoxDriver();
driver.Url = @"http://jqueryui.com/datepicker/";
driver.SwitchTo().Frame(
    driver.FindElement(By.CssSelector("iframe.demo-frame")));
SetDatepicker(driver, "#datepicker", "02/20/2002");
{% endhighlight %}

Этот способ универсален, его работоспособность не зависит от внешнего вида календарика, он работает быстрее, чем ввод текста в поле ввода, и уж тем более быстрее, чем пять-десять кликов мышкой.

Но может быть он "нечестный"? Не пропускаем ли мы при этом какие-то баги? Возможно. Если разработчики взяли "бракованную" версию jQuery, в которой API работает, а с GUI проблемы -- наши автотесты этого не заметят. Если вас это беспокоит, тогда забудьте всё вышенаписанное и продолжайте кликать мышкой.

А если вы доверяете разработчикам jQuery -- тогда можете смело использовать API, и ваши тесты станут быстрее и стабильнее.

Да, доверие творит чудеса!
