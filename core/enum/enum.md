# **Enum**

[Содержание](/CONTENTS.md#содержание)

## **Junior Level**

Enum (перечисление) в Python — это специальный тип данных, позволяющий создавать набор именованных констант, что делает
код более читаемым и защищённым от опечаток. Члены Enum являются иммутабельными синглтонами и поддерживают итерацию,
сравнение и доступ по имени или значению. Python предоставляет несколько вариантов: базовый `Enum`, `IntEnum` (
наследующий `int`), `Flag` для битовых операций и `StrEnum` (с Python 3.11) для строковых констант.

Например, вместо использования чисел 0, 1, 2 для статусов задачи можно создать Enum:

```python
from enum import Enum


class TaskStatus(Enum):
    PENDING = 0
    RUNNING = 1
    COMPLETED = 2
    FAILED = 3
```

Теперь в коде можно использовать `TaskStatus.RUNNING` вместо просто `1`. Enum предоставляет итерацию, сравнение и доступ
к членам по имени или значению. Особенно полезны они для ограничения допустимых значений параметров функции и замены
строковых констант.

[Содержание](/CONTENTS.md#содержание)

## **Middle Level**

1. **Типы Enum**:
    - `Enum`: Базовый класс для создания перечислений
    - `IntEnum`: Члены являются подклассами `int`, могут использоваться везде, где ожидается целое число
    - `Flag`, `IntFlag`: Для битовых флагов, поддерживают побитовые операции
    - `StrEnum` (Python 3.11+): Члены являются подклассами `str`

2. **Свойства членов Enum**:
    - `name`: Имя члена (строка)
    - `value`: Значение члена (может быть любого типа)
    - `__members__`: Словарь всех членов {имя: член}

3. **Автоматические значения**:
    - `auto()`: Автоматически присваивает уникальные значения (начиная с 1)
    - `@enum.unique`: Декоратор, гарантирующий уникальность значений

4. **Методы и операции**:
    - Итерация: `for status in TaskStatus:`
    - Доступ по значению: `TaskStatus(1)`
    - Доступ по имени: `TaskStatus['RUNNING']`
    - Проверка принадлежности: `isinstance(value, TaskStatus)`

5. **Расширенные возможности**:
    - Методы класса: можно добавлять методы в класс Enum
    - Свойства (property): вычисляемые атрибуты
    - Наследование: Enum может наследоваться от других классов (кроме другого Enum)

6. **Иммутабельность**: Члены Enum — синглтоны. Нельзя изменить их значение после создания.

[Содержание](/CONTENTS.md#содержание)

## **Senior Level**

В CPython 3.9+ **Enum** — **EnumType метакласс** (`Lib/enum.py`), **EnumDict** (отслеживает порядок `_member_names_`), *
*`_member_map_`** (name→member), **`__new__`/`__init__`** для создания **EnumMember** объектов, *
*`_generate_next_value_`** + `auto()`. **Singleton-подобные** immutable объекты. `Lib/enum.py`

## 1. EnumType.__new__() - метакласс создания (Lib/enum.py)

```python
class EnumType(type):
    def __new__(metacls, cls, bases, classdict):
        # 1. EnumDict уже обработал поля
        enum_dict = classdict

        # 2. Определяем тип значений (_member_type_)
        member_type, first_enum = metacls._get_mixins_(cls, bases)

        # 3. Создаём enum класс
        enum_class = super().__new__(metacls, cls, bases, classdict)

        # 4. Инициализируем структуры
        enum_class._member_names_ = []  # ['RED', 'GREEN']
        enum_class._member_map_ = {}  # {'RED': <RED>, 'GREEN': <GREEN>}
        enum_class._member_type_ = member_type
        enum_class._value_ = None  # Для StrEnum/IntEnum

        # 5. Создаём члены enum
        for member_name in enum_dict.member_names:
            value = enum_dict[member_name]
            member = metacls._create_(cls, member_name, value)
            enum_class._member_names_.append(member_name)
            enum_class._member_map_[member_name] = member

        return enum_class
```

**Объяснение для тупого человека (очень подробно):** Представь, что пишешь `class Color(Enum): RED = 1; GREEN = 2`.
Python **НЕ** создаёт обычный класс. Вместо этого **метакласс** `EnumType` берёт твой словарь `{'RED': 1, 'GREEN': 2}` и
**превращает** его в **специальные структуры**: `_member_names_ = ['RED', 'GREEN']` (порядок объявления) и
`_member_map_ = {'RED': <Color.RED>, 'GREEN': <Color.GREEN>}` (словарь для быстрого поиска). Каждый член — **отдельный
объект** `Color.RED`, а **НЕ** число `1`. Это как **фабрика**, которая берёт твои константы и делает из них **умные
объекты** с методами.

## 2. EnumDict - специальный словарь (Lib/enum.py)

```python
class EnumDict(dict):
    def __init__(self):
        super().__init__()
        self._member_names = {}  # Имена членов в порядке объявления
        self._last_values = []  # Для auto()

    def __setitem__(self, key, value):
        """Запрещаем дубликаты имён"""
        if key in self._member_names:
            raise TypeError(f'Attempted to reuse member {key!r}')
        if _is_dunder(key):
            super().__setitem__(key, value)
        else:
            self._member_names[key] = None  # Запоминаем порядок
            super().__setitem__(key, value)

    @property
    def member_names(self):
        return list(self._member_names)
```

Обычный `dict` **позволит** `class Color: RED=1; RED=2` (перезапишет). `EnumDict` *
*запрещает** дубликаты: `RED` второй раз → **TypeError**. `_member_names` хранит **порядок** объявления (
`['RED', 'GREEN']`), **НЕ** порядок хеш-таблицы. Это **очень важно** для `list(Color)` → `Color.RED, Color.GREEN`.

## 3. _create_() - создание EnumMember (EnumType)

```python
def _create_(cls, member_name, value):
    # 1. Создаём EnumMember объект
    enum_member = object.__new__(cls)

    # 2. Устанавливаем свойства
    enum_member._name_ = member_name
    enum_member._value_ = value
    enum_member.__objclass__ = cls
    enum_member.__init__(value)

    # 3. Алиасы (дубликаты значений)
    for name, canonical_member in cls._member_map_.items():
        if canonical_member._value_ == value and canonical_member is not enum_member:
            enum_member._missing_(name)  # Алиас

    return enum_member
```

`Color.RED = 1` → **новый объект** `Color.RED` с `_name_='RED'`, `_value_=1`. Если
позже `ALIEN = 1` → `Color.ALIEN` **указывает** на тот же **самый** объект `Color.RED` (алиас).
`Color.RED is Color.ALIEN` = `True`. Это **экономит память** и гарантирует **единственность**.

## 4. Enum.__new__() / __init__() членов

```python
class Enum:
    def __new__(cls, value):
        # Ищем существующий член с таким value
        for member in cls._member_map_.values():
            if member._value_ == value:
                return member

        # Новый член (только через класс!)
        member = object.__new__(cls)
        member._value_ = value
        return member

    def __init__(self, value):
        self._value_ = value

    def __repr__(self):
        return f"<{self.__class__.__name__}.{self._name_}>"
```

`Color(1)` → ищет в `_member_map_` член с `value=1` → **возвращает** `Color.RED` (
существующий объект). `Color.RED` → `'Color.RED'`. **НЕ** создаёт новые экземпляры — возвращает **сигнатоны** (единичные
объекты).

## 5. auto() + _generate_next_value_ (Lib/enum.py)

```python
def auto():
    """Генерирует значение автоматически"""
    value = _next_value_
    _next_value_ += 1
    return value


class Color(Enum):
    def _generate_next_value_(name, start, count, last_values):
        # Автоинкремент
        return count

    RED = auto()  # 1
    GREEN = auto()  # 2
    BLUE = auto()  # 3
```

`RED = auto()` → вызывает `_generate_next_value_('RED', 1, 0, [])` → возвращает `0`.
`GREEN = auto()` → `_generate_next_value_('GREEN', 1, 1, [0])` → `1`. **Глобальный счётчик** `count` (номер члена).
Можно **переопределить**: `return 3**count` → `RED=1, GREEN=3, BLUE=9`.

## 6. Enum.__iter__() / __len__()

```python
def __iter__(cls):
    return (cls._member_map_[name] for name in cls._member_names_)


def __len__(cls):
    return len(cls._member_names_)


def __getitem__(cls, name):
    return cls._member_map_[name]
```

`list(Color)` → `[Color.RED, Color.GREEN]` **в порядке объявления**. `len(Color)` →
`3`. `Color['RED']` → `Color.RED`. **`_member_names_`** гарантирует **порядок** и **быстрый поиск**.

## 7. IntEnum / StrEnum наследование

```python
class IntEnum(int, Enum):
    pass


class StrEnum(str, Enum):
    pass


class Color(IntEnum):
    RED = 1
    GREEN = 2
```

`Color.RED + 1` → `2` (потому что `IntEnum` наследует `int`). `Color.GREEN == '2'` →
`False` (потому что `_value_=2`, а **НЕ** строка). `Color.RED.value` → `1` (сырое значение).

## 8. _simple_enum() - оптимизация (3.11+)

```python
def _simple_enum(etype, seq, value=1):
    """Быстрое создание простого enum"""
    members = dict()
    for i, name in enumerate(seq):
        val = etype._generate_next_value_(name, 1, i, [])
        member = object.__new__(etype)
        member._value_ = val
        member._name_ = name
        member.__objclass__ = etype
        members[name] = member

    # Создаём класс
    cls = type(seq.__class__.__name__, (etype,), members)
    cls._member_names_ = list(members)
    cls._member_map_ = members
    return cls
```

`Enum('RED GREEN BLUE')` → **оптимизированный** путь без метакласса. **Быстрее** для
простых случаев.

## 9. Flag / IntFlag - битовые флаги

```python
class Perm(Flag):
    R = 4
    W = 2
    X = 1

    RWX = R | W | X
```

`Perm.RWX` → **новый** объект с `_value_=7`. `Perm.RWX & Perm.R` → `Perm.R`.
`Perm.RWX | Perm.W` → `Perm.RWX`. **Битовые операции** работают с `_value_`.

## 10. pickle поддержка

```python
def __reduce_ex__(self, proto):
    return (self.__class__, (self._value_,))
```

`pickle.dumps(Color.RED)` → `(Color, (1,))` → при восстановлении `Color(1)` → **тот
же** `Color.RED` объект. **НЕ** новый экземпляр!

**Enum** в CPython 3.9+ — **EnumType метакласс**, **EnumDict** (порядок + уникальность), *
*`_member_map_/ _member_names_`**, **сигнатоны** через `__new__`, **`auto()`** + `_generate_next_value_`, *
*IntEnum/StrEnum** наследование, **Flag** битовые операции.

- [Содержание](/CONTENTS.md#содержание)