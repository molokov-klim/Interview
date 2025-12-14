# **ООП**

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

- [Содержание](CONTENTS.md#содержание)

---

# **Абстракция**

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

- [Содержание](CONTENTS.md#содержание)

---

# **Инкапсуляция**

## **Junior Level**

Инкапсуляция — это принцип объектно-ориентированного программирования, который объединяет данные и методы, работающие с
этими данными, внутри одного объекта и скрывает внутренние детали реализации от внешнего мира.

Представьте, что у вас есть банковский счет. Вы можете пополнять его, снимать деньги и проверять баланс, но вам не нужно
знать, как именно банк хранит ваши данные, как они обрабатывают транзакции или как обновляют баланс. Вам предоставляют
простой интерфейс (например, банкомат или мобильное приложение), а все сложности скрыты внутри.

В Python инкапсуляция реализуется через модификаторы доступа: публичные (public), защищенные (protected) и приватные (
private) атрибуты и методы. Это помогает защитить данные от неправильного использования и обеспечивает контроль над тем,
как объект взаимодействует с внешним миром.

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

- [Содержание](CONTENTS.md#содержание)

---

# **Наследование**

## **Junior Level**

Наследование в ООП — это механизм, позволяющий создавать новый класс на основе существующего, перенимая его свойства и
методы. Новый класс называется **дочерним** (подклассом), а существующий — **родительским** (суперклассом).

Представьте, что у вас есть класс `Animal` с методами `eat()` и `sleep()`. Вы можете создать класс `Dog`, который
наследует от `Animal`, и автоматически получит эти методы. Затем вы можете добавить специфичное поведение для собаки:
метод `bark()`. Это позволяет избежать дублирования кода и создавать логические иерархии объектов.

Наследование представляет отношение **"является"** (is-a): собака является животным. Это один из основных способов
достижения полиморфизма — возможности использовать объекты разных классов через общий интерфейс.

## **Middle Level**

В Python наследование имеет несколько ключевых особенностей:

1. **Синтаксис наследования**:
   ```python
   class Parent:
       pass
   
   class Child(Parent):
       pass
   ```

2. **Множественное наследование**:
   Python поддерживает наследование от нескольких классов:
   ```python
   class Child(Parent1, Parent2, Parent3):
       pass
   ```

3. **Переопределение методов**:
   Дочерний класс может переопределять методы родительского класса, предоставляя свою реализацию.

4. **Расширение методов**:
   Использование `super()` для вызова родительской реализации:
   ```python
   def method(self):
       super().method()  # Вызов родительского метода
       # Дополнительная логика
   ```

5. **Method Resolution Order (MRO)**:
   Порядок поиска методов при множественном наследовании. Определяется алгоритмом C3 и доступен через
   `ClassName.__mro__`.

6. **Абстрактные классы и наследование**:
   Использование `abc.ABC` для создания классов, которые нельзя инстанциировать, но от которых можно наследоваться.

7. **Миксины (Mixins)**:
   Классы, предназначенные для добавления функциональности через множественное наследование. Не предназначены для
   самостоятельного использования.

8. **Доступ к родительским атрибутам**:
    - Через `super()`
    - Через явное указание имени родительского класса: `ParentClass.method(self)`

9. **Наследование встроенных типов**:
   Можно наследоваться от `list`, `dict`, `str` и других встроенных типов, но это требует осторожности.

10. **Композиция vs наследование**:
    Наследование создаёт жёсткую связь "является", композиция — гибкую связь "имеет". Композиция часто предпочтительнее.

## **Senior Level**

В CPython 3.9+ **наследование** реализовано через **PyTypeObject.tp_bases** (кортеж баз), **tp_base** (один общий
базовый), **tp_mro** (готовый MRO C3), и работу `PyType_Ready()` + `type.__call__` + `LOAD_SUPER_ATTR` для
`super()`.[1][2]

***

## PyTypeObject и хранение баз

```c
// Упрощённая часть PyTypeObject (Objects/typeobject.c)
typedef struct _typeobject {
    PyObject_VAR_HEAD              // refcnt, type="type", имя класса
    const char *tp_name;           // "MyClass"
    Py_ssize_t tp_basicsize;       // размер экземпляра
    Py_ssize_t tp_itemsize;        // для переменной длины

    // ... много слотов опущено ...

    PyObject *tp_bases;            // tuple базовых классов (B, C)
    PyTypeObject *tp_base;         // один "основной" базовый тип
    PyObject *tp_mro;              // tuple MRO: (D, B, C, object)
    PyObject *tp_dict;             // __dict__ самого класса
    PyObject *tp_subclasses;       // список слабых ссылок на подклассы

    unsigned long tp_flags;        // флаги (Py_TPFLAGS_HAVE_GC, BASETYPE и т.п.)
} PyTypeObject;
```

- `tp_bases` — кортеж всех указанных баз (`(B, C)` у `class D(B, C): ...`).
- `tp_base` — “основный” один базовый тип (обычно первый в `tp_bases`, либо `object`).
- `tp_mro` — кортеж **линейного порядка разрешения методов** (`(D, B, C, object)` для ромбовидного наследования).
- `tp_subclasses` — связка “родитель знает, какие у него дети” (для `__subclasses__()`).

каждый класс в CPython — это **C-структура** `PyTypeObject`, где есть: имя класса, размеры объектов, список родителей,
MRO, словарь методов, список детей. Наследование пишется в питоне, но *хранится* как поля `tp_bases/tp_mro` в этой
структуре.

***

## Создание класса и заполнение tp_bases (compiler_class)

```c
// Python/compile.c — генерация кода для class
static int
compiler_class(struct compiler *c, location loc, stmt_ty s) {
    // s->v.ClassDef.name = имя класса (identifier)
    // s->v.ClassDef.bases = список выражений базовых классов

    // 1) скомпилировать выражения баз, они окажутся на стеке
    VISIT_SEQ(c, expr, s->v.ClassDef.bases);  // LOAD_NAME B, LOAD_NAME C, ...

    // 2) собрать их в tuple
    ADDOP_I(c, loc, BUILD_TUPLE, PySequence_SIZE(s->v.ClassDef.bases));
    // теперь на стеке tuple(bases)

    // 3) создать namespace (обычно dict)
    ADDOP(c, loc, LOAD_BUILD_CLASS);       // функция builtins.__build_class__
    // ... компиляция тела класса в отдельную функцию-прослойку ...

    // 4) вызвать __build_class__(body_func, name, bases_tuple, **kw)
    // возвращённый объект – готовый класс (= PyTypeObject на C стороне)
    ADDOP(c, loc, CALL_FUNCTION_EX, 1);    // CALL_FUNCTION_EX оparg=1: есть *args

    // 5) сохранить класс в locals
    ADDOP_NAME(c, loc, STORE_NAME, s->v.ClassDef.name, names);

    return 1;
}
```

`class D(B, C): ...` компилируется в вызов `__build_class__`, которому передаётся: функция-тело класса, строка `"D"`,
кортеж `(B, C)` и kwargs (метакласс, если был). Дальше всё делает C-реализация `type`/метакласса.

***

## type.__new__ и PyType_Ready: инициализация наследования

```c
// Objects/typeobject.c — создание нового типа (класса)
static PyObject *
type_new(PyTypeObject *metatype, PyObject *args, PyObject *kwds)
{
    // args: (name, bases, dict)
    PyObject *name, *bases, *dict;
    // ... разбор args ...
    // создаём "сырую" структуру PyHeapTypeObject
    PyHeapTypeObject *et = (PyHeapTypeObject *)PyType_GenericAlloc(metatype, 0);
    PyTypeObject *type = &et->ht_type;

    // заполняем tp_name, tp_bases, tp_dict
    type->tp_name = /* C-строка имени */;
    Py_INCREF(bases);
    type->tp_bases = bases;
    Py_INCREF(dict);
    type->tp_dict = dict;

    // вычисляем tp_base = подходящий один базовый
    type->tp_base = best_base(bases);    // обычно первый в bases

    // готовим тип (mro, слоты, flags и т.п.)
    if (PyType_Ready(type) < 0) {
        Py_DECREF(et);
        return NULL;
    }
    return (PyObject *)type;
}
```

`type.__new__` выделяет память под новый класс, кладёт туда имя, кортеж баз, словарь методов, выбирает один “главный”
базовый (`best_base`) и потом зовёт `PyType_Ready`, чтобы посчитать MRO и прочее. До `PyType_Ready` класс “сырой”,
после — “готовый”.

***

## MRO: mro_internal и C3-алгоритм

```c
// Objects/typeobject.c
static int
mro_internal(PyTypeObject *type)
{
    PyObject *mro;

    // Если метакласс переопределил mro() — вызвать его
    PyObject *meth = lookup_mro(type, &_Py_ID(mro)); // type.mro?
    if (meth != NULL) {
        mro = PyObject_CallNoArgs(meth);  // собственный mro()
        // проверка что это tuple типов...
    }
    else {
        // стандартный C3 linearization
        mro = mro_implementation(type);   // см ниже
    }

    if (mro == NULL)
        return -1;

    Py_XSETREF(type->tp_mro, mro);  // tp_mro = кортеж типов
    return 0;
}

// C3 linearization: D(B,C), B(A), C(A), A(object)
static PyObject *
mro_implementation(PyTypeObject *type)
{
    // строит списки: [bases], [mro(base1)], [mro(base2)], ...
    // и затем делает C3 merge, выбирая головные элементы,
    // которые не встречаются в хвостах других списков.
}
```

когда класс создаётся, нужно решить **в каком порядке** искать методы при множественном наследовании. MRO C3
гарантирует:

- дочерний класс **всегда** раньше родителей,
- порядок родителей **сохраняется**,
- каждый класс встречается **один раз**.

Это всё заранее кладётся в `tp_mro = (D, B, C, A, object)`.

***

## Поиск атрибутов по MRO: _PyType_Lookup

```c
// Objects/typeobject.c
PyObject *
_PyType_Lookup(PyTypeObject *type, PyObject *name)
{
    PyObject *mro = type->tp_mro;      // кортеж типов
    Py_ssize_t i, n = PyTuple_GET_SIZE(mro);

    for (i = 0; i < n; i++) {
        PyTypeObject *base = (PyTypeObject *)PyTuple_GET_ITEM(mro, i);
        PyObject *dict = base->tp_dict;            // __dict__ класса
        PyObject *res;

        // обычный поиск в dict
        res = PyDict_GetItemWithError(dict, name);
        if (res != NULL) {
            return res;  // нашли атрибут на этом уровне MRO
        }
        if (PyErr_Occurred()) {
            return NULL;
        }
    }
    return NULL;  // атрибут не найден
}
```

`obj.method` → `PyObject_GetAttr` → `_PyType_Lookup(type(obj), 'method')` → идём по `tp_mro`: сначала класс самого
объекта, потом его родителя, и т.д., пока не `object`. **Первое совпадение** выигрывает.

***

## Наследование реализаций слотов (tp_* из базового типа)

```c
// Objects/typeobject.c — после вычисления MRO
static int
type_ready_set_bases_and_slots(PyTypeObject *type)
{
    // берём "base" как tp_base (один главный базовый)
    PyTypeObject *base = type->tp_base;

    // наследуем слоты, если они не определены в потомке
    if (type->tp_as_number == NULL)
        type->tp_as_number = base->tp_as_number;
    if (type->tp_as_sequence == NULL)
        type->tp_as_sequence = base->tp_as_sequence;
    if (type->tp_as_mapping == NULL)
        type->tp_as_mapping = base->tp_as_mapping;
    // и т.д. для tp_iter, tp_call, tp_hash, tp_str, ...
}
```

если ты не переопределил, например, `__len__` в своём классе, а родитель — `list`, то у твоего класса `tp_as_sequence`
будет указывать на **те же** C-функции, что и у `PyList_Type`. То есть потомок **по-настоящему наследует** реализацию
методов на C-уровне, а не только имена.

***

## type.__call__ и вызов конструктора с наследованием

```c
// Objects/typeobject.c
static PyObject *
type_call(PyTypeObject *type, PyObject *args, PyObject *kwds)
{
    // 1) вызываем tp_new (может быть у потомка или у базового)
    PyObject *obj = type->tp_new(type, args, kwds);
    if (obj == NULL)
        return NULL;

    // 2) вызываем tp_init (конструктор), если есть
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

`D()` при наследовании ведёт себя так:

- сначала вызывается `D.__new__` (если его нет — унаследованный от родителя),
- затем `D.__init__` (если не переопределён — срабатывает родительский).  
  С точки зрения C — это два указателя `tp_new` и `tp_init`, которые либо свои, либо скопированы из базового класса.

***

## super() и наследование: LOAD_SUPER_ATTR

```python
class B:
    def f(self): ...


class C(B):
    def f(self):
        super().f()
```

```
# Байткод C.f (3.11+ условно):
  0 LOAD_FAST           0 (self)
  2 LOAD_GLOBAL         0 (super)
  4 LOAD_FAST           0 (self)
  6 LOAD_CONST          1 (C)
  8 CALL                2        # super(C, self)
 10 LOAD_ATTR           1 (f)    # ищем f начиная после C в MRO
 12 CALL                0
 14 RETURN_VALUE
```

Центр механизма — суперкласс `_PySuper_Type`:

```c
// Objects/typeobject.c/Objects/classobject.c
typedef struct {
    PyObject_HEAD
    PyTypeObject *type;   // текущий класс (C)
    PyObject *obj;        // экземпляр (self)
    PyTypeObject *obj_type; // type(self)
} superobject;

// В __getattribute__ super'а:
static PyObject *
super_getattro(superobject *su, PyObject *name)
{
    // MRO объекта
    PyObject *mro = su->obj_type->tp_mro;

    // находим индекс su->type (C) в mro и начинаем поиск с i+1
    Py_ssize_t i = index_of(mro, (PyObject *)su->type);
    for (i = i+1; i < PyTuple_GET_SIZE(mro); i++) {
        PyTypeObject *base = (PyTypeObject *)PyTuple_GET_ITEM(mro, i);
        PyObject *descr = _PyType_Lookup(base, name);
        if (descr != NULL) {
            // возвращаем bound method/descriptor
            return PyObject_Get(descr, su->obj, (PyObject *)su->obj_type);
        }
    }
    // AttributeError, если не нашли
}
```

`super(C, self).f()` ищет `f` **не в C**, а в **следующих** по MRO классах (`B`, `object`, ...). Это делается, чтобы при
множественном наследовании все родительские реализации вызывались **один раз** и в **правильном порядке**.

***

## __bases__, __mro__ и __subclasses__ на C-уровне

```c
// Возврат __bases__ (type.__getattribute__)
static PyObject *
type_get_bases(PyTypeObject *type, void *context)
{
    if (type->tp_bases == NULL)
        Py_RETURN_NONE;
    Py_INCREF(type->tp_bases);
    return type->tp_bases;
}

static PyObject *
type_get_mro(PyTypeObject *type, void *context)
{
    if (type->tp_mro == NULL)
        Py_RETURN_NONE;
    Py_INCREF(type->tp_mro);
    return type->tp_mro;
}

static PyObject *
type_get_subclasses(PyTypeObject *type, PyObject *args)
{
    // tp_subclasses — список (list) weakref'ов на дочерние классы
    return _PyWeakref_GetRefList(type->tp_subclasses);
}
```

`D.__bases__` — это просто `tp_bases` (кортеж из `PyTypeObject*`), `D.__mro__` — `tp_mro`, `D.__subclasses__()` →
развёрнутый список weakref’ов из `tp_subclasses`. Все эти связи поддерживаются автоматически при создании/удалении
типов.

***

## Наследование встроенных типов (пример list)

```c
// Include/cpython/listobject.h
PyTypeObject PyList_Type = {
    PyVarObject_HEAD_INIT(&PyType_Type, 0)
    "list",                         // tp_name
    sizeof(PyListObject),           // tp_basicsize
    0,                              // tp_itemsize
    (destructor)list_dealloc,       // tp_dealloc
    // ...
    &list_as_sequence,              // tp_as_sequence
    &list_as_mapping,               // tp_as_mapping
    // ...
    &PyBaseObject_Type,             // tp_base = object
    // ...
};
```

Если написать:

```python
class MyList(list):
    pass
```

то создаётся `PyHeapTypeObject`, у которого:

- `tp_base` = `&PyList_Type`,
- слоты `tp_as_sequence`/`tp_as_mapping` по умолчанию **унаследованы** от листа,
- у тебя дополнительно будет свой `tp_dict` с методами класса Python’а.

экземпляр `MyList` в памяти — точно такой же массив элементов, как и `list`, плюс всё, что ты добавишь. Вся арифметика,
конкатенация, индексирование работают через `list_*` функции, потому что твой класс **наследует** их слоты.

***

Итого: наследование в CPython “под капотом” — это:

- создание нового `PyTypeObject` на базе `tp_bases`,
- вычисление `tp_mro` по C3,
- наследование функциональных слотов от `tp_base`,
- поиск атрибутов по `_PyType_Lookup` с проходом по `tp_mro`,
- `super()` реализован как объект, который стартует поиск не с текущего класса, а далее по MRO.


- [Содержание](CONTENTS.md#содержание)

---

# **Полиморфизм**

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

- [Содержание](CONTENTS.md#содержание)

---

# **Diamond Problem**

## **Junior Level**

Проблема ромбовидного наследования (Diamond Problem) — это классическая проблема в объектно-ориентированном
программировании, возникающая при множественном наследовании.
Представьте себе ромб (алмаз), где вершина — это базовый класс A, от него наследуются
два класса B и C, а от обоих наследуется класс D. Если в классе A есть метод `some_method()`, и классы B и C
переопределяют его по-разному, то возникает вопрос: какую реализацию унаследует класс D — от B или от C?

Эта проблема создает неоднозначность в поведении программы. В Python эта проблема решается с помощью четко определенного
**порядка разрешения методов (MRO - Method Resolution Order)**, который определяет, в каком порядке интерпретатор будет
искать метод в иерархии наследования.

## **Middle Level**

Технически в Python проблема решается алгоритмом **C3 linearization**, который гарантирует детерминированный и
предсказуемый порядок поиска методов.

1. **MRO (Method Resolution Order):**
    - MRO — это порядок, в котором Python ищет методы в иерархии классов. Его можно посмотреть через атрибут класса
      `__mro__` или метод `mro()`.
    - Алгоритм C3 гарантирует, что:
        - Каждый класс встречается в MRO ровно один раз.
        - Подклассы идут перед своими суперклассами.
        - Порядок наследования, указанный в определении класса, сохраняется.

2. **Пример:**
   Для иерархии A -> B, A -> C, B -> D, C -> D, MRO для D будет `[D, B, C, A, object]`. При вызове метода из D Python
   будет искать его в этом порядке.

3. **Механизм `super()`:**
    - `super()` — это не вызов родительского класса в традиционном понимании, а вызов следующего класса в MRO.
    - Это позволяет реализовать **кооперативное множественное наследование**, где каждый класс в цепочке может добавить
      свою функциональность, вызывая `super()`.
    - При правильном использовании `super()` во всех классах иерархии (A, B, C, D) метод будет вызван у каждого из них,
      что позволяет избежать "потери" вызова.

## **Senior Level**

В CPython 3.9+ **Diamond Problem** решается **C3-линеаризацией MRO** в `PyType_Ready()` (`Objects/typeobject.c`),
которая строит **единую последовательность** `(D, B, C, A, object)` без дублирования `A`. Поиск атрибутов идёт строго по
`tp_mro`, **первый** найденный метод выигрывает. `Objects/typeobject.c`,`Python/ceval.c`

## 1. Diamond иерархия и MRO (Python/compile.c → type_new)

```python
class A:  # tp_mro = (A, object)
    def m(self): print("A")


class B(A):  # tp_mro = (B, A, object)
    pass


class C(A):  # tp_mro = (C, A, object)
    pass


class D(B, C):  # tp_mro = (D, B, C, A, object) ← C3!
    pass
```

```c
// Objects/typeobject.c — type_new() при создании D
static PyObject *type_new(PyTypeObject *metatype, PyObject *args, PyObject *kwds) {
    PyObject *name, *bases, *dict;  // ("D", (B, C), {...})
    
    // ... парсинг args ...
    
    PyHeapTypeObject *et = (PyHeapTypeObject*)PyType_GenericAlloc(metatype, 0);
    PyTypeObject *type = &et->ht_type;  // новый тип D
    
    type->tp_name = PyUnicode_AsUTF8(name);    // "D"
    Py_INCREF(bases);                          // tp_bases = (B, C)
    type->tp_bases = bases;
    Py_INCREF(dict);
    type->tp_dict = dict;                      // методы D
    
    type->tp_base = best_base(bases);          // B (первый в tp_bases)
    
    if (PyType_Ready(type) < 0) {              // ← СЮДА C3-магия!
        Py_DECREF(et);
        return NULL;
    }
    return (PyObject*)type;
}
```

При `class D(B, C):` создаётся новый `PyTypeObject *D`, куда кладут `tp_bases=(B,C)`. Затем
`PyType_Ready(D)` **вычисляет** `tp_mro=(D,B,C,A,object)` по алгоритму C3 и сохраняет его. **НЕТ** дублирования `A`!

## 2. C3-линеаризация: mro_internal() и merge

```c
// Objects/typeobject.c — вычисление MRO для D
static int mro_internal(PyTypeObject *type) {
    PyObject *mro;  // результат: tuple(D, B, C, A, object)
    
    // если метакласс переопределил mro() — вызываем его
    PyObject *mro_meth = _PyType_Lookup(type, &_Py_ID(mro));
    if (mro_meth != NULL) {
        mro = PyObject_CallNoArgs(mro_meth);
        // проверяем, что это корректный tuple типов
        if (!PyTuple_Check(mro) || !mro_is_linearized(mro)) {
            PyErr_SetString(PyExc_TypeError, "bad mro");
            return -1;
        }
    } else {
        // стандартный C3
        mro = mro_implementation(type);
    }
    
    if (mro == NULL)
        return -1;
    
    Py_XSETREF(type->tp_mro, mro);     // D->tp_mro = (D,B,C,A,object)
    return 0;
}
```

`mro_internal(D)` строит **линейный список** классов для поиска методов. **C3 гарантирует**:
`D` первый, `B` перед `C` (порядок в `bases`), `A` **один раз** (не дублируется), `object` последний.

## 3. Алгоритм C3 merge (упрощённо в mro_implementation)

```c
// Псевдокод C3 для D(B,C) где B(A), C(A):
static PyObject *mro_implementation(PyTypeObject *type) {
    PyObject *result = PyList_New(0);  // [D]
    PyList_Append(result, (PyObject*)type);  // result = [D]
    
    // L1 = mro(B) = [B, A]
    // L2 = mro(C) = [C, A]  
    // L3 = bases(D) = [B, C]
    PyObject *L[3] = {mro_of_bases[0], mro_of_bases[1], bases_tuple};
    
    while (true) {
        PyObject *candidates = find_heads(L);  // головы списков: B, C
        
        if (PyList_GET_SIZE(candidates) == 0) {
            // цикл! TypeError
            PyErr_SetString(PyExc_TypeError, "Cannot create a consistent MRO");
            return NULL;
        }
        
        PyObject *candidate = choose_head(candidates);  // B (левый первый!)
        PyList_Append(result, candidate);               // result = [D, B]
        
        // удаляем B из всех L[] где он встречается
        remove_from_lists(L, candidate);
        
        if (all_lists_empty(L))
            break;
    }
    
    return PyList_AsTuple(result);  // (D, B, C, A, object)
}
```

C3 — это **алгоритм слияния списков**. Берём **головы** всех MRO родителей (`B` из `[B,A]`, `C`
из `[C,A]`). Выбираем **левую первую** (`B`), добавляем в результат, **удаляем** `B` из всех списков. Повторяем: головы
`C`, добавляем `C`, потом `A`.

## 4. Поиск метода по MRO: _PyType_Lookup

```c
// Objects/typeobject.c — поиск d.m() идёт СТРОГО по tp_mro!
PyObject *_PyType_Lookup(PyTypeObject *type, PyObject *name) {
    Py_ssize_t i, n;
    PyObject *mro = type->tp_mro;  // (D, B, C, A, object)
    n = PyTuple_GET_SIZE(mro);
    
    for (i = 0; i < n; i++) {                    // 0:D, 1:B, 2:C, 3:A
        PyTypeObject *base = PyTuple_GET_ITEM(mro, i);
        PyObject *dict = base->tp_dict;          // __dict__ класса
        
        PyObject *res = PyDict_GetItemWithError(dict, name);
        if (res != NULL) {
            Py_INCREF(res);
            return res;                          // ← ПЕРВЫЙ НАЙДЕННЫЙ!
        }
        if (PyErr_Occurred())
            return NULL;
    }
    return NULL;  // AttributeError
}
```

`d.m()` → `PyObject_GetAttr(d, 'm')` → `_PyType_Lookup(D, 'm')` → идём по `D->tp_mro`:
`D.tp_dict['m']`? нет → `B.tp_dict['m']`? нет → `C.tp_dict['m']`? нет → `A.tp_dict['m']`? **ДА** → возвращаем метод
`A.m`. **НЕ** доходим до второго `A` — его нет в MRO!

## 5. Байткод вызова: LOAD_ATTR по MRO

```python
d = D()
d.m()  # ищет m по MRO: D→B→C→A→object
```

```
# Байткод:
  0 LOAD_GLOBAL         0 (D)
  2 CALL                0
  4 STORE_FAST          0 (d)
  6 LOAD_FAST           0 (d)
  8 LOAD_ATTR           0 (m)     # ← _PyType_Lookup(D, 'm') → A.m!
 10 CALL                0
```

В `Python/ceval.c` (LOAD_ATTR):

```c
case LOAD_ATTR: {
    PyObject *name = GETITEM(names, oparg);    // PyUnicode "m"
    PyObject *owner = POP();                   // d (экземпляр D)
    
    PyObject *res = PyObject_GetAttr(owner, name);  // GenericGetAttr
    PUSH(res);
    DISPATCH();
}
```

`LOAD_ATTR 'm'` вызывает `PyObject_GetAttr(d, 'm')` → `instance_getattro(d, 'm')` →
`_PyType_Lookup(D, 'm')` по MRO → **находит в A** → bound method → `CALL`.

## 6. super() в Diamond: пропуск текущего класса

```python
class B(A):
    def m(self):
        super().m()  # ищет начиная ПОСЛЕ B в MRO!


class C(A):
    def m(self):
        super().m()
```

```c
// Objects/typeobject.c — super(B, self).m()
static PyObject *super_getattro(superobject *su, PyObject *name) {
    PyObject *mro = su->obj_type->tp_mro;      // (D,B,C,A,object)
    Py_ssize_t i = find_index(mro, su->type);  // индекс B = 1
    
    // начинаем поиск С i+1 = 2 (C!)
    for (i++; i < PyTuple_GET_SIZE(mro); i++) {
        PyTypeObject *cls = PyTuple_GET_ITEM(mro, i);
        PyObject *method = _PyType_Lookup(cls, name);
        if (method != NULL) {
            // descriptor_get(method, self, D)
            return method;
        }
    }
    PyErr_Format(PyExc_AttributeError, "no attr '%U'", name);
    return NULL;
}
```

`super(B,self).m()` в MRO `(D,B,C,A)` **стартует с C** (пропускает B и раньше). Поэтому
`B.m() → C.m() → A.m()`, **НЕ** `B.m() → A.m()` дважды!

## 7. Ошибка C3: когда MRO невозможен

```python
class X: pass


class Y: pass


class A(X): pass


class B(Y): pass


class C(A, B): pass  # OK: (C,A,X,B,Y,object)


class D(B, A): pass  # TypeError! C3 fail
```

```c
// В mro_implementation при цикле:
if (no_candidates()) {
    PyErr_SetString(PyExc_TypeError,
        "Cannot create a consistent method resolution order (MRO) "
        "for bases ...");
    return NULL;
}
```

Если порядок баз создаёт **цикл** (нельзя выбрать голову без нарушения монотонности),
`PyType_Ready(D)` выбрасывает `TypeError`. C3 **отказывается** строить некорректный MRO.

## 8. Проверка MRO корректности (type_mro_is_valid)

```c
// Objects/typeobject.c — после C3
static int mro_is_linearized(PyObject *mro) {
    Py_ssize_t i, n = PyTuple_GET_SIZE(mro);
    
    // 1. все элементы — типы
    // 2. type всегда ДО своих базовых
    for (i = 0; i < n; i++) {
        PyTypeObject *type = PyTuple_GET_ITEM(mro, i);
        Py_ssize_t j;
        for (j = 0; j < PyTuple_GET_SIZE(type->tp_bases); j++) {
            PyObject *base = PyTuple_GET_ITEM(type->tp_bases, j);
            if (index_of(mro, base) < i) {  // base раньше type? FAIL!
                return 0;
            }
        }
    }
    return 1;
}
```

После построения `(D,B,C,A)` проверяем: `B` после `D`? да. `C` после `D`? да. `A` после `B` и
`C`? да. **MRO валиден**.

## Итог Diamond в CPython

**Diamond Problem** **НЕ существует** в Python 3.9+ благодаря:

1. **C3-линеаризации** в `PyType_Ready()` → `tp_mro=(D,B,C,A,object)` **без дублирования**.
2. **Линейный поиск** `_PyType_Lookup()` по `tp_mro` → **первый** найденный атрибут.
3. **`super()`** пропускает текущий класс → вызывает **следующий** по MRO.
4. **TypeError** при невозможности C3 → отказ от неоднозначного наследования.

Всё вычисляется **один раз** при создании класса, хранится в `tp_mro`, используется **быстро** при каждом `LOAD_ATTR`.

- [Содержание](CONTENTS.md#содержание)

---

# **Магические методы**

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

- [Содержание](CONTENTS.md#содержание)

---

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

- [Содержание](CONTENTS.md#содержание)

---

# **Protocol**

## **Junior Level*

Представьте, что вы описываете не конкретный предмет, а его роль. Например, "то, что можно включить". Под это описание
подходит и лампа, и компьютер, и телевизор — у всех есть кнопка "включить". Вам неважно, что это за устройство внутри;
важно, что оно поддерживает операцию "включения".

В Python **Протокол (Protocol)** — это именно такое формальное описание роли или поведения. Он говорит: "Если у объекта
есть вот такие методы и атрибуты, то он автоматически считается подходящим для определенной цели, даже если он не был
изначально для этого предназначен". Это продвинутая, "официальная" версия принципа утиной типизации ("если ходит как
утка и крякает как утка, то это утка"). В отличие от Abstract Base Class (ABC), где класс должен явно заявить "я
наследуюсь от этого чертежа", протокол работает на уровне "если ты имеешь эти черты, то ты соответствуешь".

## **Middle Level**

Технически `Protocol` — это конструкция системы типизации, определенная в PEP 544 и доступная в модуле `typing`. Его
ключевые аспекты:

1. **Структурная, а не номинативная типизация:** Класс соответствует протоколу, если его **структура** (набор методов и
   атрибутов с правильными сигнатурами) совпадает с протоколом. Ему не нужно явно наследоваться от протокола. Это
   фундаментальное отличие от ABC, где требуется явное номинативное объявление (наследование или регистрация).

2. **Синтаксис и `@runtime_checkable`:** Протокол определяется как класс, наследующийся от `typing.Protocol`. Его методы
   часто помечаются как абстрактные, используя `...` (ellipsis) в теле. По умолчанию протоколы используются **только
   статическими анализаторами типов** (mypy, pyright). Чтобы позволить проверку соответствия протоколу во время
   выполнения (`isinstance(obj, MyProtocol)`), протокол необходимо декорировать `@typing.runtime_checkable`. Однако
   такая проверка ограничена: она проверяет только **наличие** указанных атрибутов, но не их сигнатуры или типы.

3. **Generic и вариативность:** Протоколы могут быть параметризованы (Generic), что позволяет описывать типы, зависящие
   от других типов (например, `Iterable[T]`). Они также поддерживают ковариантность и контравариантность через параметры
   `covariant=True`/`contravariant=True`, что критично для точного описания отношений между сложными типами.

4. **Встроенные протоколы:** Модуль `typing` и `collections.abc` предоставляют множество встроенных протоколов:
   `SupportsInt`, `SupportsBytes`, `ContextManager`, `Iterable`, `Sized` и др. Их использование в аннотациях делает код
   гораздо более выразительным и безопасным.

## **Senior Level**

`Protocol` в CPython — это чистый Python‑код в `Lib/typing.py`: специальный базовый класс c метаклассом
`typing._ProtocolMeta`, который помечает класс как протокольный (структурный) и запоминает множество его членов для
рантайм‑проверок `isinstance`/`issubclass` при включённом `@runtime_checkable`. Никакой спецподдержки в байткоде или
интерпретаторе нет, всё на уровне обычных классов/метаклассов.

## Базовая реализация Protocol в typing.py

В `Lib/typing.py` `Protocol` определён примерно так (упрощённо):

- Класс `Protocol` наследует от `Generic` и использует `_ProtocolMeta` как метакласс:

  ```python
  class Protocol(metaclass=_ProtocolMeta):
      pass
  ```

- `_ProtocolMeta` сам наследует от `abc.ABCMeta` (или `type` в старых версиях), переопределяя `__instancecheck__`/
  `__subclasscheck__` и `__new__`.
- При создании класса‑протокола `class P(Protocol): ...` управление идёт в `_ProtocolMeta.__new__`.

`_ProtocolMeta.__new__(mcls, name, bases, namespace, **kwargs)` делает:

1. Создаёт класс через базовый метакласс (`abc.ABCMeta.__new__`/`type.__new__`).
2. Помечает класс как протокол, устанавливая флаги:

    - `cls._is_protocol = True`;
    - `cls.__protocol_attrs__ = frozenset(<имена членов>)`.

3. Список протокольных членов вычисляется сбором всех атрибутов, объявленных в теле класса и его базовых протоколах,
   исключая служебные (`__mro__`, `__dict__` и т.п.).

Это всё — обычный Python‑метакласс: `class` компилируется как вызов `ProtocolMeta.__new__`/`__init__`, как и для любого
метакласса.

## Сбор членов протокола

Внутри `_ProtocolMeta.__new__` используется helper вроде `_get_protocol_attrs(cls)`:

- Берётся `cls.__dict__` и `__mro__` (кроме `Protocol` и `Generic`).
- Отбираются имена, у которых:
    - нет префикса `'__'` или явно разрешены (`__call__` и т.п.);
    - значение не является `typing.ClassVar`/`Final` и не помечено как приватное.
- Результат — `frozenset({'method1', 'attr2', ...})` — это интерфейс протокола.

Это множество не используется интерпретатором, а нужно только для `@runtime_checkable` и утилит
`typing_extensions.get_protocol_members`.

## runtime_checkable и __instancecheck__/__subclasscheck__

По умолчанию `Protocol` **не** должен использоваться с `isinstance`/`issubclass` — это только для статического анализа.
Декоратор `@typing.runtime_checkable` меняет поведение:

- `runtime_checkable(proto_cls)` оборачивает класс, проверяет, что это протокол (`_is_protocol=True`), и ставит ему флаг
  `_is_runtime_protocol = True`.

`_ProtocolMeta.__instancecheck__(cls, obj)` реализует:

1. Если `not getattr(cls, "_is_runtime_protocol", False)` — вызывает обычный `abc.ABCMeta.__instancecheck__` (по
   наследованию) → `False`/`TypeError`.
2. Иначе проверяет объект структурно:

    - Для каждого имени в `cls.__protocol_attrs__`:
        - через `getattr(obj, name, _marker)` проверяет наличие;
        - для методов/функций — только наличие, без точной проверки сигнатур на рантайме.

3. Если все имена есть → `True`, иначе `False`.

`_ProtocolMeta.__subclasscheck__(cls, sub)` для runtime‑протоколов делает аналогичный анализ по `dir(sub)`/MRO, но опять
же полностью в `typing.py`, без участия VM.

Сам `isinstance(x, P)` в VM вызывает `PyObject_IsInstance` → `cls.__instancecheck__`, т.е. всё поведение под контролем
`_ProtocolMeta`.

## Generic Protocol и параметры типов

`Protocol` часто комбинируется с `Generic`:

```python
class P(Protocol[T]):
    def f(self, x: T) -> T: ...
```

На уровне реализации:

- `Protocol` наследует от `Generic`, а `_ProtocolMeta` наследует от `typing._GenericAlias`‑совместимого метакласса,
  поэтому `P[int]` создаёт `typing._GenericAlias(P, (int,))`.
- При `P[int]` вызывается `Protocol.__class_getitem__`, унаследованный от `Generic`, который создаёт alias‑объект и не
  меняет поведение рантайм‑проверок; параметры типов хранятся в `__args__`/`__origin__` alias’а.

Для рантайма это всего лишь ещё один generic alias, без логики в байткоде.

## Ограничения на конструктор и наследование

`_ProtocolMeta.__call__` *не* запрещает инстанцирование протоколов (в отличие от `ABCMeta`) — протоколы используются как
обычные классы, но для статического анализа важен только их интерфейс.

Однако `typing` накладывает ограничения при наследовании/параметризации:

- В `Protocol.__mro__` не допускаются конфликтующие базы (миксы с обычными классами, не основанными на Protocol) —
  `typing` проверяет это в `__init_subclass__`/`_check_protocol`.
- При субскрипции (`Protocol[...]`) `_ProtocolMeta.__getitem__` проверяет, что все параметры — `TypeVar` (для объявления
  generic‑протокола), а не конкретные типы.

Эти проверки — обычные Python‑вычисления в `typing.py`, без участия VM.

## typing_extensions.Protocol

В старых версиях CPython большинство логики протоколов жило в `typing_extensions.Protocol`, а позже мигрировало в stdlib
`typing.Protocol`.

- `typing_extensions.Protocol` имеет свой metaclass `_ProtocolMeta` с почти идентичным кодом (иногда даже бэкпортит
  патчи из будущих версий CPython, например поведение `__init__` в 3.11).
- Реальный рантайм‑поведение зависит от того, какой именно класс стоит в MRO (stdlib vs extensions); при смешивании
  обоих в MRO возможны несостыковки, но это всё решается в чистом Python‑коде, без изменений интерпретатора.

***

Итого, `Protocol` в CPython — это обычные Python‑классы с метаклассом `_ProtocolMeta`, который при создании класса
собирает множество имён членов и, при `@runtime_checkable`, реализует структурные `isinstance`/`issubclass` через
переопределение `__instancecheck__`/`__subclasscheck__`. Никакого спецбайткода, дополнительных флагов в `PyTypeObject`
или логики в `ceval.c` под протоколы нет.

- [Содержание](CONTENTS.md#содержание)

---

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

- [Содержание](CONTENTS.md#содержание)

---

# **Наследование vs композиция**

## **Junior Level**

Наследование и композиция — это два фундаментальных подхода к организации кода в объектно-ориентированном
программировании, и выбор между ними определяет, насколько гибкой, понятной и удобной для поддержки будет ваша система.

Основное различие лежит в типе отношений, которые они моделируют:

* **Наследование** описывает отношение **«является»** (is-a). Например, `Dog` (Собака) наследует от `Animal` (Животное),
  потому что собака *является* конкретным видом животного. Это позволяет `Dog` автоматически получить общие для всех
  животных свойства и методы (например, `eat()` или `sleep()`), а также добавить свои уникальные (`bark()`).
* **Композиция** описывает отношение **«имеет»** (has-a). Класс `Car` (Автомобиль) не является двигателем, но он *имеет*
  его как составную часть. Двигатель — это независимый компонент, с которым автомобиль взаимодействует через четко
  определенный интерфейс.

**Ключевое правило современной разработки: предпочитайте композицию наследованию.** Наследование создает жесткую,
статическую связь, подобную родственной. Изменения в «родительском» классе могут неожиданно «сломать» всех его
«потомков». Композиция же строит более гибкие, договорные отношения: вы можете заменить «двигатель» на другой, не
переделывая всю «машину». Для тестирования это преимущество критично — компоненты, переданные через композицию, легко
подменить на заглушки (mocks), что позволяет тестировать классы изолированно.

## **Middle Level**

### **Детали и тонкости наследования в Python**

1. **Механизм и порядок разрешения методов (MRO)**: Python поддерживает множественное наследование. Чтобы управлять
   потенциальным хаосом при поиске методов, используется строгий **MRO (Method Resolution Order)**, вычисляемый по
   алгоритму C3 (`ClassName.__mro__`). Этот порядок гарантирует, что каждый класс в иерархии будет проверен только один
   раз и предсказуемо.

2. **Инструмент `super()` и кооперативное наследование**: `super()` — это не просто вызов метода родителя. В условиях
   MRO он делегирует выполнение **следующему классу в цепочке**. Это позволяет нескольким классам-предкам (например,
   `Mixin`-ам) кооперативно участвовать в выполнении одного метода (например, `__init__`), не мешая друг другу.

3. **Наследование как потенциальное нарушение инкапсуляции**: Это ключевая критика наследования. Дочерний класс получает
   доступ к защищённым (`_protected`) членам родителя, что создает хрупкую, скрытую зависимость от его внутренней
   реализации. Тестировать такой класс сложно, так как для понимания его поведения необходимо глубоко знать детали
   работы родителя.

4. **Множественное наследование и Mixins**: Python разрешает множественное наследование, что часто используется для *
   *Mixins** — небольших классов, добавляющих конкретную функциональность (например, `JSONSerializableMixin`). Mixin не
   предназначен для использования отдельно. При их применении важно проектировать имена методов так, чтобы избежать
   конфликтов, разрешаемых через MRO.

5. **Абстрактные классы (ABC)**: Модуль `abc` позволяет создавать формальные «чертежи» — классы, объявляющие
   обязательные методы (через `@abstractmethod`). Они заставляют наследников соблюдать контракт, явно формализуя
   отношение «является».

### **Детали и тонкости композиции в Python**

1. **Композиция vs Агрегация: управление жизненным циклом**:
    * **Композиция (сильная связь)**: Компонент (например, `Heart` для `Human`) не существует отдельно. Он создаётся и
      уничтожается вместе с объектом-владельцем (обычно в `__init__`).
    * **Агрегация (слабая связь)**: Компонент (например, `Driver` для `Car`) существует независимо и передаётся объекту
      извне (как аргумент). Их жизненные циклы разделены.

2. **Композиция и структурная типизация (Протоколы)**: Вместо жесткого наследования от абстрактного класса для
   достижения полиморфизма в Python всё чаще используют **композицию с протоколами** (`typing.Protocol`). Объекту не
   нужно объявлять «я наследник `Reader`» — достаточно просто реализовать метод `read()`. Это резко снижает связанность.
   В тестах можно подставить любой объект с нужным методом, не строя сложных иерархий.

3. **Делегирование как явная форма композиции**: Паттерн **Делегирование** — это когда внешний объект (делегатор) явно
   передает выполнение задачи внутреннему объекту (делегату). В Python его можно элегантно реализовать через
   `__getattr__`, автоматически перенаправляя вызовы. Это основа для объеков-обёрток (адаптеров, прокси), которые дают
   полный контроль над взаимодействием и являются идеальной точкой для внедрения моков в тестах.

4. **Динамическое поведение**: Композиция позволяет менять поведение объекта во время выполнения программы, заменяя его
   компоненты. Этот принцип лежит в основе многих паттернов, таких как **Стратегия** (Strategy), где алгоритм можно
   «подменить на лету».

### **Сравнительный анализ для проектирования и тестирования**

* **Гибкость и связность**: Наследование фиксирует отношения на этапе компиляции, приводя к высокой связности.
  Композиция/агрегация определяют поведение во время выполнения, обеспечивая слабую связность и большую гибкость.

* **Тестируемость**: Класс, построенный на композиции, легко тестировать изолированно. Его зависимости — это просто
  аргументы, которые можно подменить. Тестирование глубокой иерархии наследования требует создания сложных фикстур и
  мокирования родительских методов, что увеличивает сложность тестов.

* **Проблема хрупкого базового класса**: Это главный риск наследования. Даже безопасное на вид изменение во внутренней
  логике родителя может сломать работу непредусмотревшего этого наследника. В больших проектах отследить такие побочные
  эффекты крайне трудно.

* **Борьба со сложностью архитектуры**: Глубокие иерархии наследования имеют тенденцию разрастаться и усложняться.
  Композиция предлагает альтернативную парадигму: строить сложную систему не через вертикальное ветвление, а через
  горизонтальную сборку из небольших, независимых и легко заменяемых компонентов. Это прямой путь к более
  поддерживаемому и надежному коду.

## **Senior Level**

## Базовая структура объектов CPython

В CPython все объекты наследуют от базовой структуры `PyObject`, которая имитирует наследование через композицию в C.
Это видно в заголовочном файле `Include/object.h`.

```c
// Include/object.h: базовая структура всех Python объектов
#define PyObject_HEAD                   PyObject ob_base;

struct _object {
    // Счетчик ссылок (может быть разбит на части для 64-битных систем)
    union {
#if SIZEOF_VOID_P > 4
        PY_INT64_T ob_refcnt_full;
        struct {
# if PY_BIG_ENDIAN
            uint16_t ob_flags;     // Флаги объекта
            uint16_t ob_overflow;  // Переполнение refcnt
            uint32_t ob_refcnt;    // Основной счетчик ссылок
# else
            uint32_t ob_refcnt;    // Основной счетчик ссылок
            uint16_t ob_overflow;  // Переполнение refcnt
            uint16_t ob_flags;     // Флаги объекта
# endif
        };
#else
        Py_ssize_t ob_refcnt;          // Простой счетчик ссылок
#endif
    };
    PyTypeObject *ob_type;             // Указатель на тип объекта
};
```

Каждый Python-объект в памяти начинается с "шапки" — счетчика ссылок (сколько мест держит ссылку
на объект) и указателя на его тип. Это как паспорт объекта: "кто я и сколько на меня ссылок". Любая конкретная
структура (число, строка, список) начинается с этой шапки, а потом идут свои поля. Благодаря этому любой указатель на
объект можно безопасно привести к `PyObject*` — первые байты всегда одинаковые.

## PyTypeObject — сердце наследования

Типы в CPython — это объекты `PyTypeObject`, которые содержат слоты (vtable) для методов. Наследование — это копирование
и переопределение этих слотов.

```c
// Objects/typeobject.c: фрагмент инициализации типа
typedef struct {
    int slot;              // Номер слота (tp_new, tp_call и т.д.)
    void *pfunc;           // Указатель на C-функцию
} PyType_Slot;

// PyTypeObject содержит сотни таких слотов
// Ключевые для наследования:
PyTypeObject {
    PyObject_HEAD           // Наследует от PyObject
    Py_ssize_t tp_basicsize; // Размер базовой части объекта
    Py_ssize_t tp_itemsize;  // Размер для VarObject
    unsigned long tp_flags;  // Флаги типа
    PyObject *tp_bases;      // Кортеж базовых классов (!!!)
    PyObject *tp_base;       // Прямой базовый класс
    PyObject *tp_dict;       // Словарь атрибутов типа
    PyObject *tp_mro;        // Method Resolution Order
    // ... сотни слотов методов: tp_new, tp_init, tp_call ...
};
```

`PyTypeObject` — это "рецепт" для создания объектов определенного типа. Он хранит список базовых
классов (`tp_bases`) и порядок поиска методов (`tp_mro`). Когда создается новый класс, CPython копирует слоты из
родителей, разрешает конфликты по MRO и создает новый тип. Это не C++ наследование — это ручное копирование таблиц
методов.

## Инициализация наследования в PyType_Ready()

Функция `PyType_Ready()` вычисляет MRO, наследует слоты и подготавливает тип. Вот ключевой фрагмент.

```c
// Objects/typeobject.c: упрощенный фрагмент PyType_Ready()
static int
type_ready(PyTypeObject *type) {
    // 1. Берем базовые классы
    PyObject *bases = lookup_tp_bases(type);  // Кортеж родителей
    
    // 2. Вычисляем MRO (порядок разрешения методов)
    if (type->tp_mro == NULL) {
        type->tp_mro = mro_internal(type);    // C3-линеаризация
        if (type->tp_mro == NULL) {
            return -1;
        }
    }
    
    // 3. Наследуем слоты методов от родителей
    for (Py_ssize_t i = 0; i < PyTuple_GET_SIZE(bases); i++) {
        PyTypeObject *base = PyTuple_GET_ITEM(bases, i);
        inherit_slots(type, base);            // Копируем слоты
    }
    
    // 4. Разрешаем конфликты по MRO
    resolve_slots(type);
    
    // 5. Устанавливаем флаги готовности
    type_add_flags(type, Py_TPFLAGS_READY);
    return 0;
}
```

При создании класса `PyType_Ready()` делает три вещи: 1) строит MRO (очередь "от кого наследовать
методы"), 2) копирует методы из родителей в свою таблицу слотов, 3) разрешает конфликты (если метод есть у нескольких
родителей — берет по приоритету MRO). После этого тип "готов" и объекты можно создавать.

## Композиция через указатели на типы

Композиция в CPython — это когда объект содержит указатели на другие типы, а не наследует их структуры. Пример:
`PyFloatObject`.

```c
// Objects/floatobject.c: Float содержит PyObject_HEAD + свои поля
typedef struct {
    PyObject_HEAD                // Наследование "слева"
    double ob_fval;              // Свое поле: значение double
} PyFloatObject;

// Использование: любой PyFloat* можно привести к PyObject*
PyFloatObject *f = ...;
PyObject *obj = (PyObject*)f;      // Безопасно!
PyTypeObject *type = obj->ob_type; // Получаем тип float
```

Композиция — это "имеет-a": float "имеет" PyObject в начале + double. Наследование — "является-a":
любой float "является" PyObject. В памяти это одно и то же — первые байты всегда PyObject. Разница в семантике:
наследование дает слоты методов автоматически, композиция — требует явных вызовов.

## MRO (Method Resolution Order) — C3 алгоритм

MRO решает, чей метод вызывать при множественном наследовании. Вычисляется рекурсивно.

```c
// Objects/typeobject.c: упрощенный mro_internal()
static PyObject *
mro_internal(PyTypeObject *type) {
    PyObject *bases = type->tp_bases;     // Список родителей
    PyObject *seqs = PyTuple_New(1 + PyTuple_GET_SIZE(bases));
    
    // Собираем MRO всех родителей
    for (Py_ssize_t i = 0; i < PyTuple_GET_SIZE(bases); i++) {
        PyTypeObject *base = PyTuple_GET_ITEM(bases, i);
        PyObject *parent_mro = lookup_tp_mro(base);
        PyTuple_SET_ITEM(seqs, i+1, Py_NewRef(parent_mro));
    }
    
    // C3-линеаризация: решает порядок с учетом приоритетов
    PyObject *mro = merge_mros(seqs);     // Алгоритм C3
    return mro;
}
```

MRO — это "очередь родителей": сначала сам класс, потом родители слева направо, исключая уже
использованных. Алгоритм C3 гарантирует, что родители сохраняют свой порядок. При вызове метода CPython идет по этой
очереди до первого совпадения: `type->tp_call`, `base1->tp_call`, `base2->tp_call`....

## __slots__ и ограничения множественного наследования

`__slots__` конфликтует с наследованием из-за фиксированных смещений в памяти.

```c
// Objects/typeobject.c: проверка слотов при наследовании
static int
check_slots(PyTypeObject *type) {
    // Если несколько родителей с __slots__, layouts конфликтуют
    for (Py_ssize_t i = 0; i < PyTuple_GET_SIZE(type->tp_bases); i++) {
        PyTypeObject *base = PyTuple_GET_ITEM(type->tp_bases, i);
        if (base->tp_dictoffset && type->tp_dictoffset &&
            base->tp_dictoffset != type->tp_dictoffset) {
            PyErr_SetString(PyExc_TypeError,
                "multiple parents with conflicting __slots__");
            return -1;
        }
    }
    return 0;
}
```

`__slots__` фиксирует места атрибутов в памяти (экономит память, убирает `__dict__`). При
наследовании смещения должны совпадать, иначе дескрипторы слотов "смотрят не туда". Композиция решает проблему — просто
держишь объект внутри без наследования структур.

## Кэш атрибутов и версии типов

CPython кэширует поиск атрибутов через `tp_version_tag`. Изменение базовых классов инвалидирует кэш рекурсивно.

```c
// Objects/typeobject.c: инвалидация при изменении иерархии
void PyType_Modified(PyTypeObject *type) {
    BEGIN_TYPE_LOCK();                    // Глобальная блокировка типов
    type_modified_unlocked(type);         // Рекурсивно по подклассам
    END_TYPE_LOCK();
}

static void type_modified_unlocked(PyTypeObject *type) {
    // Сбрасываем версию кэша
    set_version_unlocked(type, 0);
    
    // Рекурсивно инвалидируем подклассы
    PyObject *subclasses = lookup_tp_subclasses(type);
    for each subclass in subclasses {
        type_modified_unlocked(subclass);
    }
}
```

Каждый тип имеет "версию". При поиске `obj.x` CPython проверяет кэш по версии типа. Изменение
родителей → смена версии → инвалидация кэша для всего поддерева. Это дорого, поэтому наследование иерархий в runtime —
редкость.

## Итоговое сравнение под капотом

| Аспект        | Наследование                  | Композиция                          |
|---------------|-------------------------------|-------------------------------------|
| **Память**    | PyObject_HEAD + поля          | PyObject_HEAD + указатель на объект |
| **Методы**    | Автокопирование слотов по MRO | Ручной вызов `self.child.method()`  |
| **Атрибуты**  | Поиск по MRO, кэш с версиями  | Прямой доступ через указатель       |
| **Slots**     | Конфликты layouts             | Нет проблем                         |
| **Изменение** | Инвалидация всего поддерева   | Локальное                           |

Наследование быстрее для статичных иерархий (автоматическое копирование слотов), композиция гибче (без конфликтов
слотов, локальные изменения).

- [Содержание](CONTENTS.md#содержание)

---

# **Композиция и агрегация**

## **Junior Level*

Композиция и агрегация — это два способа создания отношений между объектами в объектно-ориентированном программировании.
Обе описывают ситуацию, когда один объект содержит в себе другой, но с критически важным различием в силе связи и
управлении жизненным циклом.

Представьте, что вы строите дом. **Композиция** — это как комната в доме. Комната не существует отдельно от дома. Когда
дом сносят, комната исчезает вместе с ним. Объект-владелец (дом) полностью контролирует жизнь объекта-части (комнаты). *
*Агрегация** — это как мебель в доме. Стол, стул, диван существуют независимо от дома. Их занесли в дом, а потом могут
вынести в другой дом или на склад. Объект-владелец (дом) использует объект-часть (мебель), но не управляет его рождением
и смертью.

В разработке композиция означает, что при уничтожении основного объекта уничтожаются и все его составные части.
Агрегация означает, что объекты собраны вместе, но могут жить самостоятельно. Для QA инженера понимание этого различия
помогает проектировать тестовые фикстуры и моки, правильно управлять их жизненным циклом и понимать, какие зависимости
нужно создавать заново, а какие можно переиспользовать между тестами.

## **Middle Level**

С технической точки зрения, композиция и агрегация реализуются через **атрибуты класса**, но с разной семантикой
создания и владения.

1. **Реализация:**
    * **Композиция (Composition):** Объект-часть создается **внутри** конструктора (или иного метода) объекта-владельца.
      Владелец полностью инкапсулирует создание и, как правило, не предоставляет публичных методов для замены этой
      части. Часть реализуется как внутренний, приватный атрибут.
    * **Агрегация (Aggregation):** Объект-часть создается **вне** объекта-владельца и передается ему в качестве
      аргумента (чаще всего в конструктор). Владелец сохраняет ссылку на эту часть, но не управляет ее созданием.
      Объект-часть может быть общим (разделяемым) ресурсом.

2. **Жизненный цикл:**
    * При **композиции** жизненный цикл части жестко привязан к жизненному циклу целого. Когда объект-владелец
      удаляется (например, сборщиком мусора), удаляется и объект-часть, если на него больше нет ссылок.
    * При **агрегации** жизненные циклы независимы. Удаление владельца не влечет удаление части, так как на нее могут
      оставаться ссылки из других объектов.

3. **Для AQA:**
    * **Фикстуры в Pytest:** Композиция часто используется для создания сложных, вложенных фикстур, которые существуют
      только в рамках одной тестовой сессии или модуля и автоматически очищаются. Агрегация похожа на фикстуры с
      областью видимости `session` или `package`, которые создаются один раз и переиспользуются многими тестами.
    * **Тестовые данные:** Понимание, когда создавать новый экземпляр тестовых данных для каждого кейса (композиция), а
      когда использовать общий, предсозданный набор данных (агрегация), критично для скорости и изоляции тестов.
    * **Page Object:** Внутри Page Object может существовать композиция из элементов (например, `Button`, `InputField`),
      которые не имеют смысла вне контекста этой страницы. И агрегация — например, общий `Header` или `Footer`, которые
      могут быть переданы в несколько Page Object.

4. **Отличия от наследования:** И композиция, и агрегация — это альтернативы наследованию, предпочитаемые в современном
   дизайне ("предпочитай композицию наследованию"). Они обеспечивают большую гибкость и слабую связанность.

## **Senior Level**

## Базовые структуры композиции в CPython

Композиция в CPython реализуется через **встраивание структур** (embedding) — `PyObject_HEAD` + поля + другие
`PyObject`. Агрегация — через **указатели** на объекты. Разница в управлении памятью и временем жизни.

```c
// Include/object.h: базовые макросы для композиции
#define PyObject_HEAD                   PyObject ob_base;  // Шапка: refcnt + type
#define PyObject_VAR_HEAD               PyVarObject ob_base; // Шапка + размер

struct _object {                       // PyObject — минимальный объект
    union {                            // Счетчик ссылок (refcnt)
#if SIZEOF_VOID_P > 4
        PY_INT64_T ob_refcnt_full;     // 64-битный refcnt
        struct {
# if PY_BIG_ENDIAN
            uint16_t ob_flags;         // Флаги объекта
            uint16_t ob_overflow;      // Переполнение refcnt
            uint32_t ob_refcnt;        // Основной refcnt
# else
            uint32_t ob_refcnt;        // Основной refcnt
            uint16_t ob_overflow;      // Переполнение refcnt
            uint16_t ob_flags;         // Флаги объекта
# endif
        };
#else
        Py_ssize_t ob_refcnt;          // 32-битный refcnt
#endif
    };
    PyTypeObject *ob_type;             // Указатель на тип (vtable)
};
```

`PyObject_HEAD` — это **обязательная "шапка"** в начале **каждого** Python-объекта (8-16 байт). Она содержит: 1) *
*refcnt** (сколько ссылок держит объект живым), 2) **ob_type** (указатель на таблицу методов типа). Любая структура типа
начинается с этой шапки. Композиция = шапка + свои поля + другие шапки.

## Пример композиции: PyTupleObject (строгая композиция)

Кортеж содержит **встроенный массив PyObject*** — классическая композиция "имеет-A".

```c
// Include/tupleobject.h + Objects/tupleobject.c
typedef struct {
    PyObject_VAR_HEAD                // Шапка PyObject + ob_size (кол-во элементов)
    PyObject *ob_item[1];            // Массив объектов (гибкий размер!)
} PyTupleObject;

// Реальная структура в памяти: PyObject_HEAD + Py_ssize_t ob_size + PyObject** об_item
// Размер выделяется: sizeof(PyTupleObject) + (size-1)*sizeof(PyObject*)
```

Кортеж `(1, "a", [])` в памяти — **один большой блок**: шапка кортежа + число элементов (3) + **3 указателя** на объекты
1, "a", []. Это **композиция**: кортеж **владеет** массивом указателей. Когда refcnt кортежа → 0, **все указатели
остаются жить** (их refcnt не трогаем).

## Пример агрегации: PyDictObject (слабая композиция)

Словарь содержит **указатели** на хэш-таблицу PyDictKeysObject — агрегация.

```c
// Objects/dictobject.c: PyDictObject (Python 3.9+)
typedef struct {
    PyObject_HEAD                    // Шапка словаря
    Py_ssize_t ma_used;              // Кол-во используемых слотов
    PyDictKeysObject *ma_keys;       // УКАЗАТЕЛЬ на отдельный объект ключей!!!
    PyObject **ma_values;            // Указатель на массив значений (опционально)
} PyDictObject;

typedef struct {
    Py_ssize_t dk_size;              // Размер хэш-таблицы
    PyDictUnicodeEntry *dk_entries;  // Массив записей (ключ+хэш)
    vectorcallfunc vectorcall;       // Методы для vectorcall
} PyDictKeysObject;
```

Словарь `{"a": 1}` — **два объекта**: 1) PyDictObject (шапка + указатель на ключи), 2) PyDictKeysObject (отдельная
хэш-таблица). Это **агрегация**: словарь **ссылается** на таблицу ключей, но **не владеет** ею. Таблица ключей может
использоваться **несколькими** словарями (shared keys optimization).

## Создание составных объектов: PyTuple_New()

Композиция создается **атомарно** — выделяется память под всю структуру сразу.

```c
// Objects/tupleobject.c: PyTuple_New()
PyObject *
PyTuple_New(Py_ssize_t size) {
    PyTupleObject *op;                 // Указатель на новый кортеж
    Py_ssize_t nbytes;                 // Общий размер в байтах
    
    if (size < 0) {                    // Проверка отрицательного размера
        PyErr_BadInternalCall();
        return NULL;
    }
    
    // Вычисляем размер: шапка + (size-1)*sizeof(PyObject*)
    nbytes = size * sizeof(PyObject *) + sizeof(PyTupleObject) - sizeof(PyObject *);
    
    // Выделяем память атомарно
    op = PyObject_GC_NewVar(PyTupleObject, &PyTuple_Type, size);
    if (op == NULL)                    // Ошибка выделения
        return NULL;
        
    // Инициализируем все указатели NULL (zero-filling)
    for (Py_ssize_t i = 0; i < size; i++)
        op->ob_item[i] = NULL;         // Каждый слот = NULL
    
    PyObject_GC_Track(op);             // Регистрируем в GC
    return (PyObject *) op;
}
```

`tuple(1,2,3)` → CPython **одним malloc()** выделяет **весь блок** (шапка + 3 указателя). Потом заполняет указатели
PyLong(1), PyLong(2), PyLong(3). **Композиция = единый блок памяти**. Когда refcnt → 0, **один free()** освобождает
всё.

## Разница в деструкторах: tp_dealloc

Композиция освобождает **свои** поля, агрегация — **НЕ трогает** подчиненные объекты.

```c
// Objects/tupleobject.c: tp_dealloc для PyTuple_Type
static void
tuple_dealloc(PyTupleObject *op) {
    Py_ssize_t len = Py_SIZE(op);      // Длина кортежа
    PyObject **items = op->ob_item;    // Указатель на массив
    
    PyObject_GC_UnTrack(op);           // Убираем из GC
    Py_TRASHCAN_SAFE_BEGIN(op)         // Защита от рекурсии
    
    // НЕ освобождаем ob_item[i]! Это агрегированные объекты
    // Просто обнуляем указатели (для отладки)
    while (--len >= 0) {
        Py_CLEAR(items[len]);          // Снижаем refcnt элементов
    }
    
    Py_TYPE(op)->tp_free((PyObject*)op); // free() всей структуры
    Py_TRASHCAN_SAFE_END(op)
}
```

Кортеж умирает → **НЕ трогает** содержимое (1,2,3 живут дальше), только **снижает их refcnt** и **освобождает свой блок
**. Словарь при смерти **НЕ трогает** ma_keys (агрегация). **Композиция** бы трогала встроенные объекты.

## __slots__ как экстремальная композиция

`__slots__` создает **фиксированную композицию** без `__dict__` — экономит память.

```c
// Objects/typeobject.c: обработка __slots__ в PyType_Ready()
static int
slotptr_cmp(PyObject *slot1, PyObject *slot2) {
    // Сортируем слоты по алфавиту для детерминизма
    return PyUnicode_Compare(slot1, slot2);
}

static PyObject *
collect_slots(PyTypeObject *type) {
    PyObject *slots = PyObject_GetAttrString((PyObject*)type, "__slots__");
    if (!slots) return NULL;
    
    // Сортируем и вычисляем смещения атрибутов
    Py_ssize_t nslots = PyList_GET_SIZE(slots);
    for (Py_ssize_t i = 0; i < nslots; i++) {
        PyObject *name = PyList_GET_ITEM(slots, i);
        PyMemberDef *member = create_member(name);  // Создаем дескриптор
        // member->offset = смещение в памяти экземпляра
    }
    
    type->tp_basicsize += nslots * sizeof(PyObject*); // Фиксируем размер
    return slots;
}
```

`__slots__ = ['x', 'y']` → CPython создает **фиксированные поля** сразу после PyObject_HEAD:
`[PyObject_HEAD | PyObject* x | PyObject* y]`. **Нет `__dict__`**, нет хэш-таблицы. **Чистая композиция** с известным
layout'ом памяти.

## Сравнение композиции и агрегации под капотом

| Аспект          | Композиция (встраивание)   | Агрегация (указатели)            |
|-----------------|----------------------------|----------------------------------|
| **Память**      | Единый malloc/free         | Несколько malloc (dict + keys)   |
| **Время жизни** | Владелец управляет всем    | Подчиненные живут независимо     |
| **Размер**      | sizeof(HEAD) + поля + HEAD | sizeof(HEAD) + sizeof(PyObject*) |
| **GC**          | Рекурсивно все поля        | Только указатели (не владеет)    |
| **Shared**      | Невозможно                 | Возможно (dict keys)             |

**Композиция** = "встроить структуру целиком", **агрегация** = "указатель на чужой объект". В CPython **tuple =
композиция** (встроенный массив), **dict = агрегация** (отдельные keys/values).

- [Содержание](CONTENTS.md#содержание)

---

# **Связность и связанность**

## **Junior Level*

Связность и связанность — это две фундаментальные концепции качества кода, которые описывают, насколько хорошо
организованы компоненты внутри модуля и насколько сильно они зависят друг от друга.

**Связность (Cohesion)** отвечает на вопрос: "Насколько хорошо элементы внутри одного модуля (класса, функции) связаны
между собой и работают для достижения одной четкой цели?". Высокая связность — это хорошо. Это означает, что модуль
делает что-то одно, целостное и понятное. Например, модуль `MathUtils`, который содержит только функции для
математических вычислений, обладает высокой связностью. А модуль `Utils`, который смешивает функции для работы со
строками, отправки email и логирования, имеет низкую связность — это "мусорная корзина", которую трудно поддерживать.

**Связанность (Coupling)** отвечает на вопрос: "Насколько сильно один модуль зависит от внутреннего устройства другого
модуля?". Низкая связанность — это хорошо. Это означает, что модули взаимодействуют через четкие, простые интерфейсы и
могут изменяться независимо друг от друга. Если же модуль "залезает" в приватные детали другого модуля, то любое
изменение в одном сломает другой — это высокая связанность (сильная связь).

Простой принцип: нужно стремиться к **высокой связности и низкой связанности**. Для QA инженера это напрямую влияет на
тестируемость: модули с высокой связностью легко тестировать изолированно, а при низкой связанности можно легко заменять
зависимости на моки.

## **Middle Level**

С технической точки зрения эти концепции реализуются через конкретные практики проектирования и имеют четкие индикаторы.

1. **Типы связности (от худшего к лучшему):**
    * **Случайная (Coincidental):** Элементы собраны вместе случайно (например, модуль `MiscHelpers`). Тестировать такое
      невозможно — нет единой ответственности.
    * **Логическая (Logical):** Элементы сгруппированы по категории (например, класс `DataProcessor`, который
      обрабатывает и CSV, и JSON, и XML). Тесты становятся размазанными и хрупкими.
    * **Временная (Temporal):** Элементы выполняются в одно время (например, функция `initialize_all()`, которая
      настраивает логгер, БД и кеш). Приводит к сложным фикстурам и непредсказуемым побочным эффектам в тестах.
    * **Процедурная (Procedural):** Элементы объединены последовательностью шагов (например, функция, которая читает
      файл, парсит данные и сохраняет в БД). В тестах приходится эмулировать всю последовательность.
    * **Коммуникационная (Communicational):** Элементы работают с одними и теми же данными (например, класс
      `CustomerReport`, который и вычисляет статистику, и форматирует отчет). Уже лучше, но еще есть смесь
      ответственностей.
    * **Последовательная (Sequential):** Выход одного элемента является входом для другого (конвейер). Хорошо для
      тестирования каждого шага.
    * **Функциональная (Functional):** Все элементы вносят вклад в выполнение одной четкой задачи — **идеал**. Класс
      `InvoiceCalculator` только считает, `InvoiceFormatter` только форматирует. Юнит-тесты пишутся легко и
      изолированно.

2. **Типы связанности (от худшего к лучшему):**
    * **Содержание (Content):** Модуль напрямую обращается к приватным данным или коду другого модуля (нарушение
      инкапсуляции). Тесты становятся хрупкими к любым изменениям.
    * **Общая (Common):** Модули используют глобальные данные. Для тестов это катастрофа — состояние тестов влияет друг
      на друга, параллельный запуск невозможен.
    * **Внешняя (External):** Модули зависят от внешнего формата данных или протокола. Требует сложных интеграционных
      тестов и моков.
    * **Управление (Control):** Один модуль управляет логикой другого (например, передача флагов). Усложняет
      тестирование, так как нужно проверять множество ветвлений.
    * **Структурная (Stamp):** Модуль принимает сложную структуру данных, но использует только часть полей. Создает
      скрытые зависимости и усложняет создание тестовых данных.
    * **Данных (Data):** Модули взаимодействуют через минимальный интерфейс (например, передача примитивных значений). *
      *Идеал** для тестирования — зависимости легко заглушить.

3. **Инструменты для достижения:**
    * **Принцип единственной ответственности (SRP)** ведет к высокой связности.
    * **Инверсия зависимостей (DIP)** через абстракции (ABC, Protocol) ведет к низкой связанности.
    * **Закон Деметры ("не разговаривай с незнакомцами")** снижает связанность.

## **Senior Level**

В CPython 3.9+ **связность** (cohesion) — **монолитные типы** `PyTypeObject` с полным набором слотов (`tp_as_*`), *
*слабая связанность** (loose coupling) — **reference counting** + **generational GC** (`Modules/gcmodule.c`), *
*циклические зависимости** — `gc_refs` в цикло-детекции, **модульная связанность** — `import.c` граф
импортов. `Modules/gcmodule.c`,`Python/import.c`,`Objects/typeobject.c`

## 1. Высокая связность: монолитный PyTypeObject

```c
// Include/cpython/object.h — ВСЁ В ОДНОМ типе!
typedef struct _typeobject {
    PyObject_VAR_HEAD           // refcnt, type=type
    const char *tp_name;        // имя
    Py_ssize_t tp_basicsize;    // размер
    
    /* ЧИСЛОВЫЕ операции */
    struct PyNumberMethods *tp_as_number;
    
    /* ПОСЛЕДОВАТЕЛЬНОСТИ */
    struct PySequenceMethods *tp_as_sequence;
    
    /* СЛОВАРНЫЙ интерфейс */
    struct PyMappingMethods *tp_as_mapping;
    
    /* CALL, STR, REPR, HASH */
    ternaryfunc tp_call;
    reprfunc tp_str;
    reprfunc tp_repr;
    hashfunc tp_hash;
    
    /* АТРИБУТЫ */
    getattrofunc tp_getattro;
    setattrofunc tp_setattro;
    
    /* GC, ITERATOR */
    traverseproc tp_traverse;
    inquiry tp_clear;
    iterfunc tp_iter;
    
    /* НАСЛЕДОВАНИЕ */
    PyObject *tp_bases;
    PyTypeObject *tp_base;
    PyObject *tp_mro;
    
    /* 60+ слотов! */
} PyTypeObject;
```

**Высокая связность**! Один `PyTypeObject` содержит **ВСЕ** функции типа: арифметика + последовательности + вызов + GC +
наследование. **НЕ** разделены — **монолит**! `list` имеет `tp_as_sequence + tp_as_mapping + tp_as_number + tp_iter`.

## 2. Слабая связанность: Reference Counting (независимое удаление)

```c
// Objects/object.c — Py_DECREF независим!
Py_ssize_t _Py_RefTotal;

void Py_DECREF(PyObject *op) {
    PyObject *parent;  // НЕ НУЖЕН!
    
    // op->ob_refcnt-- (атомарно)
    Py_ssize_t refcnt = --op->ob_refcnt;
    
    if (refcnt == 0) {
        // УДАЛЯЕМ НЕЗАМЕДЛИТЕЛЬНО!
        // НЕ сообщаем никому!
        _Py_Dealloc(op);  // tp_dealloc(op)
        _Py_RefTotal--;
    }
}
```

**Слабая связанность**! При `del obj` → `Py_DECREF(obj)` → если `ob_refcnt==0` → **НЕЗАМЕДЛИТЕЛЬНО** `tp_dealloc(obj)`.
**НИКТО** не знает о существовании `obj` — **независимое уничтожение**!

## 3. Циклическая связанность: GC цикл-детекция (gcmodule.c)

```c
// Modules/gcmodule.c — цикл a→b→a
typedef struct {
    PyObject_HEAD
    uintptr_t gc_refs;     // ← временный счётчик для GC!
    char gc_state;         // GCSTATE_REACHABLE/UNREACHABLE/VISITED
    PyObject *_gc_next;    // ← в списке генерации!
    PyObject *_gc_prev;
} PyGC_Head;

void _PyGC_Collect(void) {
    // 1. MARK PHASE: gc_refs = ob_refcnt
    for (obj = gen0_head; obj; obj = obj->_gc_next) {
        obj->gc_refs = Py_REFCNT(obj);  // копируем refcnt
        obj->gc_state = GCSTATE_UNCOLLECTED;
    }
    
    // 2. TRAVERSE PHASE: вычитаем внутренние ссылки
    for (obj = gen0_head; obj; obj = obj->_gc_next) {
        if (obj->gc_refs > 0) {  // живой?
            traverse_children(obj);  // obj→child: child.gc_refs--
        }
    }
    
    // 3. SWEEP PHASE: gc_refs==0 → цикл → удаляем!
    for (obj = gen0_head; obj; obj = next) {
        if (obj->gc_refs == 0 && is_unreachable(obj)) {
            PyObject_GC_Del(obj);
        }
    }
}
```

**Циклическая связанность**! `a→b→a` → `ob_refcnt(a)=1, ob_refcnt(b)=1` (refcount **НЕ падает**). **GC** копирует
`gc_refs=ob_refcnt`, вычитает **внутренние** ссылки → `gc_refs=0` → **цикл найден** → `tp_clear()` + `tp_dealloc()`.

## 4. Генерационный GC: слабая связанность поколений

```c
// Modules/gcmodule.c — 3 поколения
struct gc_generation {
    PyGC_Head head;        // doubly-linked list
    Py_ssize_t threshold;  // порог для коллекции
    Py_ssize_t count;      // счётчик объектов
};

struct _gc_runtime_state {
    struct gc_generation generations[3];  // gen0, gen1, gen2
};

// Новые объекты → gen0
void PyObject_GC_Track(PyObject *op) {
    _PyGC_Head *gchead = &_PyGC_HEAD(op);
    generation0 = &_PyRuntime.gc.generations[0];
    link_into_list(generation0, gchead);  // gen0!
    generation0->count++;
}
```

**Слабая связанность поколений**! Новые объекты → **gen0** (collect every 700 objs). Выжившие → **gen1** (collect every
10k). Долгожители → **gen2**. **Молодые НЕ зависят** от старых!

## 5. Модульная связанность: import.c граф зависимостей

```c
// Python/import.c — обнаружение циклических импортов
typedef struct {
    PyObject *md_dict;     // module.__dict__
    unsigned int md_state; // MD_STATE_* importing/prepared etc.
    int md_weaklist;       // слабые ссылки
} PyModuleObject;

static int
set_importing_module(PyThreadState *tstate, PyObject *name, int set) {
    PyObject *modules = tstate->interp->modules;  // sys.modules
    
    PyModuleObject *module = PyDict_GetItemWithError(modules, name);
    if (!module)
        return 0;
    
    if (set) {
        if (module->md_state & MD_STATE_IMPORTING) {
            // ЦИКЛ! A→B→A
            PyErr_Format(PyExc_ImportError,
                "circular import: %R", name);
            return -1;
        }
        module->md_state |= MD_STATE_IMPORTING;
    }
    return 0;
}
```

**Циклическая модульная связанность**! `import a; import b` где `a→b`, `b→a` → `md_state |= MD_STATE_IMPORTING` → *
*ImportError**! **Графовая зависимость** отслеживается в `sys.modules`.

## 6. Связность атрибутов: __slots__ vs __dict__

```c
// Objects/typeobject.c — фиксированная структура = высокая связность
int PyType_Ready(PyTypeObject *type) {
    if (type->tp_dictoffset == 0) {  // __slots__!
        // ФИКСИРОВАННЫЕ поля: высокая связность
        type->tp_flags |= Py_TPFLAGS_HAVE_SLOTS;
    } else {
        // __dict__: динамическая связность
        type->tp_getattro = PyObject_GenericGetAttr;
    }
}
```

**`__slots__`** — **высокая связность**! Фиксированные поля `[x][y]` по offset. **`__dict__`** — **слабая связность**!
Динамический поиск в хеш-таблице.

## 7. Связность MRO: C3-линеаризация (строгий порядок)

```c
// Objects/typeobject.c — жёсткая последовательность
static int mro_internal(PyTypeObject *type) {
    PyObject *mro = mro_implementation(type);  // C3!
    
    // МОНОТОННАЯ СВЯЗННОСТЬ: child ДО parents
    if (!mro_is_linearized(mro)) {
        PyErr_SetString(PyExc_TypeError, "non-monotonic MRO");
        return -1;
    }
    
    type->tp_mro = mro;  // фиксированный порядок поиска!
}
```

**Жёсткая связность MRO**! `(D,B,C,A)` — **строгий порядок**. `D.method()` → **ТОЛЬКО** первый найденный в `tp_mro`. *
*НЕ** параллельный поиск!

## 8. Связность слотов: наследование tp_* (Objects/typeobject.c)

```c
// Наследование слотов = сильная связанность
static void inherit_slots(PyTypeObject *type) {
    PyTypeObject *base = type->tp_base;
    
    // КОПИРУЕМ указатели! Сильная связанность
    if (!type->tp_as_number) type->tp_as_number = base->tp_as_number;
    if (!type->tp_as_sequence) type->tp_as_sequence = base->tp_as_sequence;
    if (!type->tp_call) type->tp_call = base->tp_call;
    // ...
}
```

**Сильная связанность**! `MyList(list)` → `MyList.tp_as_sequence = list.tp_as_sequence` (**тот же указатель**!).
`len(MyList())` → **та же C-функция** `list_length()`!

## 9. Байткод и связность: LOAD_ATTR фиксированный путь

```python
class A: x = 1


class B(A): pass


b = B()
b.x
```

```
# Байткод:
  6 LOAD_FAST           0 (b)
  8 LOAD_ATTR           0 (x)     # ← фиксированный путь по MRO!
```

В `ceval.c`:

```c
case LOAD_ATTR: {
    PyObject *name = GETITEM(names, oparg);  // "x"
    PyObject *owner = POP();                 // b
    
    // ФИКСИРОВАННЫЙ путь: _PyType_Lookup(B, "x") → A.x
    PyObject *res = PyObject_GetAttr(owner, name);
    PUSH(res);
}
```

**Статическая связность**! `LOAD_ATTR 0 (x)` — **константа** в байткоде. **НЕ** динамический поиск — **фиксированный
offset** в `names` массиве!

## Итог связности/связанности в CPython 3.9+

| Тип связности          | Реализация                | Характеристика                       |
|------------------------|---------------------------|--------------------------------------|
| **Высокая (cohesion)** | `PyTypeObject` монолит    | 60+ слотов в одном объекте [1]       |
| **Слабая (loose)**     | `Py_DECREF` независимо    | refcnt==0 → немедленное удаление [2] |
| **Циклическая**        | `gc_refs` в GC            | `a→b→a` → цикл-детекция [3]          |
| **Модульная**          | `MD_STATE_IMPORTING`      | `import a→b→a` → ImportError [4]     |
| **Структурная**        | `__slots__` vs `__dict__` | фиксированные поля vs хеш [5]        |
| **Наследственная**     | `tp_mro` C3               | монотонный порядок [5]               |
| **Слотовая**           | `inherit_slots()`         | копирование указателей [5]           |

**CPython** балансирует: **высокая связность внутри типов**, **слабая между объектами**, **циклы** — GC, **модули** —
import-граф!

- [Содержание](CONTENTS.md#содержание)

---

# **SOLID**

## **Junior Level*

SOLID — это набор из пяти ключевых принципов проектирования в объектно-ориентированном программировании, которые
помогают создавать гибкий, поддерживаемый и расширяемый код. Каждая буква акронима представляет отдельный принцип:

**S - Single Responsibility Principle (Принцип единственной ответственности):** Каждый класс или модуль должен иметь
только одну причину для изменения. Он должен отвечать за одну конкретную задачу или ответственность.

**O - Open/Closed Principle (Принцип открытости/закрытости):** Классы должны быть открыты для расширения, но закрыты для
модификации. Это означает, что мы можем добавлять новое поведение, не меняя существующий код.

**L - Liskov Substitution Principle (Принцип подстановки Барбары Лисков):** Объекты в программе должны быть заменяемы на
экземпляры их подтипов без изменения корректности программы. Проще говоря: если что-то работает с родительским классом,
это должно работать и с любым его наследником.

**I - Interface Segregation Principle (Принцип разделения интерфейса):** Лучше иметь много специализированных
интерфейсов, чем один универсальный. Клиенты не должны зависеть от методов, которые они не используют.

**D - Dependency Inversion Principle (Принцип инверсии зависимостей):** Модули верхнего уровня не должны зависеть от
модулей нижнего уровня. Оба должны зависеть от абстракций. Абстракции не должны зависеть от деталей — детали должны
зависеть от абстракций.

Для QA инженера понимание SOLID критично, потому что код, написанный по этим принципам, гораздо легче тестировать: он
лучше изолирован, зависимости явные и заменяемые, а поведение предсказуемо.

## **Middle Level**

С технической точки зрения, каждый принцип SOLID реализуется через конкретные паттерны и механизмы Python.

1. **SRP (Single Responsibility):** На уровне модуля (файла .py) это означает, что модуль должен экспортировать
   логически связанный набор функций/классов. На уровне класса — класс должен иметь минимальное количество публичных
   методов, связанных одной целью. Нарушение SRP приводит к God-объектам, которые невозможно адекватно покрыть
   юнит-тестами. Метрика: если вы не можете назвать ответственность класса одним коротким предложением без союза "и" —
   принцип нарушен.

2. **OCP (Open/Closed):** В Python реализуется через:
    * **Наследование и полиморфизм:** Создание подклассов для добавления поведения.
    * **Композицию и паттерн "Стратегия":** Передача поведения через callable-объекты.
    * **Декораторы:** Обертывание функций без изменения их исходного кода.
    * **Абстрактные базовые классы (ABC):** Определение интерфейсов для расширения.
      Код, соответствующий OCP, позволяет добавлять новые тестовые сценарии и проверки без модификации ядра тестового
      фреймворка.

3. **LSP (Liskov Substitution):** Это контрактный принцип. В Python он реализуется через:
    * **Соблюдение сигнатур методов:** Подкласс не должен ужесточать предусловия (требовать больше) или ослаблять
      постусловия (обещать меньше).
    * **Сохранение инвариантов:** Состояние объекта после операций должно оставаться валидным с точки зрения базового
      класса.
    * **Исключения:** Подкласс не должен выбрасывать новые типы исключений, не являющиеся подтипами исключений базового
      класса.
      Нарушение LSP — классическая причина падения тестов при замене реализации. Для QA это означает, что моки и стабы
      должны точно следовать контракту реальных объектов.

4. **ISP (Interface Segregation):** В Python, где нет формальных интерфейсов, принцип реализуется через:
    * **Абстрактные базовые классы (ABC)** с минимальным набором абстрактных методов.
    * **Протоколы (Protocol),** которые позволяют описывать узкие, специфичные наборы методов.
    * **Миксины (Mixins)** — классы, предоставляющие конкретную функциональность.
      Следствие для тестирования: нам не нужно мокировать гигантские интерфейсы, достаточно реализовать только
      используемую часть.

5. **DIP (Dependency Inversion):** Практическая реализация:
    * **Зависимость от абстракций:** Вместо `ConcreteService` в коде указывается `AbstractService` (ABC или Protocol).
    * **Внедрение зависимостей (DI):** Зависимости передаются извне (через конструктор, сеттеры, контекст), а не
      создаются внутри класса.
    * **IoC-контейнеры** (в продвинутых случаях) для автоматического управления зависимостями.
      Это краеугольный камень тестируемости: он позволяет легко подменять реальные сервисы моками в тестах.

## **Senior Level**

### **Пример 1: Single Responsibility Principle (SRP)**

#### ❌ **Неправильно: God Object в тестовом фреймворке**

```python
class TestFramework:
    """Нарушение SRP: класс делает слишком много"""

    def __init__(self):
        self.driver = None
        self.config = {}
        self.test_data = {}
        self.report_data = []
        self.logger = None

    def setup_driver(self):
        # Создание драйвера
        pass

    def load_config(self, path):
        # Загрузка конфигурации
        pass

    def generate_test_data(self):
        # Генерация тестовых данных
        pass

    def run_test(self, test_case):
        # Запуск теста
        pass

    def generate_report(self):
        # Генерация отчета
        pass

    def send_email(self):
        # Отправка email
        pass

    def cleanup(self):
        # Очистка
        pass

    def backup_results(self):
        # Бэкап результатов
        pass
```

#### ✅ **Правильно: Разделение ответственностей**

```python
from abc import ABC, abstractmethod
from typing import Protocol


class WebDriverProvider(Protocol):
    def get_driver(self): ...

    def quit_driver(self): ...


class ConfigLoader:
    def __init__(self, path: str):
        self.path = path

    def load(self) -> dict:
        """Только загрузка конфигурации"""
        with open(self.path, 'r') as f:
            return json.load(f)


class TestDataGenerator:
    def __init__(self, strategy: DataGenerationStrategy):
        self.strategy = strategy

    def generate(self) -> dict:
        """Только генерация тестовых данных"""
        return self.strategy.execute()


class TestRunner:
    def __init__(self, driver_provider: WebDriverProvider):
        self.driver_provider = driver_provider

    def run(self, test_case: TestCase) -> TestResult:
        """Только запуск теста"""
        driver = self.driver_provider.get_driver()
        # логика теста
        return result


class ReportGenerator:
    def __init__(self, formatter: ReportFormatter):
        self.formatter = formatter

    def generate(self, results: list[TestResult]) -> Report:
        """Только генерация отчета"""
        return self.formatter.format(results)


# Композиция компонентов
class TestSuite:
    def __init__(
            self,
            config_loader: ConfigLoader,
            data_generator: TestDataGenerator,
            test_runner: TestRunner,
            report_generator: ReportGenerator
    ):
        self.config_loader = config_loader
        self.data_generator = data_generator
        self.test_runner = test_runner
        self.report_generator = report_generator

    def execute(self):
        config = self.config_loader.load()
        test_data = self.data_generator.generate()
        # ... выполнение
```

### **Пример 2: Open/Closed Principle (OCP)**

#### ❌ **Неправильно: Модификация при добавлении новой проверки**

```python
class AssertionChecker:
    """Нарушение OCP: нужно модифицировать при добавлении новых проверок"""

    def check(self, actual, expected, check_type):
        if check_type == "equals":
            return actual == expected
        elif check_type == "contains":
            return expected in actual
        elif check_type == "greater":
            return actual > expected
        # Добавление новой проверки требует изменения кода
        # elif check_type == "regex":
        #     return bool(re.match(expected, actual))
        else:
            raise ValueError(f"Unknown check type: {check_type}")
```

#### ✅ **Правильно: Расширение через новые классы**

```python
from abc import ABC, abstractmethod
from typing import Any


class CheckStrategy(ABC):
    @abstractmethod
    def execute(self, actual: Any, expected: Any) -> bool:
        pass


class EqualsCheck(CheckStrategy):
    def execute(self, actual, expected):
        return actual == expected


class ContainsCheck(CheckStrategy):
    def execute(self, actual, expected):
        return expected in actual


class RegexCheck(CheckStrategy):
    def execute(self, actual, expected):
        import re
        return bool(re.match(expected, actual))


class CustomCheck(CheckStrategy):
    def __init__(self, custom_func):
        self.custom_func = custom_func

    def execute(self, actual, expected):
        return self.custom_func(actual, expected)


class AssertionChecker:
    """Соблюдение OCP: расширяется без модификации"""

    def __init__(self):
        self._strategies = {}
        self._register_default_strategies()

    def _register_default_strategies(self):
        self.register("equals", EqualsCheck())
        self.register("contains", ContainsCheck())

    def register(self, name: str, strategy: CheckStrategy):
        """Добавление новой стратегии без изменения кода"""
        self._strategies[name] = strategy

    def check(self, name: str, actual: Any, expected: Any) -> bool:
        if name not in self._strategies:
            raise ValueError(f"Unknown check: {name}")
        return self._strategies[name].execute(actual, expected)


# Использование
checker = AssertionChecker()
checker.register("regex", RegexCheck())  # Расширение без модификации

# Новую кастомную проверку можно добавить динамически
checker.register("custom", CustomCheck(lambda a, e: len(a) > e))
```

### **Пример 3: Liskov Substitution Principle (LSP)**

#### ❌ **Неправильно: Нарушение контракта базового класса**

```python
class Database:
    def connect(self, timeout: int = 10) -> bool:
        """Возвращает True при успешном подключении"""
        # Логика подключения
        return True

    def query(self, sql: str) -> list:
        """Возвращает список результатов"""
        return []


class MockDatabase(Database):
    def connect(self, timeout: int = 10) -> bool:
        """Нарушение LSP: изменяет контракт - требует больше"""
        if timeout < 20:  # Ужесточение предусловия
            raise ValueError("Mock requires timeout >= 20")
        return True

    def query(self, sql: str) -> list:
        """Нарушение LSP: изменяет контракт - возвращает другой тип"""
        return {}  # Должен возвращать list, а возвращает dict


# В тестах это приведет к падению
def test_database_operations(db: Database):
    db.connect(15)  # Упадет с MockDatabase
    results = db.query("SELECT * FROM users")
    # Упадет при попытке использовать results как list
```

#### ✅ **Правильно: Соблюдение контракта**

```python
from typing import Protocol, runtime_checkable


@runtime_checkable
class DatabaseProtocol(Protocol):
    def connect(self, timeout: int) -> bool: ...

    def query(self, sql: str) -> list: ...


class RealDatabase:
    def connect(self, timeout: int = 10) -> bool:
        print(f"Connecting with timeout {timeout}")
        return True

    def query(self, sql: str) -> list:
        print(f"Executing: {sql}")
        return ["result1", "result2"]


class MockDatabase:
    def connect(self, timeout: int = 10) -> bool:
        """Соблюдает контракт: те же предусловия"""
        print(f"Mock connecting with timeout {timeout}")
        return True  # Всегда успешно

    def query(self, sql: str) -> list:
        """Соблюдает контракт: возвращает list"""
        print(f"Mock executing: {sql}")
        return ["mock_result1", "mock_result2"]  # Корректный тип


class InMemoryDatabase:
    def __init__(self):
        self.data = {}

    def connect(self, timeout: int = 10) -> bool:
        """Соблюдает контракт"""
        return True  # Всегда успешно

    def query(self, sql: str) -> list:
        """Соблюдает контракт"""
        # Парсинг SQL и работа с памятью
        return list(self.data.values())


# Все реализации могут быть использованы взаимозаменяемо
def run_database_test(db: DatabaseProtocol):
    assert isinstance(db, DatabaseProtocol)  # Проверка протокола

    success = db.connect(15)
    assert success is True

    results = db.query("SELECT * FROM users")
    assert isinstance(results, list)  # Гарантировано

    return results


# Все реализации работают корректно
for db in [RealDatabase(), MockDatabase(), InMemoryDatabase()]:
    run_database_test(db)
```

### **Пример 4: Interface Segregation Principle (ISP)**

#### ❌ **Неправильно: "Толстый" интерфейс**

```python
class TestFrameworkInterface:
    """Нарушение ISP: заставляет реализовывать ненужные методы"""

    def setup_driver(self):
        pass

    def teardown_driver(self):
        pass

    def load_config(self):
        pass

    def parse_results(self):
        pass

    def generate_report(self):
        pass

    def send_notification(self):
        pass

    def backup_results(self):
        pass

    def cleanup_temp_files(self):
        pass


class SimpleTestRunner(TestFrameworkInterface):
    """Вынужден реализовывать все методы, хотя нужны только некоторые"""

    def setup_driver(self):
        # Нужно
        pass

    def teardown_driver(self):
        # Нужно
        pass

    def load_config(self):
        # Нужно
        pass

    def parse_results(self):
        # Не нужно, но вынужден реализовать
        raise NotImplementedError("Not needed for simple runner")

    def generate_report(self):
        # Не нужно, но вынужден реализовать
        raise NotImplementedError("Not needed for simple runner")

    # ... остальные ненужные методы
```

#### ✅ **Правильно: Разделенные интерфейсы**

```python
from typing import Protocol


class DriverManager(Protocol):
    def setup_driver(self): ...

    def teardown_driver(self): ...


class ConfigLoader(Protocol):
    def load_config(self) -> dict: ...


class TestExecutor(Protocol):
    def execute_test(self, test_case) -> TestResult: ...


class ResultParser(Protocol):
    def parse_results(self, results) -> ParsedResults: ...


class ReportGenerator(Protocol):
    def generate_report(self, data) -> Report: ...


class NotificationSender(Protocol):
    def send_notification(self, message): ...


# Классы реализуют только нужные интерфейсы
class SimpleTestRunner:
    def __init__(
            self,
            driver_manager: DriverManager,
            config_loader: ConfigLoader,
            test_executor: TestExecutor
    ):
        self.driver_manager = driver_manager
        self.config_loader = config_loader
        self.test_executor = test_executor

    def run(self):
        # Использует только нужные зависимости
        config = self.config_loader.load_config()
        self.driver_manager.setup_driver()
        # ... выполнение тестов
        self.driver_manager.teardown_driver()


class FullFeaturedRunner(SimpleTestRunner):
    def __init__(
            self,
            driver_manager: DriverManager,
            config_loader: ConfigLoader,
            test_executor: TestExecutor,
            result_parser: ResultParser,
            report_generator: ReportGenerator,
            notifier: NotificationSender
    ):
        super().__init__(driver_manager, config_loader, test_executor)
        self.result_parser = result_parser
        self.report_generator = report_generator
        self.notifier = notifier

    def run_with_reporting(self):
        results = self.run()
        parsed = self.result_parser.parse_results(results)
        report = self.report_generator.generate_report(parsed)
        self.notifier.send_notification(f"Report: {report}")
```

### **Пример 5: Dependency Inversion Principle (DIP)**

#### ❌ **Неправильно: Зависимость от конкретных реализаций**

```python
import requests
import smtplib
import json


class TestReporter:
    """Нарушение DIP: зависит от конкретных библиотек"""

    def __init__(self):
        # Прямая зависимость от конкретных реализаций
        self.http_client = requests.Session()
        self.email_client = smtplib.SMTP()
        self.json_parser = json

    def send_to_webhook(self, url: str, data: dict):
        """Жесткая привязка к requests"""
        response = self.http_client.post(url, json=data)
        return response.status_code == 200

    def send_email(self, to: str, subject: str, body: str):
        """Жесткая привязка к smtplib"""
        self.email_client.connect()
        # ... логика отправки
        self.email_client.quit()

    def parse_json(self, text: str):
        """Жесткая привязка к json модулю"""
        return self.json_parser.loads(text)
```

#### ✅ **Правильно: Зависимость от абстракций**

```python
from typing import Protocol, Any
from abc import ABC, abstractmethod


class HttpClient(Protocol):
    def post(self, url: str, data: dict) -> Any: ...

    def get(self, url: str) -> Any: ...


class EmailClient(Protocol):
    def connect(self): ...

    def send(self, to: str, subject: str, body: str) -> bool: ...

    def disconnect(self): ...


class JsonParser(Protocol):
    def loads(self, text: str) -> dict: ...

    def dumps(self, obj: dict) -> str: ...


class RequestsHttpClient:
    """Конкретная реализация для production"""

    def __init__(self):
        import requests
        self.session = requests.Session()

    def post(self, url: str, data: dict):
        return self.session.post(url, json=data)

    def get(self, url: str):
        return self.session.get(url)


class MockHttpClient:
    """Реализация для тестирования"""

    def __init__(self, mock_responses=None):
        self.mock_responses = mock_responses or {}
        self.calls = []

    def post(self, url: str, data: dict):
        self.calls.append(("POST", url, data))
        return self.mock_responses.get(url, MockResponse(200))

    def get(self, url: str):
        self.calls.append(("GET", url))
        return self.mock_responses.get(url, MockResponse(200))


class TestReporter:
    """Соблюдение DIP: зависит от абстракций"""

    def __init__(
            self,
            http_client: HttpClient,
            email_client: EmailClient,
            json_parser: JsonParser
    ):
        self.http_client = http_client
        self.email_client = email_client
        self.json_parser = json_parser

    def send_to_webhook(self, url: str, data: dict) -> bool:
        """Работает с любой реализацией HttpClient"""
        response = self.http_client.post(url, data)
        return getattr(response, 'status_code', 200) == 200

    def send_email(self, to: str, subject: str, body: str) -> bool:
        """Работает с любой реализацией EmailClient"""
        self.email_client.connect()
        success = self.email_client.send(to, subject, body)
        self.email_client.disconnect()
        return success


# В production
production_reporter = TestReporter(
    http_client=RequestsHttpClient(),
    email_client=RealEmailClient(),
    json_parser=StandardJsonParser()
)

# В тестах
mock_reporter = TestReporter(
    http_client=MockHttpClient(),
    email_client=MockEmailClient(),
    json_parser=MockJsonParser()
)
```

### **Пример 6: Комплексный пример - Фреймворк для API тестирования**

#### ❌ **Неправильно: Монолитная архитектура с нарушениями SOLID**

```python
class APITestFramework:
    """Монолит с множеством нарушений SOLID"""

    def __init__(self):
        self.requests = __import__('requests')
        self.json = __import__('json')
        self.config = self._load_config()
        self.auth = self._setup_auth()
        self.results = []
        self.reports = []

    def run_test(self, endpoint, method="GET", data=None):
        # SRP: слишком много ответственностей
        # DIP: прямая зависимость от requests
        # OCP: сложно добавить новые типы тестов

        url = self.config['base_url'] + endpoint

        # Жестко закодированная логика
        if method == "GET":
            response = self.requests.get(url, auth=self.auth)
        elif method == "POST":
            response = self.requests.post(url, json=data, auth=self.auth)
        # ... другие методы

        # Разная логика обработки в одном методе
        if response.status_code == 200:
            self._log_success(response)
        else:
            self._log_failure(response)

        # Генерация отчета в том же методе
        self._generate_report(response)

        return response
```

#### ✅ **Правильно: SOLID-архитектура**

```python
from abc import ABC, abstractmethod
from typing import Protocol, Optional, Any
from dataclasses import dataclass
import json as json_module


# ===== АБСТРАКЦИИ (DIP) =====
class HttpClient(Protocol):
    def request(self, method: str, url: str, **kwargs) -> Any: ...


class Authenticator(Protocol):
    def authenticate(self, request: dict) -> dict: ...


class ResponseValidator(Protocol):
    def validate(self, response) -> bool: ...


class ReportGenerator(Protocol):
    def generate(self, test_result) -> str: ...


# ===== SRP: Каждый класс с одной ответственностью =====
@dataclass
class TestRequest:
    endpoint: str
    method: str = "GET"
    data: Optional[dict] = None
    headers: Optional[dict] = None


@dataclass
class TestResult:
    request: TestRequest
    response: Any
    success: bool
    duration: float


class RequestBuilder:
    """Только сборка запросов"""

    def __init__(self, base_url: str):
        self.base_url = base_url

    def build(self, test_request: TestRequest) -> dict:
        return {
            'method': test_request.method,
            'url': f"{self.base_url}{test_request.endpoint}",
            'json': test_request.data,
            'headers': test_request.headers or {}
        }


class TestExecutor:
    """Только выполнение тестов"""

    def __init__(
            self,
            http_client: HttpClient,
            authenticator: Authenticator,
            request_builder: RequestBuilder
    ):
        self.http_client = http_client
        self.authenticator = authenticator
        self.request_builder = request_builder

    def execute(self, test_request: TestRequest) -> TestResult:
        import time
        start = time.time()

        # Подготовка запроса
        request_data = self.request_builder.build(test_request)
        authenticated_request = self.authenticator.authenticate(request_data)

        # Выполнение
        response = self.http_client.request(**authenticated_request)

        duration = time.time() - start

        return TestResult(
            request=test_request,
            response=response,
            success=200 <= getattr(response, 'status_code', 0) < 300,
            duration=duration
        )


# ===== OCP: Легко расширяемые валидаторы =====
class StatusCodeValidator:
    def validate(self, response) -> bool:
        return 200 <= response.status_code < 300


class JsonSchemaValidator:
    def __init__(self, schema: dict):
        self.schema = schema

    def validate(self, response) -> bool:
        import jsonschema
        try:
            jsonschema.validate(response.json(), self.schema)
            return True
        except:
            return False


class CompositeValidator:
    """Композиция валидаторов"""

    def __init__(self, validators: list[ResponseValidator]):
        self.validators = validators

    def validate(self, response) -> bool:
        return all(v.validate(response) for v in self.validators)


# ===== LSP: Взаимозаменяемые аутентификаторы =====
class BasicAuthenticator:
    def authenticate(self, request: dict) -> dict:
        request['auth'] = ('user', 'pass')
        return request


class TokenAuthenticator:
    def __init__(self, token: str):
        self.token = token

    def authenticate(self, request: dict) -> dict:
        headers = request.get('headers', {})
        headers['Authorization'] = f'Bearer {self.token}'
        request['headers'] = headers
        return request


class MockAuthenticator:
    def authenticate(self, request: dict) -> dict:
        return request  # Без аутентификации для тестов


# ===== ISP: Специализированные генераторы отчетов =====
class JsonReportGenerator:
    def generate(self, test_result: TestResult) -> str:
        return json_module.dumps({
            'success': test_result.success,
            'duration': test_result.duration,
            'status': getattr(test_result.response, 'status_code', None)
        })


class HtmlReportGenerator:
    def generate(self, test_result: TestResult) -> str:
        return f"""
        <html>
            <body>
                <h1>Test Result</h1>
                <p>Success: {test_result.success}</p>
                <p>Duration: {test_result.duration:.2f}s</p>
            </body>
        </html>
        """


# ===== ФИНАЛЬНАЯ КОМПОЗИЦИЯ =====
class APITestFramework:
    """SOLID-совместимый фреймворк"""

    def __init__(
            self,
            base_url: str,
            http_client: HttpClient,
            authenticator: Authenticator,
            validator: ResponseValidator,
            report_generator: ReportGenerator
    ):
        self.request_builder = RequestBuilder(base_url)
        self.executor = TestExecutor(http_client, authenticator, self.request_builder)
        self.validator = validator
        self.report_generator = report_generator
        self.results = []

    def run_test(self, test_request: TestRequest) -> dict:
        # Выполнение
        result = self.executor.execute(test_request)

        # Валидация
        result.success = self.validator.validate(result.response)

        # Отчет
        report = self.report_generator.generate(result)

        self.results.append(result)
        return {
            'result': result,
            'report': report,
            'valid': result.success
        }

    # Легко добавить новые методы без изменения существующего кода (OCP)
    def run_test_suite(self, requests: list[TestRequest]):
        return [self.run_test(req) for req in requests]


# ===== ИСПОЛЬЗОВАНИЕ С РАЗНЫМИ КОНФИГУРАЦИЯМИ =====

# Production конфигурация
production_framework = APITestFramework(
    base_url="https://api.example.com",
    http_client=RequestsHttpClient(),
    authenticator=TokenAuthenticator("real-token"),
    validator=CompositeValidator([
        StatusCodeValidator(),
        JsonSchemaValidator({"type": "object"})
    ]),
    report_generator=JsonReportGenerator()
)

# Test конфигурация
test_framework = APITestFramework(
    base_url="https://test-api.example.com",
    http_client=MockHttpClient(),
    authenticator=MockAuthenticator(),
    validator=StatusCodeValidator(),
    report_generator=HtmlReportGenerator()
)

# Конфигурация для нагрузочного тестирования
load_framework = APITestFramework(
    base_url="https://api.example.com",
    http_client=AsyncHttpClient(),  # Новая реализация без изменения фреймворка
    authenticator=BasicAuthenticator(),
    validator=StatusCodeValidator(),
    report_generator=CsvReportGenerator()  # Новый генератор без изменения фреймворка
)
```

- [Содержание](CONTENTS.md#содержание)

---

# **Метапрограммирование**

## **Junior Level**

Метапрограммирование — это искусство написания программ, которые анализируют, генерируют или изменяют другие программы (
включая самих себя) во время выполнения. Если обычный код оперирует данными (числами, строками, объектами), то
метапрограммирование оперирует **кодом как данными**.

Представьте себе фабрику, которая не производит товары, а проектирует и собирает другие фабрики. Метапрограммирование в
Python — это и есть такая «фабрика фабрик», дающая вам инструменты для управления структурой и поведением кода на более
высоком, абстрактном уровне.

Простейшие формы метапрограммирования:

* **Декораторы** (`@decorator`): Это «обёртки», которые меняют поведение функции или класса, не трогая их исходный код.
  Например, `@staticmethod` или `@property` изменяют способ вызова метода.
* **Динамическое создание классов**: С помощью встроенной функции `type()` можно создать новый класс прямо в процессе
  выполнения программы, сгенерировав его имя, атрибуты и методы «на лету».
* **Изменение объектов**: В Python можно добавить новый метод или атрибут к уже существующему объекту после его
  создания. Эта гибкость — фундаментальное свойство языка, открывающее двери для метапрограммирования.

Метапрограммирование позволяет создавать элегантные абстракции, автоматизировать рутинный код (
например, генерацию геттеров/сеттеров), внедрять сквозную логику (логирование, проверку прав) и строить мощные
фреймворки (такие как ORM Django или система валидации Pydantic), где структура кода динамически определяется на основе
моделей или конфигураций.

## **Middle Level**

Метапрограммирование в Python — это не единая техника, а целый арсенал инструментов разного уровня воздействия: от
точечной модификации функций до управления самой природой создания классов. Их можно представить как матрешку: каждый
следующий инструмент работает на более глубоком уровне абстракции.

### **1. Декораторы: Модификация поведения «снаружи»**

Декоратор — это функция высшего порядка, принимающая в качестве аргумента другую функцию (или класс) и возвращающая
новую, модифицированную версию. Синтаксис `@` — это лишь удобная запись.

* **Суть**: Декоратор действует как «обёртка» или «плагин», добавляющий дополнительную логику *до* или *после* вызова
  исходной функции, не изменяя её ядра.
* **Эволюция**: Декораторы могут быть вложенными, принимать собственные аргументы (`@retry(attempts=3)`) и применяться к
  методам класса. Они стали стандартным способом реализации аспектно-ориентированного программирования в Python для
  задач кеширования, логирования, аутентификации.

### **2. Дескрипторы: Управление доступом к атрибутам**

Дескрипторы — это объекты, реализующие протокол с методами `__get__`, `__set__` и `__delete__`. Когда вы обращаетесь к
атрибуту объекта, Python проверяет, не является ли этот атрибут дескриптором, и если да, то вызывает соответствующий
метод.

* **Суть**: Дескрипторы перехватывают операции чтения, записи или удаления атрибута, позволяя вам внедрить свою логику (
  валидацию, ленивые вычисления, логирование доступа).
* **Где встречается**: Вся система `@property`, `@classmethod`, `@staticmethod`, а также механизм `__slots__` построены
  на дескрипторах. Это мощный инструмент для создания умных атрибутов, поведение которых вы можете полностью
  контролировать.

### **3. Метаклассы: Контроль над созданием классов**

Если класс — это «чертёж» для создания объектов, то метакласс — это «чертёж» для создания классов. По умолчанию все
классы создаются метаклассом `type`.

* **Суть**: Создав собственный метакласс (наследовав от `type`), вы можете перехватить момент создания класса (до
  появления первого его экземпляра). В этот момент можно проанализировать, модифицировать или валидировать атрибуты и
  методы будущего класса, зарегистрировать его в реестре или автоматически добавить новые методы.
* **Применение**: ORM-фреймворки (например, Django) используют метаклассы для трансформации объявлений моделей
  `class User(models.Model)` в сложные объекты с доступом к базе данных. Это инструмент для глубокой, архитектурной
  магии, когда требуется, чтобы классы вели себя особым, нестандартным образом с самого рождения.

### **Сопутствующие мощные (и опасные) техники**

* **Интроспекция**: Возможность программы исследовать свою собственную структуру во время выполнения. Функции `dir()`,
  `getattr()`, `hasattr()`, а также модуль `inspect` позволяют «заглянуть внутрь» объектов, узнать их методы, аргументы,
  исходный код. Это основа для многих продвинутых инструментов: систем автоматического обнаружения тестов, фреймворков
  dependency injection, интерактивных дебаггеров и автодокументирования.
* **Monkey Patching (Обучение обезьян)**: Динамическое изменение модулей или классов *после* их загрузки в память, во
  время выполнения программы. Позволяет «исправлять» поведение стороннего кода, добавлять фичи или, что наиболее часто,
  подменять реальные объекты на **моки (mock-объекты)** в целях тестирования (как это делает библиотека
  `unittest.mock`). Это крайне мощный, но и рискованный приём, так как может привести к трудноотлавливаемым побочным
  эффектам, если делать это без чёткого понимания области видимости изменений.
* **Динамическое выполнение кода (`eval`/`exec`)**: Функции `eval()` (вычисляет выражение) и `exec()` (выполняет блок
  кода) интерпретируют строки как инструкции Python. Это самый радикальный вид метапрограммирования, позволяющий
  исполнять код, сгенерированный самой программой. **Крайне опасен** при использовании с непроверенными
  пользовательскими данными (уязвимость инъекции кода). Применяется во внутренних DSL (предметно-ориентированных
  языках), сложных системах конфигурации или шаблонизаторах, где безопасность входных данных гарантирована.

### **Иерархия и синергия**

Эти инструменты образуют иерархию по глубине воздействия:

1. **Декораторы** меняют уже готовые функции/классы.
2. **Дескрипторы** управляют доступом к отдельным атрибутам внутри класса.
3. **Метаклассы** определяют саму природу и процесс создания классов.

При этом они часто работают вместе: метакласс может использовать дескрипторы для создания «умных» атрибутов в новом
классе, а декоратор — применять к методам этого класса. Понимание этой экосистемы открывает путь к созданию
выразительных, эффективных и элегантных абстракций, которые делают Python таким мощным языком для построения сложных
фреймворков.

## **Senior Level**

## Метаклассы в CPython — PyType_Type

Метапрограммирование в CPython реализуется через **PyType_Type** — тип, который создает все классы.
`class A(metaclass=MyMeta):` → `MyMeta.__new__(name, bases, dct)` вызывается на C-уровне.

```c
// Objects/typeobject.c: PyType_Type — метакласс всех типов
PyTypeObject PyType_Type = {
    PyVarObject_HEAD_INIT(&PyType_Type, 0)  // Сам себя инициализирует!
    "type",                                 // tp_name = "type"
    sizeof(PyHeapTypeObject),               // tp_basicsize для heap-типов
    0,                                      // tp_itemsize
    0,                                      // tp_dealloc (не удаляется)
    0,                                      // tp_vectorcall_offset
    0,                                      // tp_getattr
    0,                                      // tp_setattr
    0,                                      // tp_as_async
    type_repr,                              // tp_repr(self) → "<class 'int'>"
    0,                                      // tp_as_number
    0,                                      // tp_as_sequence
    0,                                      // tp_as_mapping
    type_hash,                              // tp_hash(type) → hash(tp_name)
    type_call,                              // tp_call(type, args, kwargs) ← КЛЮЧЕВОЕ!
    0,                                      // tp_str
    type_getattro,                          // tp_getattro(type, name)
    type_setattro,                          // tp_setattro(type, name, value)
    0,                                      // tp_as_buffer
    Py_TPFLAGS_DEFAULT | Py_TPFLAGS_HAVE_GC |
    Py_TPFLAGS_BASETYPE | Py_TPFLAGS_TYPE_SUBCLASS,  // Флаги метакласса
    type_doc,                               // tp_doc
    0,                                      // tp_traverse
    0,                                      // tp_clear
    type_richcompare,                       // tp_richcompare(type1, type2)
    0,                                      // tp_weaklistoffset
    type_mro,                               // tp_mro() → MRO список
    type_get_getattr,                       // tp_getattro слоты
    type_get_setattr,                       // tp_setattro слоты
    0,                                      // tp_as_async
    type_new,                               // tp_new(type, args, kwargs) ← СОЗДАНИЕ КЛАССА!
};
```

`PyType_Type` — это **"тип типов"**. Когда пишешь `class A:`, CPython вызывает
`PyType_Type.tp_call()` → `type_new()` → создает `PyTypeObject* A`. Метакласс `MyMeta` перехватывает этот вызов:
`MyMeta.tp_new()` вместо `type_new()`. **Все классы — объекты типа `type`**.

## type_call() — точка входа метапрограммирования

`type(args)` → `type_call(type, args, kwargs)` → цепочка `__call__` → `__new__` → `__init__`.

```c
// Objects/typeobject.c: type_call() — обработка class A(): 
static PyObject *
type_call(PyTypeObject *type, PyObject *args, PyObject *kwds) {
    PyThreadState *tstate = _PyThreadState_GET();  // Текущее состояние потока
    
    if (!PyArg_UnpackTuple(args, "type", 1, 3,      // args = (name, bases, dct)
                           &PyTuple_GET_ITEM(args, 0), &type->tp_bases, 
                           &type->tp_dict))         // Распаковка аргументов
        return NULL;
        
    // 1. Вызываем tp_prepare() метакласса (namespace preparation)
    if (type->tp_flags & Py_TPFLAGS_HEAPTYPE) {
        PyObject *prepare = _PyDict_GetItemIdWithError(  // Ищем __prepare__
            lookup_tp_dict(type), &_Py_ID(__prepare__));
        if (prepare && prepare != Py_None) {
            args = PyTuple_Pack(3, Py_NewRef(type),     // Вызываем __prepare__
                                Py_NewRef(type->tp_bases), 
                                Py_NewRef(type->tp_dict));
            PyObject *prepared = PyObject_CallObject(prepare, args);
            Py_DECREF(args);
            if (prepared == NULL) return NULL;
            type->tp_dict = prepared;  // Заменяем dct на результат
        }
    }
    
    // 2. Создаем класс: type_new()
    PyObject *obj = type_new(type, args, kwds);    // КРИТИЧЕСКАЯ ТОЧКА!
    return obj;
}
```

`class A():` → Python компилятор генерирует `(name="A", bases=(), dct={})` →
`type_call(PyType_Type, args)` → `type_new()` создает `PyTypeObject* A`. Если метакласс — сначала
`__prepare__(metaclass, name, bases)` готовит namespace (обычно dict), потом `__new__` создает класс.

## type_new() — фабрика классов CPython

`type_new(metaclass, name, bases, dct)` — сердце метапрограммирования. Создает `PyHeapTypeObject`.

```c
// Objects/typeobject.c: type_new() — создание PyTypeObject
static PyObject *
type_new(PyTypeObject *metaclass, PyObject *args, PyObject *kwds) {
    // Проверяем аргументы: type(name, bases, dct)
    PyObject *name, *bases, *dict;
    if (!PyArg_UnpackTuple(args, "", 1, 3, &name, &bases, &dict))
        return NULL;
        
    // 1. Создаем базовую PyHeapTypeObject
    PyHeapTypeObject *type = (PyHeapTypeObject *)metaclass->tp_alloc(metaclass, 0);
    if (type == NULL) return NULL;
    
    // 2. Инициализируем PyObject_HEAD
    Py_SET_TYPE(type, metaclass);              // ob_type = metaclass
    type->ht_name = name;                      // __name__ = "A"
    Py_INCREF(name);
    type->ht_qualname = name;                  // __qualname__ = "A"
    Py_INCREF(name);
    
    // 3. Устанавливаем bases и вычисляем MRO
    type->ht_bases = bases;                    // __bases__ = bases
    Py_INCREF(bases);
    set_tp_bases((PyTypeObject *)type, bases, 1);
    
    // 4. Копируем атрибуты из dct
    if (type_ready((PyTypeObject *)type) < 0) { // PyType_Ready() — MRO + слоты!
        Py_DECREF(type);
        return NULL;
    }
    
    // 5. Вызываем __init__ метакласса (опционально)
    if (PyDict_Contains(lookup_tp_dict(metaclass), &_Py_ID(__init__))) {
        if (metaclass->tp_init((PyObject *)type, args, kwds) < 0) {
            Py_DECREF(type);
            return NULL;
        }
    }
    
    return (PyObject *)type;  // Новый класс готов!
}
```

`MyMeta.__new__("A", (), {})` → выделяется память под `PyHeapTypeObject` (расширенный
PyTypeObject) → копируются name/bases/dct → `PyType_Ready()` вычисляет MRO и слоты → `__init__` метакласса (если есть).
**Результат — полноценный PyTypeObject**.

## PyType_Ready() — финальная подготовка метакласса

Вычисляет MRO, наследует слоты, проверяет конфликты `__slots__`, кэширует атрибуты.

```c
// Objects/typeobject.c: PyType_Ready() — готовим класс к использованию
static int
type_ready(PyTypeObject *type) {
    BEGIN_TYPE_LOCK();                         // Блокируем все типы
    
    // 1. Проверяем, не готов ли уже
    if (type->tp_flags & Py_TPFLAGS_READY) {
        END_TYPE_LOCK();
        return 0;
    }
    start_readying(type);                      // Помечаем "готовится"
    
    // 2. Вычисляем базовый класс (первый из tp_bases)
    PyObject *bases = lookup_tp_bases(type);
    if (PyTuple_GET_SIZE(bases) > 0) {
        type->tp_base = (PyTypeObject *)PyTuple_GET_ITEM(bases, 0);
    }
    
    // 3. Строим MRO (C3-линеаризация)
    if (type->tp_mro == NULL) {
        type->tp_mro = mro_internal(type);     // Рекурсивно по bases
        if (type->tp_mro == NULL) goto error;
    }
    
    // 4. Наследуем слоты методов (tp_call, tp_new и т.д.)
    inherit_special(type);                     // Копируем из родителей
    resolve_slots_inheritable(type);           // Разрешаем конфликты
    
    // 5. Обрабатываем __slots__ (фиксированные атрибуты)
    if (collect_methods(type) < 0) goto error; // Слоты в tp_getset
    
    // 6. Устанавливаем флаги и версию кэша
    type_add_flags(type, Py_TPFLAGS_READY);
    assign_version_tag(_PyInterpreterState_GET(), type);  // tp_version_tag
    
    stop_readying(type);
    END_TYPE_LOCK();
    return 0;
    
error:
    stop_readying(type);
    type_clear(type);  // Откат изменений
    END_TYPE_LOCK();
    return -1;
}
```

После `__new__` метакласса CPython вызывает `PyType_Ready()`:

1) строит MRO (очередь родителей)
2) копирует методы из родителей
3) обрабатывает `__slots__`
4) вычисляет смещения атрибутов
5) помечает "готов". **Только после этого класс можно использовать**

## Разрешение метаклассов — сложный алгоритм

При `class C(A, metaclass=M):` CPython решает, какой метакласс использовать.

```c
// Objects/typeobject.c: find_metaclass() — выбор метакласса
static PyObject *
find_metaclass(PyThreadState *tstate, PyObject *bases, PyObject *metaclass, 
               PyObject **metaclass_override) {
    // 1. Ищем metaclass= в bases
    for (Py_ssize_t i = 0; i < PyTuple_GET_SIZE(bases); i++) {
        PyObject *base = PyTuple_GET_ITEM(bases, i);
        PyObject *base_meta = lookup_tp_dict(_PyType_CAST(base));
        PyObject *base_m = _PyDict_GetItemId(base_meta, &_Py_ID(__class__));
        if (base_m != metaclass && PyType_Check(base_m)) {
            Py_SETREF(metaclass, base_m);      // Берем метакласс базы
        }
    }
    
    // 2. Проверяем совместимость MRO
    Py_ssize_t n = PyTuple_GET_SIZE(bases);
    for (Py_ssize_t i = 0; i < n; i++) {
        PyTypeObject *b = _PyType_CAST(PyTuple_GET_ITEM(bases, i));
        PyObject *mro = lookup_tp_mro(b);
        Py_ssize_t nmro = PyTuple_GET_SIZE(mro);
        for (Py_ssize_t j = 0; j < nmro; j++) {
            PyObject *base_m = PyTuple_GET_ITEM(mro, j);
            if (base_m == metaclass) continue;
            if (!PyType_Check(base_m)) continue;
            PyObject *bm_meta = lookup_tp_dict((PyTypeObject *)base_m);
            PyObject *bm_m = _PyDict_GetItemId(bm_meta, &_Py_ID(__class__));
            if (bm_m != metaclass && PyType_IsSubtype((PyTypeObject *)bm_m, 
                                                      metaclass)) {
                PyErr_SetString(PyExc_TypeError, 
                    "metaclass conflict: super() cannot be used");
                return NULL;
            }
        }
    }
    *metaclass_override = metaclass;
    return Py_NewRef(metaclass);
}
```

`class C(A, metaclass=M):` → берет метакласс из `A.__class__`, если он есть. Проверяет *
*совместимость MRO**: все метаклассы в MRO родителей должны быть **супертипами** `M`. Иначе
`TypeError: metaclass conflict`. **ABCMeta + type** работают благодаря иерархии метаклассов.

## Сравнение метапрограммирования уровней

| Уровень         | C-реализация                    | Python-реализация                           |
|-----------------|---------------------------------|---------------------------------------------|
| **__prepare__** | `type_call()` ищет в tp_dict    | `metaclass.__prepare__(name, bases)`        |
| **__new__**     | `type_new()` + `PyType_Ready()` | `metaclass.__new__(name, bases, dct)`       |
| **__init__**    | После `PyType_Ready()`          | `metaclass.__init__(cls, name, bases, dct)` |
| **Кэш**         | `tp_version_tag` инвалидация    | Автоматически через MRO                     |
| **MRO**         | `mro_internal()` C3             | Через `type.mro()`                          |

Метаклассы — **мощный хук** в создание классов. CPython реализует их через слоты `PyType_Type`, делая **любой класс
изменяемым на лету**.

- [Содержание](CONTENTS.md#содержание)

---

# **Миксины**

## **Junior Level**

Миксины (mixins) — это специальные классы в Python, предназначенные для добавления конкретной функциональности к другим
классам через множественное наследование. Если представить основной класс как основное блюдо, то миксины — это специи
или добавки, которые придают ему дополнительные свойства и возможности.

Миксины не предназначены для самостоятельного использования — они не являются полноценными объектами, а служат для "
подмешивания" методов и атрибутов к другим классам. Например, можно создать миксин `LoggingMixin`, который добавляет
методы логирования, и использовать его в разных классах, где нужна эта функциональность.

Для QA инженера миксины полезны при создании тестовых фреймворков и утилит: можно вынести общую функциональность (
например, работу с базой данных, генерацию тестовых данных, создание отчетов) в миксины и переиспользовать их в разных
тестовых классах.

## **Middle Level**

Технически миксины — это обычные классы Python, которые следуют определенным соглашениям при использовании в
множественном наследовании.

1. **Синтаксис и использование:**
    - Миксины включаются в цепочку наследования перед основным классом.
    - Обычно имеют суффикс `Mixin` в названии для ясности.
    - Не вызывают `super().__init__()` в своем `__init__`, если не предназначены для участия в MRO (Method Resolution
      Order) инициализации.

2. **Метод разрешения порядка (MRO):**
    - При множественном наследовании Python использует алгоритм C3 для определения порядка поиска методов.
    - Миксины должны быть спроектированы так, чтобы не конфликтовать с методами основных классов и других миксинов.
    - Важно правильно располагать миксины в списке наследования: обычно миксины идут слева перед основными классами.

3. **Особенности проектирования:**
    - **Одна ответственность:** Каждый миксин должен добавлять одну четкую функциональность.
    - **Независимость:** Миксины должны быть максимально независимы от деталей реализации классов, к которым они
      подмешиваются.
    - **Гибкость:** Миксины могут определять абстрактные методы (`@abstractmethod`), которые должны быть реализованы в
      основном классе.

4. **Для AQA:**
    - **Тестовые утилиты:** Создание миксинов для общих операций: `DatabaseMixin` (для работы с БД), `APIClientMixin` (
      для HTTP-запросов), `ScreenshotMixin` (для создания скриншотов при падении теста).
    - **Расширение Page Object:** Добавление функциональности к Page Object через миксины (например, `ModalDialogMixin`
      для работы с модальными окнами).
    - **Кастомизация тестовых классов:** В `unittest.TestCase` можно использовать миксины для добавления методов
      подготовки данных, ассертов.

5. **Примеры в экосистеме Python:**
    - Django использует миксины для CBV (Class-Based Views).
    - DRF (Django REST Framework) активно использует миксины для ViewSets.
    - В тестировании: `unittest.TestCase` можно расширять миксинами.

## **Senior Level**

## Миксины в CPython — множественное наследование + MRO

Миксины в CPython — это **обычные классы в множественном наследовании**, обрабатываемые через **C3 MRO** (Method
Resolution Order). Нет специальной поддержки — миксины работают как любой `class A(Base, Mixin1, Mixin2):`.

```c
// Objects/typeobject.c: mro_internal() — C3 алгоритм для миксинов
static PyObject *
mro_internal(PyTypeObject *type) {
    PyObject *bases = lookup_tp_bases(type);   // Кортеж баз: (Base, Mixin1, Mixin2)
    Py_ssize_t nbase = PyTuple_GET_SIZE(bases); // Кол-во родителей
    
    if (nbase == 0) {                          // Нет родителей → только type + object
        return mro_build([type, object]);
    }
    
    PyObject *result = NULL;                   // Результирующий MRO
    PyObject *sequences = PyTuple_New(nbase + 1); // Список MRO родителей
    PyTuple_SET_ITEM(sequences, 0, Py_NewRef((PyObject *)type)); // 1-й: сам класс
    
    // 2. Собираем MRO всех родителей (рекурсивно)
    for (Py_ssize_t i = 0; i < nbase; i++) {
        PyTypeObject *base = PyTuple_GET_ITEM(bases, i); // Base, Mixin1...
        PyObject *parent_mro = lookup_tp_mro(base);       // Их MRO
        if (parent_mro == NULL) goto error;
        PyTuple_SET_ITEM(sequences, i+1, Py_NewRef(parent_mro));
    }
    
    // 3. C3 линеаризация: миксины вставляются слева направо
    result = c3_merge(sequences, linearization_error);  // Алгоритм C3
    Py_DECREF(sequences);
    return result;
    
error:
    Py_DECREF(sequences);
    return NULL;
}
```

**Простыми словами для тупых**: `class A(Base, LoggingMixin, ValidationMixin):` → CPython строит MRO:
`[A, Base, LoggingMixin, ValidationMixin, object]`. **C3 алгоритм** гарантирует: 1) миксины идут **по порядку слева
направо**, 2) **НЕ дублируются**, 3) сохраняют **свой MRO**. При `A().method()` ищет:
`A.method → Base.method → LoggingMixin.method → ValidationMixin.method`.

## PyType_Ready() — наследование слотов миксинов

При создании класса CPython **копирует слоты** (методы) из миксинов по MRO, **разрешая конфликты**.

```c
// Objects/typeobject.c: inherit_special() — копирование слотов из миксинов
static int
inherit_special(PyTypeObject *type) {
    PyObject *mro = lookup_tp_mro(type);       // MRO: [A, Base, LoggingMixin...]
    
    // Идем по MRO, копируем слоты (tp_call, tp_new, tp_init...)
    for (Py_ssize_t i = 0; i < PyTuple_GET_SIZE(mro); i++) {
        PyTypeObject *base = PyTuple_GET_ITEM(mro, i); // LoggingMixin, ValidationMixin...
        
        // Копируем inheritable слоты (tp_doc, tp_new, tp_init, tp_call)
        if (base->tp_new && !type->tp_new)
            type->tp_new = base->tp_new;       // Берем из миксина
            
        if (base->tp_init && !type->tp_init)
            type->tp_init = base->tp_init;
            
        if (base->tp_call && !type->tp_call)
            type->tp_call = base->tp_call;
            
        // Пропускаем non-inheritable: tp_repr, tp_hash (нужны все или ничего)
    }
    
    // Разрешаем конфликты: последний по MRO побеждает
    resolve_slot_conflicts(type);
    return 0;
}
```

Каждый миксин имеет **таблицу слотов** (vtable): `tp_call`, `tp_new`, `tp_repr`... CPython идет по MRO и **копирует
слоты** в итоговую таблицу класса. **Последний миксин по MRO** переопределяет предыдущие. `A().__call__()` →
`A.tp_call` (из последнего миксина с `tp_call`).

## Конфликты слотов миксинов — разрешение по MRO

Если миксины определяют **одинаковые слоты**, побеждает **последний по MRO**.

```c
// Objects/typeobject.c: resolve_slots_inheritable()
static void
resolve_slots_inheritable(PyTypeObject *type) {
    PyObject *mro = lookup_tp_mro(type);
    Py_ssize_t nmro = PyTuple_GET_SIZE(mro);
    
    // Для каждого inheritable слота (tp_new, tp_init...)
    for (int slot = 0; slot < PyType_SlotCount; slot++) {
        if (!slot_is_inheritable(slot)) continue;
        
        void *last_value = NULL;               // Последнее значение по MRO
        
        // Ищем по MRO справа налево (последний побеждает)
        for (Py_ssize_t i = nmro - 1; i >= 0; i--) {
            PyTypeObject *base = PyTuple_GET_ITEM(mro, i);
            void *slot_value = *(void**)((char*)base + slot_offset[slot]);
            
            if (slot_value != NULL) {          // Найден непустой слот
                last_value = slot_value;       // Запоминаем последний
                break;
            }
        }
        
        // Записываем победивший слот
        if (last_value)
            *(void**)((char*)type + slot_offset[slot]) = last_value;
    }
}
```

`LoggingMixin.tp_repr`, `ValidationMixin.tp_repr` → **ValidationMixin** (правее в `bases`) дает финальный `tp_repr`. *
*MRO решает ВСЕ конфликты**: атрибуты, методы, слоты. Миксины должны быть **"тонкими"** (1 ответственность), иначе MRO
становится непредсказуемым.

## __slots__ в миксинах — запрещено множественное наследование

Миксины с `__slots__` **конфликтуют** из-за фиксированных смещений памяти.

```c
// Objects/typeobject.c: check_slots_multiple_bases()
static int
check_multiple_inheritance_slots(PyTypeObject *type) {
    PyObject *bases = lookup_tp_bases(type);   // (Base, LoggingSlotsMixin)
    Py_ssize_t nbase = PyTuple_GET_SIZE(bases);
    
    Py_ssize_t dictoffset = type->tp_dictoffset;  // Смещение __dict__
    
    // Считаем миксины с __slots__
    int slots_count = 0;
    for (Py_ssize_t i = 0; i < nbase; i++) {
        PyTypeObject *base = PyTuple_GET_ITEM(bases, i);
        if (base->tp_dictoffset != 0) {        // Есть __slots__
            slots_count++;
            if (base->tp_dictoffset != dictoffset) {
                PyErr_SetString(PyExc_TypeError,
                    "multiple bases with __slots__ have incompatible slots");
                return -1;                     // Конфликт смещений!
            }
        }
    }
    
    if (slots_count > 1) {
        PyErr_SetString(PyExc_TypeError,
            "can't derive from multiple bases with __slots__");
        return -1;
    }
    return 0;
}
```

`__slots__ = ['x']` → фиксирует `x` на `offsetof(PyObject_HEAD, x)`. Два миксина с `__slots__` → **разные смещения** →
дескрипторы "смотрят не туда". **Правило**: **только 1 родитель** с `__slots__`. Миксины должны использовать **обычные
атрибуты** или **композицию**.

## Кэш атрибутов миксинов — tp_version_tag

Поиск `A().method` кэшируется по `tp_version_tag`. Изменение миксинов **инвалидирует** весь поддерево.

```c
// Objects/typeobject.c: type_modified_unlocked() при изменении миксина
void PyType_Modified(PyTypeObject *mixin) {
    BEGIN_TYPE_LOCK();                         // Блокируем типы
    type_modified_unlocked(mixin);             // Рекурсивно по подклассам
    END_TYPE_LOCK();
}

static void
type_modified_unlocked(PyTypeObject *type) {
    ASSERT_TYPE_LOCK_HELD();
    
    // 1. Инвалидируем КЭШ этого типа
    set_version_unlocked(type, 0);             // tp_version_tag = 0
    
    // 2. Рекурсивно инвалидируем ПОДКЛАССЫ (использующие миксин!)
    PyObject *subclasses = lookup_tp_subclasses(type);
    if (subclasses) {
        Py_ssize_t pos = 0;
        PyObject *ref;
        while (PyDict_Next(subclasses, &pos, NULL, &ref)) {
            PyTypeObject *subclass = type_from_ref(ref);
            if (subclass) {
                type_modified_unlocked(subclass);  // A, B, C... все!
                Py_DECREF(subclass);
            }
        }
    }
}
```

Изменил `LoggingMixin.method()` → **ВСЕ классы** `class X(..., LoggingMixin):` получают `tp_version_tag=0` → **полная
инвалидация кэша**. `X().method` каждый раз ищет **заново** по MRO. **Дорого**, поэтому миксины **статичны**.

## super() в миксинах — MRO цепочка

`super().method()` в миксинах следует **MRO**, пропуская самого себя.

```c
// Objects/superobject.c: super_get() + метод разрешения
static PyObject *
super_descr_get(PyWrapperDescrObject *wrapper, PyObject *self, PyObject *instance) {
    PyTypeObject *type = wrapper->d_type;      // Класс миксина
    PyObject *mro = lookup_tp_mro(type);       // MRO миксина
    
    // Находим следующий класс ПОСЛЕ текущего в MRO
    Py_ssize_t i;
    for (i = 0; i < PyTuple_GET_SIZE(mro); i++) {
        if (PyTuple_GET_ITEM(mro, i) == (PyObject *)type) {
            break;                             // Найден LoggingMixin
        }
    }
    PyTypeObject *next_type = PyTuple_GET_ITEM(mro, i+1); // Следующий!
    
    // Создаем super(next_type, instance)
    return _PySuper_Create(next_type, instance);
}
```

В `LoggingMixin.method()` `super().method()` → **следующий** класс в MRO получает вызов. **Цепочка**:
`A.method() → LoggingMixin.method() → super() → ValidationMixin.method() → Base.method()`. **Правильный порядок** без
дублирования.

## Сравнение миксинов под капотом

| Аспект                 | Миксин (MI + MRO)       | Композиция            |
|------------------------|-------------------------|-----------------------|
| **Поиск методов**      | Авто по MRO             | `self.mixin.method()` |
| **Слоты**              | Копирование + конфликты | Отсутствуют           |
| **__slots__**          | Только 1 родитель       | Полная свобода        |
| **Изменения**          | Инвалидация поддерева   | Локально              |
| **super()**            | Автоматическая цепочка  | Ручная                |
| **Производительность** | Кэш + MRO поиск         | Прямой вызов          |

**Миксины** = **множественное наследование с дисциплиной**: слева направо, 1 ответственность, без `__slots__`. CPython
обрабатывает их через **универсальный MRO + слоты**, без специального кода.

- [Содержание](CONTENTS.md#содержание)

---

# Делегирование

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# Фабричные методы и абстрактные фабрики

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# Адаптеры и мосты

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# Наблюдатель (Observer)

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# Стратегия (Strategy)

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# Шаблонный метод (Template Method)

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)
