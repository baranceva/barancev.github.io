---
layout: article
title: ...как прочитать много данных с веб-страницы?
date: 2017-01-30T17:05:15+03:00
excerpt: Постарайтесь сократить количество обращений к браузеру.
tags: [selenium]
image:
  feature: 2017-01-30-read-data-from-web-page/table-wide.png
  credit: Looker Data Sciences, Inc.
  creditlink: https://looker.com/docs/exploring-data/using-table-calculations
  teaser: 2017-01-30-read-data-from-web-page/table.png
  thumb:
comments: true
toc: true
ads: true
---
## Постановка проблемы

Автотесты, использующие Selenium, работают медленно уже потому, что пользовательский интерфейс сам по себе тяжёлый и неповоротливый. Действия через него выполняются дольше по сравнению с сетевыми запросами или обращениями к API, потому что результат нужно визуализировать, на это тратится время и ресурсы.

Увы, на этом проблемы с производительностью не заканчиваются. Чтение данных из пользовательского интерфейса тоже может занять много времени, если это делать неправильно.

Давайте рассмотрим такую задачу. Есть [страница с информацией о столицах разных стран](https://www.spotthelost.com/country-capital-currency.php) в виде таблицы, нужно прочитать эту информацию и представить её в виде словаря (ассоциативного массива).

Прямолинейный способ решения выглядит так (код на языке Python, открытие страницы вынесено за пределы функции):

{% highlight python %}
def by_rows(driver):
    capitals = {}
    table = driver.find_element_by_css_selector("table#container table")
    rows = table.find_elements_by_tag_name("tr")[1:]
    for row in rows:
        cells = row.find_elements_by_tag_name("td")
        capitals[cells[0].text] = cells[1].text
    return capitals
{% endhighlight %}

Нашли таблицу, получили список строк (исключая заголовок), в цикле каждую строку разбили на ячейки, взяли текст ячеек и построили словарь из полученных данных.

Вроде бы всё логично, но... очень медленно.

Выполнение этой функции в браузере Internel Explorer 11 на моей машине занимает примерно 30 секунд. Полминуты! Никаких действий, только чтение данных.

Почему так долго? Потому что выполняется много обращений к браузеру.

В таблице содержится 196 строк с данными (заголовок не считаем). Следовательно, для чтения названий стран и столиц из ячеек потребуется 196*2=392 обращения к браузеру. Ещё 196 обращений нужно для разбиения каждой строки на ячейки. И ещё одно для поиска таблицы, но это уже незаметно на общем фоне.

Итого 589 обращений к браузеру. Каждое занимает примерно 50 миллисекунд, но их много.

## Быстрый браузер

Одна из причин низкой производительности -- медленный браузер (или драйвер браузера, но в данном случае это неважно).

Что будет, если вместо Internet Explorer взять Chrome, который среди пользователей Selenium считается самым быстрым?

Время выполнения сразу падает до 6 секунд. Неплохо!

А как думаете, сколько времени будет загружать данные Firefox? Тоже 6 секунд? 10? 15?

Неправильно. 3 секунды! В два раза быстрее, чем Chrome! И в десять раз быстрее IE (результаты получены на Firefox Nightly 54.0a1 + geckodriver 0.13.0).

|                     | IE 11      | Chrome     | Firefox    |
| ------------------- | ---------- | ---------- | ---------- |
| **by_rows**         | 29.3835    | 6.3087     | 2.9712     |

Конечно, это не означает, что Firefox во всех случаях обгоняет Chrome, но данные со страницы он читает явно лучше.

## Чтение по вертикали

А что делать тем, кто вынужден использовать медленный браузер? Предположим, что ваше приложение работает только в Internet Explorer. Так иногда случается с "энтерпрайз"-приложениями. И -- вот совпадение! -- там часто бывает много данных.

Надо придумать способ уменьшить количество обращений к браузеру.

Для получения ячеек таблицы вовсе необязательно анализировать каждую строку таблицы и тратить на это 196 обращений. Можно читать таблицу по столбцам, а не по строкам, ведь нам нужно загрузить данные всего из двух столбцов:

{% highlight python %}
def by_cols(driver):
    table = driver.find_element_by_css_selector("table#container table")
    col1 = list(map(lambda el: el.text,
                    table.find_elements_by_xpath("./tbody/tr/td[1]")[1:]))
    col2 = list(map(lambda el: el.text,
                    table.find_elements_by_xpath("./tbody/tr/td[2]")[1:]))
    return dict(zip(col1, col2))
{% endhighlight %}

Загружаем список ячеек в первом и втором столбцах -- два обращения к браузеру, из каждой ячейки извлекаем текст -- ещё 392 обращения к браузеру, а потом "спариваем" два получившихся списка строк.

Итого 394 обращения. В прошлый раз, напоминаю, было 589.

При таком способе загрузки данных можно ожидать сокращения времени примерно в полтора раза для всех браузеров. Так и есть:

|                     | IE 11      | Chrome     | Firefox    |
| ------------------- | ---------- | ---------- | ---------- |
| **by_rows**         | 29.3835    | 6.3087     | 2.9712     |
| **by_cols**         | 19.9307    | 3.9300     | 2.0570     |

## Мимо браузера

Если Selenium работает так медленно -- напрашивается мысль совсем от него отказаться, получать "сырой" исходный код страницы с сервера при помощи какого-нибудь HTTP-клиента и анализировать его:

{% highlight python %}
import requests
from lxml import etree, html

def by_http_client():
    capitals = {}
    response = requests.get("https://www.spotthelost.com/country-capital-currency.php")
    page = html.fromstring(response.text)
    table = page.cssselect("table#container table")[0]
    rows = table.cssselect("tr")[1:]
    for row in rows:
        cells = row.cssselect("td")
        capitals[etree.tostring(cells[0], method="text")] = cells[1].text
    return capitals
{% endhighlight %}

Этот способ действительно хорошо работает, но только для простых приложений. Если для получения данных нужно предварительно выполнить какие-то сложные действия в браузере, и если это всё реализовано с использованием AJAX -- симулировать это при помощи низкоуровневого HTTP-клиента крайне сложно. Создание сценариев становится трудоёмким.

Вот если у приложения есть специальный программный интерфейс (API), нацеленный на получение нужных данных -- тогда да, обязательно надо его использовать. Но если нет -- может быть стоит вернуться обратно в браузер и поискать там другие возможности ускорения?

## Use the source, Luke!

Что мы сделали в предыдущем примере? Мы получили исходный код страницы в обход браузера. Но почему бы не взять его прямо из браузера?

{% highlight python %}
from lxml import etree, html

def by_page_source(driver):
    capitals = {}
    page = html.fromstring(driver.page_source)
    table = page.cssselect("table#container table")[0]
    rows = table.cssselect("tr")[1:]
    for row in rows:
        cells = row.cssselect("td")
        capitals[etree.tostring(cells[0], method="text")] = cells[1].text
    return capitals
{% endhighlight %}

Теперь Selenium не используется для поиска элементов на странице и для получения их текста. Мы обращаемся к нему один единственный раз, чтобы загрузить код страницы в формате HTML, а затем при помощи совсем другого инструмента (в данном случае lxml) анализируем этот код.

Дополнительное преимущество по сравнению с HTTP-клиентом заключается в том, что из браузера мы получаем именно тот код, который в данный момент там отрисован, это "слепок" DOM (дерева элементов страницы). Он может отличаться от кода страницы, присланного сервером, потому что на страницу могли загружаться какие-то дополнительные данные средствами JavaScript.

А что с производительностью? Смотрите сами:

|                     | IE 11      | Chrome     | Firefox    |
| ------------------- | ---------- | ---------- | ---------- |
| **by_rows**         | 29.3835    | 6.3087     | 2.9712     |
| **by_cols**         | 19.9307    | 3.9300     | 2.0570     |
| **by_page_source**  | **0.0360** | **0.0210** | **0.0192** |

Фантастика! Мы получаем нужные данные за сотые доли секунды в любом браузере!

Для Internet Explorer ускорение в 1000 раз по сравнению с первым способом. Для "быстрого" Firefox производительность улучшилась "всего лишь" в 100 раз. И при увеличении количества данных эта разница будет только возрастать.

Возникает резонный вопрос -- а если из браузера взять исходный код не всей страницы целиком, а только нужной таблицы -- может быть это ещё сильнее повысит производительность?

{% highlight python %}
from lxml import etree, html

def by_table_source(driver):
    capitals = {}
    table = driver.find_element_by_css_selector("table#container table")
    table_tree = html.fromstring(table.get_attribute("outerHTML"))
    rows = table_tree.cssselect("tr")[1:]
    for row in rows:
        cells = row.cssselect("td")
        capitals[etree.tostring(cells[0], method="text")] = cells[1].text
    return capitals
{% endhighlight %}

А вот и нет!

|                     | IE 11      | Chrome     | Firefox    |
| ------------------- | ---------- | ---------- | ---------- |
| **by_rows**         | 29.3835    | 6.3087     | 2.9712     |
| **by_cols**         | 19.9307    | 3.9300     | 2.0570     |
| **by_page_source**  | **0.0360** | **0.0210** | **0.0192** |
| **by_table_source** | 0.1180     | 0.02818    | 0.0278     |

То ли Selenium тратит дополнительное время на поиск таблицы, то ли браузер код страницы строит быстрее, чем код отдельного элемента. Не знаю. Но факт тот, что это снижает производительность, а не увеличивает. Хотя не исключено, что на других страницах ситуация поменяется.

## JavaScript

Есть ещё один способ получить нужные данные за одно единственное обращение к браузеру -- выполнить фрагмент JavaScript-кода, который сразу вернёт то, что надо:

{% highlight python %}
def by_js(driver):
    array = driver.execute_script(
        "capitals = [];" +
        "table = document.querySelector('table#container table');" +
        "rows = table.querySelectorAll('tr');" +
        "for (var i = 1; i < rows.length; ++i) {" +
        "cells = rows[i].querySelectorAll('td');" +
        "capitals.push([cells[0].textContent, cells[1].textContent]);" +
        "};" +
        "return capitals;")
    return {pair[0]: pair[1] for pair in array}
{% endhighlight %}

Операции, которые ранее выполнялись на языке Python -- поиск элементов на странице, получение текста элементов -- теперь переписаны на JavaScript и перенесены на сторону браузера. Правда, реализация Selenium для Python не умеет правильно обрабатывать ассоциативные массивы, возвращаемые из функции execute_script, поэтому пришлось вернуть список пар, который затем средствами Python превращается в словарь. Если бы мы писали, например, на Java, можно было бы немного упростить выполняемый JavaScript-код.

Давайте посмотрим, насколько быстро работает этот способ:

|                     | IE 11      | Chrome     | Firefox    |
| ------------------- | ---------- | ---------- | ---------- |
| **by_rows**         | 29.3835    | 6.3087     | 2.9712     |
| **by_cols**         | 19.9307    | 3.9300     | 2.0570     |
| **by_page_source**  | **0.0360** | **0.0210** | **0.0192** |
| **by_table_source** | 0.1180     | 0.02818    | 0.0278     |
| **by_js**           | 0.1783     | **0.0077** | 0.0201     |

Для IE и Firefox разница невелика, но посмотрите, что вытворяет Chrome -- он ускорился ещё в несколько раз и теперь уж точно стал самым быстрым!

## Резюме

Если сценарий работает медленно -- это не приговор. Ищите участки кода, которые "тормозят", и оптимизируйте их.

Конечно, ускорить работу тестируемого приложения вы не сможете. Но если в низкой производительности виноваты тесты -- их можно и нужно исправлять.