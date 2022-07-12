# Управляемые атрибуты  
Существует четыре методики доступа:
* Методы `__getattr__` и `__setattr__`, предназначенные для направления операций извлечения неопределенных атрибутов и операций присваения всех атрибутов обобщенным методам обработчиков.
* Метод `__getattribute__`, предназначенный для направления операций извлечения всех атрибутов обобщенному методу.
* Встроенная функция `property`, предназначенная для направления операций доступа к конкретному атрибуту функциям обработчиков получения и установки.
* Дескрипторный протокол, предназначенный для направления операций доступа к конкретному атрибуту экземплярам классов с произвольными методами обработчиков получения и установки и являющийся основой для других инструментов, таких как свойства и слоты.

Последние две методики применяются к _конкретным_ атрибутам, тогда как первые две являются достаточно обобщенными, чтобы использоваться основными на делегировании промежуточными классами, которые обязаны направлять внутренним объектам запросы _произвольных_ атрибутов.
## Свойства  
Свойства создаются с помощью встроенной функции `property` и присваиваются атрибутам класса в точности как функции методов. Соответственно, подобно другим атрибутам класса они наследуются подклассами и экземплярами. Их функции перехвата доступа снабжаются аргументом экземпляра `self`, который дает доступ к информации состояния и атрибутам класса, доступным в переданном экземпляре.  
Свойство позволяет контролировать операции извлечения и присваения и свободно делать вычисляемым атрибут, представляющий собой хранилище простых данных; работа существубщего кода при этом не нарушается. Свойства связанны с дескрипторами; по существу они считаются их ограниченной формой.
### __Основы__  
Свойство создается присваением результата вызова встроенной функции атрибуту класса:  
```
атрибут = property(fget, fset, fdel, doc)
```

Ни один из аргументов встроенной функции `property` не является обязательным, и все они получают стандартное значение `None`, если не передается иное. Для первых трех аргументов `None` означает, что соответствующая операция не поддерживается, и попытка ее выполнить приведет к генерации исключения `AttributeError`.  
* `fget` - функция для перехвата операций извлечения атрибута.
* `fset` - функция для перехвата операций присвоения атрибута.
* `fdel` - функция для перехвата операций удаления атрибута.

Формально все три аргумента принимают **любой** вызываемый объект, включая метод класса, имеющий первый аргумент для получения уточняемого экземпляра.  
`fget` возвращает вычисленное значение атрибута, `fset` и `fdel` ничег оне возвращают (на самом деле `None`).  
Аргумент `doc` принимает строку документации для атрибута, если ее наличие желательно; в противном случае свойство копирует документацию фукнции `fget`, которая обычно по умолчанию установлена в `None`.  
Вызов встроенной функции `property` возвращает объект свойства, который присвается  имени атрибута, подлежащего управлению в области видимости класса, где он будет наследоваться каждым экземпляром.
### __Пример__  
```python
class Person:
    def __init__(self, name):
        self._name = name

    def getName(self):
        print("fethc...")       #Извлечение
        return self._name

    def setName(self, name):
        print("change...")      #Изменение
        self._name = name

    def delName(self):
        print("remove...")      #Удаление
        del self._name
    
    name = property(getName, setName, delName, "Hello doc")
```
### __Вычисляемые атрибуты__  
Один из способов примения свойств- динамически вычислять значение атрибута при попытке его извлечения:
```python
class ProSquare:
    def __init__(self, value):
        self._value = value

    def getX(self):
        return self._value ** 2

    def setX(self, value):
        self._value = value

    def delX(self):
        del self._value

    x = property(getX, setX, delX)
```
### __Реализация свойств с помощью декораторов__  
Встроенная функция `property` может использоваться в качестве декоратора для определения функции, которая будет запускаться автоматически при извлечении атрибута:
```python
class Person:
    @property
    def name(self): ...     #Поторная привязка: name = property(name)
```  
Во время выполнения декорированный метод автоматически передается первым аргнументом встроенной функции `property`. В данном случае декоратор является синткасических сахаром, т.к. можно задать более явно:
```python
class Person:
    def name(self): ...
    name = property(name)
```  
### __Декораторы для установки и удаления__  
Объекты свойств имеют также методы `getter`, `setter` и `deleter`, которые назначают соответствующие методы доступа к свойству и возвращают копию самго свойства. Таки м образом их можно применять для указания компонентов свойств, декорируя нормальные методы:
```python
class Person:
    def __init__(self, name):
        self._name = name

    @property
    def name(self):
        print("fetch...")
        return self._name

    @name.setter
    def name(self, name):
        print("change...")
        self._name = name

    @name.deleter
    def name(self):
        print("remove")
        del self._name
```  

## Дескрипторы  
_Дескрипторы_ предлагает альтернативный способ перехвата доступа к атрибутам. В действительности свойство _ялвяется_ разновидностью дескриптора- говоря фрмально, встроенная функция `property` предсавляет собой лишь упрощенный способ создания особого типа дескриптора, который запускает функции методов при доступу к атрибуту.  
Дескрипторы создаются в виде независимых _классов_ и присваиваются атрибутам классов в точности как функции методов. Подобно любым другм атрибутам классов они наследуются подклассами и экземплярами. Их методы перехвата доступа снабжаются как `self` для экземпляра самого дескриптора, так и экземпляром клиентского класса, чей атрибут ссылается на объект дескриптора. По указанной причине они могут предохранять собственную информацию состояния, а также информацию состояния экземпляра клиентского класса. Например, дескриптор может вызывать метод, доступные в клиентском классе, плюс определенные в нем методы, специфичные для дескриптора.  

Как и свойство, дескриптор управляет одним конкретным атрибутом. Хотя дескриптор не способен перехватывать все операции доступа к атрибуту обобщенным образом, он обеспечивает контроль над операциями извлечения и присваивания и позволяет свободно изменять имя атрибута с простого хранилища данных на вычисляемый атрибт, не нарушая рабоут существующего кода.  

В отличие от свойств дескрипторы охватвают более широкие границы и предоставляют более универсальный инструмент. Скажем, из-за того, что дескрипторы реализуются как нормальные классы, они имеют собственное состояние, способны принимать участие в иерархиях наследования дескрипторов, могут применять композицию для агрегирования объектов и предлагают естественную структуру для написания кода внутренних методов и строк документации по атрибутам.  
### __Основы__ 
Дескрипторы реализуются как отдельные классы и предоставляют особым образом именованные методы доступа для операций доступа к атрибутам, которые они перехватвают. Методы извлечения, устновки и удаления автоматически запускаются, когда в отношении атрибута, которому присвоен экземпляр класса дескриптора, выполняется соответствующая операция доступа.
```python
class Descriptor:
    "doc string goes here"
    def __get__(self, instance, owner): ...

    def __set__(self, instance, value): ...
    
    def __delete__(self, instance): ...
```  

Классы с любым таким методом считаются дескрипторами, а их методы будут специальными, когда один из их экземпляров присваивается атрибуту другого класса - при доступе к атрибуту методы автоматически вызываются. Если любой из методов отсутствует, то обычно это значит, что соответствующий вид доступа не поддерживается. Тем не менее, в отличие от свойств пропуск `__set__` повзоляет присваивать значение имени атрибута дескрипора и, следовательно, переопределять его в экземпляре, тем самым _скрывая_ дескриптор- чтобы сделать атрибут, допускащим _только чтение_, потребуется определить `__set__` для перехвата операций  присваения и генерации исключения.
### __Аргументы методов дескриптора__  
Все три метода дескриптора принимают экземпляр класса дескриптора (`self`) и экземпляр клиентского класса, к которому присоединен экземпляр дескриптора (`instance`).  
Метод доступа `__get__` принимает аргумент, указывающий класс, к которому писоединен экземпляр дескриптора. Его аргумент `instance` представляет собой либо экземпляр, через который осуществляется доступ к атрибуту (для `instance.attr`), либо `None`, когда досутп к атрибутов производится напрямую через класс владельца (для `class.attr`). Первый вариант обычно вычисляет значение для опеации досупа через экземлпяр, а второй обычно возвращает `self`, если поддерживается доступ к объекту дескриптора.  

```python


class Descriptor:
    "doc string goes here"
    def __get__(self, instance, owner):
        print(self, instance, owner, sep="\n")


class Subject:
    attr = Descriptor()

def test():
    ex = Subject()
    ex.attr             #ex.attr -> Descriptor.__get__(Subject.attr, ex, Subject)
```  

Дескриптору известно, что к нему обращаютстя напрямую, если его аргументом экземпляра окащывается `None`.
### __Дескрипторы атрибутов только для чтения__  
В отличие от своств, отсутствие метода `__set__` в дескрипторе недостаточно для того, чтобы сделать атрибут допускающим только чтение, т.к. имени дескриптора может быть присвоено значение в экземпляре.  
```python


class D: 
    def __get__(*args):
        print("get")


class C:
    a = D()


def test():
    ex = C()
    ex.a            #>>>get
    C.a             #>>>get
    ex.a = 99       #Сохраняется в ex, скрывая C.a
    ex.a            #>>>99
```  

Чтобы сделать основанный на дескрипторе атрибут допускающим только чтение, необходимо перехватывать операцию присваения в классе дескриптора и генерировать исключение для предотвращения присвоения атрибту.  
```python


class D: 
    def __get__(*args):
        print("get")
    
    def __set__(*args):
        raise AttributeError


class C:
    a = D()


def test():
    ex = C()
    ex.a            #>>>get
    C.a             #>>>get
    ex.a = 99       #Сохраняется в ex, скрывая C.a
    ex.a            #>>>AttributeError
```  
### __Пример__  
```python


class Name:
    def __get__(self, instance, owner):
        print("fetch...")                   #Извлечение
        return instance._name
    
    def __set__(self, instance, value):
        print("change")                     #Изменение
        instance._name = value

    def __delete__(self, instance):
        print("remove")
        del instance._name


class Person:
    def __init__(self, name):
        self._name = name
    
    name = Name()


def test():
    bob = Person("Bob Kotik")
    print(bob.name)
    bob.name = "Robert"
    print(bob.name)
    del bob.name
```
> Дескриптор обязн присваиваться атрибут __класса__. Дескриптор не будет работать, если взамен присвоить его атрибуту экземпляра `self`

Экземпляр класса дескриптора является атрибутом клиентского класса, а потому наследуется всеми экземплярами класса и его подклассов.  
### __Вычисляемые атрибуты__  
На практике дескрипторы могут применяться для вычисления значений атрибутов при каждом извлечении.
```python


class DecsSquare:
    def __init__(self, value):              #Каждый дескриптор имеет свое состояния
        self.value = value
    
    def __get__(self, instance, owner):     #При извлечении атрибта
        return self.value ** 2 
    
    def __set__(self, instance, value):     #При присваении атрибута
        self.value = value                  #Операция удаления и строка документации отсутствуют


class Client1:
    X = DecsSquare(3)


class Client2:
    X = DecsSquare(4)


def test():
    ex1 = Client1()
    ex2 = Client2()
    print(f"ex1 : {ex1.X}")     #3**2 = 9
    ex1.X =5 
    print(f"ex1 : {ex1.X}")     #5**2 = 25
    print(f"ex2 : {ex2.X}")     #4**2 = 16
```  
### __Использование информации состояния в дескрипторах__  
Фактически дескрипторы могут использовать состояние экземпляра клиентского класса и сотояние самого дескриптора, а также любое их сочетание.  
* Состояние дескриптора применяется для управления либо данными, используемыми при внутренней работе дескриптора, либо данными, которые охватывают все экземпляры. Оно может варьироваться в зависимости от места пявления атрибута (часто в зависимости от клиентского класса).
* Состояние экземпляра хранит информацию, связанную и возможно созданную клиентским классом. Оно может варьироваться в зависимости от экземпляра.  
Другими словами, состояние дескриптора представляет собой данные для каждого дескриптора, а состояние экземпляра- данные для каждого экземпляра клиента.  

Для дескриптора вполне реально хранить или применять атрибт, присоединенный к экземпляру клиентского класса, а не к самому себе:
```python


class InstState:
    def __get__(self, instance, owner):
        print("InstState get")
        return instance._X * 10

    def __set__(self, instance, value):
        print("InstState set")
        instance._X = value


class CalcAttrs:        #Клиентский класс
    X = InstState()     #Атрибут класса дескриптора
    Y = 3               #Атрибут класса
    def __init__(self):
        self._X = 2     #Атрибут экземпляра
        self.Z = 4      #Атрибут экземпляра


def test():
    ex = CalcAttrs()
    print(ex.X, ex.Y, ex.Z)     #X вычисляется, остальные нет
    ex.X = 5                    #Присваение X перехватывается
    CalcAttrs.Y = 6             #Y повторно присваивается в классе
    ex.Z = 7                    #Z присваивается в экземпляре
    print(ex.X, ex.Y, ex.Z)
    ex2 = CalcAttrs()           #X и Z отличабтся от ex
    print(ex2.X, ex2.Y, ex2.Z)
```  
Но также дескриптор может присоединять информацию к собственному экземпляру:
```python


class DescState:
    def __init__(self, value):
        print("Init!")
        self.value = value

    def __get__(self, instance, owner):
        print("DescState get")
        return self.value * 10

    def __set__(self, instance, value):
        print("DescState set")
        self.value = value


class CalcAttrs:        #Клиентский класс
    X = DescState(2)     #Атрибут класса дескриптора
    Y = 3               #Атрибут класса
    def __init__(self):
        self.Z = 4      #Атрибут экземпляра


def test():
    ex = CalcAttrs()
    print(ex.X, ex.Y, ex.Z)     #X вычисляется, остальные нет
    ex.X = 5                    #Присваение X перехватывается
    CalcAttrs.Y = 6             #Y повторно присваивается в классе
    ex.Z = 7                    #Z присваивается в экземпляре
    print(ex.X, ex.Y, ex.Z)
    ex2 = CalcAttrs()           #X использует разделяемые данные подобно Y
    print(ex2.X, ex2.Y, ex2.Z)
```  
## `__getattr__` и `__getattribute__`  
Методы `__getattr__` и `__getattribute__` перехватывают доступ к произвольным именам (в отличие от свойств и дескрипторов), поэтому они применяются в более широких ролях вроде делегирования.  

Перехват извлечения атрибутов имеет две разновидности, реализуемые посредством двух разных методов:
* Метод `__getattr__` запускается для неопределенных атрибутов (атрибут не зранится в экземпляре и не наслеуется).
* Метод `__getattribute__` запускается для каждого атрибута (может вызвать рекурсивный цикл).  

Эти два метода являеются представителями набора методов перехвата атрибутов, куда также входят `__setattr__` и `__delattr__`.   

Эти методы относятся к универсальному протоколу _перегрузки операций_.  

Эти методы могут использоваться для реализации объектов оболочек, которые управляют всем доступом к внутреннему объекту.  
### __Основы__  
```python
def __getattr__(self, name) : # При извлечении неопределенных атрибутов [obj.паше]
def __getattribute__ (self, name) : # При извлечении всех атрибутов [obj .name]
def __setattr__ (self, name, value) : # При присваивании всех атрибутов [obj .name=value]
def __delattr__(self, name) : # При удалении всех атрибутов [del obj.name]
```  

Два метода извлечения обычно возвращают значение атрибута, другие два ничего не возвращают (`None`).  

### __Пример__  

```python


class Person:
    def __init__(self, name):
        self._name = name                   #Запускается __setattr__
    
    def __getattr__(self, attrname):        #При obj.[неопределенный_атрибут]
        print(f"get {attrname}")
        if attrname == "name":              #Перехват имени name: не хранится в экземпляре
            return self._name               #Зацикливания нет: реальный атрибут
        else:
            raise AttributeError            #Остальные являются ошибками
    
    def __setattr__(self, attrname, value): #При obj.[любой_атрибут] = value
        print(f"set {attrname}")
        if attrname == "name":
            attrname = "_name"              #Установка внутреннего имени
        self.__dict__[attrname] = value     #Избегание зацикливания

    def __delattr__(self, attrname):        #При del obj.[любой_атрибут]
        print(f"del {attrname}")
        if attrname == "name":
            attrname = "_name"
        del self.__dict__[attrname]         #Избегание зацикливания


def test():
    ex = Person("Evgeniy Chernetckiy")      #Экземпляр ex имеет управляемый атрибут
    print(ex.name)                          #Запускается __getattr__
    ex.name = "Python developer"            #Запускается __setattr__
    print(ex.name)
    del ex.name                             #Запускается __delattr__
```  
В отличие от дескрипторов и свойств здесь отсутствует прямое понятие указания документации для нашего атрибута; управляемые атрибуты при таком подходе существуют только внутри методов перехвата, а не как отдельные объекты.  
### __Использование `__getattribute__`__  

Для получения точно таких же результатов посредством `__getattribute__` заменяется метод `__ getattr__` в примере приведенным далее кодом; поскольку он перехватывает извлечение всех атрибутов, в данной версии необходимо избегать зацикливания, передавая новые операции извлечения суперклассу, и в целом нельзя
предполагать, что неизвестные имена являются ошибками:  
```python
def __getattribute__(self, attrname):   #При obj.[любой_атрибут]
        print(f"get {attrname}")
        if attrname == "name":              #Перехват всех имен
            attrname = "_name"              #Отображение на внутреннее имя
        return object.__getattribute__(self, attrname)  #Избегание зацикливания
```  

Запуск кода после внесения изменений дает похожий вывод, но мы имеем добавочный вызов `__getattribute__` для операции извлечения в `__setattr__` (в первый
раз возникшей в __ init__).  

### __Вычисляемые атрибуты__  

```python


class AttrSquare:
    def __init__(self, value):
        self._value = value                 #Запускается __setattr__!
    
    def __getattr__(self, attrname):        #При операции извлечения неопределенных атрибутов
        if attrname == "value":
            return self._value ** 2
        else:
            raise AttributeError
    
    def __setattr__(self, attrname, value): #При операции присваения всех атрибутов
        if attrname == "value":
            attrname = "_value"
        self.__dict__[attrname] = value
    

def test():
    ex = AttrSquare(2)
    print(ex.value)                         #2 ** 2 = 4
    ex.value = 5                            #__setattr__
    print(ex.value)                         #5 ** 2 = 25
```  

__Использование `__getattribute__`__  

```python


class AttrSquare:
    def __init__(self, value):
        self._value = value                 #Запускается __setattr__!
    
    def __getattribute__(self, attrname):   #При операции извлечения любого атрибута
        if attrname == "value":
            return self._value ** 2         #Снова запускается __getattribute__
        else:
            return object.__getattribute__(self, attrname)
    
    def __setattr__(self, attrname, value): #При операции присваения всех атрибутов
        if attrname == "value":
            attrname = "_value"
        object.__setattr__(self, attrname, value)
    

def test():
    ex = AttrSquare(2)
    print(ex.value)                         #2 ** 2 = 4
    ex.value = 5                            #__setattr__
    print(ex.value)                         #5 ** 2 = 25
```  

В методах класса происходит неявное перенаправление:
* `self._value = value` внутри конструктора запускает `__setattr__`;
* `self.value` внутри `__getattribute__` снова запускает `__getattribute__`.  

### __Сравнение `__getattr__` и `__getattribute__`__  


```python


class GetAttr:
    attr1 = 1
    def __init__(self):
        self.attr2 = 2
    
    def __getattr__(self, attrname):
        if attrname == "attr3":
            return 3
        else:
            raise AttributeError


class GetAttribute:
    attr1 = 1
    def __init__(self):
        self.attr2 = 2
    
    def __getattribute__(self, attrname):
        print(f"get {attrname}")
        if attrname == "attr3":
            return 3
        else:
            return object.__getattribute__(self, attrname)


def test():
    ex1 = GetAttr()
    ex2 = GetAttribute()
    print(ex1.attr1)
    print(ex1.attr2)
    print(ex1.attr3)
    print("*" * 20)
    print(ex2.attr1)
    print(ex2.attr2)
    print(ex2.attr3)
```  
* Версия с `__getattr__` перехватывает только операции извлечения атрибута `attr3` т.к. он не определен.
* Версия с `__getattribute__` перехватывает операции извлечения всех атрибутов и обязана направлять те, которыми она не управляет, методу извлечения из суперкласса, чтобы избежать появления циклов.  

### __Сравнение методик управления__  

Свойства:  

```python


class Powers:
    def __init__(self, square, cube):
        self._square = square
        self._cube = cube
    
    def getSquare(self):
        return self._square ** 2
    
    def setSquare(self, value):
        self._square = value
    
    def getCube(self):
        return self._cube ** 3
    
    def setCube(self, value):
        self._cube = value

    square = property(getSquare, setSquare)
    cube = property(getCube, setCube)


def test():
    ex = Powers(2, 3)
    print(ex.square)    #4
    print(ex.cube)      #27
    ex.square = 3
    ex.cube = 2
    print(ex.square)    #9
    print(ex.cube)      #8
```  

Свойства через декораторы:  

```python


class Powers:
    def __init__(self, square, cube):
        self._square = square
        self._cube = cube

    @property
    def square(self):
        return self._square ** 2
    
    @square.setter
    def square(self, value):
        self._square = value

    @property
    def cube(self):
        return self._cube ** 3
    
    @cube.setter
    def cube(self, value):
        self._cube = value


def test():
    ex = Powers(2, 3)
    print(ex.square)    #4
    print(ex.cube)      #27
    ex.square = 3
    ex.cube = 2
    print(ex.square)    #9
    print(ex.cube)      #8
```  

Дескрипторы:  

```python


class DecsSquare:
    def __get__(self, instance, owner):
        return instance._square ** 2
    
    def __set__(self, instance, value):
        instance._square = value


class DecsCube:
    def __get__(self, instance, owner):
        return instance._cube ** 3

    def __set__(self, instance, value):
        instance._cube = value


class Powers:
    square = DecsSquare()
    cube = DecsCube()

    def __init__(self, square, cube):
        self._square = square
        self._cube = cube

    
def test():
    ex = Powers(2, 3)
    print(ex.square)    #4
    print(ex.cube)      #27
    ex.square = 3
    ex.cube = 2
    print(ex.square)    #9
    print(ex.cube)      #8
```  

`__getattr__`  

```python  


class Powers:
    def __init__(self, square, cube):
        self._square = square
        self._cube = cube

    def __getattr__(self, attrname):
        if attrname == "square":
            return self._square ** 2
        elif attrname == "cube":
            return self._cube ** 3
        else:
            raise AttributeError

    def __setattr__(self, attrname, value):
        if attrname == "square":
            attrname = "_square"
        elif attrname == "cube":
            attrname = "_cube"
        self.__dict__[attrname] = value


def test():
    ex = Powers(2, 3)
    print(ex.square)    #4
    print(ex.cube)      #27
    ex.square = 3
    ex.cube = 2
    print(ex.square)    #9
    print(ex.cube)      #8
```  

`__getattribute__`  

```python



class Powers:
    def __init__(self, square, cube):
        self._square = square
        self._cube = cube

    def __getattribute__(self, attrname):
        if attrname == "square":
            return self._square ** 2
        elif attrname == "cube":
            return self._cube ** 3
        else:
            return object.__getattribute__(self, attrname)
    
    def __setattr__(self, attrname, value):             #Демонстрируются разные методы
        if attrname == "square":
            self.__dict__["_square"] = value            #Вызывает __getattribute__
        elif attrname == "cube":
            object.__setattr__(self, "_cube", value)    #Через __setattr__ суперкласса
        else:
            object.__setattr__(self, attrname, value)


def test():
    ex = Powers(2, 3)
    print(ex.square)    #4
    print(ex.cube)      #27
    ex.square = 3
    ex.cube = 2
    print(ex.square)    #9
    print(ex.cube)      #8
```  

### __Перехват атрибутов для встроенных операций__  

Методы `__getattr__` и `__getatribute__` перехватывают операции извлечения соответственно неопределенных и всех атрибутов. Сказанное справедливо для в отношении _нормально именованных_ и _явно вызываемых_ атрибутов. Для атрибутов имен методов, неявно извлекаемых _встроенными_ операциями, такие методы могут _вообще не запускаться_. Это означает, что вызовы методов перегрузки операций не могут делегироватсья внутренним объектам, если только классы оболочек самостоятельно не переопределяют данные методы.  

Операции извлечения атрибутов ждя методов `__str__`, `__add__` и `__getitem__`, запускаемы неявно, в Python 3.X не направляются методам перехвата атрибутов. В Python 3.X ни `__getattr__`, ни `__getatribute__` не знапускаются для таких атрибутов. __Данные атрибуты ищутся сразу в классе__.  

__Пример__:  

```python


class GetAttr:
    eggs = 88
    def __init__(self):
        self.spam = 77
    
    def __len__(self):
        print("__len__: 42")
        return 42
    
    def __getattr__(self, attrname):
        print(f"getattr: {attrname}")
        if attrname == "__str__":
            return lambda *args: "[Getattr str]"
        else:
            return lambda *args: None


class GetAttribute:
    eggs = 88
    def __init__(self):
        self.spam = 77

    def __len__(self):
        print("__len__: 42")
        return 42

    def __getattribute__(self, attrname):
        print(f"getattribute {attrname}")
        if attrname == "__str__":
            return lambda *args: "[Getattribute str]"
        else:
            return lambda *args: None


def test():
    for _class in (GetAttr, GetAttribute):
        print(_class.__name__)
        ex = _class()
        ex.eggs
        ex.spam
        ex.other
        len(ex)
        try: ex[0]
        except: print("fail []")

        try: ex + 99
        except: print("fail +")

        try: ex()
        except: print("fail ()")

        ex.__call__()
        print(ex.__str__())
        print(ex)
```  

__Вывод__:  

```
GetAttr
getattr: other
__len__: 42
fail []
fail +
fail ()
getattr: __call__
<__main__.GetAttr object at 0x0000019F75A38FA0>
<__main__.GetAttr object at 0x0000019F75A38FA0>
GetAttribute
getattribute eggs
getattribute spam
getattribute other
__len__: 42
fail []
fail +
fail ()
getattribute __call__
getattribute __str__
[Getattribute str]
<__main__.GetAttribute object at 0x0000019F75A38D60>
```
## Пример: проверка достоверности атрибутов  

Для каждой методики используется одинаковый тест

### __Использование свойтсв для проверки достоверности__  

Важно: присвоение в методе `__init__` также вызывает свойство!

```python


class CardHolde:
    acctlen = 8
    retireage = 59.5
    def __init__(self, acct, name, age, addr):
        self.acct = acct
        self.name = name
        self.age = age
        self.addr = addr
    
    @property
    def name(self):
        return self.__name
    
    @name.setter
    def name(self, value):
        value = value.lower().replace(" ", "_")
        self.__name = value

    @property
    def age(self):
        return self.__age

    @age.setter
    def age(self, value):
        if value < 0 or value > 150:
            raise ValueError("invalid age")
        else:
            self.__age = value

    @property
    def acct(self):
        return self.__acct[:-3] + "***"
    
    @acct.setter
    def acct(self, value):
        value = value.replace("-", "")
        if len(value) != self.acctlen:
            raise TypeError("invalid acct number")
        else:
            self.__acct = value

    @property
    def remain(self):
        return self.retireage - self.age


def printholder(who):
    print(who.acct, who.name, who.age, who.remain, who.addr, sep="/")


def test():
    bob = CardHolde("1234-5678", "Bob Smith", 40, "123 main st")
    printholder(bob)
    bob.name = "Bob Q. Smith"
    bob.age = 50
    bob.acct = "23-45-67-89"
    printholder(bob)

    sue = CardHolde("5678-12-34", "Sue Jines", 35, "124 main st")
    printholder(sue)
    try:
        sue.age = 200
    except:
        print("Bad age for Sue")

    try:
        sue.remain = 5
    except:
        print("Cant set sue.remain")
    
    try:
        sue.acct = "1234567"
    except:
        print("Bad acct for Sue")
```  

__Вывод__:  

```
12345***/bob_smith/40/19.5/123 main st
23456***/bob_q._smith/50/9.5/123 main st
56781***/sue_jines/35/24.5/124 main st
Bad age for Sue
Cant set sue.remain
Bad acct for Sue
```  

### __Использование дескрипторов для проверки достоверности__  

В отличие от свойств дескрипторы могут также иметь собственое состояние и являются универсальной схемой.

Важно: операции присваения значения атрибутам внутри метода конструктора `__init__` запускают методы `__set__` дескриптора. Также важным является тот факт, что в этой версии необходимо перехватывать операции присваения значений имени `remain` в его дескрипторе и генерировать исключение, иначе операция присваения этому атрибуту молча создаст атрибут экземпляра, котоырй скроет дескриптор атрибута класса.  

__Вариант 1: прверка достоверности с помощью разделяемого состояния экземпляра дескриптора:__  

```python


class Name:
    def __get__(self, instance, owner):
        return self.name

    def __set__(self, instance, value):
        self.name = value.lower().replace(" ", "_")


class Age:
    def __get__(self, instance, owner):
        return self.age
    
    def __set__(self, instance, value):
        if value < 0 or value > 150:
            raise ValueError("invalid age")
        else:
            self.age = value

class Acct:
    def __get__(self, instance, owner):
        return self.acct[:-3] + "***"

    def __set__(self, instance, value):
        value = value.replace("-", "")
        if len(value) != instance.acctlen:
            raise TypeError("invalid acct")
        else:
            self.acct = value


class Remain:
    def __get__(self, instance, owner):
        return instance.retireage - instance.age

    def __set__(self, instance, value):
        raise AttributeError("cannot set remain")
    


class CardHolde:
    acctlen = 8
    retireage = 59.5

    def __init__(self, acct, name, age, addr):
        self.acct = acct
        self.name = name
        self.age = age
        self.addr = addr
    
    acct = Acct()
    name = Name()
    age = Age()
    remain = Remain()


def printholder(who):
    print(who.acct, who.name, who.age, who.remain, who.addr, sep="/")


def test():
    bob = CardHolde("1234-5678", "Bob Smith", 40, "123 main st")
    printholder(bob)
    bob.name = "Bob Q. Smith"
    bob.age = 50
    bob.acct = "23-45-67-89"
    printholder(bob)

    sue = CardHolde("5678-12-34", "Sue Jines", 35, "124 main st")
    printholder(sue)
    try:
        sue.age = 200
    except:
        print("Bad age for Sue")

    try:
        sue.remain = 5
    except:
        print("Cant set sue.remain")
    
    try:
        sue.acct = "1234567"
    except:
        print("Bad acct for Sue")
```  

__Вывод__:  

```
12345***/bob_smith/40/19.5/123 main st
23456***/bob_q._smith/50/9.5/123 main st
56781***/sue_jines/35/24.5/124 main st
Bad age for Sue
Cant set sue.remain
Bad acct for Sue
```  

__Вариант 2: проверка достоверностис помощью состояния для каждого экземпляра клиентского класса:__

```python


class Acct:
    def __get__(self, instance, owner):
        return instance.__acct[:-3] + "***"

    def __set__(self, instance, value):
        value = value.replace("-", "")
        if len(value) != instance.acctlen:
            raise ValueError("invalid acct")
        else:
            instance.__acct = value


class Name:
    def __get__(self, instance, owner):
        return instance.__name

    def __set__(self, instance, value):
        instance.__name = value.lower().replace(" ", "_")


class Age:
    def __get__(self, instance, owner):
        return instance.__age

    def __set__(self, instance, value):
        if value < 0 or value > 150:
            raise ValueError("invalid age")
        else:
            instance.__age = value


class Remain:
    def __get__(self, instance, owner):
        return instance.retireage - instance.age

    def __set__(*args):
        raise AttributeError("cannot set remain")


class CardHolde:
    acctlen = 8
    retireage = 59.5

    def __init__(self, acct, name, age, addr):
        self.acct = acct
        self.name = name
        self.age = age
        self.addr = addr
    
    acct = Acct()
    name = Name()
    age = Age()
    remain = Remain()


def printholder(who):
    print(who.acct, who.name, who.age, who.remain, who.addr, sep="/")


def test():
    bob = CardHolde("1234-5678", "Bob Smith", 40, "123 main st")
    printholder(bob)
    bob.name = "Bob Q. Smith"
    bob.age = 50
    bob.acct = "23-45-67-89"
    printholder(bob)

    sue = CardHolde("5678-12-34", "Sue Jines", 35, "124 main st")
    printholder(sue)
    try:
        sue.age = 200
    except:
        print("Bad age for Sue")

    try:
        sue.remain = 5
    except:
        print("Cant set sue.remain")
    
    try:
        sue.acct = "1234567"
    except:
        print("Bad acct for Sue")
    print(sue.__dict__)
```  

__Вывод:__

```
12345***/bob_smith/40/19.5/123 main st
23456***/bob_q._smith/50/9.5/123 main st
56781***/sue_jines/35/24.5/124 main st
Bad age for Sue
Cant set sue.remain
Bad acct for Sue
{'_Acct__acct': '56781234', '_Name__name': 'sue_jines', '_Age__age': 35, 'addr': '124 main st'}
```  

### __Использование `__getattr__` для проверки достоверности__

```python
class CardHolde:
    acctlen = 8
    retireage = 59.8

    def __init__(self, acct, name, age, addr):
        self._acct = acct
        self.name = name
        self.age = age

        self.addr = addr

    def __getattr__(self, attrname):
        if attrname == "acct":
            return self._acct[:-3] + "***"
        elif attrname == "remain":
            return self.acctlen - self.age
        else:
            raise AttributeError(attrname)
    
    def __setattr__(self, attrname, value):
        if attrname == "acct":
            attrname = "_acct"
            value = value.replace("-", "")
            if len(value) != self.acctlen:
                raise ValueError("invalid acct")
        elif attrname == "name":
            value = value.lower().replace(" ", "_")
        elif attrname == "age":
            if value < 0 or value > 150:
                raise ValueError("invalid age")
        elif attrname == "remain":
            raise AttributeError("cannot set remain")
        self.__dict__[attrname] = value


def printholder(who):
    print(who.acct, who.name, who.age, who.remain, who.addr, sep="/")


def test():
    bob = CardHolde("1234-5678", "Bob Smith", 40, "123 main st")
    printholder(bob)
    bob.name = "Bob Q. Smith"
    bob.age = 50
    bob.acct = "23-45-67-89"
    printholder(bob)

    sue = CardHolde("5678-12-34", "Sue Jines", 35, "124 main st")
    printholder(sue)
    try:
        sue.age = 200
    except:
        print("Bad age for Sue")

    try:
        sue.remain = 5
    except:
        print("Cant set sue.remain")
    
    try:
        sue.acct = "1234567"
    except:
        print("Bad acct for Sue")
    print(sue.__dict__)
```  

__Вывод:__  

```
1234-5***/bob_smith/40/-32/123 main st
23456***/bob_q._smith/50/-42/123 main st
5678-12***/sue_jines/35/-27/124 main st
Bad age for Sue
Cant set sue.remain
Bad acct for Sue
{'_acct': '5678-12-34', 'name': 'sue_jines', 'age': 35, 'addr': '124 main st'}
```  

### __Использование `__getattribute__` для проверки достоверности__  

```python


class CardHolde:
    acctlen = 8
    retireage = 59.5

    def __init__(self, acct, name, age, addr):
        self.acct = acct
        self.name = name
        self.age = age

        self.addr = addr

    def __getattribute__(self, attrname):
        if attrname == "acct":
            return object.__getattribute__(self, "acct")[:-3] + "***"
        elif attrname == "remain":
            return object.__getattribute__(self, "retireage") - object.__getattribute__(self, "age")
        else:
            return object.__getattribute__(self, attrname)
    
    def __setattr__(self, attrname, value):
        if attrname == "acct":
            #attrname = "_acct" Не требуется, т.к. обрабадывается каждый запрос
            value = value.replace("-", "")
            if len(value) != self.acctlen:
                raise ValueError("invalid acct")
        elif attrname == "name":
            value = value.lower().replace(" ", "_")
        elif attrname == "age":
            if value < 0 or value > 150:
                raise ValueError("invalid age")
        elif attrname == "remain":
            raise AttributeError("cannot set remain")
        self.__dict__[attrname] = value


def printholder(who):
    print(who.acct, who.name, who.age, who.remain, who.addr, sep="/")


def test():
    bob = CardHolde("1234-5678", "Bob Smith", 40, "123 main st")
    printholder(bob)
    bob.name = "Bob Q. Smith"
    bob.age = 50
    bob.acct = "23-45-67-89"
    printholder(bob)

    sue = CardHolde("5678-12-34", "Sue Jines", 35, "124 main st")
    printholder(sue)
    try:
        sue.age = 200
    except:
        print("Bad age for Sue")

    try:
        sue.remain = 5
    except:
        print("Cant set sue.remain")
    
    try:
        sue.acct = "1234567"
    except:
        print("Bad acct for Sue")
    print(sue.__dict__)
```  

__Вывод:__  

```
12345***/bob_smith/40/19.5/123 main st
23456***/bob_q._smith/50/9.5/123 main st
56781***/sue_jines/35/24.5/124 main st
Bad age for Sue
Cant set sue.remain
Bad acct for Sue
{'acct': '56781234', 'name': 'sue_jines', 'age': 35, 'addr': '124 main st'}
```