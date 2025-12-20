# **ООП**

- [Содержание](/CONTENTS.md#содержание)

## **Junior Level**

Объектно-ориентированное программирование (ООП) — это подход к разработке программ, где основными строительными блоками
являются объекты, которые объединяют данные и методы для работы с ними.

Основные концепции ООП:

1. **Класс** — чертёж или шаблон для создания объектов (например, класс `Dog`).
2. **Объект** — конкретный экземпляр класса (например, `my_dog = Dog()`).
3. **Инкапсуляция** — объединение данных и методов в одном объекте и сокрытие внутренней реализации от внешнего мира.
4. **Наследование** — возможность создавать новые классы на основе существующих, перенимая их свойства и методы (
   например, класс `Poodle` наследует от `Dog`).
5. **Полиморфизм** — возможность использовать объекты разных классов через одинаковый интерфейс (например, методы
   `speak()` для `Dog` и `Cat` могут работать по-разному).
6. **Абстракция** - это концепция, которая позволяет выделить существенные характеристики объекта, игнорируя
   несущественные детали реализации. Она помогает работать со сложными системами, представляя их в виде упрощённых
   моделей.

ООП помогает организовать код, делает его более понятным, удобным для повторного использования и изменения.

- [Содержание](/CONTENTS.md#содержание)

## **Middle Level**

В Python ООП реализовано динамически и поддерживает все классические принципы, а также некоторые уникальные особенности:

**1. Динамическая природа классов и объектов**:

- Классы являются объектами первого класса (можно присваивать переменным, передавать в функции).
- Атрибуты и методы могут добавляться, изменяться или удаляться во время выполнения.
- Поддержка динамического создания классов через `type()`.

**2. Множественное наследование и MRO**:

- Python поддерживает множественное наследование.
- Алгоритм C3 Linearization определяет порядок разрешения методов (Method Resolution Order, MRO).
- `super()` используется для корректного вызова методов родительских классов.

**3. Дескрипторы и свойства**:

- **Дескрипторы** — объекты, реализующие протокол `__get__`, `__set__`, `__delete__`. Лежат в основе свойств, методов,
  статических методов и методов класса.
- **Свойства (property)** — позволяют использовать getter, setter и deleter для атрибутов, сохраняя синтаксис доступа к
  атрибуту.

**4. Абстрактные классы**:

- Модуль `abc` позволяет создавать абстрактные классы и методы.
- Абстрактный класс не может быть инстанциирован, требует переопределения абстрактных методов в дочерних классах.

**5. Магические методы (dunder methods)**:

- Методы вида `__init__`, `__str__`, `__add__` и т.д.
- Позволяют переопределять поведение операторов и встроенных функций для объектов пользовательских классов.

**6. Метаклассы**:

- Классы, экземпляры которых являются другими классами.
- Позволяют вмешиваться в процесс создания класса, добавлять валидацию, регистрацию и т.д.

**7. Принципы SOLID в Python**:

- **Single Responsibility**: Каждый класс должен иметь одну причину для изменения.
- **Open/Closed**: Классы должны быть открыты для расширения, но закрыты для изменения.
- **Liskov Substitution**: Объекты базового класса должны быть заменяемы объектами производных классов без изменения
  корректности программы.
- **Interface Segregation**: Много специализированных интерфейсов лучше одного универсального.
- **Dependency Inversion**: Зависимости должны строиться на абстракциях, а не на деталях.

**8. Протоколы и утиная типизация**:

- Python использует утиную типизацию: интерфейс объекта определяется его поведением (наличием методов), а не явным
  объявлением.
- **Протоколы** — неформальные интерфейсы, например, протокол итератора требует методы `__iter__` и `__next__`.

- [Содержание](/CONTENTS.md#содержание)

## **Senior Level**

В CPython 3.9+ **ООП** — **PyTypeObject** (классы как объекты), **MRO** (`tp_mro` tuple), **descriptor протокол** (
`__get__`/`__set__`), **`PyType_Ready()`** инициализация, **`super()`** → `_PySuperObject`, **tp_call** для
`Class()`. `Objects/typeobject.c`,`Objects/classobject.c`,`Python/compile.c`

## 1. PyTypeObject структура (Include/cpython/object.h)

```c
typedef struct _typeobject {
    PyObject_VAR_HEAD         // ob_base = PyVarObject_HEAD_INIT + tp_name
    Py_ssize_t tp_basicsize;  // Размер экземпляра (без переменных полей)
    Py_ssize_t tp_itemsize;   // Размер для массивов (list/tuple=sizeof(PyObject*))
    
    // Методы типа
    destructor tp_dealloc;          // __del__
    printfunc tp_print;             // print(obj)
    getattrfunc tp_getattr;         // __getattr__
    setattrfunc tp_setattr;         // __setattr__
    PyAsyncMethods *tp_as_async;    // async методы
    reprfunc tp_repr;               // repr()
    
    // Кэш атрибутов
    Py_ssize_t tp_hash;             // Кэш хеша типа
    
    // Итерация
    unaryfunc tp_call;              // obj() вызов
    reprfunc tp_str;                // str()
    getattrofunc tp_getattro;       // obj.name поиск
    setattrofunc tp_setattro;       // obj.name = value
    
    // Дескрипторный протокол
    PyBufferProcs *tp_as_buffer;    // buffer протокол
    
    unsigned long tp_flags;         // Py_TPFLAGS_BASETYPE и т.д.
    
    const char *tp_doc;             // __doc__
    traversal tp_traverse;          // GC traverse
    inquiry tp_clear;               // GC clear
    richcmpfunc tp_richcompare;     // ==
    
    // MRO и наследование
    Py_ssize_t tp_weaklistoffset;   // Слабые ссылки
    Py_ssize_t tp_dictoffset;       // __dict__ offset
    Py_ssize_t tp_init;             // __init__
    Py_ssize_t tp_alloc;            // __alloc__
    vectornewfunc tp_new;           // __new__
    freefunc tp_free;               // tp_free
    
    // Классовые атрибуты
    PyObject *tp_bases;             // tuple базовых классов
    tuple mro;                      // Method Resolution Order tuple
    PyObject *tp_cache;             // Кэш атрибутов
    PyObject *tp_subclasses;        // Список подклассов
    PyObject *tp_weaklist;          // Слабые ссылки
    destructor tp_del;              // __del__
    
    // 3.8+ Finalization
    destructor tp_version_tag;      // Версия кэша
    destructor tp_finalize;         // __del__
} PyTypeObject;
```

`class MyClass:` → **PyTypeObject** размером ~300 байт в памяти.
`MyClass.__bases__` = `(object,)`, `MyClass.mro` = `(MyClass, object)`. `MyClass()` → `MyClass.tp_new(MyClass)` →
`MyClass.tp_alloc()` → `MyClass.tp_init()`. **Всё** в одном объекте!

## 2. PyType_Ready() - инициализация класса (Objects/typeobject.c)

```c
int PyType_Ready(PyTypeObject *type) {
    // 1. Инициализируем базовые поля
    if (type->tp_flags & Py_TPFLAGS_READY) {
        return 0;                      // Уже готов
    }
    
    // 2. Вычисляем MRO (Method Resolution Order)
    if (mro_internal(type) < 0) {
        return -1;
    }
    
    // 3. Инициализируем dictoffset (для __dict__)
    if (type->tp_dictoffset && 
        type->tp_dictoffset != _PyObject_DictOffset()) {
        PyErr_Format(PyExc_TypeError,
            "instance dictoffset must be %zd, not %zd",
            _PyObject_DictOffset(), type->tp_dictoffset);
        return -1;
    }
    
    // 4. Инициализируем слабые ссылки
    if (type->tp_weaklistoffset && 
        !PyType_SUPPORTS_WEAKREFS(type)) {
        return -1;
    }
    
    // 5. Устанавливаем флаги READY
    type->tp_flags |= Py_TPFLAGS_READY;
    
    // 6. Обновляем кэш подклассов
    update_all_slots(type);
    
    return 0;
}
```

После компиляции `class MyClass:` Python вызывает `PyType_Ready(&MyClass_Type)`. *
*Главное** — `mro_internal()` вычисляет `(MyClass, object)` → `MyClass.tp_mro`. Проверяет `__dict__` offset (всегда 184
байта в PyDictObject*). Устанавливает `Py_TPFLAGS_READY` флаг. **Только после этого** класс готов к использованию!

## 3. mro_internal() - вычисление MRO (typeobject.c)

```c
static int mro_internal(PyTypeObject *type) {
    PyObject *mro;
    PyObject *bases;
    
    // Базовый случай
    if (PyTuple_GET_SIZE(type->tp_bases) == 0) {
        mro = PyTuple_New(1, (PyObject *)type);
        type->tp_mro = mro;
        return 0;
    }
    
    // C3 линейнаязация (псевдокод)
    mro = PyTuple_New(0);
    todo = list(type->tp_bases);
    
    while (todo) {
        candidate = find_candidate(todo);  // Первый общий класс
        if (!candidate) {
            // Diamond problem!
            PyErr_SetObject(PyExc_TypeError, "MRO conflict");
            return -1;
        }
        append(candidate, mro);
        remove_from_todos(candidate, todo);
    }
    
    type->tp_mro = mro;
    return 0;
}
```

`class D(B, C):` → MRO должен быть `D, B, C, object`. **C3 алгоритм** решает *
*алмазную проблему** наследования. Ищет **первый** класс, **присутствующий** во **всех** путях. **Гарантирует**:
дочерний **перед** родителями, **линейный** порядок. `super()` следует **этому** порядку!

## 4. obj.name атрибутный поиск (Objects/object.c)

```c
PyObject *PyObject_GenericGetAttr(PyObject *obj, PyObject *name) {
    PyTypeObject *tp = Py_TYPE(obj);
    PyObject *descr;
    
    // 1. Ищем дескриптор в типе
    descr = _PyType_Lookup(tp, name);
    if (descr != NULL) {
        descr_getfunc f = Py_TYPE(descr)->tp_descr_get;
        if (f != NULL) {
            // Дескриптор! Вызываем __get__
            PyObject *res = f(descr, obj, (PyObject *)tp);
            if (res != NULL) {
                return res;
            }
        }
    }
    
    // 2. Ищем в __dict__ экземпляра
    if (tp->tp_dictoffset != 0) {
        PyObject **dictptr = _PyObject_GetDictPtr(obj);
        if (dictptr && *dictptr != NULL) {
            PyObject *res = PyDict_GetItem(*dictptr, name);
            if (res != NULL) {
                Py_INCREF(res);
                return res;
            }
        }
    }
    
    // 3. Ищем в __dict__ типа
    if (PyType_HasFeature(tp, Py_TPFLAGS_HAVE_CLASS)) {
        PyObject *res = _PyType_Lookup(tp, name);
        if (res != NULL) {
            Py_INCREF(res);
            return res;
        }
    }
    
    // AttributeError
    PyErr_Format(PyExc_AttributeError,
        "'%.50s' object has no attribute '%.400s'",
        tp->tp_name, PyObject_GETITEM(name));
    return NULL;
}
```

`obj.name` → **3 места поиска**: 1) **класс** (property/method), 2) **__dict__
экземпляра**, 3) **__dict__ класса**. **Дескрипторы** (`property`) имеют **приоритет** — вызывается
`property.__get__(obj)`. `__dict__` — **PyDictObject** по offset в `tp_dictoffset`.

## 5. super() байткод + _PySuperObject (Objects/classobject.c)

```python
class B:
    def f(self): pass


class C:
    def f(self): pass


class D(B, C):
    def f(self):
        super().f()  # B.f(self)
```

```python
# Байткод D.f():
0
LOAD_SUPER_ATTR
0  # super().__getattr__('f')
2
LOAD_FAST
0(self)
4
CALL
1  # B.f(self)
```

**LOAD_SUPER_ATTR (ceval.c):**

```c
case LOAD_SUPER_ATTR: {
    PyObject *super = GETITEM(names, oparg>>8);     // super объект
    PyObject *attr = GETITEM(names, oparg&0xFF);   // 'f'
    
    // super = _PySuperObject(type=D, obj=self)
    _PySuperObject *s = (_PySuperObject*)super;
    
    // Ищем в MRO после D
    PyTypeObject *start = s->type;
    PyObject *mro = start->tp_mro;
    
    for (Py_ssize_t i = PySequence_Index(mro, (PyObject*)start) + 1; 
         i < PyTuple_GET_SIZE(mro); i++) {
        PyTypeObject *cls = PyTuple_GET_ITEM(mro, i);
        PyObject *res = _PyType_Lookup(cls, attr);
        if (res != NULL) {
            // НАЙДЕН! B.f
            Py_INCREF(res);
            PUSH(res);
            DISPATCH();
        }
    }
    
    PyErr_SetObject(PyExc_AttributeError, attr);
    goto error;
}
```

`super().f()` → `LOAD_SUPER_ATTR` → `_PySuperObject{D, self}` → **MRO**
`D, B, C, object` → **пропускаем D** → ищем в `B, C, object` → **B.f найден** → возвращаем **bound method** `B.f(self)`.
**Автоматически** вызывает **правильный** родитель!

## 6. Класс создание: compiler_class() (Python/compile.c)

```c
static int compiler_class(struct compiler *c, location loc, stmt_ty s) {
    identifier name = s->v.ClassDef.name;     // 'MyClass'
    asdl_expr_seq *bases = s->v.ClassDef.bases;  // (object,)
    asdl_keyword_seq *keywords = s->v.ClassDef.keywords;
    
    // 1. Создаём namespace для класса
    if (!compiler_enter_scope(c, name, COMPILER_SCOPE_CLASSBLOCK, s, loc)) {
        return -1;
    }
    
    // 2. Базовые классы
    VISIT_SEQ(c, expr, bases);                 // LOAD_GLOBAL object
    
    // 3. Ключевые аргументы (metaclass)
    if (keywords) {
        VISIT_KEYWORDS(c, keywords);
    }
    
    // 4. Тело класса
    VISIT_SEQ(c, stmt, s->v.ClassDef.body);
    
    // 5. Генерируем: MyClass = type(name, (object,), {})
    ADDINSTR(c, loc, BUILD_TUPLE, 1);          // bases tuple
    ADDINSTR(c, loc, LOAD_CONST, add_empty_dict);  // {}
    ADDINSTR(c, loc, CALL_FUNCTION_EX, 1);     // type(name, bases, {})
    ADDINSTR(c, loc, STORE_NAME, add(name));   // MyClass =
    
    compiler_exit_scope(c);
    return 0;
}
```

`class MyClass(object): pass` → байткод: `bases=(object,)`,
`dict={}, CALL_FUNCTION_EX(1)` → **вызов** `type('MyClass', (object,), {})` → **PyTypeObject** создания!

## 7. type.__call__() = Class() создание (Objects/typeobject.c)

```c
static PyObject *type_call(PyTypeObject *type, PyObject *args, PyObject *kwds) {
    PyObject *obj;
    
    // 1. Выделяем экземпляр
    if (type->tp_new == NULL) {
        PyErr_Format(PyExc_TypeError,
            "cannot create '%.100s' instances",
            type->tp_name);
        return NULL;
    }
    
    obj = type->tp_new(type, args, kwds);  // MyClass.__new__()
    if (obj == NULL) {
        return NULL;
    }
    
    // 2. Инициализируем
    if (PyType_HasFeature(type, Py_TPFLAGS_HAVE_INIT) &&
        type->tp_init != NULL) {
        int res = type->tp_init(obj, args, kwds);  // MyClass.__init__()
        if (res < 0) {
            Py_DECREF(obj);
            return NULL;
        }
    }
    
    return obj;
}
```

`MyClass()` → `type_call(&MyClass_Type)` → `MyClass.tp_new(MyClass)` → **malloc** →
`MyClass.tp_init(obj)` → **готовый экземпляр**. По умолчанию `object.__new__()` + `object.__init__()`.

## 8. Property дескриптор (Objects/descrobject.c)

```c
static PyObject *property_getter(PyObject *self, PyObject *obj, PyObject *type) {
    // self = property(fget=name)
    PyObject *func = ((PyPropertyObject *)self)->prop_get;
    
    if (func == NULL) {
        PyErr_SetString(PyExc_AttributeError, "unreadable property");
        return NULL;
    }
    
    // Вызываем getter функцию
    PyObject *args = PyTuple_New(1, obj);  // (self,)
    PyObject *result = PyObject_Call(func, args, NULL);
    Py_DECREF(args);
    return result;
}
```

`class C: @property def x(self): return 42` → **C.x** = **PyPropertyObject** с
`prop_get=fget`. `c = C(); c.x` → `property_getter(property, c, C)` → `fget(c)` → `42`. **Автоматически**!

**ООП** в CPython 3.9+ — **PyTypeObject** (класс=тип), **MRO** C3 алгоритм, **descriptor протокол** (`__get__`),
`PyType_Ready()`, **`super()`** → MRO поиск, `type.__call__()` → `__new__`+`__init__`, **property** дескрипторы.

- [Содержание](/CONTENTS.md#содержание)