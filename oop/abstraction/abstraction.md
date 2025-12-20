# **Абстракция**

[Содержание](/CONTENTS.md#содержание)

## **Junior Level**

Абстракция в объектно-ориентированном программировании — это процесс выделения существенных характеристик объекта и
игнорирования несущественных деталей. Представьте, что вы заказываете пиццу: вас интересует её состав и цена, но не
детали того, как её готовят на кухне или доставляют курьером. Вы абстрагируетесь от сложных процессов, фокусируясь
только на том, что важно для вас как заказчика.

В программировании абстракция позволяет создавать модели реальных объектов, которые содержат только те свойства и
методы, которые нужны для решения конкретной задачи. Например, в программе для библиотеки класс "Книга" может иметь
свойства "автор", "название", "год издания" и методы "взять", "вернуть". Но он не будет включать такие детали, как "вес
книги", "цвет обложки" или "материал страниц", если они не важны для работы библиотечной системы.

Абстракция делает код проще, понятнее и легче для изменения.

[Содержание](/CONTENTS.md#содержание)

## **Middle Level**

В Python абстракция реализуется через несколько механизмов, каждый из которых предоставляет свой уровень сокрытия
деталей:

1. **Абстрактные классы**:
    - Определяются с помощью модуля `abc` (abstract base classes).
    - Не могут быть инстанциированы напрямую.
    - Содержат абстрактные методы (помеченные `@abstractmethod`), которые должны быть реализованы в дочерних классах.
    - `@abstractproperty` (устаревшее в Python 3.3+) и комбинация `@property` с `@abstractmethod`.

2. **Интерфейсы**:
    - В Python нет явных интерфейсов как в Java/C#, но их роль выполняют абстрактные классы и протоколы.
    - Протоколы — неформальные интерфейсы, основанные на утиной типизации.

3. **Инкапсуляция как средство абстракции**:
    - `_single_underscore`: защищённый атрибут (соглашение).
    - `__double_underscore`: приватный атрибут с name mangling.
    - Свойства (`@property`) для контроля доступа к атрибутам.

4. **Уровни абстракции**:
    - **Высокоуровневые абстракции**: интерфейсы, абстрактные классы.
    - **Среднеуровневые**: конкретные классы с детализированным поведением.
    - **Низкоуровневые**: вспомогательные классы, детали реализации.

5. **Шаблоны проектирования, основанные на абстракции**:
    - **Фабричный метод**: абстрагирует процесс создания объектов.
    - **Мост (Bridge)**: разделяет абстракцию и реализацию.
    - **Стратегия (Strategy)**: абстрагирует семейство алгоритмов.

6. **Абстракция данных**:
    - Создание типов данных, которые скрывают свою внутреннюю структуру.
    - Предоставление только операций для работы с данными.

[Содержание](/CONTENTS.md#содержание)

## **Senior Level**

В CPython 3.9+ **абстракция** — **ABC** (`Lib/abc.py` + `Lib/_collections_abc.py`) с **ABCMeta** метаклассом, *
*`abstractmethod`** (`__isabstractmethod__=True`), **`register()`** виртуальное наследование, *
*`__subclasscheck__`/`__instancecheck__`** переопределение `issubclass`/`isinstance`, **structural typing** через
`typing.Protocol`. `Lib/abc.py`,`Objects/abstract.c`

## 1. ABCMeta.__new__() - метакласс ABC (Lib/abc.py)

```python
class ABCMeta(type):
    def __new__(mcls, name, bases, namespace, **kwargs):
        # 1. Создаём обычный класс
        cls = super().__new__(mcls, name, bases, namespace, **kwargs)

        # 2. Отмечаем абстрактные методы
        abstracts = []
        for key, value in namespace.items():
            if getattr(value, '__isabstractmethod__', False):
                abstracts.append(key)

        # 3. Если есть абстрактные методы - класс абстрактный
        cls.__abstractmethods__ = set(abstracts)

        # 4. Регистрируем в _abc_registry
        _abc_registry[cls] = None

        return cls
```

`class MyABC(ABC): @abstractmethod def method(self): pass` → **ABCMeta** видит `__isabstractmethod__=True` на `method` →
`MyABC.__abstractmethods__ = {'method'}`. **ABCMeta** — **обычный метакласс**, но **добавляет** `__abstractmethods__` и
регистрирует в глобальном `_abc_registry` словаре.

## 2. @abstractmethod декоратор (Lib/abc.py)

```python
def abstractmethod(funcobj):
    """Устанавливает __isabstractmethod__ = True"""
    funcobj.__isabstractmethod__ = True
    return funcobj
```

`@abstractmethod def f(): pass` → `f.__isabstractmethod__ = True`. **Метакласс** сканирует `__dict__` класса, находит
все такие методы → **помечает класс абстрактным**. **Очень просто** — один атрибут!

## 3. ABCMeta.__subclasscheck__() - переопределение issubclass()

```python
def __subclasscheck__(cls, subclass):
    # 1. Обычная проверка наследования
    if cls in getattr(subclass, '__mro__', ()):
        return True

    # 2. Проверяем register()
    for s in subclass.__mro__:
        if cls in _abc_impl_cache.get(s, ()):
            return True

    # 3. Проверяем реализацию абстрактных методов
    ok = all(cls.__abstractmethod_subset(obj) for obj in subclass.__mro__)
    if ok:
        _abc_impl_cache[subclass].add(cls)
        return True

    return False
```

`issubclass(MyClass, Sequence)` → **НЕ** смотрит на наследование! Проверяет: 1) **наследуется** ли от
зарегистрированного класса, 2) **реализует** ли все методы `Sequence` (`__len__`, `__getitem__`). `MyList(Sequence)` → *
*True** даже **без** `class MyList(Sequence):`!

## 4. ABC.register() - виртуальное наследование (Lib/abc.py)

```python
def register(cls, subclass):
    """Регистрирует subclass как виртуальный"""
    if not issubclass(subclass, cls):
        for obj in subclass.__mro__:
            if cls in _abc_cache.get(obj, ()):
                break
        else:
            raise RuntimeError('Refusing to register %r as subclass of %r' %
                               (subclass, cls))

    # Добавляем в кэш
    _abc_impl_cache.setdefault(subclass, set()).add(cls)
    _abc_impl_cache.setdefault(cls, set()).add(subclass)

    return subclass
```

`Sequence.register(MyList)` → `issubclass(MyList, Sequence) = True` **навсегда**! `MyList` **НЕ** наследует код
`Sequence`, но **поддерживает** протокол (`__len__`, `__getitem__`). **Duck typing** + **явная регистрация**.

## 5. collections.abc.Sequence реализация (Lib/_collections_abc.py)

```python
class Sequence(Collection):
    """Последовательности: list, tuple, str"""
    __slots__ = 'register', '__abstractmethods__'

    @abstractmethod
    def __getitem__(self, index):
        """Получить элемент по индексу"""
        return NotImplemented

    @abstractmethod
    def __len__(self):
        """Длина последовательности"""
        return NotImplemented

    def __iter__(self):
        """Итератор"""
        return SequenceIter(self)

    def __contains__(self, value):
        """Проверка вхождения"""
        for item in self:
            if item == value:
                return True
        return False
```

`Sequence` **НЕ** содержит кода! Только **список абстрактных методов** (`__len__`, `__getitem__`).
`isinstance(my_list, Sequence)` → проверяет **наличие** этих методов. **Реализация** в `list.c` (`list_length`,
`list_subscript`).

## 6. PySequence_Check() C API (Objects/abstract.c)

```c
int PySequence_Check(PyObject *s) {
    PyTypeObject *tp = Py_TYPE(s);
    
    // 1. Быстрая проверка по tp_as_sequence
    if (tp->tp_as_sequence && tp->tp_as_sequence->sq_item != NULL) {
        return 1;                          // Реальная последовательность
    }
    
    // 2. ABC проверка
    if (PyType_FastSubclass(tp, Py_TPFLAGS_SEQUENCE)) {
        return 1;
    }
    
    // 3. Проверяем __getitem__
    PyObject *item = PyObject_GetItem(s, PyLong_FromLong(0));
    Py_XDECREF(item);
    if (!PyErr_Occurred()) {
        return 1;                          // Поддерживает __getitem__
    }
    PyErr_Clear();
    
    return 0;
}
```

`len(my_list)` → `PySequence_Check(my_list)` → `PyList_Type.tp_as_sequence->sq_length != NULL` → **True**. **Быстрый
путь** через C слоты, **медленный** через `__getitem__`.

## 7. list_subscript() - реализация __getitem__ (Objects/listobject.c)

```c
static PyObject *list_subscript(PyListObject *self, PyObject *item) {
    if (PyLong_Check(item)) {
        Py_ssize_t i = PyLong_AsSsize_t(item);
        if (i < 0)
            i += Py_SIZE(self);
        if (i >= 0 && i < Py_SIZE(self)) {
            return Py_NewRef(self->ob_item[i]);
        }
    }
    PyErr_SetString(PyExc_IndexError, "list index out of range");
    return NULL;
}
```

`my_list[0]` → `PyObject_GetItem(my_list, 0)` → `list_subscript(my_list, 0)` → `my_list->ob_item[0]`. **Прямой доступ**
к массиву указателей!

## 8. typing.Protocol - structural typing (typing.py)

```python
class Protocol(Generic, metaclass=ProtocolMeta):
    """Структурная типизация - duck typing"""
    __slots__ = ()

    def __instancecheck__(cls, instance):
        # Проверяем наличие методов по сигнатуре
        return _is_protocol_compatible(instance, cls.__dict__)

    def __subclasscheck__(cls, subclass):
        # Проверяем наличие абстрактных методов
        return _is_protocol_compatible(subclass, cls.__dict__)


def _is_protocol_compatible(obj, protocol_attrs):
    for attr_name, attr_value in protocol_attrs.items():
        if attr_name.startswith('_'):
            continue
        obj_attr = getattr(obj, attr_name, None)
        if obj_attr != attr_value:
            return False
    return True
```

`class HasLen(Protocol): def __len__(self) -> int: ...` → `isinstance(my_list, HasLen)` → **НЕ** наследование! Проверяет
**наличие** `__len__` с **правильной** сигнатурой. **Структурная типизация** — работает **любая** реализация!

## 9. PyList_Type.tp_as_sequence - C протокол (Include/listobject.h)

```c
static PySequenceMethods list_as_sequence = {
    .sq_length = list_length,          // len(lst)
    .sq_concat = list_concat,          // lst + lst2
    .sq_repeat = list_repeat,          // lst * 3
    .sq_item = list_subscript,         // lst[i]
    .sq_slice = list_slice,            // lst[1:3]
    .sq_ass_item = list_ass_item,      // lst[i] = x
    .sq_ass_slice = list_ass_slice,    // lst[1:3] = [...]
    .sq_contains = list_contains,      // x in lst
    .sq_inplace_concat = list_inplace_concat,
    .sq_inplace_repeat = list_inplace_repeat,
};
```

`PyList_Type.tp_as_sequence = &list_as_sequence` → `len(lst)` → `PySequence_Length(lst)` →
`lst->tp_as_sequence->sq_length(lst)` → `list_length()`. **Слоты** — **таблица указателей** на C функции!

## 10. Байткод: isinstance(obj, Sequence) (ceval.c)

```c
case ISINSTANCE: {
    PyObject *inst = TOP();            // obj
    PyObject *cls = SECOND();          // Sequence
    
    int retval = PyObject_IsInstance(inst, cls);  // Вызывает __instancecheck__
    
    Py_DECREF(inst);
    Py_DECREF(cls);
    
    if (retval < 0) {
        goto error;
    }
    
    Py_SETREF(SECOND(), PyBool_FromLong(retval));
    DISPATCH();
}
```

`isinstance(lst, Sequence)` → `PyObject_IsInstance(lst, Sequence)` → `Sequence.__instancecheck__(lst)` → **True**. *
*Делегирует** ABC метаклассу!

**Абстракция** в CPython 3.9+ — **ABCMeta** (`__subclasscheck__`), **`abstractmethod`** (`__isabstractmethod__`), *
*`register()`** виртуальное наследование, **`tp_as_sequence`** C протоколы, **`typing.Protocol`** structural typing, *
*`PySequence_Check()`** duck typing.

- [Содержание](/CONTENTS.md#содержание)