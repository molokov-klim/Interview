# **copy() и deepcopy()**

[Содержание](/CONTENTS.md#содержание)

## **Junior Level**

Когда вам нужно создать независимую копию объекта в Python, на помощь приходят функции `copy()` и `deepcopy()` из модуля
`copy`. Разница между ними напоминает разницу между созданием ярлыка на папку или её копирование.

`copy()` создаёт поверхностную (shallow) копию — новый "контейнер", который, однако, продолжает ссылаться на те же
внутренние объекты, что и оригинал. Представьте, что вы сделали копию папки с документами: сама папка новая, но лежащие
в ней файлы — те же самые. Если вы измените содержимое файла в копии папки, изменения отразятся и в оригинале, потому
что это один и тот же физический файл.

`deepcopy()` выполняет глубокое копирование — она рекурсивно создаёт полностью независимые копии всех объектов, включая
вложенные структуры. Это похоже на то, как если бы вы не только создали новую папку, но и перепечатали каждый документ
внутри неё. После такой операции изменения в копии никак не затронут оригинал, и наоборот.

Выбор между этими двумя функциями зависит от ситуации. Если вы работаете с неизменяемыми объектами (числа, строки,
кортежи) или уверены, что не будете менять вложенные элементы, достаточно `copy()`. Когда же требуется полная изоляция —
например, для тестовых данных, конфигураций или сложных изменяемых структур — без `deepcopy()` не обойтись.

[Содержание](/CONTENTS.md#содержание)

## **Middle Level**

Под поверхностью этих функций скрывается довольно интересная механика. Поверхностное копирование создаёт новый объект
того же типа, но не углубляется в его содержимое — все вложенные элементы остаются общими с оригиналом. Это происходит
потому, что `copy()` копирует только ссылки, а не сами объекты. Такой подход эффективен по памяти и времени, но требует
осторожности: изменение вложенного изменяемого объекта в копии повлияет на оригинал.

Глубокое копирование устроено сложнее. Функция `deepcopy()` рекурсивно обходит всю структуру объекта, создавая новые
экземпляры для каждого встреченного изменяемого объекта. Она умеет корректно обрабатывать даже циклические ссылки (когда
объекты ссылаются друг на друга), используя внутренний словарь для отслеживания уже скопированных объектов и
предотвращения бесконечной рекурсии.

Обе функции уважают специальные методы объектов: если класс определяет `__copy__()` или `__deepcopy__()`, используются
эти реализации, что позволяет контролировать процесс копирования. Это особенно полезно для объектов со сложным
внутренним состоянием, например, открытых файлов или сетевых соединений, которые не могут быть просто продублированы.

Стоит помнить о некоторых особенностях. Некоторые объекты (модули, классы, функции) не копируются, а возвращаются как
есть, поскольку обычно в этом нет смысла. Также `deepcopy()` может быть довольно медленной для больших и сложных
структур из-за рекурсивного обхода и создания множества новых объектов.

На практике глубокое копирование незаменимо при работе с многомерными структурами данных, графами или любыми объектами,
где важна полная независимость копии. Поверхностное же копирование часто используется для создания "снимков" состояния
объекта в определённый момент, когда достаточно скопировать только верхний уровень структуры.

[Содержание](/CONTENTS.md#содержание)

## **Senior Level**

`copy.copy()` и `copy.deepcopy()` — это чистый Python‑код в `Lib/copy.py`, который реализует общий протокол копирования
через поиск специальных методов, тип‑специфические таблицы и рекурсивный обход структуры с мемоизацией. Байткода как
такового там немного (обычные вызовы функций и атрибутов), вся «магия» — в логике диспетчеризации.

## copy.copy(): выбор стратегии

В CPython 3.9+ **copy.copy()** использует **tp_traverse** + `__copy__()` + **shallow copy** (`PyObject_Malloc`), *
*copy.deepcopy()** — **рекурсивный _deepcopy_dispatch** с **memo dict** (id→copy), `_reconstruct()` для классов, защита
от циклов. `Lib/copy.py`,`Objects/object.c`

## 1. copy.copy() - Lib/copy.py (Python C API)

```python
def copy(x):
    """Shallow copy operation on arbitrary Python objects."""
    cls = type(x)

    # 1. __copy__() метод класса
    copier = _copy_dispatch.get(cls)
    if copier is not None:
        return copier(x)

    try:
        # 2. Пробуем __copy__()
        cpy = x.__copy__()
        if hasattr(cpy, '__dict__'):
            cpy.__dict__.update(x.__dict__)
        return cpy
    except AttributeError:
        pass

    # 3. Встроенные типы
    if isinstance(x, dict):
        return dict(x)
    if isinstance(x, list):
        return list(x)
    if isinstance(x, set):
        return set(x)
    # ...

    # 4. Fallback: PyObject_CallMethod("__getnewargs__")
    newargs = _copy_call_getnewargs(x)
    return _reconstruct(x, len(newargs), newargs, ())
```

`copy(lst)` → `list(lst)` (shallow), `copy(obj)` → `obj.__copy__()` или
`type(obj)(*obj.__getnewargs__())`. **Не рекурсивно** — копирует только верхний уровень.

## 2. copy.deepcopy() - рекурсивный dispatch (Lib/copy.py)

```python
def deepcopy(x, memo=None):
    """Deep copy operation on arbitrary Python objects."""
    if memo is None:
        memo = {}

    d = id(x)
    y = memo.get(d, None)  # Уже копировали?
    if y is not None:
        return y  # Цикл! Возвращаем копию

    cls = type(x)

    # 1. __deepcopy__()
    copier = _deepcopy_dispatch.get(cls)
    if copier is not None:
        y = copier(x, memo)

    # 2. Встроенные типы
    elif cls in _deepcopy_dispatch:
        y = _deepcopy_dispatch[cls](x, memo)

    # 3. Классы с __reduce__
    elif hasattr(x, '__reduce_ex__'):
        args = x.__reduce_ex__(2)[1]
        y = _reconstruct(x, 0, args, (), None, None, memo)

    else:
        # 4. Fallback: __getnewargs__ + __dict__ + __slots__
        reductor = getattr(x, "__reduce_ex__", None)
        if reductor is not None:
            args = reductor(4)[1]
        else:
            args = _copy_call_getnewargs(x)
        y = _reconstruct(x, len(args), args, (), None, None, memo)

    memo[d] = y  # Записываем в memo
    return y
```

`deepcopy(obj)` рекурсивно копирует **все вложенные объекты**. `memo[id(obj)]=copy`
предотвращает **бесконечный цикл** при `l.append(l)`.

## 3. _deepcopy_dispatch таблица (Lib/copy.py)

```python
_deepcopy_dispatch = d = {}
d[dict] = _deepcopy_dict
d[list] = _deepcopy_list
d[set] = _deepcopy_set
d[tuple] = _deepcopy_tuple
d[frozenset] = _deepcopy_frozenset


def _deepcopy_dict(x, memo):
    y = {}
    memo[id(x)] = y
    for key, value in x.items():
        y[deepcopy(key, memo)] = deepcopy(value, memo)  # Рекурсия!
    return y


def _deepcopy_list(x, memo):
    y = []
    memo[id(x)] = y
    append = y.append  # Быстрый append
    for item in x:
        append(deepcopy(item, memo))  # Рекурсия!
    return y
```

`_deepcopy_dict({a: [1]})` → `{copy(a): copy([1])}` → рекурсивно копирует **каждый**
ключ/значение. `memo` сохраняет уже сделанные копии.

## 4. PyList_Type.tp_richcompare для list.copy()

```c
static PyObject *list_copy(PyListObject *self) {
    Py_ssize_t len = Py_SIZE(self);
    PyObject **src = self->ob_item;
    PyObject **dest;
    
    // Создаём новый список той же ёмкости
    PyListObject *newlist = _PyList_New(len);  # Выделяем ob_item[len+overalloc]
    if (newlist == NULL)
        return NULL;
    
    dest = newlist->ob_item;
    
    // Копируем указатели (shallow!)
    for (Py_ssize_t i = 0; i < len; i++) {
        PyObject *v = src[i];
        Py_INCREF(v);              // +refcnt на каждый элемент
        dest[i] = v;
    }
    
    return (PyObject *)newlist;
}
```

`lst.copy()` → `_PyList_New(len)` → копирует **указатели** `ob_item[]` с `Py_INCREF()`. *
*Shallow** — вложенные списки **не копируются**.

## 5. _PyDict_NewPresized() для dict.copy() (3.9+)

```c
PyObject *_PyDict_NewPresized(Py_ssize_t expected_size) {
    PyDictObject *mp;
    Py_ssize_t table_size = _PyDict_NewPresizedTableSize(expected_size);
    
    mp = PyObject_GC_New(PyDictObject, &PyDict_Type);
    if (mp == NULL)
        return NULL;
    
    mp->ma_used = 0;
    mp->ma_version_tag = DICT_NEXT_VERSION();
    mp->ma_table = PyMem_Calloc(table_size, sizeof(PyDictUnicodeEntry));
    mp->ma_keys = NULL;  // Создадим позже
    mp->ma_values = NULL;
    
    // Выделяем ma_keys той же ёмкости
    if (dictkeys_new(mp, table_size) < 0) {
        PyDictObject_Clear(mp);
        Py_DECREF(mp);
        return NULL;
    }
    
    _PyObject_GC_TRACK(mp);
    return (PyObject *)mp;
}
```

`dict.copy()` → новый PyDictObject с **пустой** `ma_keys` + `ma_values`, затем
`dict_update(new, old)` копирует все пары **shallow**.

## 6. _reconstruct() для пользовательских классов

```python
def _reconstruct(x, len_args, args, state, listitems, dictitems, memo):
    # 1. Создаём новый экземпляр
    cls = type(x)
    newobj = object.__new__(cls)

    # 2. Восстанавливаем состояние
    if state is not None:
        state = deepcopy(state, memo)
        newobj.__dict__ = state
    elif hasattr(newobj, '__slots__'):
        # __slots__ копируем вручную
        for key, value in state.items():
            setattr(newobj, key, deepcopy(value, memo))

    # 3. Восстанавливаем mutable атрибуты
    if len_args > 0:
        newobj.__init__(*deepcopy(args, memo))

    return newobj
```

`deepcopy(MyClass())` → `MyClass.__new__()` → `deepcopy(state)` → `__dict__` →
`__init__(*args)`. **Полная копия состояния**.

## 7. Защита от рекурсии: memo[id(obj)]

```python
l = [1, 2]
l.append(l)  # Цикл!
deep_l = deepcopy(l, memo={})

# Первый вызов: memo={} пустой
y = _deepcopy_list(l, memo)
memo[id(l)] = y  # Записали!

# Рекурсивный вызов l[3] = l:
item = deepcopy(l[3], memo)  # l[3] == l
d = id(l[3])  # == id(l)
y = memo.get(d)  # НАЙДЕН! Возвращаем копию
```

`memo[id(original)]=copy` **предотвращает** копирование уже обработанных объектов.
`[1, [1, [1, ...]]]` копируется **один раз**.

## 8. C-level: PyObject_Malloc для shallow copy

```c
static PyObject *generic_copy(PyObject *self) {
    PyObject *newobj;
    
    // 1. Выделяем память той же структуры
    newobj = PyType_GenericAlloc(Py_TYPE(self), 1);
    if (newobj == NULL)
        return NULL;
    
    // 2. Копируем PyObject_HEAD
    Py_INCREF(Py_TYPE(self));
    newobj->ob_refcnt = 1;
    newobj->ob_type = Py_TYPE(self);
    
    // 3. Копируем __dict__ shallow
    if (Py_TYPE(self)->tp_dictoffset != 0) {
        PyObject **dictptr = _PyObject_GetDictPtr(self);
        if (dictptr != NULL && *dictptr != NULL) {
            PyObject *dict = PyDict_Copy(*dictptr);  # Shallow dict
            if (dict == NULL) {
                Py_DECREF(newobj);
                return NULL;
            }
            _PyObject_SetDict(newobj, dict);
        }
    }
    
    return newobj;
}
```

`__copy__()` → `PyType_GenericAlloc()` → копирует `ob_type` + `__dict__.copy()` (shallow). *
*Не трогает атрибуты**.

## 9. Не копируемые типы (deepcopy игнорирует)

```python
# Модули, функции, файлы, сокеты НЕ копируются
def _deepcopy_func(x, memo):
    return x  # Возвращаем оригинал!


def _deepcopy_module(x, memo):
    return x


_deepcopy_dispatch[type(open('file.txt'))] = _deepcopy_file_like
```

`deepcopy(open())` → **оригинал** (нельзя клонировать дескриптор). Функции/классы — **shallow
** (refcnt++).

**copy.copy()/deepcopy()** в CPython 3.9+ — **Lib/copy.py** с `_deepcopy_dispatch`, **memo[id→copy]** против циклов,
`PyList_New()`/`PyDict_NewPresized()` + `Py_INCREF()` (shallow), `_reconstruct()` для классов, `__copy__()`/
`__deepcopy__()` хуки.

- [Содержание](/CONTENTS.md#содержание)