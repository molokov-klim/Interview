# **Магические методы**

[Содержание](/CONTENTS.md#содержание)

## **Junior Level**

Магические методы в Python — это специальные методы, которые начинаются и заканчиваются двойным подчеркиванием (dunder),
например `__init__`, `__str__`, `__add__`. Они позволяют настраивать поведение объектов при выполнении стандартных
операций: создание, вывод на печать, сложение, сравнение и т.д.

Когда вы пишете `obj + other`, Python внутри вызывает `obj.__add__(other)`. Если в вашем классе определен этот метод, вы
можете задать, как именно будет работать сложение для ваших объектов. Это делает код интуитивно понятным и позволяет
вашим объектам вести себя как встроенные типы данных.

Самые распространенные магические методы:

- `__init__` — инициализатор объекта (конструктор)
- `__str__` — строковое представление для человека (`str(obj)`, `print(obj)`)
- `__repr__` — строковое представление для разработчика (показывается в консоли)
- `__len__` — поддержка функции `len(obj)`
- `__getitem__`, `__setitem__` — доступ по индексу или ключу: `obj[key]`

Использование магических методов делает ваш код более "питоничным" и понятным.

[Содержание](/CONTENTS.md#содержание)

## **Middle Level**

Магические методы — это основа протоколов в Python, реализующих полиморфизм через утиную типизацию. Они делятся на
несколько категорий:

1. **Методы жизненного цикла**:
    - `__new__` — создание объекта (вызывается до `__init__`)
    - `__init__` — инициализация объекта
    - `__del__` — финализатор (вызывается перед удалением объекта)

2. **Строковые представления**:
    - `__str__` — для неформального представления (читабельного для человека)
    - `__repr__` — для формального представления (должно позволять воссоздать объект)
    - `__format__` — для поддержки `format(obj, spec)`
    - `__bytes__` — для `bytes(obj)`

3. **Протоколы сравнения**:
    - `__eq__`, `__ne__` — равенство/неравенство (`==`, `!=`)
    - `__lt__`, `__le__`, `__gt__`, `__ge__` — сравнения (`<`, `<=`, `>`, `>=`)
    - `__hash__` — вычисление хеша (обязателен для использования в множествах и как ключ словаря)

4. **Протоколы числовых операций**:
    - Арифметические: `__add__`, `__sub__`, `__mul__`, `__truediv__`
    - Унарные: `__neg__`, `__pos__`, `__abs__`
    - С преобразованием типов: `__int__`, `__float__`, `__bool__`

5. **Протоколы коллекций**:
    - `__len__` — длина
    - `__getitem__`, `__setitem__`, `__delitem__` — доступ по индексу/ключу
    - `__contains__` — поддержка оператора `in`
    - `__iter__`, `__next__` — итерация

6. **Протоколы вызова и контекста**:
    - `__call__` — вызов объекта как функции
    - `__enter__`, `__exit__` — контекстные менеджеры (оператор `with`)

7. **Дескрипторы и атрибуты**:
    - `__getattr__`, `__setattr__`, `__delattr__` — доступ к атрибутам
    - `__getattribute__` — перехват всех обращений к атрибутам
    - `__dir__` — список атрибутов для `dir(obj)`

8. **Сериализация**:
    - `__getstate__`, `__setstate__` — для pickle
    - `__reduce__`, `__reduce_ex__` — кастомная сериализация

[Содержание](/CONTENTS.md#содержание)

## **Senior Level**

В CPython 3.9+ **магические методы** (`__init__`, `__str__`, `__add__` и т.д.) — это **обычные атрибуты** в `tp_dict`
класса, которые **находятся** по MRO через `_PyType_Lookup()` и **вызываются** через **дескрипторный протокол** (
`tp_descr_get`) или **слоты** `tp_*` (`tp_call`,
`tp_as_number->nb_add`). [Objects/typeobject.c][Objects/object.c][Objects/descrobject.c]

## 1. Хранение магических методов в tp_dict

```c
// Objects/typeobject.c — после компиляции класса
static int
PyType_Ready(PyTypeObject *type)
{
    // ... MRO, bases ...
    
    // tp_dict содержит ВСЕ атрибуты класса, включая __init__, __str__
    PyObject *dict = type->tp_dict;  // {"__init__": func, "__str__": func, ...}
    
    // Магические методы ищутся как _PyType_Lookup(type, "__str__")
    // → PyDict_GetItem(dict, "__str__") → PyCFunctionObject или PyFunctionObject
}
```

`__init__`, `__str__` — **не специальные** поля в `PyTypeObject`. Это **обычные ключи** в `type.__dict__` (`tp_dict`).
`PyObject_GetAttr(MyClass, '__str__')` → `_PyType_Lookup(MyClass, '__str__')` → `tp_dict['__str__']`.

## 2. Поиск магического метода: _PyType_Lookup + дескриптор

```c
// Objects/typeobject.c
PyObject *
_PyType_Lookup(PyTypeObject *type, PyObject *name)  // name = "__str__"
{
    PyObject *mro = type->tp_mro;  // (MyClass, object)
    Py_ssize_t i, n = PyTuple_GET_SIZE(mro);
    
    for (i = 0; i < n; i++) {
        PyTypeObject *cls = PyTuple_GET_ITEM(mro, i);
        PyObject *res = PyDict_GetItem(cls->tp_dict, name);  // cls.__dict__["__str__"]
        if (res != NULL) {
            Py_INCREF(res);
            return res;  // ← НАШЛИ __str__!
        }
    }
    return NULL;
}
```

```c
// Objects/object.c — obj.__str__()
PyObject *PyObject_Str(PyObject *v)
{
    PyObject *res = NULL;
    
    // 1. Ищем __str__ по MRO
    PyObject *str_meth = _PyType_Lookup(Py_TYPE(v), &_Py_ID(__str__));
    if (str_meth != NULL) {
        res = call_method(v, str_meth);  // obj.__str__()
        if (res != NULL)
            return res;
        PyErr_Clear();  // __str__ вернул ошибку → fallback
    }
    
    // 2. Fallback: __repr__
    PyObject *repr_meth = _PyType_Lookup(Py_TYPE(v), &_Py_ID(__repr__));
    if (repr_meth != NULL) {
        return call_method(v, repr_meth);
    }
    
    // 3. Fallback: tp_str слот типа
    reprfunc f = Py_TYPE(v)->tp_repr;
    if (f != NULL)
        return f(v);
    
    PyErr_Format(PyExc_TypeError, "'%.200s' object is not repr-able", ...);
    return NULL;
}
```

`str(obj)` → `PyObject_Str(obj)` → `_PyType_Lookup(type(obj), '__str__')` → находит `MyClass.__str__` в `tp_dict` →
вызывает как **обычный метод** `obj.__str__()`.

## 3. Дескрипторный протокол для магических методов

```c
// Objects/descrobject.c — метод как дескриптор
typedef struct {
    PyObject_HEAD
    PyFunctionObject *func;        // def __str__(self): ...
} PyMethodDescrObject;

static PyObject *
method_descr_get(PyMethodDescrObject *descr, PyObject *obj, PyObject *type)
{
    if (obj == NULL) {
        Py_INCREF(descr);
        return (PyObject *)descr;  // unbound method
    }
    
    // Создаём bound method: meth.__func__ = descr->func, meth.__self__ = obj
    return PyMethod_New(descr->func, obj, type);
}
```

`MyClass.__str__` — это **PyMethodDescrObject**. При `obj.__str__` срабатывает `tp_descr_get` → создаётся **bound method
** (с `self=obj`). При `MyClass.__str__` (без obj) — **unbound method**.

## 4. Байткод вызова магического метода

```python
class C:
    def __str__(self):
        return "C!"


c = C()
print(str(c))  # → c.__str__()
```

```
# Байткод str(c):
  0 LOAD_GLOBAL         0 (str)      # builtin str()
  2 LOAD_FAST           0 (c)
  4 CALL                1
  6 CALL                1 (print)

# Внутри str(c) → PyObject_Str(c) → LOAD_ATTR '__str__' → CALL
```

В `Python/ceval.c` (LOAD_ATTR для `__str__`):

```c
case LOAD_ATTR: {
    PyObject *name = GETITEM(names, oparg);  // "__str__"
    PyObject *owner = TOP();                 // c
    
    PyObject *res = PyObject_GetAttr(owner, name);  // c.__str__
    Py_DECREF(TOP());  // снимаем c
    SET_TOP(res);      // кладём bound method
    DISPATCH();
}
```

`str(c)` → `PyObject_Str(c)` → `LOAD_ATTR '__str__'` → `PyObject_GetAttr(c, '__str__')` → **bound method** →
`CALL_FUNCTION`.

## 5. Слоты tp_* для магических методов (ускорение)

```c
// Include/cpython/object.h
typedef struct _typeobject {
    reprfunc tp_repr;              // __repr__
    reprfunc tp_str;               // __str__
    objobjproc tp_richcompare;     // __eq__, __lt__
    
    /* Number: __add__, __mul__ */
    struct {
        binaryfunc nb_add;         // obj + other
        /* ... */
    } *tp_as_number;
    
    /* Sequence: __len__, __getitem__ */
    struct {
        lenfunc sq_length;         // len(obj)
        ssizeargfunc sq_item;      // obj[i]
    } *tp_as_sequence;
} PyTypeObject;
```

```c
// Objects/longobject.c — int.__add__
static PyObject *
long_add(PyLongObject *a, PyObject *bb)
{
    PyLongObject *b;
    // ... реализация сложения больших чисел ...
}

// PyLong_Type.tp_as_number->nb_add = long_add;
```

Для **встроенных типов** (`int`, `list`) магические методы **НЕ** в `tp_dict`, а в **C-слотах** (`tp_str`,
`tp_as_number->nb_add`). `1 + 2` → `PyNumber_Add(1, 2)` → `PyLong_Type.tp_as_number->nb_add` → **C-код**, **без поиска
по MRO**!

## 6. PyObject_Str: полный алгоритм с fallback'ами

```c
// Objects/abstract.c
PyObject *PyObject_Str(PyObject *v)
{
    PyObject *res = NULL;
    PyTypeObject *tp = Py_TYPE(v);
    
    // 1. Быстрый путь: tp_str слот (только для встроенных типов)
    if (tp->tp_str != NULL) {
        res = tp->tp_str(v);
        if (res != NULL)
            return res;
        PyErr_Clear();
    }
    
    // 2. Медленный путь: obj.__str__()
    PyObject *meth = _PyObject_LookupAttr(v, &_Py_ID(__str__));
    if (meth != NULL) {
        res = PyObject_CallOneArg(meth, v);
        Py_DECREF(meth);
        if (res != NULL)
            return res;
        PyErr_Clear();
    }
    
    // 3. Fallback: obj.__repr__()
    meth = _PyObject_LookupAttr(v, &_Py_ID(__repr__));
    if (meth != NULL) {
        res = PyObject_CallOneArg(meth, v);
        Py_DECREF(meth);
        return res ? res : NULL;
    }
    
    // 4. Финальный fallback: "<obj object at 0x...>"
    return PyUnicode_FromFormat("<%s object at %p>", tp->tp_name, v);
}
```

`str(obj)` пробует **4 пути по скорости**:

1. `type.tp_str` (C-функция, **самый быстрый**).
2. `obj.__str__()` (Python-метод).
3. `obj.__repr__()` (fallback).
4. Сырой `"<obj at 0x...>"`.

## 7. Арифметические магические методы через слоты

```c
// Objects/abstract.c — универсальное сложение
PyObject *PyNumber_Add(PyObject *v, PyObject *w)
{
    PyTypeObject *vt = Py_TYPE(v), *wt = Py_TYPE(w);
    
    // 1. v.__add__(w)
    if (vt->tp_as_number && vt->tp_as_number->nb_add) {
        PyObject *res = vt->tp_as_number->nb_add(v, w);
        if (res != Py_NotImplemented)
            return res;
    }
    
    // 2. w.__radd__(v)
    if (wt->tp_as_number && wt->tp_as_number->nb_add) {
        PyObject *res = wt->tp_as_number->nb_add(v, w);
        if (res != Py_NotImplemented)
            return res;
    }
    
    PyErr_Format(PyExc_TypeError, "unsupported operand type(s) for +: '%s' and '%s'",
                 vt->tp_name, wt->tp_name);
    return NULL;
}
```

`a + b` → `PyNumber_Add(a, b)` → **СНАЧАЛА** `type(a).tp_as_number.nb_add(a, b)` (т.е. `a.__add__(b)`), **ПОТОМ**
`type(b).tp_as_number.nb_add(a, b)` (`b.__radd__(a)`). **НЕ** ищет `__add__` в `tp_dict`!

## 8. Создание bound method из магического метода

```c
// Objects/classobject.c
PyObject *
PyMethod_New(PyFunctionObject *func, PyObject *self, PyObject *cls)
{
    PyMethodObject *meth = PyObject_GC_New(PyMethodObject, &PyMethod_Type);
    if (meth == NULL)
        return NULL;
    
    Py_INCREF(func);
    meth->im_func = func;      // __str__ функция
    
    Py_XINCREF(self);
    meth->im_self = self;      // экземпляр obj
    
    Py_XINCREF(cls);
    meth->im_class = cls;      // MyClass
    
    PyObject_GC_Track(meth);
    return (PyObject *)meth;
}
```

`c.__str__` → `PyMethod_New(__str__, c, C)` → **bound method** с тремя полями: `im_func` (код `__str__`), `im_self` (=
`c`), `im_class` (=`C`). При вызове: `im_func(im_self)`.

## 9. __init__ и tp_init слот

```c
// Objects/typeobject.c — type.__call__()
static PyObject *
type_call(PyTypeObject *type, PyObject *args, PyObject *kwds)
{
    // 1. tp_new: создаём "пустой" объект
    PyObject *obj = type->tp_new(type, args, kwds);
    if (obj == NULL)
        return NULL;
    
    // 2. tp_init: вызываем __init__(obj, *args, **kwds)
    if (type->tp_init != NULL) {
        int res = type->tp_init(obj, args, kwds);
        if (res < 0) {
            Py_DECREF(obj);
            return NULL;
        }
    }
    
    return obj;
}
```

`C()` → `type_call(C, (), {})` → `C.tp_new(C)` (создаёт объект) → `C.tp_init(obj)` → ищет `__init__` в `tp_dict` →
вызывает `obj.__init__()`.

## Итог работы магических методов в CPython

**Магические методы** работают через **3 механизма**:

1. **Слоты `tp_*`** (`tp_str`, `tp_as_number->nb_add`) — **быстрые C-функции** для встроенных типов.
2. **Поиск в `tp_dict`** по MRO (`_PyType_Lookup`) — Python-методы `__str__`, `__add__`.
3. **Дескрипторный протокол** — превращение в **bound method** с `self`.

Байткод `str(c)` → `LOAD_ATTR '__str__'` → **bound method** → `CALL`. **ВСЁ одинаково** для всех магических методов!

- [Содержание](/CONTENTS.md#содержание)