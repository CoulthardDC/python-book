# Декораторы  

Декорирование представляет собой указания управляющего или дополняющего кода для функции и классов. Сами декораторы принимают вид вызываемых объектов (напримерб функций), обрабатывающих другие вызываемые объекты. Декораторы имеют две связанные друг с другом формы:  

* Декораторы функций делают повторное привязывание имен во время определения функций, предоставляя уровень логики, который может управлять функциями и методами или последующими обращениями к ним.
* Декораторы классов делают повторное привязывание имен во время определения классов, предоставляя уровень логики, который может управлять классами и экземплярами, созднными при последующих обращениях к классам.  

Т.е. декораторы предоставляют способ вставки автоматически запускаемого кода в конце операторов определения функции и классов - в конце `def` для декораторов функций и в конце `class` для декораторов классов.  

## Управление вызовами и экземплярами  

В типичной ситуации автоматически запускаемый код может применяться для дополнения обращений к функциям и классам. Это организуется за счет ввода в действие объектов оболочек (известных как _посредники_), предназначенных для вызова в более позднее время.  


* **_Посредники вызовов_**

  Декораторы функций вводят в действие объекты оболочек, которые позволяют перехватывать последующие _вызовы функций_ и обрабатывают их по мере надобности, обычно передавая сами вызовы исходной функции для выполнения управляемого действия.  

* **_Посредники интерфейсов_**  

  Декораторы классов вводят в действие объекты оболочек, которые позволяют перехватывать последующие _вызовы для создания экземпляров_ и при необходимости обрабатывать их, обычно передавая сами вызовы исходному классу для создания управляемого экземпляра.  

  Декораторы достигают таких эффектов за счет автоматической повторной привязки имен функций и классов к другим вызываемым объектам в конце операторов `def` и `class`. При обращении в более позднее время привязанные вызываемые объекты могут выполнять задачи наподобие отсеживания и измерения времени вызовов функций, управления доступом к атрибутам экземпляров и т.д.  

## Управления функциями и классами  

Хотя чаще всего необходимо использование объектов оболочек для перехвата последующих обращений к функциям и классам, это не единственный способ применения декораторов.  

* **_Администраторы функций_**  
  Декораторы функций могут использоваться для управления _объектами функций_ взамен или в дополнение к их последующим вызовам, скажем, чтобы регистрировать функции в API-интерфейсах.  

* **_Администраторы классов_**  
  Декораторы классов могут использоваться для непосредственного управления _объектами классов_ взамен или в дополнение к вызовам, создающим экземпляры, например, чтобы дополнить класс новыми методами. Такая роль плотно пересекается с ролью _метаклассов_ (в разделе о них есть дополнение). Оба инструмента зпускаются в конце процесса создания классов.  

## Основы  

## __Декораторы функций__  

Декоратор записывается перед оператором def, определяющим функцию или метод, и состоит из символа `@`, за которым находится ссылка на _метафункцию_ - функцию (или другой вызываемый объект), управляющую другой функцией.  

```python
@decorator    #Декорирование функции
def F(arg):
  ...
F(99)         #Вызов функции
```
Эквивалентно:  

```python
def F(arg):
  ...
F = decorator(F)  #Повторная привязка имени функции к результату декоратора
F(99)             #Вызывается декоратор decorator(F)(99)
```

`decorator`- это вызываемый объект с одним аргументом, который возвращает вызываемый объект с таким же количеством аргументов, как у функции F (если не саму F).

### Реализация  

Декоратор является _вызываемым объектом, возвращающим вызываемый объект_.  

Например, чтобы использовать протокол декорирования для управления функцией сразу после ее создания, можно реализовать декоратор такого вида:  

```python
def decorator(F):
  #Обработка функции F
  return F
@decorator
def func():...    #func = decorator(func)
```

В более типичной ситуации для вставки логики перехвата последующих обращений к функции можно реализовать декоратор, который возвращает объект, отличающийся от исходной функции - посредник для вызова в более позднее время:  

```python
def decorator(F):
  #Сохранение либо использование функции F
  #Возвращение другого вызываемого объекта:
  #вложенного def, класса с __call__ и т.д.
@decorator
def func(): ...   #func = decorator(func)
``` 

Декоратор вызывается во время декорирования, а вызываемый объект, который он возвращает, вызывается при обращении к исходному имени функции в будущем. Сам декоратор принимает декорированную функцию; возвращенный вызываемый объект принимает любые аргументы, переданные позже имени декорированной функции. При надлежащей реализации все работает аналогично методам уровня класса: в первом аргументе возвращенного вызываемого объекта оказывается подразумеваемый объект экземпляра.  
Пример ниже воплозает эту идею: декоратор возвращает объект-оболочку, который предозраняет исходную функцию в объемлющей области видимости:  

```python
def decorator(F):
  def wrapper(*args):
    F(*args)      #Использование функции F и аргументов args
  return wrapper

@decorator          #func = decorator(func)
def func(x, y): ... #func передается функции F декоратора
...
func(6, 7)          #6, 7 передается аргументу *args оболчки
```  

Когда происходит обращение к имени `func`, в действительности вызывается функция `wrapper`, возвращенная декоратором; функция `wrapper` затем может запустить исходную функцию `func`, т.к. она по прежнему доступна в _объемлющей обалсти видимости_. При реалихации подобным образом каждая декорированная функция создает новую область видимости для перехвата состояния.  

Чтобы сделать тоже самое с помощью классов, необходимо перегрузить метод `__call__` и применить атрибуты экземпляра вместо объемлющей области видимости:  

```python
class decorator:
  def __init__(self, func):     #При декорировании
    self.func = func
  
  def __call__(self, *args):    #При вызове внутренней функции
    self.func(*args)            #Вызывает исходную функцию


@decorator
def func(x, y): ...             #func = decorator(func)
                                #func передается self
func(6, 7)                      #Вызов метода __call__; 6, 7 передаются методу с помощью *args
```  

Когда происходит обращение к имени `func`, вызывается метод `__call__` экземпляра, созданного декоратором `decorator`. Метод `__call__` затем запускает исходную функцию `func`, потому что она доступна в атрибуте экземпляра. В случае такой реализациикаждая декорированная функция выпускает новый экземпляр для предохранения состояния.  

### __Поддержка декорирования методов__  

С реализацией, основанной на классе, связан один важный момент. Она не работает в случае применения к функциям методов на уровне класса:  

```python

class decorator:
  def __init__(self, func):     #func - метод без экземпляра
    self.func = func

  def __call__(self, *args):    #self - экземпляр декоратора
    self.func(*args)            #self.func(*args) терпит неудачу! Экземпляр C не находится в *args


class C:
  @decorator
  def method(self, x, y): ...
```  

Проблема заключается в том, что аргумент `self` в методе `__call__` декоратора получает экземпляр класса `decorator`, когда декоратор вызывается позже, и экземпляр класса `C` никогда не помещается в `*args`.В результате направление вызова исходному методу становится невозможным — объект декоратора хранит исходную функцию метода, но не располагает экземпляром, чтобы ей передать.  

Для поддержки и функций, и методов лучше подойдет альтернатива в виде вложенной функции:  

```python


def decorator(func):
    def wrapped(*args):
        return func(*args)
    return wrapped


class SomeClass:
    @decorator
    def func(self, arg):
        return arg ** 2
```

Формально такая версия с вложенной функцией работает из-за того, что Python создает объект связанного метода и потому передает экземпляр целевого класса аргументу `self`, __только когда атрибут метода ссылается на простую функцию__. Когда взамен он ссылается на экземпляр вызываемого класса, то данный жкземпляр передается `self`, чятобы предоставить доступ к собственной информации состояния.  

## __Декораторы классов__  
Декораторы классов имеют роль, которая частично совпадает с ролью _метаклассов_, однако обеспечивают более простой способ достижения тех же целей.  

Декораторы классов имеют одинаковый синтаксис с декораторами функций. Тем не менее, вместо помещения в оболочку индивидуальных функций или методов декораторы классов управляют классами или снабжают вызовы, создающие экземпляры, дополнительной логикой, которая управляет экземплярами, созданными из класса, или дополняет их. Во второй роли они могут управлять полными объектными интерфейсами.  

### __Использование__  

```python
@decorator  #Декорирование класса
class C:
  ...
X  = C(99)  #Создание экземпляра
```  

Класс автоматически передается функции декоратора, а ее результат присваивается имени класса:  

```python
class C:
  ...
C = decorator(C)
X = C(99)
```  

Совокупный эффект зключается в том, что при следующем обращении к имени класса для создания экземпляра, будет запущег возвращенный декоратором вызываемый объект, который может обращатсья к самому исходному классу или нет.  

### __Реализация__  

Новые декораторы классов реализуются с помощью многих тех же методик, которые применялись с декораторами функций, хотя часть их могут включать в себя два уровня дополнения — для управления как вызовами, создающими экземпляры, так и доступом к интерфейсу экземпляра. Поскольку декоратор классов также является вызываемым, объектом, который возвращает вызываемый объект, его будет вполне достаточно для большинства сочетаний функций и классов. Однако как бы декоратор классов ни был реализован, его результат запускается при последующем создании экземпляра.  

Например для простого управления классом:

```python
def decorator(C):
  #Обработка класса C
  return C

@decorator
class C: ...
```  

Чтобы взамен вставить уровень оболочки, который перехватывает будущие вызовы, создающие экземпляры, возвращается другой вызываемый объект:  

```python
def decorator(C):
  #Сохранение либо использование класса С
  #Возвращение другого вызываемого объекта:
  #вложенного def, класса с __call__ и т.д.

@decortor
class C: ...
```  

Вызываемый объект, возвращаемый таким декоратором классов, обычно создает и возвращает новый экземпляр исходного класса, каким-то образом дополненный для управления его интерфейсом.  

Следующий декоратор вставляет обхект, который перехватывает доступ к неопределенным атрибутам экземпляра класса:  

```python
def decorator(cls):
    class Wrapper:
        def __init__(self, *args):
            self.wrapped = cls(*args)
        
        def __getattr__(self, name):
            return getattr(self.wrapped, name)
    return Wrapper


@decorator
class C:
    def __init__(self, x, y):
        self.attr = "spam"

def test():
    ex = C(6, 7)
    print(ex.attr)
```  

В приведенном примере декоратор повторно привязывает имя класса к другому классу, который предозраняет исходный класс в объемлющей области видимости и при обращении к исходному классу сздает и внедряет в него экземпляр. Когда позже из экземпляра извлекается какой-нибудь атрибут, операция перехватывается методом __ getattr__ объекта-оболочки и ее выполнение делегируется внедренному экземпляру исходного класса. Кроме того, каждый декорированный класс создает новую область видимости, которая запоминает исходный класс.  

Подобно декораторам функцй декораторы классов обычно реализуются в виде "фабричных" функций, создающих и вовзращающих вызываемые объекты, в форме классов, которые используют методы `__init__` или `__call__` для перехвата операций вызова, либо в виде какой-то их комбинации. Фабричные функции, как правило, предозраняют состояние в ссылках из объемлющих областей видимости, а классы - в атрибутах.  

### __Поддержка множества экземпляров__  

Как и с декораторами функций, для дкораторов классов одни комбинации типов работают лучше, чем другие. Далее приведена ошибочная альтернативка декоратора класса из предыдущего примера:  

```python
class Decorator:
    def __init__(self, C):
        print("INIT")
        self.C = C

    def __call__(self, *args):
        self.wrapped = self.C(*args)
        return self

    def __getattr__(self, attrname):
        return getattr(self.wrapped, attrname)


@Decorator                      #SomeClass = Decorator(SomeClass) -> Decortor.__init__(self, SomeClass) => SomeClass - экземпляр класса
class SomeClass:
    pass


def test():
    print("ex:")
    ex = SomeClass()            #ex = SomeClass.__call__() => Создает атрибут экземпляра класса Decorator self.wrapped
    print("ex1")
    ex1 = SomeClass()           #ex1 = Someclass.__call__() - ССЫЛАЮТСЯ С ex НА ОДИН ЭКЗЕМПЛЯР КЛАССА Decorator. => Изменяет атрибут экземпляра класса Decorator self.wrapped
```  

Данная версия терпит неудачу при обработке _множества экземпляров_ заданного класа - каждый вызов, создающий экземпляр, переписывает ранее сохраненный экземпляр. Исходная версия не поддерживает множество экземпляров, потому что каждый вызов, создающий экземпляр, создаст новый независимый объект-оболчку. В более общем случае любая из следующих схем поддерживает множество __внутренних экземпляров__:  

```python
def decorator(C):
  class Wrapper:
    def __init__(self, *args):
      self.wrapped = C(*args)
  return Wrapper


  class WrapperL ...
  def decorator(C):
    def onCall(*args):
      return Wrapper(C(*args))
    return onCall
```  

## Вложение декораторов  

Для поддержки множества вложенных шагов дополнения декораторный синтаксис позволяет добавить к декорированной функции или методу несколько уровней логики оболочки. Когда применяется такая возможность, каждый декоратор должен находиться в собственной строке. Декораторный синтаксис в форме:  

```python
@A
@B
@C
def f(...):
  ...
```  

выполняется аналогично такому синтаксису:  

```python
def f(...):
  ...
f = A(B(C(f)))
```  

Исходная функция проходит через три разных декоратора, а результирующий выхываемый объект присваивается исходному имени. Каждый декоратор обрабатывает результат предыдущего декоратора.  

Декоратор, указанный последним, применяется первым и является наиболее глубоко вложенным декоратором, когда позже поисходит вызов имени исходной функции.  

Как и в случае с декораторами функций, множество декораторов классов дают в результате множество вызовов вложенных функцй и возможно множество уровней и шагов логики оболочки вокру вызовов, создающих экземпляры. Скажем, следующий код:  

```python
@spam
@eggs
class C:
  ...
X = C()
```  

Эквивалентен такому коду:  

```python
class C:
  ...
C = spam(eggs(C))
X = C
```  

И снова каждый декоратор волен возвращать либо исходный класс, либо вставленный объект-оболочку. Благодаря объектам-оболочкам, когда в конечном итоге запрашивается экземпляр исходного класса С, вызов перенаправляется объектам уровня оболочки, предоставляемым декораторами spam и eggs, которые способны исполнять совершенно разные роли — например, они могут отслеживать и проверять допустимость доступа к атрибутам, причем оба шага выполнялись бы при будущих запросах.  

## Аргументы декораторов  
Аргументы декораторов нередко подразумевают наличие _трех уровней вызываемых объектов_: вызываемого объекта для приема аргументов декоратора, возвращающего вызываемый объект, который служит в качестве декоратора и возвращает вызываемый объект для обработки обращений к исходной функции или классу. Каждый из трех уровней может быть функцией либо классом и предохранять состояние в форме ссылок из объемлющих областей видимости или атрибутов класса.  

# Реализация декораторов функций  

Здесь содержатся рабочие примеры, кторые демонстрируют изложенные концепции декораторов.  

## Отслеживание вызовов  

Ниже определяется и применяется декоратор функции, который подсчитывает количесвто вызовов декорированной функции и для каждого вызова выводит трассировочное сообщение:

```python


class tracer:
    def __init__(self, func):
        self.func = func
        self.calls = 0

    def __call__(self, *args):
        self.calls += 1
        print(f"call {self.calls} to {self.func.__name__}")
        self.func(*args)

@tracer                 #spam = tracer(spam)
def spam (a, b, c):
    print(a + b + c)

spam(1, 2, 3)
spam("a", "b", "c")
print(spam.calls)
print(spam)
```  
__Важно__: каждая функция, декорированная классом `tracer` , будет создавать новый экземпляр с собственным объектом созраненной функции и счетчиком вызовов.  

## Варианты предохранения состояния для декораторов  

В последнем примере предыдущего раздела поднята важная проблема. Декораторы функций предлагают разнообразные варианты предохранения информации состояния, предоставленной во время декорирования, для применения в течение фактического вызова функции. Как правило, они должны поддерживать множество декорированных объектов и множество вызовов, но есть несколько способов достижения таких целей: для предохранения состояния можно использовать атрибуты экземпляров, глобальные переменные, нелокальные переменные замыканий и атрибуты функций.  

### __Атрибуты экземпляров классов__  

Ниже приводится расширенная версия предыдущего примера с добавленной поддержкой _ключевых_ аргуметов посредством синтаксиcа  `**`, которая к тому же _возвращает_ результат внутренней функции для охвата большего числа сценариев применения.  

```python


class tracer:
    def __init__(self, func):
        self.func = func
        self.calls = 0

    def __call__(self, *args, **kargs):
        self.calls += 1
        print(f"call {self.calls} to {self.func.__name__}")
        return self.func(*args, **kargs)


@tracer
def spam(a, b, c):
    print(a + b + c)


@tracer
def eggs(x, y):
    print (x ** y)


spam(1, 2, 3)
spam (a=4, b=5, c=6)

eggs(2, 16)
eggs(4, y=4)
```  

Для явного хранения состояния здесь используются _атрибуты экземпляра класса_. Внутренняя информация и счетчик вызовов предоставляют информацию _для каждого экземпляра_ - каждый случай декоррования получает собственную копию.  

Однако не смотря на удобство данной схемы, по прежнему остается недостаток с применением к методам. Позже этот недостаток будет устранен.  

### __Объемлющие области видимости и глобальные переменные__:  

_Функции замыканий_ - со ссылками из объемлющих областей видимости def и вложенными операторами def - часто могут обеспечивать тот же самый эффект, особенно для статических данных вроде декорированной исходной функции. Однако в этом примере также понадобтся счетчик в объемлющей области видимости, который _изменяется_ при каждом вызове.  

```python
def tracer(func):
    calls = 0
    def wrapper(*args, **kwargs):
        nonlocal calls
        calls += 1
        print(f"call {calls} to {func.__name__}")
        return func(*args, **kwargs)
    return wrapper


@tracer
def spam(a, b, c):
    print(a + b + c)


@tracer
def eggs(x, y):
    print(x ** y)


spam (1, 2, 3)
spam (a=4, b=5, c=6)

eggs(2, 16)
eggs(4, y=4)
```  

### __Атрибуты функций__  
Во всех версиях Python мы можем писоединить к функциям произвольные атрибуты за счет присваивания им значений с помощью выражения вида `функция.атрибут = значение`. Поскольку фабричная функция при любом вызове создаст новую функцию.  

# Декорирование методов  

```python
class tracer:
    def __init__(self, func):
        self.func = func
        self.calls = 0

    def __call__(self, *args, **kargs):
        self.calls += 1
        print(f"call {self.calls} to {self.func.__name__}")
        return self.func(*args, **kargs)
```  

В случае применения к методу класса такой декоратор терпит неудачу, т.к. `self` является экземпляром класса декоратора, а экземпляр декорированного целевого класса вообзе не помещается в `*args`.  

```python


class tracer:
    def __init__(self, func):
        self.func = func
        self.calls = 0

    def __call__(self, *args):
        self.calls += 1
        print(f"call {self.calls} to {self.func.__name__}")
        return self.func(*args)


class SomeClass:
    @tracer
    def someMethod(self, x, y):
        return x ** y


def test():
    ex = SomeClass()
    print(ex.someMethod(2, 2))
```  

__Вывод__:  

```
call 1 to someMethod
Traceback (most recent call last):
  File "c:\Users\coult\Documents\learning2\property\decorator.py", line 32, in <module>
    test()
  File "c:\Users\coult\Documents\learning2\property\decorator.py", line 28, in test
    print(ex.someMethod(2, 2))
  File "c:\Users\coult\Documents\learning2\property\decorator.py", line 11, in __call__
    return self.func(*args)
TypeError: someMethod() missing 1 required positional argument: 'y'
```  

Суть проблемы кроется в аргументе `self` метода `__call__` класса `tracer` - он является экземпляром `tracer`. В коде нам необходимы оба экземпляра: экземпляр `tracer` для состояния декоратора и экземпляр `SomeClass` для перенаправления на исходный метод. В дейсттвительности `self` обязан быть объектом `tracer`, чтобы предоставить доступ к информации состояния `tracer` (его атрибутам `calls` и `func`); это справедливо для декорирования как простой функции, так и метода.  

Когда имя декорированного метода повторно привязывается к объекту экземпляра класса с помощью `__call__`, интерпретатор Python передает в `self` только _экземпляр_ `tracer`;  

> __То есть сохраняя метод в качестве атрибута экземпляра декоратора он становится несвязанным методом (`<class 'function'>` а не `<class 'method'>`) и требует явной передачи экземпляра `self` целевого класса (класса, которому и принадлежит метод).__  

## __Использование дескрипторов для декорирования методов__  

```python


class tracer:
    def __init__(self, func):
        self.func = func
        self.calls = 0
    
    def __call__(self, *args, **kwargs):
        self.calls += 1
        print(f"call {self.calls} to {self.func.__name__}")
        return self.func(*args, **kwargs)

    def __get__(self, instance, owner):
        return wrapper(self, instance)


class wrapper:
    def __init__(self, desc, object):
        self.desc = desc
        self.obj = object

    def __call__(self, *args, **kwargs):
        return self.desc(self.obj, *args, **kwargs)


class SomeClass:
    def __init__(self, name):
        self.name = name

    @tracer
    def lastName(self):
        return self.name.split()[-1]


def test():
    ex = SomeClass("Evgeniy Chernetskiy")
    print(ex.lastName())
```  

Текущая версия декоратора tracer работает аналогично предшествующей версии с вложенными функциями, но действие варьируется в зависимости от контекста применения.  

* Декориованные функции вызывают только его метод `__call__` и никогда метод `__get__`.
* Декорированные методы вызыва. сначала его метод `__get__`, чтобы выполнить извлечение имени метода (`I.method()`); возвращенный метод `__get__` объект хранит экземпляр целевого класса и затем вызывается с целью завершения выражения вызова, тем самым запуская метод `__call__` декоратора (для()).  

Например, вызов в тестовом коде:
```python
sue.giveRaise (. 10) # Запускается__ get__ , затем__ call__
```
выполняет сначала tracer.__ get__ , потому что атрибут giveRaise в классе Person был повторно привязан к дескриптору декоратором функций методов. Затем выражение вызова запускает метод __ call__ возвращенного объекта wrapper, который в свою очередь вызывает tracer.__ call__ . Другими словами, вызовы декорированных методов инициируют процесс из четырех шагов: tracer.__ get__ , за которым
следуют три операции вызова — wrapper.__ call__ , tracer.__ call__ и в заключение исходного декорированного метода.  

В качестве альтернативы для обеспечения того же эффекта можно применить вложенную функцию:  

```python


class tracer:
    def __init__(self, func):
        self.func = func
        self.calls = 0

    def __get__(self, instance, owner):
        def wrapper(*args, **kwargs):
            return self(instance, *args, **kwargs)
        return wrapper

    def __call__(self, *args,  **kwargs):
        self.calls += 1
        print(f"call {self.calls} to {self.func.__name__}")
        return self.func(*args, **kwargs)


class SomeClass:
    
    @tracer
    def someMethod(self, x, y):
        return x ** y


def test():
    ex = SomeClass()
    print(ex.someMethod(2, 2))
```  

## Добавление аргументов к декоратору  

Декораторы могут быть более конфигурируемыми - например в универсальном инструменте может быть полезной возможность предоставления выходной метки/отключения трассировки сообщений. Пример добавления метки:  

```python


def timer(label=""):
    def decorator(func):
        def onCall(*args):
            ...
            func(*args)
            print(label)
        return onCall
    return decorator

@timer('==>')               #someFunc = timer("==>")(someFunc)
def someFunc(N): ...

someFunc(...)
```  

В коде добавляется объемлющая область видимости, чтобы предохранить аргумент декоратора для использования в будущем вызове. Когда функция `someFunc` определена, интерпретатор Python на самом деле вызывает `decorator`- результат выполнения `timer` перед тем, как фактически происходит декорирование, - со значением `label`, доступным в объемлющей области видимости декоратора. То есть `timer` возвращает декоратор, запоминающий аргумент декоратора и исходную функцию, а также возвращающий вызываемый объект `onCall`, который в конечном итоге обращается к исходной функции при вызовах в более позднее время. Поскольку такая структура создает новые функции `decorator` и `onCall`, их объемлющие области видимости сохраняют состояние для каждого случая декорирования.  

### Измерение времени с аргументами декоратора  

```python
import time


def timer(label="", trace=True):              #При аргументах декоратора: созранение аргументов

    class Timer:
        def __init__(self, func):             #При декорировании @: сохранение декорированной функции
            self.func = func
            self.alltime = 0
        
        def __call__(self, *args, **kwargs):  #При вызовах: вызов исходной функции
            start = time.time()
            result = self.func(*args, **kwargs)
            elapsed = time.time() - start
            self.alltime += elapsed
            if trace:
                print(f"{label} {self.func.__name__}: {elapsed}, {self.alltime}")
            return result
    return Timer


@timer(label="[CCC]==>")
def listcomp(N):
    return [x ** 2 for x in range(N)]

@timer(trace=True, label="[MMM]==>")
def mapcall(N):
    return list(map((lambda x: x), range(N)))


def test():
    for func in (listcomp, mapcall):
        result = func(5)
        func(50000)
        func(500000)
        func(1000000)
        print(result)
        print(f"alltime: {func.alltime}\n")


if __name__ == "__main__":
    test()
```  

Реализация еще и для методов (через декоратор и `__getattr__`):  

```python
import time


def timer(label="", trace=True):

    class Timer:
        def __init__(self, func):
            self.func = func
            self.alltime = 0
        
        def __get__(self, instance, owner):
            return Wrapper(self, instance)
        
        def __call__(self, *args):
            start = time.time()
            result = self.func(*args)
            elapsed = time.time() - start
            self.alltime += elapsed
            if trace:
                print(f"{label} {self.func.__name__}: {elapsed}, {self.alltime}")
            return result
    return Timer

class Wrapper:
    def __init__(self, dec, obj):
        self.dec = dec
        self.obj = obj

    def __call__(self, *args):
        return self.dec(self.obj, *args)

    def __getattr__(self, attrname):
        return getattr(self.dec, attrname)


@timer(label="[CCC]==>")
def listcomp(N):
    return [x ** 2 for x in range(N)]

@timer(trace=True, label="[MMM]==>")
def mapcall(N):
    return list(map((lambda x: x), range(N)))


class SomeClass:
    @timer(trace=True, label="CLASS==>")
    def someMethod(self, N):
        return [x ** 2 for x in range(N)]


def test():
    ex = SomeClass()
    for func in (listcomp, mapcall, ex.someMethod):
        result = func(5)
        func(50000)
        func(500000)
        func(1000000)
        print(result)
        print(f"alltime: {func.alltime}\n")


if __name__ == "__main__":
    test()
``` 

Реализация через объемлющие области видимости (и атрибуты функции):  

```python
import time

def timer(label="", trace=True):
    def decorator(func):
        def onCall(*args):
            start = time.time()
            result = func(*args)
            elapsed = time.time() - start
            onCall.alltime += elapsed
            if trace:
                print(f"{label} {func.__name__}: {elapsed}, {onCall.alltime}")
            return result
        onCall.alltime = 0
        return onCall
    return decorator


@timer(label="[CCC]==>")
def listcomp(N):
    return [x ** 2 for x in range(N)]

@timer(trace=True, label="[MMM]==>")
def mapcall(N):
    return list(map((lambda x: x), range(N)))


class SomeClass:
    @timer(trace=True, label="CLASS==>")
    def someMethod(self, N):
        return [x ** 2 for x in range(N)]


def test():
    ex = SomeClass()
    for func in (listcomp, mapcall, ex.someMethod):
        result = func(5)
        func(50000)
        func(500000)
        func(1000000)
        print(result)
        print(f"alltime: {func.alltime}\n")


if __name__ == "__main__":
    test()
```  

## Реализация декораторов класса  

Декораторы классов применяются либо для управления самими классами, либо для перехвата вызовов, создающих экземпляры, с целью управления экземплярами.

## Классы одиночки  

Из за перехвата вызовов, создающих экземпляры, декораторы мгут использоваться либо для управления всеми экземплярами, либо для дополнения интерфейсов этих экземпляров. Далее идет пример декоратора класса, который управляет всеми экземплярами класса. В примере реализован паттерн __"ОДИНОЧКА"__, при котором в любой момент времени существует самое большее один экземпляр класса. Функция `singleton` определяет и возвращает функцию для управления экземплярами, а посредствам синтаксиса `@` целевой класс помещается в оболчку данной функции.  

```python
instances ={}

def singleton(aClass):
    def onCall(*args, **kwargs):
        if aClass not in instances:
            instances[aClass] = aClass(*args, **kwargs)
        return instances[aClass]
    return onCall


@singleton
class SomeClass:
    def __init__(self, val):
        self.val = val 


@singleton 
class Person:                                               #Person = singleton(Person)
    def __init__(self, name, job="", pay=0):
        self.name = name
        self.job = job
        self.pay = pay

    def personPay(self):
        return self.pay


def test():
    ex = Person("Evgeniy", "python developer", 3000)        #ex = Person(...) => ex = onCall(...)
    print(ex.name, ex.personPay())
    ex1 = Person("Dmitry", "java developer", 5000)
    print(ex1.name, ex1.personPay())
    
    x = SomeClass(val=41)
    print (x.val)
    y = SomeClass(val=55)
    print(y.val)
```  

Теперь, когда класс `Person` или `Spam` используется в будущем с целью создания экземпляра, предоставленный декоратором уровень логики оболочки направляет вызовы, создающие экземпляры, функции `onCall`, которая гарантирует наличие единственного экземпляра для каждого класса независимо от того, сколько таких вызовов было сделано.  

Можно сделать через `nonlocal`:  

```python
def singleton(aClass):
    instance = None
    def onCall(*args, **kwargs):
        nonlocal instance
        if instance is None:
            instance = aClass(*args, **kwargs)
        return instance
    return onCall
```  

Через атрибуты функции:  

```python
def singleton(aClass):
    def onCall(*args, **kwargs):
        if onCall.instance is None:
            onCall.instance = aClass(*args, **kwargs)
        return onCall.instance
    onCall.instance = None
    return onCall
```  

Через класс:  

```python
class singleton:
    def __init__(self, aClass):
        self.aClass = aClass
        self.instance = None
    
    def __call__(self, *args, **kwargs):
        if self.instance == None:
            self.instance = self.aClass(*args, **kwargs)
        return self.instance
```  

## Отслеживание объектных интерфейсов  

Еще один способ использования декораторов класса (помимо управления всеми экземплярами декарируемого класса) предусматривает дополнение интерфейска _каждого_ созданного экземпляра. Декораторы классов могут по существу устанавливать для экземпляров уровень логики оболочки или “посредника”, который каким-то образом управляет доступом к их интерфейсам.  

Метод перегрузки операций__ getattr__ - способ оборачивания полных объектных интерфейсов внедренных экземпляров с целью реализации кодовой схемы делегирования. Похожие примеры встречались во время раскрытия темы управляемых атрибутов. Метод __ getattr__ запускается при извлечении неопределенного имени атрибута; мы можем использовать такую привязку для перехвата обращений к методам в классе контроллера и их передачи внедренному объекту.  

Пример кода делегирования без декорирования:  

```python
class Wrapper:
    def __init__(self, object):
        self.object = object

    def __getattr__(self, attrname):
        print(f"Trace: {attrname}")
        return getattr(self.object, attrname)


def test():
    ex = Wrapper([1, 2, 3])
    ex.append(4)
    print(ex.object)
```  

Данный приме отслеживает доступ к атрибутам, осуществляемый снаружи класса внутреннего объекта; операции доступа внутри методов внутреннего объекта не перехватываются и выполняются нормально по определению. Поведение такой модели полного интерфейса отличается от поведения декораторов функций, которые оборачивают только один конкретный метод.  

### __Отслеживание интерфейсов с помощью декораторов классов__  

Предыдущий пример можно реализовать с помощью декоратора класса:  

```python
def tracer(aClass):
    class Wrapper:
        def __init__(self, *args, **kwargs):
            self.fetches = 0
            self.wrapped = aClass(*args, **kwargs)
        
        def __getattr__(self, attrname):
            print(f"Trace {attrname}")
            self.fetches += 1
            return getattr(self.wrapped, attrname)
    return Wrapper

@tracer
class Spam:
    def display(self):
        print("Some data")


@tracer
class Person:                                                           #Person = tracer(Person) == Wrapper
    def __init__(self, name, job="", pay=0):
        self.name = name
        self.job = job
        self.pay = pay

    def personPay(self):
        return self.pay


def test():
    ex = Person("Evgenniy", job="python developer", pay=100000)         #ex = Person(...) == ex = Wrapper(...)
    print(ex.personPay())
```  

# Пример: "закрытые" и "открытые" атрибуты  

Реализация Private:  

```python
"""
Защита для атрибутов, извлекаемых из экземпляра класса.
Пример использования приведен в коде самотестирования в конце файла.

Doubler = Private("data", "size")(Doubler)
Private возвращает onDecorator, onDecorator возвращает onInstance,
а в кажом экземпляре onInstance внедрен экземпляр Doubler
"""
traceMe = False

def trace(*args):
    if traceMe:
        print(f"[{' '.join(map(str, args))}]")

def Private(*privates):
    def onDecorator(aClass):
        class onInstace:

            def __init__(self, *args, **kwargs):
                self.wrapped = aClass(*args, **kwargs)                              #wrapped в атрибуте экземпляра

            def __getattr__(self, attrname):                                        #Внутри wrapped перехват выпоняться не будет
                trace("get", attrname)
                if attrname in privates:
                    raise TypeError(f"private attribute fecth: {attrname}")
                else:
                    return getattr(self.wrapped, attrname)

            def __setattr__(self, attrname, value):                                 #Операции доступа извне (внутри wrapper не будут выполняться)
                trace("set", attrname)
                if attrname == "wrapped":                                           #Разрешить доступ к собственным атрибутам
                    self.__dict__[attrname] = value
                elif attrname in privates:
                    raise TypeError(f"private attribute change {attrname}")
                else:
                    setattr(self.wrapped, attrname, value)                          #Атрибуты внутреннего объекта
        return onInstace
    return onDecorator


if __name__ == "__main__":
    traceMe = True


    @Private("data", "size")
    class Doubler:
        def __init__(self, label, start):
            self.label = label
            self.data = start

        def size(self):
            return len(self.data)

        def double(self):
            for i in range(self.size()):
                self.data[i] = self.data[i] * 2

        def display(self):
            print(f"{self.label} => {self.data}")
    

    X = Doubler("X is", [1, 2, 3])
    Y = Doubler("Y is", [-10, -20, -30])
    #Весь сделующий код выполняется успешно
    print(X.label)
    X.display(); X.double(); X.display()

    print(Y.label)
    Y.display(); Y.double
    Y.label = "Spam"
    Y.display()
    #Весь следующий код терпит неудачу
    try:
        print(X.size())
    except TypeError as ex:
        print(ex)

    try:
        print(X.data)
    except TypeError as ex:
        print(ex)

    try:
        X.data = [1, 1, 1]
    except TypeError as ex:
        print(ex)

    try:
        X.size = lambda S: 0
    except TypeError as ex:
        print(ex)

    try:
        print(Y.data)
    except TypeError as ex:
        print(ex)

    try:
        print(Y.size())
    except TypeError as ex:
        print(ex)
```  

Реализация Private и Public (добавление 4го уровня логики):  


```python
traceMe = False

def trace(*args):
    if traceMe:
        print(f"[{' '.join(map(str, args))}]")


def accessControl(failIf):
    def onDecorator(aClass):
        class onInstance:
            def __init__(self, *args, **kwargs):
                self.__wrapper = aClass(*args, **kwargs)

            def __getattr__(self, attrname):
                if failIf(attrname):
                    raise TypeError(f"private attribute fecth: {attrname}")
                else:
                    return getattr(self.__wrapper, attrname)

            def __setattr__(self, attrname, value):
                if attrname == "_onInstance__wrapper":
                    self.__dict__[attrname] = value
                elif failIf(attrname):
                    raise TypeError(f"private attribute change {attrname}")
                else:
                    setattr(self.__wrapper, attrname, value)
        return onInstance
    return onDecorator


def Private(*attributes):
    return accessControl(failIf=(lambda attr: attr in attributes))


def Public(*attributes):
    return accessControl(failIf=(lambda attr: attr not in attributes))


if __name__ == "__main__":
    @Private("age")
    class Person:
        def __init__(self, name, age):
            self.name = name
            self.age = age
    
    ex = Person("Evgeniy", 23)
    print(ex.name)
    ex.name = "Jenya"
    print(ex.name)
    try:
        print(ex.age)
    except TypeError as ex:
        print(ex)

    @Public("name")
    class Person:
        def __init__(self, name, age):
            self.name = name
            self.age = age

    ex = Person("Evgeniy", 23)
    print(ex.name)
    ex.name = "Jenya"
    print(ex.name)
    try:
        print(ex.age)
    except TypeError as ex:
        print(ex)
```  

# Пример: проверка допустимости аргументов  

Базовый функционал:  

```python


from typing import Type


def rengetest(*argchecks):
    def onDecorator(func):
        if not __debug__:
            return func
        else:
            def onCall(*args):
                for (ix, low, high) in argchecks:
                    if args[ix] < low or args[ix] > high:
                        raise TypeError(f"Argiment {ix} not in {low} .. {high}")
                return func(*args)
            return onCall
    return onDecorator
    


class Person:
    def __init__(self, name, job, pay):
        self.name = name
        self.job = job
        self.pay = pay

    @rengetest([1, 0.0, 1.0])
    def giveRise(self, percent):
        self.pay *= 1 + percent


def test():
    ex = Person("Evgeniy Chernetskiy", "python developer", 2021)
    ex.giveRise(0.1)
    try:
        ex.giveRise(2)
    except TypeError as exe:
        print(exe)


if __name__ == "__main__":
    test()
``` 

Версия для позиционных аргументов, ключевых аргументов и аргументов со значением по умолчанию:  

```python
"""
Декоратор функций, проверяющий вхождения в диапазон аргументов, переданных любой функции или методу.
Аргументы указываются декоратору по ключам. В фактическом вызове аргуметы могут передаваться по позициям
или по ключам, а аргументы со стандартными значениями ращрешено пропускать. Примеры сценариев использования
в конце файла.
"""

trace = True
def rangetest(**argschecks):
    def onDecorator(func):
        if not __debug__:
            return func
        else:
            code = func.__code__
            allargs = code.co_varnames[:code.co_argcount]
            funcname = func.__name__
            def onCall(*pargs, **kwargs):
                """
                Все элементы pargs соответсвуют первым N
                ожидаемых аргументов позициям.
                Отсальные должны быть в kwargs или быть опущены
                из-за наличия стандартных значений
                """
                expected = list(allargs)
                positional = expected[:len(pargs)]
                for (argname, (low, high)) in argschecks.items():
                    #Для аргументов подлежащих проверке
                    if argname in kwargs:
                        #Был передан по имени
                        if kwargs[argname] < low or kwargs[argname] > high:
                            #Аргумент не находится в диапазоне
                            raise TypeError(f"{funcname} argument {argname} not in {low}..{high}")
                    elif argname in positional:
                        #Был передан по позиции
                        position = positional.index(argname)
                        if pargs[position] < low or pargs[position] > high:
                            raise TypeError(f"{funcname} argument {argname} not in {low}..{high}")
                    else:
                        #Предполагается, что не был передан: имеет стандартное значение
                        if trace:
                            #Аргумент имеет стандартное значение
                            print(f"Argument {argname} defaulted")
                return func(*pargs, **kwargs)
            return onCall
    return onDecorator


if __name__ == "__main__":
    #Тестирование функций с позиционными и ключевыми аргументами
    @rangetest(age=(0,120))
    def persinfo(name, age):
        print(f"{name} is {age} years old")

    @rangetest(M=(1, 12), D=(1, 31), Y=(0,2021))
    def birthday(M, D, Y):
        print(f"birthday = {M}/{D}/{Y}")

    persinfo("bob", 40)
    persinfo(age=40, name="Bob")
    birthday(5, D=1, Y=1917)

    try:
        persinfo("Bob", 150)
    except TypeError as Ex:
        print(Ex)

    try:
        persinfo(age=150, name="Bob")
    except TypeError as Ex:
        print(Ex)

    try:
        birthday(5, D=40, Y=2021)
    except TypeError as Ex:
        print(Ex)
```