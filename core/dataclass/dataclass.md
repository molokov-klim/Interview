# **Dataclass**

[Содержание](/CONTENTS.md#содержание)

## **Junior Level**

Dataclass — это декоратор из модуля `dataclasses`, который автоматически добавляет в класс стандартные методы, избавляя
разработчика от написания шаблонного кода. Если обычный класс, предназначенный в основном для хранения данных, требует
ручного определения методов `__init__`, `__repr__` и `__eq__`, то dataclass делает это за вас одной строкой.

Представьте, что вам нужно создать простой класс для представления точки с координатами. Без dataclass пришлось бы
писать несколько методов, которые часто выглядят одинаково для разных классов. С dataclass достаточно объявить поля с
аннотациями типов, и всё необходимое появится автоматически. Это делает код чище, уменьшает вероятность ошибок и
упрощает поддержку.

Например, класс `Point` можно определить всего в две строки, и он сразу будет иметь конструктор, читаемое строковое
представление и корректное сравнение экземпляров по значению полей. Dataclass также предлагает гибкие настройки: можно
создавать неизменяемые классы, управлять участием полей в сравнении, задавать значения по умолчанию и даже определять
поля, которые вычисляются динамически.

[Содержание](/CONTENTS.md#содержание)

## **Middle Level**

Под капотом dataclass анализирует аннотации полей класса и на их основе генерирует необходимые методы. Помимо очевидных
`__init__`, `__repr__` и `__eq__`, он может создавать и другие специальные методы в зависимости от параметров
декоратора. Например, при `order=True` генерируются методы сравнения (`__lt__`, `__le__`, `__gt__`, `__ge__`), что
позволяет упорядочивать экземпляры. При `frozen=True` экземпляры становятся неизменяемыми, и любая попытка изменить поле
вызывает исключение `FrozenInstanceError`. Это полезно для создания объектов, которые должны оставаться постоянными,
например, для использования в качестве ключей словаря.

Функция `field()` предоставляет тонкий контроль над каждым полем. Через неё можно указать, должно ли поле участвовать в
конструкторе, строковом представлении, сравнении или вычислении хеша. Особенно важны параметры `default` для значений по
умолчанию и `default_factory` для изменяемых объектов — например, чтобы каждый экземпляр получал свой собственный
список, а не ссылался на один и тот же. Параметр `metadata` позволяет прикреплять к полям произвольные метаданные,
которые не влияют на поведение dataclass, но могут быть использованы сторонними инструментами для валидации или
генерации документации.

Dataclass поддерживает наследование, собирая поля от всех родительских классов в порядке MRO (Method Resolution Order).
Однако при наследовании требуется аккуратность с значениями по умолчанию, чтобы избежать проблем с порядком аргументов в
конструкторе. Для дополнительной инициализации или валидации можно определить метод `__post_init__()`, который
вызывается автоматически после основного конструктора.

Начиная с Python 3.10, dataclass поддерживает паттерн-матчинг через параметр `match_args=True`, что делает экземпляры
класса удобными для использования в конструкциях `match/case`. Также при использовании `slots=True` dataclass может
генерировать классы с оптимизацией памяти, аналогичные классам с `__slots__`, что особенно полезно при создании большого
количества экземпляров.

В целом, dataclass — это не просто удобное сокращение кода, а полноценный инструмент для создания надёжных и эффективных
классов данных, который сочетает простоту использования с широкими возможностями настройки под конкретные задачи.

1. **Автоматически генерируемые методы**:
    - `__init__()`: Инициализатор с параметрами для всех полей
    - `__repr__()`: Читаемое строковое представление
    - `__eq__()`: Сравнение по значениям всех полей
    - `__ne__()`: Автоматически на основе `__eq__`
    - Опционально (через параметры декоратора):
        - `__lt__()`, `__le__()`, `__gt__()`, `__ge__()`: при `order=True`
        - `__hash__()`: при `frozen=True` или явном указании

2. **Параметры декоратора `@dataclass`**:
    - `init=True`: Генерация `__init__`
    - `repr=True`: Генерация `__repr__`
    - `eq=True`: Генерация `__eq__`
    - `order=False`: Генерация методов сравнения
    - `frozen=False`: Запрет изменения полей после создания
    - `unsafe_hash=False`: Принудительная генерация `__hash__`
    - `match_args=True`: Поддержка паттерн-матчинга (Python 3.10+)

3. **Поле `field()`**: Функция для тонкой настройки полей:
    - `default`: Значение по умолчанию
    - `default_factory`: Фабрика для изменяемых значений по умолчанию
    - `init`: Включать ли поле в `__init__`
    - `repr`: Включать ли поле в `__repr__`
    - `compare`: Участвует ли поле в сравнении
    - `hash`: Участвует ли поле в вычислении хеша
    - `metadata`: Произвольные метаданные

[Содержание](/CONTENTS.md#содержание)

## **Senior Level**

В CPython 3.9+ **dataclass** — **Python декоратор** `Lib/dataclasses.py` с `_process_class()`, генерирует **специальные
методы** (`__init__`, `__repr__`, `__eq__`) через `_create_fn()` + `exec()`, использует **metaclass** `DataclassType`. *
*PEP 557**. `Lib/dataclasses.py`

## 1. @dataclass декоратор (Lib/dataclasses.py)

```python
def dataclass(cls=None, *, init=True, repr=True, eq=True, order=False,
              unsafe_hash=False, frozen=False, match_args=True,
              kw_only=False, slots=False, weakref_slot=False):
    def wrap(cls):
        # 1. Собираем поля класса
        fields = fields(cls)

        # 2. Генерируем специальные методы
        ns = dict(cls.__dict__)

        # __init__
        if init:
            __init__ = _init_fn(fields, locals())
            ns['__init__'] = __init__

        # __repr__
        if repr:
            __repr__ = _repr_fn(fields, globals())
            ns['__repr__'] = __repr__

        # __eq__
        if eq:
            __eq__ = _eq_fn(fields, globals())
            ns['__eq__'] = __eq__

        # __hash__ / __setattr__
        if frozen:
            ns['__setattr__'] = _frozen_setattr()
            ns['__delattr__'] = _frozen_delattr()

        # 3. Создаём новый класс
        return type(cls)(cls.__name__, cls.__bases__, ns)

    return wrap if cls is None else wrap(cls)
```

**Объяснение для тупого человека (очень подробно):** Представь, что у тебя класс `Person` с полями `name: str` и
`age: int`. Когда ты пишешь `@dataclass`, Python **не меняет** твой исходный класс. Вместо этого он **создаёт новый
класс** с **такими же** полями, но **добавляет** готовые методы `__init__`, `__repr__`, `__eq__`. Это как **фабрика**,
которая берёт твой чертеж дома и **достраивает** коммуникации, двери, окна. Ты написал только стены — Python добавил
остальное. `_process_class()` — это **главная фабрика**, которая анализирует аннотации `: str`, `: int` и понимает, что
это **поля dataclass'а**.

## 2. fields() - сбор полей класса

```python
def fields(cls):
    flds = []
    bases = list(cls.__bases__)

    # Ищем __dataclass_fields__
    ann = getattr(cls, '__annotations__', {})
    for name, type_ in ann.items():
        if isinstance(type_, Field):
            flds.append(type_)
        else:
            # Обычная аннотация -> Field(name, type_)
            flds.append(Field(name, type_, default=MISSING))

    # Наследование
    for b in bases[::-1]:  # MRO
        flds = fields(b) + flds

    return tuple(flds)
```

Python **сканирует** `__annotations__` = `{'name': str, 'age': int}`. Каждое имя *
*становится Field**. Если написал `name: str = "Bob"` — это **default значение**. Если класс **наследуется** от другого
dataclass — **берёт** поля родителя **первым**. Представь, что собираешь конструктор LEGO: сначала **база** от
родителей, потом **твои** кубики сверху.

## 3. _init_fn() - генерация __init__

```python
def _init_fn(fields, locals):
    # Сигнатура: def __init__(self, name: str, age: int = 0):
    sig_lines = ['def __init__(self, /, ']

    # Параметры из полей
    for idx, field in enumerate(fields):
        if field.init_arg is False:
            continue
        arg = field.name
        if field.default is not MISSING:
            arg += '=' + repr(field.default)
        sig_lines.append(arg)

    sig = ', '.join(sig_lines) + '):'

    # Тело: self.name = name
    body_lines = []
    for field in fields:
        if field.init_arg:
            line = f'self.{field.name} = {field.name}'
            body_lines.append(line)

    # exec() создаёт функцию
    fn = _create_fn('__init__', ('self',),
                    [sig] + body_lines,
                    locals=locals)
    return fn
```

Для `@dataclass class Person: name: str; age: int = 0` создаётся **строка кода**:

```
def __init__(self, /, name: str, age: int = 0):
    self.name = name
    self.age = 0
```

Затем `exec()` **компилирует** эту строку в **настоящую** Python функцию и **вставляет** в класс. Это **магия**: твой
класс получает **готовый** `__init__`, который принимает **именно те параметры**, что ты объявил.

## 4. _create_fn() - компиляция кода в функцию

```python
def _create_fn(name, args, body_lines, globals=None, locals=None,
               return_type=None):
    # Формируем полный код функции
    code = ['def ' + name + '(' + ', '.join(args) + '):']
    code.extend('    ' + line for line in body_lines)

    # Компилируем в PyCodeObject
    src = '\n'.join(code)
    co = compile(src, '<dataclasses>', 'exec')

    # Создаём функцию
    fn = types.FunctionType(co, globals, name, (), closure=None)
    return fn
```

`_create_fn()` — **мини-компилятор**. Берёт строки
`def __init__(self, name): self.name=name`, **склеивает** в одну большую строку, вызывает `compile()` → **PyCodeObject
**, затем `types.FunctionType()` → **готовую функцию**. Это **точно так же**, как если бы ты написал эту функцию *
*руками** — никакого отличия в скорости!

## 5. __repr__ генерация

```python
def _repr_fn(fields, globals):
    # 'Person(name=\'Bob\', age=30)'
    body_lines = []
    body_lines.append('r = self.__class__.__name__ + \'(\'')

    for field in fields:
        if field.repr:
            line = f"r += '{field.name}=' + repr({field.name}) + ', '"
            body_lines.append(line)

    # Убираем последнюю запятую
    body_lines.append('return r.rstrip(\", \") + \')\'')

    return _create_fn('__repr__', ('self',), body_lines, globals)
```

`print(Person)` → `'Person(name=\'Bob\', age=30)'`. Python **проходит** по всем
полям, для **каждого** делает `name='Bob'`, **склеивает** через запятую. **Точно** как `str(dict)`, но **для твоего
класса**. Если поле `repr=False` — **пропускает** его.

## 6. DataclassType метакласс

```python
class DataclassType(type):
    def __new__(cls, name, bases, ns):
        # Автоматическая обработка @dataclass
        if hasattr(ns, '__dataclass_transform__'):
            return super().__new__(cls, name, bases, ns)

        # Обрабатываем поля
        fields = []
        for base in bases[::-1]:
            fields.extend(getattr(base, '__dataclass_fields__', {}))

        # Добавляем __dataclass_fields__
        ns['__dataclass_fields__'] = dict(fields)

        return super().__new__(cls, name, bases, ns)
```

**Метакласс** — это **фабрика классов**. Когда Python видит `class Person:`, он *
*спрашивает** метакласс: "как создать этот класс?". `DataclassType` **добавляет** `__dataclass_fields__` =
`{'name': Field(...), 'age': Field(...)}`. Это **словарь полей** для наследования.

## 7. Field() - дескриптор полей

```python
class Field:
    __slots__ = ('name', 'type', 'default', 'default_factory',
                 'init', 'repr', 'eq', 'order', 'hash',
                 'compare', 'metadata', 'init_arg')

    def __init__(self, default, *, default_factory, init, repr, eq,
                 order, unsafe_hash, frozen, compare, metadata):
        self.name = None
        self.default = default
        self.default_factory = default_factory
        self.init = init
        self.repr = repr
        self.eq = eq
        self.order = order
        self.hash = unsafe_hash
        self.compare = compare
        self.metadata = metadata
```

`name: str = field(default_factory=list)` → **Field объект** с флагами `repr=True`,
`init=True`, `default_factory=<class 'list'>`. Когда `__init__` видит `default_factory` — вызывает `list()` вместо
копирования значения.

## 8. __post_init__() поддержка

```python
# Автоматический вызов после __init__
def _init_fn(fields, locals):
    # ... обычный __init__ ...

    # Добавляем __post_init__ если есть
    if '__post_init__' in locals:
        body_lines.append('self.__post_init__()')
```

Dataclass **всегда** вызывает `self.__post_init__()` **после** заполнения полей.
Полезно для **валидации**: `def __post_init__(self): assert self.age >= 0`.

## 9. frozen=True - неизменяемый dataclass

```python
def _frozen_setattr():
    return _create_fn('__setattr__', ('self', 'name', 'value'),
                      ['if _is_dataclass_field(self, name):',
                       '    raise FrozenInstanceError(f"cannot assign to {name}")',
                       'object.__setattr__(self, name, value)'])
```

`@dataclass(frozen=True)` → `__setattr__` **блокирует** изменение полей.
`p.name = "new"` → **FrozenInstanceError**. **Только** `object.__setattr__` для служебных полей.

## 10. Наследование dataclass

```python
@dataclass
class Person:
    name: str


@dataclass
class Employee(Person):
    salary: int
```

**Результат:** `__init__(self, name: str, salary: int)`. **Поля родителей** идут **первыми**.

**Dataclass** в CPython 3.9+ — **Lib/dataclasses.py** декоратор, **_process_class()** анализ `__annotations__`, **
_create_fn() + exec()** генерация `__init__`/`__repr__`, **Field дескрипторы**, **DataclassType метакласс**, `frozen`/
`__post_init__` поддержка.

- [Содержание](/CONTENTS.md#содержание)