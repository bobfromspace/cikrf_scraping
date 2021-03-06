# cikrf_scraping

*Как скачивать результаты выборов с сайта ЦИК РФ с помощью языка `R`?*

Цель - показать, как с помощью статистического пакета `R` за короткое время можно превратить неудобные для ручного сбора результаты выборов на сайте ЦИК РФ в таблицу, готовую к статистическому анализу.

Для иллюстрации я использую результаты выборов в Государственную Думу 2016г. по УИКам Москвы (на момент 13 июня 2017 они отсутствовали в базе данных "Календаря выборов" движения Голос). Кроме этого, я описываю, как можно собирать информацию о результатах выборов на уровне регионов или даже УИКов в нескольких регионах в рамках одних выборов.

## Ориентация

* [Вместо чего?](#Разные-способы-сбора-информации-о-выборах)

* [Для кого?](#Целевая-аудитория)

* [Собственно, содержимое](#Организация-проекта)

* [Уровень пользователей](#ПО-и-уровень-знаний)

* [Важные комментарии](#Напоследок)

## Разные способы сбора информации о выборах

Часто те, кто анализирует результаты выборов, собирают электоральную статистику с сайта ЦИК руками, однако, есть ряд других более удобных способов.

* Сам ЦИК предоставляет возможность работать с [открытыми данными](http://www.cikrf.ru/opendata/ "Открытые данные - ЦИК РФ"). Стоит учесть, что охват небольшой.

* [Проект "Календарь выборов"](http://els.golosinfo.org/ru/elections "Выборы - Голос") движения "Голос" совместно с [Сергеем Шпилькиным](http://podmoskovnik.livejournal.com/profile "podmoskovnik - Профиль || Живой Журнал") даёт доступ к результатам федеральных, региональных и муниципальных выборов. Проект полезный, и я бы первым делом искала результаты выборов именно в их базе, поскольку она кажется наиболее полной на данный момент. Тем не менее, возможности создателей не безграничны, так что файлы с данными представлены не по всем региональным и муниципальным выборам.

* Иван Бегтин недавно опубликовал [lazyscraper](https://github.com/ivbeg/lazyscraper "ivbeg/lazyscraper: Lazy helper tool to make easier scraping with simple tasks"), скрипт в Питоне. С его помощью можно собирать различную информацию с страницы в Интернете, а работает он из коммандной строки. Можно собрать информацию о результатах выбора хотя бы с одной избирательной комиссии не вручную.

Знатоки Питона догадываются, как расширить функционал этого скрипта так, чтобы можно было собирать информацию с ряда страниц, но я всего лишь знаю, как пользоваться функцией `lapply()`.

* Наконец, можно написать код на любом другом знакомом языке самостоятельно. Например, в `R`.

## Целевая аудитория

Полагаю, эту репозиторию найдут полезной те, кто профессионально и не очень интересуется выборами в России в общем и содержимым сайта ЦИК РФ в частности. Особенно если приходится собирать статистику по выборам.

## Организация проекта

1. Отдельно доступны файлы, содержащие ссылки на результаты выборов в ГД 2016 по [пропорциональной системе](https://1drv.ms/u/s!AsB08ZBv5_PG3CqZ8euWHLD29_om "Файл с ссылками на результаты по пропорциональной системе на OneDrive") и [одномандантным округам](https://1drv.ms/u/s!AsB08ZBv5_PG3Cl_SQ7TpoZPZDVF "Файл с ссылками на результаты по одномандатным округам на OneDrive"). Они здесь скорее на всякий случай.

2. [Сбор ссылок на электронные версии протоколов по УИКам](link_collection.md).

3. [Непосредственный web scraping результатов](uik_collection.md).

## ПО и уровень знаний

### Программы и другие инструменты

* [`R`](https://cran.gis-lab.info/ "Ссылка на страницу скачивания программы R") или [RStudio](https://www.rstudio.com/products/rstudio/download/ "Ссылка на страницу скачивания RStudio").

* Браузер с возможностью просмотра кода страницы ([Список браузеров с объяснениями, по-английски](https://www.computerhope.com/issues/ch000746.htm "How to view the HTML source code of a web page - Computer code"))

### Рекомендации по чтению

Для тех, кто чувствует себя не так уверенно в отношении возможностей овладевания материалом, я бы рекомендовала сначала ознакомиться или же освежить свои знания со следующим материалом

1. в отношении `R`:

* Типы данных, в частности, списки ([отрывок из книги Евгения Балдина, 2008г.](http://www.inp.nsk.su/~baldin/DataAnalysis/R/R-03-datatype.pdf "Типы данных в R и принципы работы с ними - Евгений Балдин")).

* Функции в общем ([что это такое и как их писать, по-английски](https://www.datacamp.com/community/tutorials/functions-in-r-a-tutorial#gs.jE38w40 "A Tutorial on Using Functions in R! - Data Camp")).

* Функции применения функций к ряду случаев, так называемая "группа apply" ([большая обзорная статья, по-английски](https://www.datacamp.com/community/tutorials/r-tutorial-apply-family#gs.BtFBXV0 "Tutorial on the R Apply Family - Data Camp")).

 * Зачем и как работает пакет `{magrittr}`. Чаще всего используется `%>%` - подробное введение к нему и не только [здесь, по-английски](https://cran.r-project.org/web/packages/magrittr/vignettes/magrittr.html "Abstract || {magrittr} || CRAN").

 * Как работать с пакетом `{data.table}`. Есть хорошее введение Владислава Грозина [в секции Документов в ВКонтакте](https://vk.com/spbrug "Saint Petersburg R User Group | ВКонтакте") (я не сижу в ВК, поэтому более точную ссылку дать не могу). Помимо этого есть [перевод англоязычного введения в содержимое пакета Андреем Огурцовым](https://bookdown.org/statist_/DataTableManual/ "Руководство по data.table"), очень наглядное пособие.

2. в отношении веб-разработки:

* CSS-указатель ([коты в помощь овладеванию базовыми навыками, по-английски](https://robots.thoughtbot.com/basic-css-selectors-explained-with-cats "Basic CSS Selector Syntax Explained Using Cats - Mike Borsare; THOUGHTBOT")). 

* regex/регулярные выражения ([небольшое, но полное учебное пособие, по-английски](https://regexone.com/ "RegexOne. Learn Regular Expressions with simple, interactive exercises")). Когда-то помогло и мне.

* Знания о методах GET и POST лишними тоже не будут. [W3schools.com дают](https://www.w3schools.com/tags/ref_httpmethods.asp "HTTP Methods GET vs POST") базовую ясную информацию по этому предмету.

## Напоследок

Скачивание большого объёма данных требует времени, поэтому не надо огорчаться, если всё срабатывает не за пару секунд.

Использованное решение перевода из списка списков в базу данных c помощью внешнего указателя в [файле с скачианием информации с сраницы](uik_collection.Rmd) было частично взято с сайта [Stack Overflow](https://stackoverflow.com/ "Stack Exchange"), однако, в настоящее время я не могу найти ссылку на страницу, чтобы сделать референцию как положено по лицензии [cc by-ca 3.0](https://creativecommons.org/licenses/by-sa/3.0/ "Attribution-ShareAlike 3.0 Unported"). Как только это произойдёт, я обязательно обновлю информацию.

Я люблю получать письма на *nilchenko[at]eu.spb.ru*. Буду также рада, если кто-то решит продолжить это дело, усовершенствовав код или подход в целом. Возможно, даже напишет пакет для `R`!
