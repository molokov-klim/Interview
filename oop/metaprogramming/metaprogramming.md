# **Метапрограммирование**

[Содержание](/CONTENTS.md#содержание)

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

[Содержание](/CONTENTS.md#содержание)

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

[Содержание](/CONTENTS.md#содержание)

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

- [Содержание](/CONTENTS.md#содержание)