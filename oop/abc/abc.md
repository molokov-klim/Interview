# **ABC**

## **Junior Level*

Представьте, что вы архитектор, который разрабатывает проект дома (интерфейс, контракт). Вы создаете подробный чертеж,
где указано: "здесь должна быть дверь", "здесь должно быть не менее двух окон", "обязательно должна быть крыша". Однако
сам чертеж — это не дом, по нему нельзя жить. Вы просто задаете стандарт, которому должен следовать любой построенный по
этому проекту дом.

В Python ABC (Abstract Base Classes) — это и есть такие "чертежи" для классов. Это специальный механизм, который
позволяет нам декларативно сказать: "Любой класс, который будет моим наследником, ОБЯЗАН реализовать конкретные методы (
например, `save()`, `load()`, `validate()`)". Если наследник не реализует все обязательные методы, Python не даст
создать его экземпляр и сразу укажет на ошибку. Это делает код предсказуемым, помогает избежать ошибок в рантайме и
четко документирует ожидания от объекта. Для QA инженера это мощный инструмент для стандартизации тестовых утилит,
плагинов и проверки контрактов в системе.

[Содержание](/CONTENTS.md#содержание)

## **Middle Level**

На техническом уровне ABC — это классы, наследующиеся от встроенного класса `abc.ABC` и использующие декоратор
`@abstractmethod` (а также `@abstractclassmethod`, `@abstractstaticmethod`, `@abstractproperty`) для пометки абстрактных
методов. Ключевые аспекты:

1. **Инстанцирование:** Попытка создать экземпляр класса, у которого есть хотя бы один не переопределенный
   `@abstractmethod`, немедленно вызовет исключение `TypeError`. Это проверка происходит в момент вызова `__new__`
   класса, еще до вызова `__init__`.

2. **Регистрация (Registration):** Помимо явного наследования, класс может быть "зарегистрирован" как виртуальный
   подкласс ABC с помощью метода `register()`. После этого `isinstance()` и `issubclass()` будут возвращать для него
   `True`, но при этом **не проводится проверка на наличие абстрактных методов!** Это "мягкий" способ заявить о
   соответствии интерфейсу, полезный для интеграции сторонних или унаследованных классов.

3. **`__subclasshook__`:** Это специальный метод класса (`@classmethod`), который позволяет кастомизировать логику
   проверки `issubclass()`. Метод может проверять наличие у класса требуемых атрибутов или методов (через `hasattr`), а
   не только факт прямого наследования или регистрации. Это самый гибкий и "питонический" способ определения виртуальных
   подклассов.

4. **Встроенные ABCs в `collections.abc`:** Модуль предоставляет богатейший набор готовых абстрактных классов для
   стандартных протоколов: `Iterable`, `Iterator`, `Container`, `Sequence`, `Mapping`, `Callable` и т.д. Использование
   `isinstance(obj, collections.abc.Sequence)` вместо `isinstance(obj, list)` делает код полиморфным и независимым от
   конкретной реализации.

[Содержание](/CONTENTS.md#содержание)

## **Senior Level**

В CPython 3.9+ **ABC** (Abstract Base Classes) реализованы через **C-модуль `_abc`** (`Modules/_abc.c`), **регистрацию**
в `_abc_registry` (weakref dict), **кэш** `_abc_cache`/`_abc_negative_cache`, перехват `isinstance()`/`issubclass()`
через `__instancecheck__`/`__subclasscheck__` на `ABCMeta`. Абстрактные методы — флаг `__abstractmethods__` в
`tp_dict`. `Modules/_abc.c`,`Objects/abstract.c`

## 1. Структура _abc_data в C-модуле

```c
// Modules/_abc.c — глобальное состояние ABC на модуль
typedef struct {
    PyObject *_abc_registry;       // dict: ABC → set(слабые ссылки на подклассы)
    PyObject *_abc_cache;          // dict: (subclass, ABC) → True/False
    PyObject *_abc_negative_cache; // dict: (subclass, ABC) → False (не подкласс)
    uint64_t _abc_negative_cache_version; // версия кэша (меняется при register)
} _abc_data;
```

Каждый ABC-модуль имеет **3 словаря**:

- `_abc_registry` — кто кого зарегистрировал (`MyABC → {tuple, str}`).
- `_abc_cache` — положительный кэш проверок.
- `_abc_negative_cache` — отрицательный кэш.  
  **Кэш ускоряет** `isinstance()` в 1000x раз!

## 2. Регистрация подкласса: _abc_register_impl()

```c
// Modules/_abc.c — MyABC.register(tuple)
static PyObject *
_abc__abc_register_impl(PyObject *module, PyObject *self, PyObject *subclass)
{
    _abc_data *impl = _get_impl(module, self);  // получаем данные ABC
    if (impl == NULL)
        return NULL;
    
    PyObject *registry;  // set слабых ссылок
    Py_BEGIN_CRITICAL_SECTION(impl);  // атомарно
    registry = impl->_abc_registry;
    Py_END_CRITICAL_SECTION();
    
    if (registry == NULL) {
        registry = PySet_New(NULL);  // новый set
        Py_BEGIN_CRITICAL_SECTION(impl);
        Py_XSETREF(impl->_abc_registry, registry);
        Py_END_CRITICAL_SECTION();
    }
    
    // добавляем weakref(subclass) в registry
    PyObject *ref = PyWeakref_NewRef(subclass, NULL);
    PySet_Add(registry, ref);
    Py_DECREF(ref);
    
    // инвалидируем кэш
    impl->_abc_negative_cache_version++;
    
    Py_RETURN_NONE;
}
```

`MyABC.register(tuple)` → `weakref(tuple)` **добавляется** в `MyABC._abc_registry`. **НЕ** меняет `tuple.__bases__`!
`issubclass(tuple, MyABC)` смотрит **только** в registry. **Кэш сбрасывается** (`_abc_negative_cache_version++`).

## 3. Проверка issubclass: _abc_abc_subclasscheck()

```c
// Modules/_abc.c — issubclass(MyClass, MyABC)
static int
_abc_abc_subclasscheck_impl(PyObject *self, PyObject *subclass)
{
    _abc_data *impl = _get_impl(NULL, self);
    if (impl == NULL)
        return -1;
    
    // 1. Проверяем кэш
    int cached = check_cache(impl, subclass, self);
    if (cached != -1)
        return cached;
    
    // 2. Обычное наследование (MRO)
    if (PyObject_IsSubclass(subclass, (PyObject *)Py_TYPE(self)))
        return 1;
    
    // 3. Регистрация в _abc_registry
    PyObject *registry = impl->_abc_registry;
    if (registry != NULL && PySet_Contains(registry, subclass))
        return 1;
    
    // 4. __subclasshook__
    PyObject *hook = lookup_method(self, &_Py_ID(__subclasshook__));
    if (hook != NULL) {
        PyObject *res = PyObject_CallFunctionObjArgs(hook, subclass, NULL);
        if (res != NULL) {
            int result = PyObject_IsTrue(res);
            Py_DECREF(res);
            return result;
        }
    }
    
    // 5. Кэшируем False
    cache_negative_result(impl, subclass, self);
    return 0;
}
```

`issubclass(C, ABC)` проверяет **4 пути**:

1. **Кэш** (быстро).
2. **Обычное наследование** `C.__mro__` содержит `ABC`.
3. **Регистрация** `ABC._abc_registry` содержит `weakref(C)`.
4. **`ABC.__subclasshook__(C)`** (custom логика).  
   **НЕ реализует** — возвращает `False`.

## 4. Перехват isinstance/issubclass в PyObject_IsInstance()

```c
// Objects/abstract.c — isinstance(obj, ABC)
int PyObject_IsInstance(PyObject *inst, PyObject *cls)
{
    PyTypeObject *tp = Py_TYPE(inst);
    
    // 1. Обычная проверка по MRO
    if (PyTuple_Contains(tp->tp_mro, cls))
        return 1;
    
    // 2. ABC перехват через __instancecheck__
    PyObject *inst_cls = (PyObject *)Py_TYPE(cls);
    if (Py_TYPE(inst_cls) == &ABCMeta_Type) {  // это ABC!
        PyObject *meth = _PyType_Lookup(Py_TYPE(inst_cls), &_Py_ID(__instancecheck__));
        if (meth != NULL) {
            PyObject *res = PyObject_CallFunctionObjArgs(meth, cls, inst, NULL);
            int result = PyObject_IsTrue(res);
            Py_XDECREF(res);
            return result;
        }
    }
    
    return 0;
}
```

`isinstance(obj, ABC)` **СНАЧАЛА** проверяет `type(obj).__mro__`, **ПОТОМ** вызывает `ABC.__instancecheck__(obj)`. *
*ABCMeta** перехватывает и делегирует в `_abc_abc_instancecheck()`.

## 5. __abstractmethods__ и проверка абстрактности

```c
// Lib/abc.py (компиляция → tp_dict)
class MyABC(ABC):
    @abstractmethod
    def method(self):
        pass
```

В `PyType_Ready()` (Objects/typeobject.c):

```c
static int
PyType_Ready(PyTypeObject *type)
{
    // ... MRO ...
    
    // проверяем __abstractmethods__ из tp_dict
    PyObject *abstracts = PyDict_GetItem(type->tp_dict, &_Py_ID(__abstractmethods__));
    if (abstracts != NULL && PySet_GET_SIZE(abstracts) > 0) {
        // есть нереализованные абстрактные методы!
        type->tp_flags |= Py_TPFLAGS_IS_ABSTRACT;
    }
}
```

`@abstractmethod` добавляет имя метода в `MyABC.__abstractmethods__ = {'method'}`. При создании `MyABC` если set **НЕ
пустой** — класс помечается `Py_TPFLAGS_IS_ABSTRACT`. **НЕ** блокирует создание экземпляра!

## 6. Блокировка инстанцирования абстрактного класса

```c
// Objects/typeobject.c — type_call() для абстрактного класса
static PyObject *
type_call(PyTypeObject *type, PyObject *args, PyObject *kwds)
{
    // ПРОВЕРКА АБСТРАКТНОСТИ!
    if (type->tp_flags & Py_TPFLAGS_IS_ABSTRACT) {
        PyObject *abstracts = PyDict_GetItem(type->tp_dict, &_Py_ID(__abstractmethods__));
        if (abstracts != NULL && PySet_GET_SIZE(abstracts) > 0) {
            // выводим НЕ реализованные методы
            PyErr_Format(PyExc_TypeError,
                "Can't instantiate abstract class %s with abstract methods %R",
                type->tp_name, abstracts);
            return NULL;
        }
    }
    
    // обычное создание: tp_new → tp_init
    return type->tp_new(type, args, kwds);
}
```

`MyABC()` → `type_call(MyABC)` → видит `Py_TPFLAGS_IS_ABSTRACT` + `__abstractmethods__ != set()` → **TypeError** со
списком нереализованных методов!

## 7. __subclasshook__ пример (collections.abc.Container)

```python
# Lib/_collections_abc.py
class Container(ABC):
    @classmethod
    def __subclasshook__(cls, C):
        # любой класс с __contains__ — Container!
        if cls is Container:
            return hasattr(C, '__contains__')
        return NotImplemented
```

В C (`_abc_abc_subclasshook__`):

```c
PyObject *hook_res = PyObject_CallFunctionObjArgs(hook, subclass, NULL);
int result = PyObject_IsTrue(hook_res);  // True/False/NotImplemented
```

`Container.__subclasshook__(MyClass)` → `hasattr(MyClass, '__contains__')` → `True`. `MyClass` **НЕ наследует**
`Container`, **НО** `issubclass(MyClass, Container) == True`!

## 8. Байткод регистрации ABC

```python
from abc import ABC


class MyABC(ABC): pass


MyABC.register(tuple)
print(issubclass(tuple, MyABC))  # True
```

```
# Байткод MyABC.register(tuple):
  0 LOAD_GLOBAL         0 (MyABC)
  2 LOAD_GLOBAL         1 (tuple)
  4 CALL_METHOD         1          # register(tuple)
  6 POP_TOP
  8 LOAD_GLOBAL         0 (issubclass)
 10 LOAD_GLOBAL         1 (tuple)
 12 LOAD_GLOBAL         0 (MyABC)
 14 CALL_FUNCTION       2
```

`MyABC.register(tuple)` → `CALL_METHOD` → C `_abc__abc_register_impl()` → `weakref(tuple)` в `_abc_registry`.

## Итог реализации ABC в CPython 3.9+

**ABC** — **виртуальное наследование** через:

1. **C-модуль `_abc`** с `_abc_data` (registry + 2 кэша).
2. **`register()`** → weakref в `_abc_registry`.
3. **`issubclass(C, ABC)`** → кэш → MRO → registry → `__subclasshook__`.
4. **`isinstance(obj, ABC)`** → `ABC.__instancecheck__(obj)`.
5. **`@abstractmethod`** → `__abstractmethods__` set → `Py_TPFLAGS_IS_ABSTRACT`.
6. **Инстанцирование** → проверка флага + set → `TypeError`.

**НЕ меняет MRO**, **НЕ добавляет методы**, **только** перехватывает `isinstance`/`issubclass`!

- [Содержание](/CONTENTS.md#содержание)