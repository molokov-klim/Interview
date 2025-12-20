# **Паттерны проектирования**

## **Junior Level*

Паттерны проектирования — это проверенные временем решения типовых проблем в разработке программного обеспечения. Можно
сравнить их с архитектурными чертежами для строительства: вместо того чтобы каждый раз изобретать велосипед, опытные
архитекторы используют готовые схемы организации кода, которые уже доказали свою эффективность для конкретных ситуаций.

Это не готовые куски кода, а скорее концепции или шаблоны мышления о том, как структурировать взаимодействие между
компонентами системы. Например, "Фабрика" — это паттерн для создания объектов, не привязываясь к их конкретным классам,
а "Наблюдатель" описывает, как один объект может уведомлять множество других о произошедших изменениях.

Для AQA инженера понимание паттернов критически важно по нескольким причинам: во-первых, чтобы понимать архитектуру
тестируемого приложения и находить в ней слабые места; во-вторых, чтобы проектировать собственные тестовые фреймворки,
которые будут гибкими, расширяемыми и поддерживаемыми; в-третьих, чтобы говорить с разработчиками на одном языке при
обсуждении дизайна и потенциальных проблем.

[Содержание](/CONTENTS.md#содержание)

## **Middle Level**

С технической точки зрения, паттерны проектирования (особенно из классического каталога GoF — "Банды четырех") в Python
часто реализуются с учетом уникальных особенностей языка.

1. **Идиоматичность Python:** Многие классические паттерны в Python могут быть реализованы проще и элегантнее, чем в
   статически типизированных языках. Например:
    * **Стратегия (Strategy):** Вместо создания иерархии классов можно просто передавать callable-объекты (функции,
      лямбды, экземпляры с `__call__`).
    * **Декоратор (Decorator):** Прямо соответствует синтаксическому декоратору Python, который является "обёрткой"
      вокруг функции или метода, модифицирующей его поведение.
    * **Адаптер (Adapter):** Часто реализуется не через наследование, а через композицию и магический метод
      `__getattr__` для перенаправления вызовов.

2. **Категории паттернов:**
    * **Порождающие (Creational):** Решают задачу гибкого и контролируемого создания объектов (Синглтон, Фабрика,
      Строитель). В Python Синглтон часто реализуют через метакласс или модуль (сам модуль по своей природе — синглтон).
    * **Структурные (Structural):** Организуют композицию классов и объектов для образования более крупных структур (
      Адаптер, Мост, Компоновщик, Декоратор, Фасад).
    * **Поведенческие (Behavioral):** Определяют эффективные способы взаимодействия и распределения ответственности
      между объектами (Наблюдатель, Стратегия, Команда, Цепочка обязанностей, Состояние).

3. **Для AQA:**
    * **Page Object** — это специализированный структурный паттерн для UI-тестирования, по сути являющийся Фасадом,
      скрывающим детали HTML-структуры за удобным API.
    * **Фабрика** и **Строитель (Builder)** незаменимы для создания сложных тестовых данных и фикстур.
    * **Наблюдатель (Observer)** или **Издатель-Подписчик (Pub/Sub)** лежит в основе систем событийного логирования или
      сбора результатов тестов в реальном времени.
    * **Прокси (Proxy)** и **Заместитель (Mock/Stub)** — основа библиотек для мокирования (unittest.mock), позволяющих
      подменять реальные зависимости в тестах.

4. **Dependency Injection (Внедрение зависимостей):** Хотя формально не входит в каталог GoF, это ключевой архитектурный
   паттерн, который в Python часто реализуется просто через передачу зависимостей в конструктор (Constructor Injection),
   без использования тяжеловесных фреймворков. Это краеугольный камень тестируемого дизайна.

[Содержание](/CONTENTS.md#содержание)

## **Senior Level**

В CPython 3.9+ **паттерны проектирования** встроены в ядро: **Singleton** (`functools.lru_cache(maxsize=None)`), *
*Factory** (`type.__call__`/`tp_new`), **Proxy** (`weakref.proxy`), **Decorator** (`tp_call` chaining), **Visitor** (MRO
`_PyType_Lookup`), **Flyweight** (`PyType_FromSpec`/
`tp_cache`). `Objects/typeobject.c`,`Objects/weakrefobject.c`,`Lib/functools.py`

## 1. Singleton: functools.lru_cache(maxsize=None)

```c
// Lib/_functools.py → Objects/call.c (3.12+ specialization)
typedef struct {
    PyObject_HEAD
    PyObject *func;                // декорируемая функция
    Py_ssize_t maxsize;            // maxsize=None → ∞ (singleton!)
    PyObject *cache;               // dict: args → result
    PyObject *weakreflist;         // слабые ссылки
} lru_cacheobject;

// __call__ всегда возвращает ТОТ ЖЕ объект из кэша!
static PyObject *
lru_cache_call(lru_cacheobject *self, PyObject *args, PyObject *kwargs)
{
    PyObject *key = make_key(args, kwargs);    // tuple(args, kwargs)
    PyObject *result = PyDict_GetItem(self->cache, key);
    
    if (result != NULL) {
        Py_INCREF(result);                     // ← SINGLETON!
        return result;                         // тот же объект!
    }
    
    // MISS: вызываем func, кэшируем
    result = PyObject_Call(self->func, args, kwargs);
    if (result != NULL) {
        PyDict_SetItem(self->cache, key, result);  // сохраняем ссылку
    }
    return result;
}
```

`@lru_cache(maxsize=None)` — **настоящий Singleton**! При повторном вызове с теми же аргументами возвращается **ТА САМАЯ
** ссылка из `cache` dict (не копия!). **Бессмертный кэш** до GC.

## 2. Factory Method: type.__call__ → tp_new/tp_init

```c
// Objects/typeobject.c — C(1,2) = Factory!
static PyObject *
type_call(PyTypeObject *type, PyObject *args, PyObject *kwds)
{
    // 1. FACTORY: tp_new создаёт "пустой" объект
    PyObject *obj = type->tp_new(type, args, kwds);
    if (obj == NULL)
        return NULL;
    
    // 2. Инициализация (если есть __init__)
    if (type->tp_init != NULL) {
        int res = type->tp_init(obj, args, kwds);
        if (res < 0) {
            Py_DECREF(obj);
            return NULL;
        }
    }
    
    return obj;  // готовый экземпляр!
}
```

`C(1,2)` — **Factory Method**! `type_call(C)` **НЕ** всегда вызывает `C.__new__ + C.__init__`. Может использовать *
*любые** `tp_new`/`tp_init` (namedtuple, dataclass, enum). **Полный контроль** создания объектов!

## 3. Proxy: weakref.proxy (Objects/weakrefobject.c)

```c
// Objects/weakrefobject.c — Proxy к объекту
typedef struct {
    PyObject_HEAD
    PyWeakReference *wr;           // слабая ссылка на target
    PyObject *callback;            // callback при удалении target
} PyProxyObject;

static PyObject *
proxy_getattr(PyProxyObject *proxy, PyObject *name)
{
    PyObject *target = proxy->wr->wr_object;  // реальный объект
    if (target == NULL) {                      // target умер!
        PyErr_SetString(PyExc_ReferenceError, "weakly-referenced object no longer exists");
        return NULL;
    }
    return PyObject_GetAttr(target, name);     // делегируем!
}

static PyObject *
proxy_call(PyProxyObject *proxy, PyObject *args, PyObject *kwds)
{
    PyObject *target = proxy->wr->wr_object;
    if (target == NULL)
        return weakref_error();
    return PyObject_Call(target, args, kwds);  // proxy() → target()
}
```

`proxy = weakref.proxy(obj)` — **прозрачный прокси**! `proxy.attr` → `obj.attr`, `proxy()` → `obj()`. Если `obj`
удалён — `ReferenceError`. **НЕ держит** сильную ссылку!

## 4. Decorator: tp_call chaining (функции + __call__)

```c
// Objects/typeobject.c — obj() где obj имеет __call__
static PyObject *
PyObject_Call(PyObject *callable, PyObject *args, PyObject *kwds)
{
    PyTypeObject *tp = Py_TYPE(callable);
    
    // 1. Специализированные fastcall
    if (PyFunction_Check(callable))
        return _PyFunction_Vectorcall(callable, args, nargs, kwds);
    
    // 2. Обычный tp_call
    ternaryfunc call = tp->tp_call;
    if (call != NULL)
        return call(callable, args, kwds);
    
    // 3. __call__ метод
    PyObject *meth = _PyObject_LookupAttr(callable, &_Py_ID(__call__));
    if (meth != NULL) {
        PyObject *res = PyObject_Call(meth, args, kwds);
        Py_DECREF(meth);
        return res;
    }
    
    PyErr_Format(PyExc_TypeError, "'%.200s' object is not callable", tp->tp_name);
    return NULL;
}
```

`obj()` — **Decorator pattern**! `tp_call` → если есть `__call__` метод → вызывает его. `@decorator` функции работают *
*аналогично** через `tp_call`.

## 5. Visitor: MRO _PyType_Lookup (Objects/typeobject.c)

```c
// Objects/typeobject.c — поиск attr по MRO = Visitor!
PyObject *
_PyType_Lookup(PyTypeObject *type, PyObject *name)
{
    PyObject *mro = type->tp_mro;  // (D, B, C, A, object)
    Py_ssize_t i, n = PyTuple_GET_SIZE(mro);
    
    // VISITOR: посещаем каждый класс в MRO
    for (i = 0; i < n; i++) {
        PyTypeObject *base = PyTuple_GET_ITEM(mro, i);
        PyObject *res = PyDict_GetItem(base->tp_dict, name);
        if (res != NULL) {
            Py_INCREF(res);
            return res;  // первый "приёмник" visitor'а!
        }
    }
    return NULL;
}
```

`obj.method` — **Visitor pattern**! `_PyType_Lookup` **посещает** каждый класс в `tp_mro`, пока **один** не вернёт
метод. **Остановка** на первом совпадении!

## 6. Flyweight: PyType_FromSpec (повторное использование типов)

```c
// Objects/typeobject.c — создание типа один раз!
PyTypeObject *
PyType_FromSpecWithBases(PyType_Spec *spec, PyObject *bases)
{
    // 1. Ищем в глобальном кэше типов
    PyTypeObject *type = find_type_in_cache(spec->name);
    if (type != NULL)
        return type;  // ← FLYWEIGHT!
    
    // 2. Создаём новый PyHeapTypeObject
    PyHeapTypeObject *et = PyObject_MALLOC(sizeof(PyHeapTypeObject));
    type = &et->ht_type;
    
    // 3. Заполняем spec->basicsize, spec->itemsize, spec->slots
    type->tp_name = spec->name;
    type->tp_basicsize = spec->basicsize;
    
    // 4. Кэшируем глобально
    cache_type_in_global(spec->name, type);
    
    PyType_Ready(type);
    return type;
}
```

```c
// Пример C extension:
static PyType_Spec MyType_spec = {
    .name = "mymodule.MyType",
    .basicsize = sizeof(MyTypeObject),
    .itemsize = 0,
};

PyObject *my_module = PyModule_Create(&moduledef);
PyTypeObject *MyType = PyType_FromSpecWithBases(&MyType_spec, NULL);
Py_INCREF(MyType);
PyModule_AddObject(my_module, "MyType", (PyObject *)MyType);
```

**Flyweight**! Тип `MyType` создаётся **ОДИН РАЗ** в `PyType_FromSpec`, кэшируется глобально. Все `MyType()` используют
**тот же** `PyTypeObject`. **Экономия памяти**!

## 7. Prototype: copy.deepcopy (Objects/copy.c)

```c
// Objects/copy.c — глубокое копирование
PyObject *
PyObject_DeepCopy(PyObject *obj)
{
    PyTypeObject *tp = Py_TYPE(obj);
    
    // 1. __copy__ / __deepcopy__ методы
    PyObject *copier = _PyObject_LookupAttr(obj, &_Py_ID(__deepcopy__));
    if (copier != NULL) {
        PyObject *newobj = PyObject_CallOneArg(copier, obj);
        Py_DECREF(copier);
        return newobj;
    }
    
    // 2. tp_deepcopy слот типа
    if (tp->tp_traverse != NULL && tp->tp_clear != NULL) {
        binaryfunc deepcopy = tp->tp_deepcopy;
        if (deepcopy != NULL)
            return deepcopy(obj, memo_dict);
    }
    
    // 3. Рекурсивное копирование полей
    return recursive_deepcopy(obj, memo_dict);
}
```

`copy.deepcopy(obj)` — **Prototype**! Создаёт **клон** с `__deepcopy__` или через `tp_deepcopy`. `memo_dict`
предотвращает циклические копии!

## 8. Adapter: протоколы (tp_as_number, tp_as_sequence)

```c
// Include/cpython/object.h — адаптер для разных интерфейсов
struct PyNumberMethods {
    binaryfunc nb_add;             // + (адаптировано для int/float/complex)
    /* ... */
};

struct PySequenceMethods {
    lenfunc sq_length;             // len() (адаптировано для list/tuple/str)
    /* ... */
};

// PyLong_Type, PyList_Type заполняют РАЗНЫЕ адаптеры!
PyLong_Type.tp_as_number = &long_as_number;
PyList_Type.tp_as_sequence = &list_as_sequence;
```

**Adapter**! `PyNumber_Add(a, b)` работает с `int`, `float`, `Decimal` через **общий интерфейс** `tp_as_number`. `len()`
работает с `list`, `tuple`, `str` через `tp_as_sequence`.

## 9. Observer: tp_subclasses + __subclasses__()

```c
// Objects/typeobject.c — родитель знает о детях!
static PyObject *
type_get_subclasses(PyTypeObject *type, void *context)
{
    // tp_subclasses — list слабых ссылок на подклассы
    PyObject *list = _PyWeakref_GetRefList(type->tp_subclasses);
    if (list == NULL)
        Py_RETURN_NONE;
    return list;  // [SubClass1, SubClass2, ...]
}
```

**Observer**! При создании `class Sub(Base):` → `weakref(Sub)` **автоматически** добавляется в `Base.tp_subclasses`.
`Base.__subclasses__()` возвращает **список живых** подклассов!

## 10. Command: bound methods (PyMethodObject)

```c
// Objects/classobject.c — bound method = команда!
typedef struct {
    PyObject_HEAD
    PyObject *im_func;             // def method(self):
    PyObject *im_self;             // self (контекст)
    PyObject *im_class;            // класс
} PyMethodObject;

static PyObject *
method_call(PyMethodObject *meth, PyObject *args, PyObject *kwds)
{
    // Команда: meth.im_func(meth.im_self, *args)
    PyObject *self = meth->im_self;
    Py_ssize_t argc = PyTuple_GET_SIZE(args);
    PyObject *newargs = PyTuple_New(argc + 1);
    PyTuple_SET_ITEM(newargs, 0, Py_NewRef(self));
    for (Py_ssize_t i = 0; i < argc; i++)
        PyTuple_SET_ITEM(newargs, i+1, Py_NewRef(PyTuple_GET_ITEM(args, i)));
    
    PyObject *result = PyObject_Call(meth->im_func, newargs, kwds);
    Py_DECREF(newargs);
    return result;
}
```

`obj.method` — **Command**! `PyMethodObject` инкапсулирует: **функцию** (`im_func`), **контекст** (`im_self`), *
*параметры**. Вызов → `im_func(im_self, *args)`!

## Итог паттернов в CPython 3.9+

| Паттерн       | Реализация                 | Файл                      |
|---------------|----------------------------|---------------------------|
| **Singleton** | `lru_cache(maxsize=None)`  | `Objects/call.c`          |
| **Factory**   | `type.__call__` → `tp_new` | `Objects/typeobject.c`    |
| **Proxy**     | `weakref.proxy`            | `Objects/weakrefobject.c` |
| **Decorator** | `tp_call` + `__call__`     | `Objects/typeobject.c`    |
| **Visitor**   | MRO `_PyType_Lookup`       | `Objects/typeobject.c`    |
| **Flyweight** | `PyType_FromSpec` cache    | `Objects/typeobject.c`    |
| **Prototype** | `copy.deepcopy`            | `Objects/copy.c`          |
| **Adapter**   | `tp_as_*` протоколы        | `Include/objimpl.h`       |
| **Observer**  | `tp_subclasses`            | `Objects/typeobject.c`    |
| **Command**   | `PyMethodObject`           | `Objects/classobject.c`   |

**CPython** — **библиотека паттернов** на C-уровне!

- [Содержание](/CONTENTS.md#содержание)