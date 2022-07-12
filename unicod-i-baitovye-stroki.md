---
description: Описание Unicode и байтовых строк в Python 3.X
---

# 🤬 Unicod и байтовые строки

В Python 3.X для двоичных данных предоставляется альтернативный строковый тип, а для текста Unicode(включая ASCII) предоставляется нормальный строковый тип.

## Схемы кодирования:

Способ хранения символов варьируется в зависимости от того, какой набор символов используется

_**Наборы символов**_ - это стандарты, которые назначают индивидуальным символам целочисленные коды, чтобы они могли быть представлены в памяти компьютера.

Например стандарт **ASCII **отображает 'a' на целочисленное значение 97 (в шестнадцатеричном формате 0x61). ASCII предоставляет под каждый символ 8-ми битный байт (из которых используется по факту 7). Таким образом алфавит ASCII состоит из 128 символов.

**Latin-1** предоставляет все 8 бит под каждый символ, поэтому его алфавит состоит из 256 символов. Специальным символам присваиваются значения от 128 до 255 (За пределами ANSII).

**Unicode** обеспечивает более высокую гибкость. Чтобы хранить обогащенный текст подобного рода в памяти компьютера, его необходимо транслировать в низкоуровневые байты с использованием _кодировки._ Кодировки определяют правила для перевода строки символов Unicode в последовательность байтов и извлечения строки из последовательности байтов. Трансляция туда и обратно между байтами и строками определяется двумя терминами:

* кодирование- это процесс перевода строки символов в форму низкоуровневых байтов согласно заданному имени кодировки
* декодирование- это процесс перевода низкоуровневой строки байтов в форму строки символов согласно заданному имени кодировки

Для сценариев декодирование строки являются просто символами в памяти, но они могут быть закодированы в разнообразные представления байтовых строк для передачи по сети, хранения в файлах и т.д.

В некоторых кодировках процесс трансляции тривиален (например в ASCII и Latin-1). В некоторых же отображение может быть сложным и выдавать множество байтов на символ.

Например, широко применяемая кодировка **UTF-8** позволяет представлять большой диапазон символов за счет использования схемы с переменным количеством байтов. Символы с кодами ниже 128 представляются как один байт; символы с код между 128 и Ox7ff (2047) переводятся в 2 байта, где каждый байт имеет значение 128— 255, а символы с кодами выше 0x7ff переводятся в 3- или 4-байтовые последовательности со значениями байтов 128-255. Это позволяет сохранять простые строки ASCII компактными, обходить вопросы упорядочения байтов и избегать нулевых байтов, которые могут вызывать проблемы при работе с библиотеками С и сетями

ASCII является подмножеством Latin-1 и UTF-8, однако не совместима с UTF-16 и UTF-32, так  как они выделяют фиксировано 2 и 4 байта под символ соответственно.

## Хранение строк в памяти:

Кодировки применяются только когда текст хранится и передается внешне(в файле, на другом носителе и т.д.). В памяти Python всегда хранит декодированные строки текста в _нейтральном к кодировкам_ формате. Текст транслируется в специфичный для кодировки формат и обратно только при передаче во внешние текстовые файлы, байтовые строки либо API-интерфейсы с особыми требованиями к кодировке либо из них.

{% hint style="info" %}
Начиная с версии Python 3.3 для внутреннего хранения строк текста применяется схема с переменной длинной с 1, 2 и 4 байтами на символ в зависимости от содержимого строки. Размер выбирается на основе символа с наибольшим порядковым значением Unicode в представленной строке.
{% endhint %}

_**Инструменты обработки текста:**_

* Содержимое и длина строки в действительности выражаются в кодовых точках Unicode - идентифицирующих порядковых числах для символов. Например ord теперь возвращает порядковое значение кодовой точки Unicode символа, которое не обязательно будет кодом ASCII и может умещаться или не умещаться в один 8-битный байт. Аналогично len возвращает количество символов, не байтов.

_**Размер текста**_

* Одиночный символ в Unicod не обязательно отображается на одиночный байт либо при кодировании в файле, либо при хранении в памяти. Даже символы простого текста в 7-битном ASCII могут не отображаться на байты — UTF-16 использует множество байтов на символ в файлах, a Python может выделять 1, 2 или 4 байта на символ в памяти. Мышление в терминах символов позволяет нам абстрагироваться от деталей внешнего и внутреннего хранения.

Кодирование имеет отношение главным образом к файлам и передаче. После загрузки в строку в Python с текстом в памяти не связно понятие кодировка, и она является просто последовательностью символов Unicode, хранящихся обобщенным образом. Обращение к такой строке осуществляется как к строковому объекту.

## Типы строк Python:

Python 3.X имеет три типа строковых объектов:

* **str** для представления декодированного текста Unicode (в том числе ASCII)
* **bytes** для представления двоичных данных (включая декодированный текст) (каждый элемент последовательности представлен в виде численного значения в диапазоне от 0 до 255)
* **bytearray** - изменяемая разновидность bytes

## Текстовые и двоичные файлы:

В Python 3.X  проведено четкое и независимое от платформы различие между текстовыми и двоичными файлами.

_**Текстовые файлы**_

* Когда файл открывается в текстовом режиме, чтение его данных автоматически декодирует его содержимое и возвращает его как объект **str**; запись берет объект str и автоматически кодирует его перед передачей в файл. Операции чтения и записи выполняют трансляцию в соответствии со стандартной кодировкой для платформы или указанным именем кодировки. В текстовом режиме файлы также поддерживают универсальный перевод признака конца файла и дополнительные аргументы спецификации кодировки. В зависимости от имени кодировки текстовые файлы могут также автоматически обрабатывать последовательность в начале файла, обозначающую порядок следования байтов_**.**_

_**Двоичные файлы**_

* Когда файл открывается в двоичном режиме за счет добавления b (только в нижнем регистре) к аргументу строки режима во встроенном вызове **open**, чтение его данных их не декодирует, а возвращает в низкоуровневом и неизмененном виде как объект **bytes**; подобным же образом запись берет объект **bytes** и передает его в файл без изменений. В двоичном режиме файлы также принимают объект **bytearray** для содержимого, подлежащего записи в файл.

Файлы в текстовом режиме также обрабатывают последовательность с маркером порядка следования байтов (byte order mark — **BOM**), которая может появляться в начале файлов в условиях ряда схем кодирования. Скажем, в кодировках UTF-16 и UTF-32 маркер ВОМ указывает формат с обратным или прямым порядком байтов (по существу, какой из концов битовой строки более значащий). Текстовый файл UTF-8 также может включать маркер ВОМ для объявления о том, что в целом он относится к UTF-8. При чтении и записи данных с использованием этих схем кодирования интерпретатор Python может пропускать или записывать маркер ВОМ.

## Преобразование строковых типов:

* `str.encode()` и `bytes(S, encoding)` транслируют строку в форму с низкоуровневыми байтами и в процессе создают закодированный объект `bytes` из декодированного объекта `str`.
* `bytes.decode()` и `str(B, encoding)` транслируют низкоуровневые байты в форму строки и в процессе создают декодированный объект `str` из закодированного объекта `bytes.`

Оба метода `encode` и `decode`, а также вызовы `open`, используют либо явно передаваемое имя кодировки, либо принятое по умолчанию. В Python 3.X этим значением является UTF-8, однако `open` применяет значение из модуля `locale`, которое варьируется в зависимости от платформы.

### Важно:

Во-первых, разнообразные стандартные кодировки платформы доступны в модулях `sys` и `locale`, но аргумент кодировки в `bytes` обязателен, несмотря на не обязательность в `str.encode`(и `bytes.decode`).

Во-вторых, хотя вызовы `str` не требуют аргумента кодировки, как делает `bytes`, его отсутствие в вызовах `str` вовсе не означает, что будет использовать стандартная кодировка. Вызов `str` без аргумента кодировки возвращает выводимую строку объекта `bytes`, а не его преобразованную в `str` форму

## Написание текста, отличающегося от ASCII:

### Написание строк Unicode:

Для записи произвольных в строках символов Unicode, часть которых может даже не удастся набрать на клавиатуре, строковые литералы Python поддерживают шестнадцатеричные байтовые управляющие последовательности "\xNN" и управляющие последовательности Unicode вида "\uNNNN" и ” \UNNNNNNNN”. В управляющих последовательностях Unicode первая форма дает четыре шестнадцатеричные цифры для кодирования 2-байтовой (16-битной) кодовой точки символа, а вторая — восемь цифр для 4-байтовой (32-битной) кодовой точки. Байтовые строки поддерживают только шестнадцатеричные управляющие последовательности для закодированного текста и другие формы для данных, основанных на байтах.

Формально, для написания отличных от ASCII символов можно применять:

* Шестнадцатеричные управляющие последовательности или управляющие последовательности Unicode для внедрения порядковых значений кодовых точек в текстовые строки — нормальные строковые литералы в Python З.Х.
* Шестнадцатеричные управляющие последовательности для внедрения закодированного представления символов в байтовые строки — литералы в виде байтовых строк в Python 3.X.

## &#x20;Преобразование между кодировками:

```
B = b'A\xc3\x84B\xc3\xa8C'  #Текст, первоначально закодированынй в формате UTF-8
S = B.decode('UTF-8')       #Декодирование в текст Unicode согласно UTF-8
T = S.encode('cp500')       #Преобразование в закодированную строку bytes согласно EBCDIC
U = T.decode('cp500')       #Преобразование обратно в Unicode согласно EBCDIC

B = U.encode()              #Преобразование согласно стандартной кодировке (UTF-8)
```

## Использование объектов `bytes` в Python 3.X:

Объект `bytes` в Python З.Х представляет собой последовательность коротких целых чисел, каждое из которых находится в диапазоне 0-255 и потому отображается как символ `ASCII`. Он поддерживает операции над последовательностями и большинство методов, доступных в объектах `str`. Тем не менее, `bytes` не поддерживает метод format или выражение форматирования %, к тому же смешивать объекты типов `bytes` и `str` не допускается без явных преобразований.

## Использование объектов `bytearray` в Python 3.X:

В Python З.Х появился третий строковый тип, `bytearray` — изменяемая последовательность целых чисел в диапазоне от 0 до 255, — который является изменяемым вариантом типа `bytes`. Как таковой, он поддерживает те же самые строковые методы и операции над последовательностями, что и `bytes`, а также многие операции изменения на месте, поддерживаемые списками

## Использование файлов Unicode

Читать и записывать текст Unicode, сохраненный в файлах, тоже легко, т.к. вызов `open` в Python З.Х принимает кодировку для текстовых файлов и организует автоматическое выполнение требуемого кодирования и декодирования при передаче данных. Это позволяет обрабатывать разнообразный текст Unicode, который создан посредством различных кодировок, стандартных для платформы, и сохранять тот же самый текст в отличающихся кодировках для разных целей.

```
open("test.txt", "r", encoding="UTF-8").read()
```

Аргумент `encoding` передает вызову `open` желаемую кодировку текстового файла как для записи, так и для чтения.

## Маркер BOM(Надо закончить)

## Модуль `re` для сопоставления с образцом

Модуль `re` обобщен для работы с объектами любого строкового типа в Python 3.X - `str`, `bytes` и `bytearray` - и возвращает результирующие подстроки того же самого типа, что и исходная строка.

## Модуль `struct` для работы с двоичными данными

```
import struct

b = struct.pack('>i4sh', 7, b'spam', 8) #Упаковка трех объектов в строку (4-х байтовое числоб 4-х байтовая стрка и 2-х байтовое число)
print(b)
vals = struct.unpack('>i4sh', b)        #Распаковка данных
print(vals)
```

### Форматированные строки

#### Установка порядка байт, размера и выравнивания на основании символа формата



В языке Python способ упаковки строки определяется на основе первого символа строки формата. Этот символ определяет:

* порядок байт, который формируется с помощью символов **@**, **=**, **<**, **>**, **!**. Если не указать этого параметра, то принимается символ **@**;
* размер в байтах упакованных данных. В этом случае первыми используются цифры, которые обозначают число;
* выравнивание, устанавливаемое системой.



В соответствии с документацией Python в строке формата порядок байт, размер и выравнивание формируются согласно первому символу формата. Возможные первые символы формата отображены в следующей таблице.

|            |                                        |              |                  |
| ---------- | -------------------------------------- | ------------ | ---------------- |
| **Символ** | **Порядок байт**                       | **Размер**   | **Выравнивание** |
| @          | Естественный (зависит от хост-системы) | Естественный | Естественное     |
| =          | Естественной                           | Стандартный  | Отсутствует      |
| <          | little-endian                          | Стандартный  | Отсутствует      |
| >          | big-endian                             | Стандартный  | Отсутствует      |
| !          | сетевой (аналог big-endian)            | Стандартный  | Отсутствует      |

Значение порядка байт может быть одним из 4-х:

* естественной порядок (native). Этот порядок может быть или little-endian или big-endian. Данный порядок определяется в зависимости от хост-системы;
* порядок типа little-endian. При таком порядке первым обрабатывается младший байт, а затем уже старший байт;
* порядок типа big-endian. В этом случае первым обрабатывается старший байт, а затем уже младший байт;
* сетевой порядок (network), который по умолчанию равен порядку big-endian.

Размер упакованных данных может быть одним из двух:

* естественный – определяется с помощью инструкции sizeof компилятора языка C;
* стандартный – определяется на основе символа формата в соответствии с нижеследующей таблицей.

|   |                                 |                |   |
| - | ------------------------------- | -------------- | - |
| x | pad byte                        | без значения   |   |
| c | char                            | байты длиной 1 | 1 |
| b | signed char                     | integer        | 1 |
| B | unsigned char                   | integer        | 1 |
| ? | \_Bool                          | bool           | 1 |
| h | short                           | integer        | 2 |
| H | unsigned short                  | integer        | 2 |
| i | int                             | integer        | 4 |
| I | unsigned int                    | integer        | 4 |
| l | long                            | integer        | 4 |
| L | unsigned long                   | integer        | 4 |
| q | long long                       | integer        | 8 |
| Q | unsigned long long              | integer        | 8 |
| n | ssize\_t                        | integer        |   |
| N | size\_t                         | integer        |   |
| e | float (экспоненциальный формат) | float          | 2 |
| f | float                           | float          | 4 |
| d | double                          | float          | 8 |
| s | char\[]                         | bytes          |   |
| p | char\[]                         | bytes          |   |
| P | void\*                          | integer        |   |



\