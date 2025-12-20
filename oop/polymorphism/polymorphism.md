# **Полиморфизм**

[Содержание](/CONTENTS.md#содержание)

## **Junior Level**

Полиморфизм — это способность объектов с разной внутренней структурой иметь одинаковый интерфейс и реагировать на одни и
те же команды по-разному.

Представьте, что у вас есть пульт дистанционного управления, который работает с разными
устройствами: нажатие кнопки "включить" включает и телевизор, и кондиционер, и музыку, но делает это по-разному. Каждое
устройство понимает команду "включить" по-своему.

В программировании полиморфизм позволяет использовать один и тот же метод или функцию для работы с объектами разных
классов. Например, если у классов `Dog` и `Cat` есть метод `speak()`, то при вызове `dog.speak()` мы услышим "Гав!", а
при `cat.speak()` — "Мяу!". Это позволяет писать более гибкий и универсальный код.

В Python полиморфизм тесно связан с концепцией "утиной типизации" (duck typing): если объект ходит как утка и крякает
как утка, то он утка. То есть Python не проверяет тип объекта заранее, а смотрит, есть ли у него нужный метод в момент
вызова.

[Содержание](/CONTENTS.md#содержание)

## **Middle Level**

В Python полиморфизм проявляется в нескольких формах:

1. **Утиная типизация (Duck Typing)**:
    - Основной механизм полиморфизма в Python
    - Объекты используются на основе наличия методов/атрибутов, а не их типа
    - Пример: любой объект с методом `__len__()` можно передать в функцию `len()`

2. **Переопределение методов в наследовании**:
    - Дочерние классы могут переопределять методы родительских классов
    - Вызов метода у объекта дочернего класса использует переопределённую версию

3. **Перегрузка операторов**:
    - Магические методы (`__add__`, `__str__`, `__getitem__`) позволяют определить поведение операторов для
      пользовательских классов
    - Один и тот же оператор (`+`, `==`, `[]`) работает по-разному для разных типов

4. **Абстрактные базовые классы (ABC)**:
    - Определяют интерфейсы, которые должны быть реализованы
    - `collections.abc` содержит абстрактные классы для коллекций (`Iterable`, `Sequence`, `Mapping`)

5. **Протоколы**:
    - Неформальные интерфейсы, основанные на наличии определённых методов
    - Пример: протокол итератора требует методы `__iter__()` и `__next__()`

6. **Функции высшего порядка**:
    - Функции, принимающие другие функции как аргументы или возвращающие функции
    - `map()`, `filter()`, `sorted()` с параметром `key`

7. **Множественная диспетчеризация**:
    - Используется в библиотеках типа `multipledispatch`
    - Выбор реализации функции на основе типов нескольких аргументов

8. **`singledispatch` из `functools`**:
    - Позволяет создавать перегруженные функции на основе типа первого аргумента

[Содержание](/CONTENTS.md#содержание)

## **Senior Level**

В CPython 3.9+ **полиморфизм** “под капотом” – это:

1) **таблицы слотов в PyTypeObject** (`tp_as_number`, `tp_as_sequence`, `tp_call` и т.п.);
2) **общие “абстрактные” функции** (`PyNumber_Add`, `PySequence_GetItem`), которые по `Py_TYPE(obj)` берут нужную
   реализацию;
3) **дескрипторный протокол** и `tp_getattro` для полиморфных атрибутов и методов.

***

## 1. Слоты в PyTypeObject – “виртуальные таблицы”

```c
/* Упрощённый фрагмент PyTypeObject (Include/cpython/object.h) */
typedef struct _typeobject {
    PyObject_VAR_HEAD                /* стандартный заголовок */
    const char *tp_name;             /* имя типа, например "int" */
    Py_ssize_t tp_basicsize;         /* размер объекта */
    Py_ssize_t tp_itemsize;          /* для переменной длины */

    /* Числовой протокол: +, -, *, /, ... */
    struct PyNumberMethods *tp_as_number;   /* указатель на таблицу числовых операций */

    /* Последовательность: len, concat, повтор, индексирование */
    struct PySequenceMethods *tp_as_sequence; /* таблица seq-операций */

    /* Отображение: словарный интерфейс */
    struct PyMappingMethods *tp_as_mapping; /* таблица mapping-операций */

    unaryfunc tp_call;               /* вызов объекта: obj() */
    reprfunc tp_str;                 /* str(obj) */
    getattrofunc tp_getattro;        /* получение атрибута obj.attr */
    setattrofunc tp_setattro;        /* установка атрибута */

    /* ... много других слотов опущено ... */
} PyTypeObject;
```

Комментарии по строкам:

- `tp_as_number` – указатель на структуру `PyNumberMethods`, где лежат C-функции для операций вроде `+`, `-`, `*`, `==`
  и т.д.
- `tp_as_sequence` – структура `PySequenceMethods` с реализациями `len`, `obj[i]`, конкатенации и т.п.
- `tp_as_mapping` – структура `PyMappingMethods` с реализацией словарного интерфейса (`obj[key]`, `len(obj)` как
  mapping).
- `tp_call` – функция “вызвать объект”, используется для функций, классов, объектов с `__call__`.
- `tp_getattro` / `tp_setattro` – общий вход для доступа к атрибутам; конкретный тип может переопределять поведение.

“По‑человечески для тупых”: у каждого типа в CPython есть **набор указателей на функции**, которые реализуют операции.
Это аналог **vtable** в C++: есть один интерфейс “сложить два объекта”, но реализация берётся из `tp_as_number`
конкретного типа (`int`, `float`, `list` и т.п.). То есть “полиморфизм” – это просто **выбор строки из таблицы** по
`Py_TYPE(obj)`.

***

## 2. Числовой полиморфизм: PyNumber_Add и nb_add

```c
/* Include/cpython/object.h */
typedef struct {
    binaryfunc nb_add;         /* a + b */
    binaryfunc nb_subtract;    /* a - b */
    binaryfunc nb_multiply;    /* a * b */
    /* ... ещё много числовых операций ... */
} PyNumberMethods;

/* Objects/abstract.c – абстрактная операция сложения */
PyObject *PyNumber_Add(PyObject *v, PyObject *w)
{
    /* Берём типы операндов */
    PyTypeObject *vt = Py_TYPE(v);    /* тип левого */
    PyTypeObject *wt = Py_TYPE(w);    /* тип правого */

    /* Пытаемся вызвать nb_add у левого операнда */
    if (vt->tp_as_number && vt->tp_as_number->nb_add) {
        PyObject *res = vt->tp_as_number->nb_add(v, w);
        if (res != Py_NotImplemented) {
            return res;
        }
        Py_XDECREF(res);
    }

    /* Пытаемся вызвать nb_add у правого (обратное сложение) */
    if (wt->tp_as_number && wt->tp_as_number->nb_add &&
        wt != vt) {
        PyObject *res = wt->tp_as_number->nb_add(v, w);
        if (res != Py_NotImplemented) {
            return res;
        }
        Py_XDECREF(res);
    }

    /* Ни один тип не смог – выбрасываем TypeError */
    PyErr_Format(PyExc_TypeError,
                 "unsupported operand type(s) for +: '%.100s' and '%.100s'",
                 vt->tp_name, wt->tp_name);
    return NULL;
}
```

- `PyNumber_Add(a, b)` не знает, какие именно типы у `a` и `b`.
- Он смотрит на `tp_as_number->nb_add` у типа левого аргумента. Если есть – вызывает её.
- Если левый не умеет или вернул `Py_NotImplemented` – пробует правый аргумент (обратная операция, как `__radd__`).
- Если оба “не умеют” – выбрасывается `TypeError`.

когда ты пишешь `a + b`, интерпретатор **сначала смотрит**, какой у `a` тип, и есть ли у этого типа
реализация `+`. Если есть – вызывает её. Если этот тип не знает, как сложиться с типом `b`, он говорит “я не умею” (
`Py_NotImplemented`), и тогда пробуют уже реализацию `+` у типа `b`. Это и есть **динамический полиморфизм**.

***

## 3. Пример: nb_add для int и list

```c
/* Objects/longobject.c – для int */
static PyNumberMethods long_as_number = {
    .nb_add = long_add,     /* сложение больших целых */
    .nb_subtract = long_sub,
    .nb_multiply = long_mul,
    /* ... */
};

/* Objects/listobject.c – для list */
static PySequenceMethods list_as_sequence = {
    .sq_length = list_length,
    .sq_concat = list_concat,   /* lst1 + lst2 */
    /* ... */
};

/* В PyTypeObject типов */
PyTypeObject PyLong_Type = {
    /* ... */
    .tp_as_number = &long_as_number,
    /* ... */
};

PyTypeObject PyList_Type = {
    /* ... */
    .tp_as_sequence = &list_as_sequence,
    /* list не заполняет tp_as_number->nb_add ! */
};
```

- `int` реализует арифметические операции через `long_as_number.nb_add = long_add`.
- `list` **не** заполняет `tp_as_number->nb_add`, а операцию `+` реализует как последовательность (`sq_concat`), которую
  использует байткод `BINARY_ADD`, если оба операнда – последовательности.
- Разные типы реализуют “сложение” **через разные таблицы слотов**.

оператор `+` может значить “арифметику” (для чисел), а может значить “конкатенацию” (для списков,
строк). На C-уровне это реализовано тем, что у разных типов разные таблицы функций: у int – одна реализация, у list –
другая. Но Python‑код везде одинаковый: `a + b`.

***

## 4. Полиморфизм последовательностей: PySequence_GetItem

```c
/* Include/cpython/object.h */
typedef struct {
    lenfunc sq_length;            /* len(o) */
    binaryfunc sq_concat;         /* o1 + o2 */
    ssizeargfunc sq_repeat;       /* o * n */
    ssizeargfunc sq_item;         /* o[i] */
    /* ... прочие методы ... */
} PySequenceMethods;

/* Objects/abstract.c – абстрактный доступ по индексу */
PyObject *PySequence_GetItem(PyObject *s, Py_ssize_t i)
{
    PySequenceMethods *m;

    if (s == NULL)
        return null_error();

    /* Берём таблицу seq-операций у типа объекта */
    m = Py_TYPE(s)->tp_as_sequence;
    if (m && m->sq_item) {
        return m->sq_item(s, i);
    }

    /* Fallback: пробуем obj.__getitem__(i) */
    return PyObject_GetItem(s, PyLong_FromSsize_t(i));
}
```

- Для “настоящих” последовательностей (`list`, `tuple`, `str` и т.п.) заполнен `tp_as_sequence->sq_item`, и
  индексирование идёт **напрямую** в C.
- Для “псевдо‑последовательностей”, которые просто реализуют `__getitem__`, но не заполнили слоты, идёт медленный путь
  через `PyObject_GetItem`.

операция `obj[i]` **сначала** пытается использовать быстрый C‑слот (`sq_item`). Если тип
“неофициальный” и слоты не заполнены, используется “медленный” путь через обычный метод `__getitem__`. Но интерфейс (
`obj[i]`) один и тот же – полиморфизм.

***

## 5. Атрибутный полиморфизм: PyObject_GenericGetAttr + дескрипторы

```c
/* Objects/object.c */
PyObject *PyObject_GenericGetAttr(PyObject *obj, PyObject *name)
{
    PyTypeObject *tp = Py_TYPE(obj);

    /* 1. Ищем атрибут в MRO класса (включая родителей) */
    PyObject *descr = _PyType_Lookup(tp, name);
    descrgetfunc f = NULL;
    if (descr != NULL) {
        f = Py_TYPE(descr)->tp_descr_get;  /* есть ли __get__? */
        if (f != NULL && PyDescr_IsData(descr)) {
            /* data-дескриптор имеет приоритет над __dict__ */
            return f(descr, obj, (PyObject *)tp);
        }
    }

    /* 2. Ищем в __dict__ экземпляра */
    PyObject **dictptr = _PyObject_GetDictPtr(obj);
    if (dictptr && *dictptr) {
        PyObject *res = PyDict_GetItemWithError(*dictptr, name);
        if (res != NULL) {
            Py_INCREF(res);
            return res;
        }
        if (PyErr_Occurred())
            return NULL;
    }

    /* 3. Если был не-data дескриптор – вызываем его __get__ */
    if (descr != NULL && f != NULL) {
        return f(descr, obj, (PyObject *)tp);
    }

    /* 4. Если это просто атрибут класса – вернуть его как есть */
    if (descr != NULL) {
        Py_INCREF(descr);
        return descr;
    }

    /* 5. Ничего не нашли – AttributeError */
    PyErr_Format(PyExc_AttributeError,
                 "'%.50s' object has no attribute '%.400s'",
                 tp->tp_name, ...);
    return NULL;
}
```

- `_PyType_Lookup` идёт по `tp_mro` типа (наследование!) и ищет атрибут.
- Если атрибут – дескриптор (имеет `tp_descr_get`), он может вернуть *совсем другой объект* (например, bound‑метод).
- Порядок: сначала data‑дескрипторы (property с setter), потом `__dict__` экземпляра, потом non‑data дескрипторы (
  обычные методы), потом просто атрибуты класса.

`obj.attr` – это **полиморфный вызов**:

- если `attr` – property → вызывается его `fget(self)`;
- если `attr` – функция в классе → создаётся bound‑метод;
- если `attr` – просто поле в экземпляре → читается из `__dict__`.

Логика одна, а поведение зависит от конкретного типа и того, что лежит в его `__dict__`.

***

## 6. Полиморфный вызов объектов: tp_call

```c
/* Objects/abstract.c */
PyObject *PyObject_Call(PyObject *callable,
                        PyObject *args, PyObject *kwargs)
{
    PyTypeObject *tp = Py_TYPE(callable);
    ternaryfunc call = tp->tp_call;  /* указатель на функцию вызова */

    if (call == NULL) {
        PyErr_Format(PyExc_TypeError,
            "'%.200s' object is not callable",
            tp->tp_name);
        return NULL;
    }
    return call(callable, args, kwargs);
}
```

- `tp_call` реализует `()` для объектов: функции, классы, экземпляры с `__call__`.
- Для `PyFunctionObject` (`def f(...)`) в `tp_call` стоит функция, запускающая байткод.
- Для типа (`type`) – `tp_call` создаёт новый экземпляр (см. `type_call` выше).
- Для пользовательского объекта с `__call__` – `tp_call` у типа `type(object)` вызывает его метод `__call__`.

`something()` работает **одинаково** для функций, классов, объектов с `__call__`. Но на C‑уровне это
просто вызов `tp_call` конкретного типа – опять таблица из указателей.

***

## 7. Байткод полиморфного вызова: CALL / PRECALL

```python
def f(x): return x


f(10)
```

```
# Байткод (3.11+):
  0 LOAD_GLOBAL              0 (f)      # берём объект-вызываемый
  2 LOAD_CONST               1 (10)
  4 PRECALL                  1         # подготовка стека/fastcall
  6 CALL                     1         # общий вызов
  8 POP_TOP
 10 LOAD_CONST               0 (None)
 12 RETURN_VALUE
```

В интерпретаторе (`Python/ceval.c`):

```c
case CALL: {
    PyObject *callable = PEEK(oparg);      /* вызываемый объект */
    PyObject **args = stack_pointer - oparg; /* указатель на аргументы */

    /* Полиморфный fast-path: vectorcall, tp_call и т.п. */
    PyObject *res = _PyObject_Vectorcall(callable, args, oparg, NULL);
    if (res == NULL)
        goto error;

    /* снимаем callable и аргументы, кладём результат */
    STACKADJ(-(oparg+1));
    PUSH(res);
    DISPATCH();
}
```

байткод `CALL` вообще **не знает**, кого он вызывает: обычную функцию, метод, класс, объект‑функтор.
Он берёт объект и аргументы и передаёт их в `_PyObject_Vectorcall` / `PyObject_Call`, которые, глядя на тип, выбирают
правильный `tp_call` / fastcall‑функцию.

***

Итого, полиморфизм в CPython 3.9+ реализован через:

- **слоты в PyTypeObject** (`tp_as_number`, `tp_as_sequence`, `tp_call` и т.д.),
- **абстрактные функции** (`PyNumber_*`, `PySequence_*`, `PyMapping_*`), которые выбирают C‑реализацию по
  `Py_TYPE(obj)`,
- **общий механизм атрибутов** (`PyObject_GenericGetAttr` + дескрипторы),
- **полиморфный вызов** через `tp_call` и байткод `CALL`.

Всё это даёт привычный на уровне языка эффект: “одна и та же операция выполняется по‑разному в зависимости от типа
объекта”.

- [Содержание](/CONTENTS.md#содержание)