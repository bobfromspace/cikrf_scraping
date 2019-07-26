# Сбор ссылок на страницы УИКов с сайта ЦИК РФ

Открывая сайт ЦИК РФ, можно заметить, что пользование им людьми осложнено структурированностью и необходимостью кликать на кучу кнопочек. При работе машины с этим сайтом таких проблем не возникает. Наоборот, структурированность сайта обеспечивает единый алгоритм работы.

Чтобы собрать с сайта ЦИКа ссылки на УИКи, потребуется несколько пакетов. Если они не установлены, то следует выполнить команды за знаком диез (убрав диез естественно).

Первый пакет `{rvest}` потребуется, чтобы по--простому и быстро собирать данные с страниц в Интернете. `{stringr}` предлагает функции для работы с последовательностью знаков (формат character). Наконец, последний пакет `{magrittr}` позволяет уменьшить количество повторений в коде таких элементов, как название объекта, а также предоставляет возможность сделать код компактным.

```r
# install.packages("rvest")
library(rvest)
# install.packages("stringr")
library(stringr)
# install.packages("magrittr")
library(magrittr)
```

Нижеследующие создания функций немного забегают вперёд. Сначала необходимо изучить структуру страницы в Интернете, чтобы найти элементы, содежание нужную информацию. В данном случае, ссылки на страницы избирательных комиссий более низкого ранга находятся в элементе `option`. Для просмотра структуры `html`--страницы требуется воспользоваться возможностями инструментов разработчика в релевантном веб--браузере. Инструкция о том, как это сделать по [этой ссылке](https://www.computerhope.com/issues/ch000746.htm "How to view the HTML source code of a web page --- Computer code").

Первая функция извлекает информацию из вектора с адресом ссылок, а также предоставляет информацию об элементах, в которых содержатся ссылки на более низкие избирательные комиссии. Вторая функция избавляется от всех лишних знаков, которые окружают содержимое нужного элемента, в данном случае ссылки на страницу избирательной комиссии более низкого ранга.

Для скачивания потребуется функция, извлекающая страницу по ссылке, а также распознающая её содержимое. Она имеет именно такой вид, поскольку подходит для работы с УИК, а не избирательными комиссиями выше. 
```r
get_lower <- function(x) {
  x <- lapply(x,read_html)
  x <- lapply(x,function(y) html_nodes(y,"option"))
}
```

Следующая функция позволит писать меньше строк для очистки знаковых переменных. Полезна при извлечении ссылок на страницы ОИК и УИК.
```r
str_clear <- function(x) {
  x <- as.character(x) # перевод в удобный формат для работы с функциями пакета {stringr}
  x <- x[-1] # первый элемент содержит ненужную информацию, его надо удалить
  # теперь можно заняться непосредственно очисткой вектора от всего лишнего
  x <- str_replace(x,"\">.*","")
  x <- str_replace_all(x,"amp;","")
  x <- str_replace(x,"\n","")
  x <- str_replace(x,"<option value=\"","")
}
```

## Одномандатные округа

На этапе сбора ссылок различие между одномандатными округами и партийным списком не так важно, просто на результаты одной и другой избирательных систем ведут две отличающиеся ссылки.

Нужно присвоить объекту ссылку из адресной строки браузера (механизм немного различается для сайта ЦИК РФ и региональной избирательной комиссии; в случае одномандатных округов я показываю, как работает механизм с сайтом ЦИК РФ) как набор знаков (characters/string). Затем считать информацию о структуре `html`--странице, извлечь все узлы с ссылками на нижестоящие избирательные комиссии и очистить их от всех ненужных знаков.

```r
sd_link <- "http://www.vybory.izbirkom.ru/region/izbirkom?action=show&global=true&root=1000259&tvd=100100067796113&vrn=100100067795849&prver=0&pronetvd=0&region=0&sub_region=0&type=463&vibid=100100067796113"
sd_urloik <- read_html(sd_link) %>% html_nodes(.,"option") %>% str_clear(.)
```

Теперь имеется вектор с ссылками на окружные избирательные комиссии (ОИК). Каждая ОИК подразделяется на территориальные избирательные комиссии (ТИК). В ТИКах, объединённых одной ОИК, избиратели голосуют за одинаковый набор кандидатов. ТИКи в свою очередь надстроены над самым низким уровнем избирательных комиссий --- участковыми (УИК). Результаты выборов на них и требуются.

Процесс сбора ссылок на нижестоящие комиссии не отличается от сбора ссылок на ОИКи, но для сбора ссылок на УИКи требуется перейти на сайт региональной избирательной комиссии. Для этого необходимо заменить пару элементов в структуре ссылок, вставив номер региона в нужные поля.

```r
sd_urldist <- get_lower(sd_urloik) %>% lapply(.,str_clear) %>% unlist(.) 
## что нужно сделать, чтобы получить результаты УИК
sd_urldist <- str_replace(sd_urldist,"region=0","region=77") %>% str_replace(.,"sub_region=0","sub_region=77")
sd_urluik <- get_lower(sd_urldist) %>% lapply(.,str_clear) %>% unlist(.)
# saveRDS(sd_urluik,"Moscow_GD2016_SingleDistrict.Rds")
```

Получив ссылки на страницы с результатами выборов УИКов в виде вектора, можно сохранить этот файл в удобном формате. Обычно я использую внутренний формат `.Rds` в случае если мне придётся далее работать с этим файлом в `R`. Плюс этого формата: занимает мало места и сохраняет все характеристики объекта (при сохранении data frame это имеет большое значение).


## Партийный список

Теперь нужно повторить те же действия для партийного списа, но для облегчения задачи я сразу взяла ссылку с сайта региональной избирательной комиссии.

```r
p_link <- "http://www.moscow_city.vybory.izbirkom.ru/region/region/moscow_city?action=show&root=1000259&tvd=100100067796113&vrn=100100067795849&region=77&global=true&sub_region=77&prver=0&pronetvd=0&vibid=100100067796113&type=242"
p_urloik <- read_html(p_link) %>% html_nodes(.,"option") %>% str_clear(.)
p_urldist <- get_lower(p_urloik) %>% lapply(.,str_clear) %>% unlist(.)
p_urluik <- get_lower(p_urldist) %>% lapply(.,str_clear) %>% unlist(.)
# saveRDS(p_urluik,"Moscow_GD2016_Proportional.Rds")
```

Собрав ссылки, можно переходить непосредственно к сбору информации о результатах выборов на уровне УИКов и переводу в рабочий формат (data frame/table). Ниже я пишу о случаях, когда требуется сбор ссылок во многих регионах на уровне региональной избирательной комиссии или ниже.

## Сбор данных в нескольких регионах
В случае сбора данных в нескольких --- чаще всего во всех --- регионах требуются результаты либо по самим регионам, либо ниже (ОИК, ТИК, УИК). Поэтому я показываю, как с помощью `R` собрать ссылки на результаты выборов в ГД 2016 по __партийному__ списку. При сборе результатов для одномандатных округов следует учитывать большее количество ОИК.

Тем не менее, для того, чтобы собрать ссылки количество ОИК большого значения не имеет. Алгоритм работы такой же, как и во всём остальном, однако, придётся делать большее количество переходов по ссылкам на нижестоящие комиссии (избирательные комиссии субъектов РФ).

### На уровне регионов

Чтобы собрать ссылки на результаты выборов, нужно создать дополнительную функцию очистки знаковых переменных, но можно обойтись без неё, хотя места это займёт больше.

```r
str_cl2 <- function(xml_var) {
  xml_var = as.character(xml_var)%>%str_replace(.,"<.{8}","")%>%
    str_replace(.,"\">.*","")%>%str_replace_all(.,"&amp;","&")
}
```

Теперь для сбора ссылок требуется только алгоритм перехода по страницам от уровня РФ к уровню регионов. К сожалению, переход не такой прямой и короткий, как хотелось бы. Код:

```r
r_link <- "http://www.vybory.izbirkom.ru/region/region/izbirkom?action=show&root=1&tvd=100100067795854&vrn=100100067795849&region=0&global=1&sub_region=0&prver=0&pronetvd=0&vibid=100100067795854&type=242"
r_urlrik <- read_html(r_link) %>% html_nodes(.,"option") %>% str_clear(.)

# сейчас надо сделать переход на уровень комиссии региона РФ; принять во внимание надо то, что в результате ссылка будет даваться не полностью, а только частично
r_urlrik2 <- lapply(r_urlrik, read_html)%>%
  lapply(., function(x)html_nodes(x,xpath = ".//td/a[2]"))%>%
  lapply(., str_cl2)%>%unlist()
# далее потребуется создать основу ссылки
base <- "http://www.vybory.izbirkom.ru/"

# теперь можно соединить основу с остатоком ссылки
r_urlrik2 <- paste0(base,r_urlrik2)

# здесь опять делаем переход, на этот раз на страницу с результатами единого избирательного округа, а не одномандатные
r_urlrik3 <- lapply(r_urlrik2, read_html)%>%lapply(., function(x)html_nodes(x,xpath = ".//tr[16]/td/nobr/a"))%>%
  lapply(., str_cl2)%>%unlist()
  
# теперь можно собирать ссылки на странички с результатами выборов в субъектах РФ
r_urlrik4 <- lapply(r_urlrik3, read_html)%>%
  lapply(., function(x)html_nodes(x,xpath = ".//option/@value"))%>%
  lapply(.,str_cl2)%>%lapply(., function(x)str_replace(x," value=\\\"",""))%>%
  lapply(., function(x)str_replace(x,"\\\"",""))%>%unlist()
# saveRDS(r_urlrik4,"AllRegs_GD2016_Proportional.Rds")
```

### На уровне УИК

Вплоть до уровня районных избирательных комиссий всё знакомо и просто, однако, для нижестоящих УИК опять надо перейти на уровень региональных избирательных комиссий. По сути надо просто сделать несколько похожих круга действий благодаря иерархичности устройства сайта ЦИК.

```r
r_urloik = get_lower(r_urlrik4)%>%lapply(., str_clear)%>%unlist(.)
r_urloik2 = lapply(r_urloik, read_html)%>%
  lapply(., function(x)html_nodes(x,xpath = ".//tr[2]/td/a/@href"))%>%
  lapply(., str_cl2)%>%lapply(., function(x)str_replace(x," href=\\\"",""))%>%
  lapply(., function(x)str_replace(x,"\\\"",""))%>%unlist()
r_urluik = get_lower(r_urloik2)%>%lapply(., str_clear)%>%unlist()
# saveRDS(r_urluik,"AllRegs_UIK_GD2016_Proportional.Rds")
```

Таким образом, имеются ссылки для всех регионов, как на уровне региональной избирательной комиссии, так и УИК. Далее требуется извлечь из этих ссылок информацию, что будет описано в другом файле (ссылка на него в конце страницы).

Кому понравилось, могут повторить то же самое с одномандатными округами самостоятельно, это не так сложно.

## Сбор данных более, чем с одних выборов

Для этого необходимо работать с элеменами jQuery, встроенными в `html`--страницу. У меня руки не доходят разобраться с этим и AJAX, но есть пакет `{RSelenium}`, который позволяет удалённо работать с различными веб--браузерами. Если при попытке работы с [Firefox](https://www.mozilla.org/ru/firefox/new/?f=106 "Загрузить Firefox - Бесплатный браузер - Mozilla") возникают проблемы, то лучший вариант --- использовать [PhantomJS](http://phantomjs.org/ "PhantomJS").

**К другим файлам**:

* [Описание](README.md)

* [Скачивание информации](uik_collection.Rmd)