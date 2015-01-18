---
layout: article
title: ...как в Selenium 'прицепить' файл к невидимому полю ввода?
date: 2014-12-08T11:24:18+03:00
excerpt: Надо просто сделать его видимым. Хотя иногда это не просто. Но надо. Другого пути пока нет.
tags: [selenium, tricks]
image:
  feature: 2014-12-08-how-to-attach-file-to-invisible-field/invisible-no-more.png
  credit: Invisible Disabilities
  creditlink: http://invisibledisabilities.org/invisible-no-more-campaign/
  teaser: 2014-12-08-how-to-attach-file-to-invisible-field/youtube.png
  thumb:
comments: true
ads: true
---
Как вам должно быть известно, для того, чтобы "прицепить" файл к файловому полю ввода, в Selenium нужно выполнить стандартную команду sendKeys в это поле ввода. Если кому-то это не было известно -- теперь вы это знаете.

А также вам должно быть известно, что Selenium не умеет работать со скрытыми полями ввода.

Причина уважительная - пользователь не имеет возможности этого делать, значит и при симуляции поведения пользователя это тоже должно быть запрещено.

Однако в последнее время стало модно делать невидимые файловые поля ввода, с которыми Selenium работать, как уже сказано, не может.

Причина создания таких полей тоже вполне уважительная -- красота и кроссплатформенность.

Пара ссылок, которые поясняют, как это работает:

* [How to Style a HTML file upload button in Pure CSS](http://geniuscarrier.com/how-to-style-a-html-file-upload-button-in-pure-css/)
* [jQuery Custom File Upload Input](http://www.filamentgroup.com/lab/jquery-custom-file-input-book-designing-with-progressive-enhancement.html)

В двух словах, идея следующая. Поскольку файловое поле ввода выглядит некрасиво и во всех браузерах по разному, нужно его заменить на какую-нибудь красивую кнопку, которая выглядит везде одинаково. А сверху над этой кнопкой поместить то самое файловое поле ввода, а чтобы его не было видно - надо сделать его прозрачным (opacity:0). И тогда пользователь будет думать, что он кликает по красивой кнопке, хотя на самом деле он при этом кликает по находящемуся над ней файловому полю ввода, открывается диалог выбора файла и пользователь выбирает файл. В общем, дальше всё как обычно, как если бы файловое поле было видимым.

В общем, у человека всё хорошо, потому что его так легко обмануть.

Но Selenium не таков! Его обмануть непросто! Он точно знает, что поле невидимое! И работать с ним отказывается.

Баг это или фича - вопрос отдельный и неоднозначный, пока не будем заострять на нём внимание. Вместо этого попробуем ответить на простой практический вопрос: что делать? Как прикрепить файл к этому невидимому полю ввода?

Ответ прост и очевиден: надо сделать его видимым.

Тогда возникает следующий вопрос: а как сделать его видимым?

А для этого у нас есть возможность исполнять произвольный JavaScript-код, и мы ею воспользуемся, чтобы поменять стиль элемента -- убрать прозрачность, уменьшить шрифт до нормального, отменить все трансформации.

{% highlight java %}
public void unhide(WebDriver driver, WebElement element) {
  String script = "arguments[0].style.opacity=1;"
    + "arguments[0].style['transform']='translate(0px, 0px) scale(1)';"
    + "arguments[0].style['MozTransform']='translate(0px, 0px) scale(1)';"
    + "arguments[0].style['WebkitTransform']='translate(0px, 0px) scale(1)';"
    + "arguments[0].style['msTransform']='translate(0px, 0px) scale(1)';"
    + "arguments[0].style['OTransform']='translate(0px, 0px) scale(1)';"
    + "return true;";
  ((JavascriptExecutor) driver).executeScript(script, element);
}

public void attachFile(WebDriver driver, By locator, String file) {
  WebElement input = driver.findElement(locator);
  unhide(driver, input);
  input.sendKeys(file);
}
{% endhighlight %}

_(примеры кода здесь и ниже на языке Java)_

Возможно, этот список необходимых корректировок стиля неполон, может быть вам понадобится ещё дополнительно что-то делать с элементом, чтобы он стал видимым.

А теперь примеры использования на разных сайтах:

{% highlight java %}
WebDriver driver = new FirefoxDriver();

driver.get("http://blueimp.github.io/jQuery-File-Upload/basic.html");
attachFile(driver, By.id("fileupload"), "C:\\temp\\image.png");

driver.get("http://imgup.net/");
attachFile(driver, By.id("image_image"), "C:\\temp\\image.png");

driver.get("http://www.2imgs.com/");
attachFile(driver, By.id("f_file"), "C:\\temp\\image.png");
{% endhighlight %}

Работает? Вот и отлично!
