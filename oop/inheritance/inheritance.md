# **Наследование**

[Содержание](/CONTENTS.md#содержание)

## **Junior Level**

Наследование в ООП — это механизм, позволяющий создавать новый класс на основе существующего, перенимая его свойства и
методы. Новый класс называется **дочерним** (подклассом), а существующий — **родительским** (суперклассом).

Представьте, что у вас есть класс `Animal` с методами `eat()` и `sleep()`. Вы можете создать класс `Dog`, который
наследует от `Animal`, и автоматически получит эти методы. Затем вы можете добавить специфичное поведение для собаки:
метод `bark()`. Это позволяет избежать дублирования кода и создавать логические иерархии объектов.

Наследование представляет отношение **"является"** (is-a): собака является животным. Это один из основных способов
достижения полиморфизма — возможности использовать объекты разных классов через общий интерфейс.

[Содержание](/CONTENTS.md#содержание)

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

[Содержание](/CONTENTS.md#содержание)

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


- [Содержание](/CONTENTS.md#содержание)