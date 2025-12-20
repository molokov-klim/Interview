# **Инкапсуляция**

[Содержание](/CONTENTS.md#содержание)

## **Junior Level**

Инкапсуляция — это принцип объектно-ориентированного программирования, который объединяет данные и методы, работающие с
этими данными, внутри одного объекта и скрывает внутренние детали реализации от внешнего мира.

Представьте, что у вас есть банковский счет. Вы можете пополнять его, снимать деньги и проверять баланс, но вам не нужно
знать, как именно банк хранит ваши данные, как они обрабатывают транзакции или как обновляют баланс. Вам предоставляют
простой интерфейс (например, банкомат или мобильное приложение), а все сложности скрыты внутри.

В Python инкапсуляция реализуется через модификаторы доступа: публичные (public), защищенные (protected) и приватные (
private) атрибуты и методы. Это помогает защитить данные от неправильного использования и обеспечивает контроль над тем,
как объект взаимодействует с внешним миром.

[Содержание](/CONTENTS.md#содержание)

## **Middle Level**

В Python инкапсуляция имеет свои особенности из-за динамической природы языка:

1. **Уровни доступа**:
    - **Публичные (public)**: Обычные атрибуты и методы, доступные отовсюду.
    - **Защищенные (protected)**: Имена с одним подчёркиванием (`_attr`). Это соглашение, а не строгая защита. Говорит
      разработчикам: "это для внутреннего использования".
    - **Приватные (private)**: Имена с двумя подчёркиваниями (`__attr`). Python применяет **name mangling** (искажение
      имён), превращая `__attr` в `_ClassName__attr`. Это затрудняет случайный доступ, но не делает атрибут полностью
      недоступным.

2. **Свойства (property)**:
    - Декораторы `@property`, `@attr.setter`, `@attr.deleter`.
    - Позволяют контролировать доступ к атрибутам, добавляя логику при чтении, записи или удалении.
    - Сохраняют синтаксис доступа как к обычному атрибуту.

3. **Дескрипторы**:
    - Классы, реализующие протокол `__get__`, `__set__`, `__delete__`.
    - Лежат в основе свойств, методов класса, статических методов.
    - Позволяют создавать переиспользуемые механизмы управления доступом.

4. **Методы доступа (getters/setters)**:
    - В Python не принято создавать простые getters/setters для всех атрибутов.
    - Используются только при необходимости добавить логику (валидацию, вычисления).
    - Иначе нарушается принцип "We're all consenting adults here".

5. **`__slots__`**:
    - Ограничивает набор атрибутов экземпляра.
    - Экономит память, предотвращая создание `__dict__`.
    - Ограничивает динамическое добавление атрибутов.

6. **Интерфейсы и абстракции**:
    - Абстрактные классы (`abc.ABC`) определяют контракты без раскрытия реализации.
    - Протоколы (`typing.Protocol`) для структурной типизации.

[Содержание](/CONTENTS.md#содержание)

## **Senior Level**

В CPython 3.9+ **инкапсуляция** — **name mangling** (`_Class__private` в compile.c), **`__slots__`** (
`tp_dictoffset=0`), **descriptor протокол** (`property`/`@property`), **`__getattribute__`** защита, **`__dict__`** по
`tp_dictoffset`. **НЕ** есть `private/protected`. `Python/compile.c`,`Objects/object.c`,`Objects/descrobject.c`

## 1. Name mangling в компиляторе (Python/compile.c)

```c
static const char * _Py_Mangle(const char *privateobj, const char *privatename) {
    /* Name mangling: __private -> _Class__private */
    const char *p;
    size_t len_privateobj = strlen(privateobj);
    size_t len_privatename = strlen(privatename);
    size_t len = len_privateobj + len_privatename + 1;
    char *mangled = PyMem_Malloc(len + 1);
    
    if (mangled == NULL)
        return NULL;
    
    /* Skip leading underscores (_Class -> Class) */
    p = privateobj;
    while (*p == '_')
        p++;
    
    /* privateobj + privatename */
    strcpy(mangled, "_");
    strcat(mangled, p);           // _Class
    strcat(mangled, privatename); // _Class__private
    
    return mangled;
}

static identifier compiler_nameop(struct compiler *c, location loc, identifier name, expr_context_ty ctx) {
    if (ctx == Load && c->u->u_scope_type == COMPILER_SCOPE_CLASS &&
        PyUnicode_GET_LENGTH(name) >= 2 &&
        PyUnicode_READ_CHAR(name, 0) == '_' &&
        PyUnicode_READ_CHAR(name, 1) == '_') {
        
        /* __private -> _Class__private */
        identifier class_name = c->u->u_private;  // 'MyClass'
        mangled = _Py_Mangle(PyUnicode_AsUTF8(class_name), PyUnicode_AsUTF8(name));
        if (mangled == NULL)
            return NULL;
        
        name = PyUnicode_FromString(mangled);
        PyMem_Free(mangled);
    }
    return name;
}
```

`class MyClass: def __private(self): pass` → компилятор **заменяет** `__private` на `_MyClass__private` **в байткоде**.
`LOAD_NAME '__private'` → `LOAD_NAME '_MyClass__private'`. **Подкласс** `class Sub(MyClass):` ищет `__private` →
`_Sub__private` (НЕ находит родительский). **Не private** — просто **избегает конфликтов** имён!

## 2. Байткод с mangling

```python
class MyClass:
    def __private(self):
        pass


class Sub(MyClass):
    def f(self):
        self.__private()  # _Sub__private()
```

```
# Байткод Sub.f():
  0 LOAD_FAST           0 (self)
  2 LOAD_ATTR           0 (_Sub__private)  # <- mangled!
  4 CALL_FUNCTION       1
  6 POP_TOP
  8 LOAD_CONST          0 (None)
 10 RETURN_VALUE
```

В байткоде **НЕ** `self.__private()`, а `self._Sub__private()`. Компилятор **заменил** имя **статически**.
`MyClass().__private()` → **AttributeError** (ищет `_MyClass__private`).

## 3. __slots__ - без __dict__ (Objects/typeobject.c)

```c
int PyType_Ready(PyTypeObject *type) {
    // ...
    
    /* __slots__ = () -> tp_dictoffset=0 */
    if (type->tp_dictoffset == 0) {
        /* Нет __dict__ - экономим память */
        type->tp_flags |= Py_TPFLAGS_HAVE_SLOTS;
    }
    
    /* Вычисляем реальный размер экземпляра */
    if (type->tp_itemsize != 0) {
        /* Переменная длина (str, tuple) */
        type->tp_basicsize = -type->tp_basicsize;
    }
    
    type->tp_flags |= Py_TPFLAGS_READY;
    return 0;
}

PyObject *type_new(PyTypeObject *type, PyObject *args, PyObject *kwargs) {
    Py_ssize_t nslots = type->tp_dictoffset / sizeof(PyObject *);
    
    /* Выделяем память */
    PyObject *inst = PyObject_MALLOC(type->tp_basicsize + 
                                   nslots * sizeof(PyObject *));
    
    if (inst == NULL)
        return PyErr_NoMemory();
    
    /* НЕТ __dict__ если slots */
    if (type->tp_dictoffset == 0) {
        /* slots = ['name', 'age'] -> фиксированные поля */
        memset(inst + type->tp_basicsize, 0, nslots * sizeof(PyObject *));
    }
    
    return inst;
}
```

`class Point: __slots__ = ['x', 'y']` → `Point.tp_dictoffset = 0` → **НЕТ** `PyDictObject __dict__` (экономия ~64
байт!). `p.x` → **фиксированное поле** по offset (не поиск в хеш-таблице). `p.z = 1` → **AttributeError**!

## 4. __getattribute__ защита (Objects/object.c)

```c
static PyObject *instance_getattro(PyObject *self, PyObject *name) {
    PyInstanceObject *inst = (PyInstanceObject*)self;
    PyTypeObject *type = Py_TYPE(self);
    
    /* 1. __getattribute__ перехватывает */
    if (type->tp_getattro != PyObject_GenericGetAttr) {
        return type->tp_getattro(self, name);
    }
    
    /* 2. Дескрипторы класса */
    PyObject *descr = _PyType_Lookup(type, name);
    if (descr != NULL) {
        descrgetfunc f = Py_TYPE(descr)->tp_descr_get;
        if (f != NULL) {
            PyObject *res = f(descr, self, (PyObject *)type);
            return res;
        }
    }
    
    /* 3. __dict__[name] */
    PyObject **dictptr = _PyObject_GetDictPtr(self);
    if (dictptr != NULL && *dictptr != NULL) {
        PyObject *res = PyDict_GetItem(*dictptr, name);
        if (res != NULL) {
            Py_INCREF(res);
            return res;
        }
    }
    
    /* 4. __getattr__ */
    if (type->tp_getattro == PyObject_GenericGetAttr) {
        PyObject *res = PyObject_GenericGetAttr(self, name);
        if (res != NULL)
            return res;
    }
    
    PyErr_Format(PyExc_AttributeError,
        "'%.50s' object has no attribute '%.400s'",
        type->tp_name, PyUnicode_AsUTF8(name));
    return NULL;
}
```

`class C: def __getattribute__(self, name): if name=='_private': raise AttributeError` → **все** `c._private` →
`__getattribute__` → **блок**. **НЕ** доходит до `__dict__`!

## 5. Property дескриптор (Objects/descrobject.c)

```c
typedef struct {
    PyObject_HEAD
    PyGetterDescrObject *prop_get;     // fget функция
    PySetterDescrObject *prop_set;     // fset функция
    PyDeleterDescrObject *prop_del;    // fdel функция
    PyObject *prop_doc;                // __doc__
} PyPropertyObject;

static PyObject *property_get(PyPropertyObject *self, PyObject *obj, PyObject *type) {
    PyObject *func = (PyObject*)self->prop_get;
    
    if (func == NULL) {
        PyErr_SetString(PyExc_AttributeError,
            "can't get attribute");
        return NULL;
    }
    
    /* Вызываем getter(self) */
    PyObject *args = PyTuple_New(1);
    PyTuple_SET_ITEM(args, 0, Py_NewRef(obj));
    PyObject *result = PyObject_Call(func, args, NULL);
    Py_DECREF(args);
    return result;
}
```

`@property def x(self): return self._x` → **PyPropertyObject** с `prop_get=fget`. `c.x` →
`property_get(property, c, C)` → `fget(c)` → `_x`. **НЕ** хранит значение — **вычисляет**!

## 6. __slots__ layout в памяти

```c
class Point:
    __slots__ = ['x', 'y']  # 2 * PyObject* = 16 байт

# Point в памяти:
PyObject_HEAD (16 байт) + x (8 байт) + y (8 байт) = 32 байт
# Без slots: PyObject_HEAD + __dict__ (64+ байт) = 80+ байт!
```

**Обычный класс**: `[refcnt][type][__dict__ ptr][данные]`. **`__slots__`**: `[refcnt][type][x ptr][y ptr]`. **НЕТ**
хеш-таблицы `__dict__` — **фиксированные поля** по offset. **10x быстрее** доступ + **экономия памяти**.

## 7. Байткод доступа к атрибуту

```python
class C:
    def __init__(self):
        self.x = 42


c = C()
print(c.x)
```

```
# Байткод:
  0 LOAD_GLOBAL         0 (C)
  2 CALL                0
  4 STORE_FAST          0 (c)
  6 LOAD_FAST           0 (c)
  8 LOAD_ATTR           0 (x)        # c.x -> PyObject_GetAttr
 10 CALL                1 (print)
```

`LOAD_ATTR 0 (x)` → `PyObject_GetAttr(c, 'x')` → `instance_getattro(c, 'x')` → **поиск**: класс → `__dict__` →
`__getattr__`.

## 8. Защита через __setattr__ (user-level)

```python
class SafeDict(dict):
    def __setattr__(self, name, value):
        if name.startswith('_'):
            raise AttributeError(f"cannot set {name}")
        object.__setattr__(self, name, value)

    def __setitem__(self, key, value):
        if key.startswith('_'):
            raise KeyError(f"cannot set {key}")
        super().__setitem__(key, value)
```

`sd = SafeDict(); sd._secret = 1` → `__setattr__` → **блок**. `sd['_secret'] = 1` → `__setitem__` → **блок**. **Двойная
защита**!

**Инкапсуляция** в CPython 3.9+ — **name mangling** (`_Class__private` compile-time), **`__slots__`** (
`tp_dictoffset=0`), **`__getattribute__`/`__setattr__`** перехват, **property дескрипторы** (`prop_get`), *
*`PyObject_GenericGetAttr()`** порядок поиска.

- [Содержание](/CONTENTS.md#содержание)