# mini-readability
Mini readability (Tenzor)

## Описание алгоритма mini readability

### Задача

Имеется необходимость вытаскивать полезный контент из веб документов (без реклам и меню навигации)
и представлять его в виде текстового файла для чтения.
Форматирование текстовых документов имеет определенные правила и может настраиваться в отдельном
файле конфигурации. Результирующий файл необходимо сохранять на диске в определенной директории
относительно текущего расположения программы в поддиректориях. Имена поддиректорий должны
соответствовать сегментам урла заданного веб документа.

### Решение

Приложение выполнено в виде утилиты, для запуска из командной строки.
Все подключаемые модули имеются в составе системных библиотек python.
Для запуска приложения необходима версия python 3.5
Основной инструмент для это класс __HTMLParser__ входящий в состав модуля __html.parser__,
который осуществляет обработку документа и инициирует вызов определенных методов
в момент прохождения через различные этапы обработки документа (например: обработка открывающего тега).
Для реализации задачи был создан класс __ExtractorContent__, наследник класса __HTMLParser__,
в котором перекрыто поведение методов __handle_starttag__ (обработка открывающего тега),
__handle_data__ (обработка текста внутри тега), __handle_endtag__ (обработка закрывающего тега).
Эти методы вызываются в процессе обработки документа и осуществляют запись полезного контента по абзацам.
Полезный контент определяется согласно установленным тегам которые его могут содержать,
в большинстве случаев это тег \<p> и текст содержащийся внутри блока \<p> \</p>
по умолчанию считается полезным. Так же по задаче требовалось сохранить ссылки встречающиеся
в полезной информации, поэтому особому вниманию требовалась обработка открывающих тегов и их атрибутов.
Полезная информация в документе может быть разбита на несколько частей (несколько тегов \<p>)
и быть расположена в разных частях документа, но как правило ее обрамляют тегом \<div> со специфическим
классом. Учитывая все это приложению необходимо задать основной тег в котором хранится блок
с полезной информацией и теги с абзацами в которых эта информация размещена,
а так же необходимо перечислить ряд вложенных тегов который могут содержать уточняющую информацию (ссылки).

Поскольку разные сайты имеют различную структуру страниц и корректность html кода этих страниц
не всегда удовлетворяет основным правилам верстки, появилась необходимость наличия возможности
настройки тегов которые могут содержать полезный контент на определенных сайтах.
Такая возможность позволит откалибровать поведение приложения при необходимости обработки страниц
со специфической структурой.

Все основные настройки хранятся в приложении и могут быть дополнены в отдельном необязательном конфигурационном файле config.json (пример рабочего конфигурационного файла прилагается).

Описание настроек:

Список разрешенных вложенных в основной блок контента тегов. В большинстве случаев используют тег \<a>
но перечислены так же и другие для необходимости сохранения информации.

```
"list_allow_nested_tags": ["p", "a", "b", "i", "u", "span"]
```

Основной тег для заголовка.
```
"title_tag": "h1"
```

Атрибуты для основного тега заголовка.
```
"title_tag_attributes": {}
```

Основной тег контента - блок в котором расположены абзацы с полезной информацией,
как правило он специфичен для каждого сайта но в большинстве случаев актуален тег \<p>.
```
"content_tag": "p"
```

Атрибуты для основного тега. Если он имеет специфические классы или ид, то их можно определить для более точной обработки.
```
"content_tag_attributes": {}
```

Основной тег параграфа, который разделяет контент на абзацы.
```
"paragraph_tag": "p"
```

Максимальное количество символов в строке, при превышении которого текст будет переноситься на следующую строку по словам.
```
"word_wrap_column": 80
```

Настройки шаблона для определенных тегов. Имеют смысл для обработки тега \<a>
в данной задаче но есть потенциал к использованию более широких возможностей
оформления конечного результата. Параметр _data_ определяет шаблон для содержимого тега,
а все атрибуты тега описываются как есть без префиксов или постфиксов.
```
"template_tags": {
    "a": {
        #
        "_data_": "%s",
        "href": "[%s]",
    },
    "b": {
        "_data_": "**%s**",
    },
}
```

Список хостов для которых будет действовать конфиг или __True__ для всех сайтов.
Для определенных сайтов можно перечислить в виде списка (например ```["news.rambler.ru", "regnum.ru"]```).
```
"hosts": True
```

Это базовые настройки. Если для определенного сайта требуются специфические параметры по ряду настроек,
то достаточно будет указать отдельные параметры и указать для какого сайта они предназначены,
настройки которые не указаны в конфигурационном файле будут взяты из основных настроек.

Основной класс которые реализует поставленную задачу __KeeperContent__
он осуществляет доступ к классу __ExtractorContent__, передает ему актуальный набор настроек
для указанного сайта и вытаскивает готовую информацию пригодную для сохранения.
Наследником этого класса является __FileKeeperContent__ который осуществляет запись информации
в файл по определенному пути (исходя из урла).

### Результаты

`http://www.vesti.ru/doc.html?id=2699112&cid=9` - основной контент сохранен, имеется немного мусора.

`http://expert.ru/2015/12/14/kitaj-zateyal-novuyu-igru-protiv-dollara/` - контент полностью сохранен, без мусора.

`http://lenta.ru/news/2015/12/15/american/` - контент полностью сохранен, без мусора.

`http://news.rambler.ru/politics/32242487/` - контент полностью сохранен, без мусора.

`http://regnum.ru/news/accidents/2035660.html` - контент полностью сохранен, без мусора.

`http://vz.ru/news/2015/12/14/783785.html` - контент сохранен, из-за особенностей структуры документа передалось излишнее количество ссылок.

`http://ria.ru/world/20151216/1343317621.html` - основной контент сохранен, из-за особенностей структуры документа имеется большое количество мусора после контентной части.

Работа программы проверялась на версии python 3.5.0 под операционной системой Ubuntu 15.10,
а так же под операционной системой Microsoft Windows 7. Результаты работы под этими ОС не имеют серьезных отличий.

### Выводы

В качестве дальнейшего улучшения программы можно создать еще один наследник класса __KeeperContent__,
который бы занимался отправкой готового текста на определенный в настройках электронный почтовый адрес.

Так же для улучшения программы можно реализовать чтение из файла списка урлов требующих обработки
или передавать их в несколько параметров в командной строке.
В случае возникновения проблем при обработке документов, возможно предусмотреть наличие
информативных сообщений с конкретным описанием проблем.
Некоторые сайты (например vesti.ru) указывают ид материала в GET параметрах,
через это новые материалы с этих сайтов будут замещать старые, есть необходимость
улучшать программу в этом направлении и предоставлять возможность
конфигурировать правила сохранения материалов.

Основным улучшением программы может послужить использование сторонних
специализированных для обработки html документов библиотек,
которые имеют решения разного рода проблем возникающих в процессе обработки не валидных документов.