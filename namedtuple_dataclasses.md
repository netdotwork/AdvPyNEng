### collections.namedtuple, typing.NamedTuple

Data classes, как и collections.namedtuple, как и typing.NamedTuple - конструкторы классов.

namedtuple и NamedTuple - сильно ограничены.

Их стоит использовать если нужен готовый объект, который будет обладать свойствами кортежа, по умолчанию:
- итерируемый (можно перебирать атрибуты как элементы кортежа)
- с доступом к атрибутам по индексу
- неизменяемый (нельзя изменять атрибуты)

#### 1. Создаем итерируемый объект:

```python
from collections import namedtuple

RouterClass = namedtuple('Router', ['hostname', 'ip', 'ios'])
r1 = RouterClass('r1', '10.1.1.1', '15.4')
for i in r1:
    print(i)

r1
10.1.1.1
15.4
```
#### 2. Обращаемся по индексу:

```python
In [8]: r1[0]
Out[8]: 'r1'

In [9]: r1[1]
Out[9]: '10.1.1.1'

In [10]: r1[2]
Out[10]: '15.4'
```

Хороший пример использования таких объектов - формирование запросов к БД:
```python
import sqlite3
from collections import namedtuple


key = 'vlan'
value = 10
db_filename = 'dhcp_snooping.db'

keys = ['mac', 'ip', 'vlan', 'interface', 'switch']
DhcpSnoopRecord = namedtuple('DhcpSnoopRecord', keys)

conn = sqlite3.connect(db_filename)
query = 'select {} from dhcp where {} = ?'.format(','.join(keys), key)

print('-' * 40)
for row in map(DhcpSnoopRecord._make, conn.execute(query, (value,))):
    print(row.mac, row.ip, row.interface, sep='\n')
    print('-' * 40)
```

[Создание классов с помощью namedtuple](https://advpyneng.readthedocs.io/ru/latest/book/13_data_classes/namedtuple.html)

### Data classes

NamedTuple расширяет предыдущий функционал возможностью добавления собственных методов в класс, но в этом случае лучше использовать Data classes. Объект Data classes:

- изменяемый (можно изменять атрибуты, а можно отключить эту возможность)
- не итерируемый, по умолчанию, но есть встроенные методы, которыми можно собрать атрибуты в словарь или кортеж (здесь появляется возможность обращаться по индексу или по имени)
- автоматически будут созданы методы __init__, __repr__, __eq__
- атрибуты класса нужно создавать с помощью аннотации типов. А с использованием модуля pydantic это превращается в киллерфичу, т.к. типы будут автоматически проверяться
- атрибуты класса можно создавать динамически, сохраняя состояние (с помощью property или __post_init__)
- можно сравнивать (по умолчанию класс создает только метод __eq__. Объекты сравниваются как кортежи (сравниваются элементы по порядку). Дополнительно можно включить создание методов __lt__, __le__, __gt__, __ge__)

#### 1. Посмотрим на методы, которые создаются по умолчанию:

```python
In [1]: from dataclasses import dataclass

In [2]: @dataclass
   ...: class IPAddress:
   ...:     ip: str
   ...:     mask: str
   ...:

In [3]: ip1 = IPAddress("10.1.1.1", 28)

In [4]: ip1
Out[4]: IPAddress(ip='10.1.1.1', mask=28)

# Смотрим созданные атрибуты:

In [5]: vars(ip1.__class__)
Out[6]:
mappingproxy({'__module__': '__main__',
              '__annotations__': {'ip': str, 'mask': str},
              '__dict__': <attribute '__dict__' of 'IPAddress' objects>,
              '__weakref__': <attribute '__weakref__' of 'IPAddress' objects>,
              '__doc__': 'IPAddress(ip: str, mask: str)',
              '__dataclass_params__': _DataclassParams(init=True,repr=True,eq=True,order=False,unsafe_hash=False,frozen=False),
              '__dataclass_fields__': {'ip': Field(name='ip',type=<class 'str'>,default=<dataclasses._MISSING_TYPE object at 0x7f88e761e790>,default_factory=<dataclasses._MISSING_TYPE object at 0x7f88e761e790>,init=True,repr=True,hash=None,compare=True,metadata=mappingproxy({}),_field_type=_FIELD),
               'mask': Field(name='mask',type=<class 'str'>,default=<dataclasses._MISSING_TYPE object at 0x7f88e761e790>,default_factory=<dataclasses._MISSING_TYPE object at 0x7f88e761e790>,init=True,repr=True,hash=None,compare=True,metadata=mappingproxy({}),_field_type=_FIELD)},
              '__init__': <function __main__.__create_fn__.<locals>.__init__(self, ip: str, mask: str) -> None>,
              '__repr__': <function __main__.__create_fn__.<locals>.__repr__(self)>,
              '__eq__': <function __main__.__create_fn__.<locals>.__eq__(self, other)>,
              '__hash__': None})

# Видим __init__, __repr__, __eq__
```

#### 2. С помощью pydantic можно включить проверку типов. Например:

```python
In [1]: from pydantic.dataclasses import dataclass

In [2]: @dataclass
   ...: class Book:
   ...:     title: str
   ...:     price: int

In [3]: book = Book('Good Omens', price=35)

In [4]: book.__annotations__
Out[4]: {'title': str, 'price': int}

In [5]: vars(book)
Out[5]: {'title': 'Good Omens', 'price': 35, '__initialised__': True}

In [6]: book = Book('Good Omens', price='35')

In [7]: book.__annotations__
Out[7]: {'title': str, 'price': int}

In [8]: vars(book)
Out[8]: {'title': 'Good Omens', 'price': 35, '__initialised__': True}

# Можно заметить, что строка преобразуется в число
# Если строку преобразовать нельзя, возникает ошибка:

In [9]: book = Book('Good Omens', price='a')
---------------------------------------------------------------------------
ValidationError                           Traceback (most recent call last)
<ipython-input-12-c21f0df3a6ac> in <module>
----> 1 book = Book('Good Omens', price='a')

<string> in __init__(self, title, price)

~/virtenvs/3.8.4/lib/python3.8/site-packages/pydantic/dataclasses.cpython-38-x86_64-linux-gnu.so in pydantic.dataclasses._process_class._pydantic_post_init()

ValidationError: 1 validation error for Book
price
  value is not a valid integer (type=type_error.integer)
```
[Примеры использования pydantic.dataclasses](https://github.com/samuelcolvin/pydantic/tree/master/docs/examples)

#### 3. Атрибуты можно создавать динамически:

```python
In [1]: from dataclasses import dataclass

In [2]: @dataclass
   ...: class IPAddress:
   ...:     ip: str
   ...:     mask: int
   ...: 
   ...:     def __post_init__(self):
   ...:         if not isinstance(self.mask, int):
   ...:             self.mask = int(self.mask)
   ...: 

In [3]: ip1 = IPAddress("10.10.1.1.", "24")

In [4]: ip1.mask
Out[4]: 24

In [5]: ip1.mask = 28

In [6]: ip1.mask
Out[6]: 28
```

Параметры в метод __post_init__ можно передавать указанием типа:

```python
In [18]: from dataclasses import InitVar

In [19]: @dataclass
    ...: class Book:
    ...:     title: str
    ...:     author: str
    ...:     gen_desc: InitVar[bool] = True
    ...:     desc: str = None
    ...: 
    ...:     def __post_init__(self, gen_desc: bool):
    ...:         if gen_desc and self.desc is None:
    ...:             self.desc = "`%s` by %s" % (self.title, self.author)
    ...: 

In [20]: Book("Fareneheit 481", "Bradbury")
Out[20]: Book(title='Fareneheit 481', author='Bradbury', desc='`Fareneheit 481` by Bradbury')

In [21]: Book("Fareneheit 481", "Bradbury", gen_desc=False)
Out[21]: Book(title='Fareneheit 481', author='Bradbury', desc=None)

```

Атибуты можно создавать более гибко, с применением property:

```python
from dataclasses import dataclass, field, asdict
from typing import Union, Dict


@dataclass
class Book:
    title: str
    price: float
    _price: float = field(init=False, repr=False)
    quantity: int = 0
    total: float
    _total: float = field(init=False)

    @property
    def total(self):
        return round(self.price * self.quantity, 2)

    @total.setter
    def total(self, total: float) -> None:
        self._total = total

    @property
    def price(self):
        return self._price

    @price.setter
    def price(self, value: Union[int, float]) -> None:
        if not isinstance(value, (int, float)):
            raise TypeError("Значение должно быть числом")
        if not value >= 0:
            raise ValueError("Значение должно быть положительным")
        self._price = float(value)

    def to_dict(self) -> Dict[str, str]:
        return {k: v for k, v in asdict(self).items() if not k.startswith("_")}
```

В примере выше для создания экземпляра класса достаточно указать 3 атрибута: `total`, `price`, `quantity`. 
Атрибут `total` является обязательным, но вычисляется динамечски за счет property, поэтому указывать его при создании экземпляра не нужно. Но, поскольку property делает атрибут не изменяемым, по умолчанию, нужно добавить для него setter, иначе при создании класса мы получим ошибку `AttributeError: can't set attribute`. Поскольку setter никакой роли здесь играть не будет (кроме того, что даст возможность изменения/создания `total`) создают переменную-пустышку (`_total`).

Переменная `price` является обязательной при создании экземпляра класса, а её изменение будет контролироваться setter'ом. getter здесь необходим, чтобы к переменной можно было обращаться.

#### 4. Сравнение объектов Data classes

По умолчанию, в Data classes создается метод __eq__. Можно проверить объекты только на равенство. Но, если глобально разрешить остальные специальные методы сравнения (__lt__, __le__, __gt__, __ge__), то, с помощью параметра переменной `compare` можно выбрать по какому атрибуту будет выполняться сравнение экземпляров (атрибуты сравниваются по порядку по аналогии с кортежем). Например:

```python
class IPAddress:
    ip: str = field(compare=False)
    _ip: int = field(init=False, repr=False)
    mask: int

    def __post_init__(self):
        self._ip = int(ipaddress.ip_address(self.ip))


In [40]: ip1 = IPAddress('10.10.1.1', 24)

In [41]: ip2 = IPAddress('10.2.1.1', 24)

In [42]: ip_list = [ip1, ip2]

In [43]: sorted(ip_list)
Out[43]: [IPAddress(ip='10.2.1.1', mask=24), IPAddress(ip='10.10.1.1', mask=24)]

In [44]: ip1 > ip2
Out[44]: True
```

В примере выше при сравнении экземпляров сравниваются self._ip, т.к. ip мы исключили из сравнения.


#### 5. Если необходимо передать изменяемый аргумент при создании экземпляра класса, используется параметр default_factory:

```python
@dataclass
class Bookshelf:
    books: List[Book] = field(default_factory=list)
```

В default_factory передается любой вызываемый объект без аргументов (это может быть lambda)

#### 6. При наследовании атрибуты, определенные в родительском классе, будут наследованы дочерним классом

```python
In [23]: from typing import Any

In [24]: @dataclass
    ...: class BaseBook:
    ...:     title: Any = None
    ...:     author: str = None
    ...:
    ...: @dataclass
    ...: class Book(BaseBook):
    ...:     desc: str = None
    ...:     title: str = "Unknown"
    ...:

In [25]: book = Book("Farenheit 481", "Bradbury")

In [26]: book
Out[26]: Book(title='Farenheit 481', author='Bradbury', desc=None)

In [27]: vars(book)
Out[27]: {'title': 'Farenheit 481', 'author': 'Bradbury', 'desc': None}
```

#### 7. Использование __slots__:

```python
In [44]: @dataclass
    ...: class BaseBook:
    ...:      __slots__ = {"title", "author"}
    ...:      title: str
    ...:      author: str
    ...:

In [45]: book = BaseBook("Farenheit 481", "Bradbury")

In [46]: book
Out[46]: BaseBook(title='Farenheit 481', author='Bradbury')

In [47]: book.__dict__
---------------------------------------------------------------------------
AttributeError                            Traceback (most recent call last)
<ipython-input-47-f50441a761ab> in <module>
----> 1 book.__dict__

AttributeError: 'BaseBook' object has no attribute '__dict__'

In [48]: book.__slots__
Out[48]: {'author', 'title'}
```

[Наследование и __slots__ в Data classes](https://stackoverflow.com/questions/50180735/how-can-dataclasses-be-made-to-work-better-with-slots)

[Примеры](https://github.com/pyneng/advpyneng-online-2-sep-nov-2020/tree/master/examples)

[Про __slots__](https://webdevblog.ru/vvedenie-v-python-dataclasses-chast-1/)

[Введение в Data classes](https://habr.com/ru/post/415829/)
