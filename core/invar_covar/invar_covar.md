# **Инвариантность и ковариантность**

[Содержание](/CONTENTS.md#содержание)

## **Junior Level**

Инвариантность и ковариантность — это понятия из теории типов, которые описывают, как отношения между типами сохраняются
при использовании в обобщённых (generic) конструкциях.

Представьте, что у вас есть коробки для фруктов. У вас есть:

- Коробка для любых фруктов (`Box[Fruit]`)
- Коробка для яблок (`Box[Apple]`), где `Apple` — подтип `Fruit`

**Ковариантность** означает, что отношение "является подтипом" сохраняется и для коробок: если `Apple` — подтип `Fruit`,
то `Box[Apple]` — подтип `Box[Fruit]`. То есть коробку яблок можно использовать там, где ожидается коробка фруктов.

**Инвариантность** означает, что эти типы не связаны: `Box[Apple]` и `Box[Fruit]` — совершенно разные, независимые типы.
Коробку яблок нельзя использовать вместо коробки фруктов, и наоборот.

**Контравариантность** (обратное ковариантности) — когда отношение обращается: если `Apple` — подтип `Fruit`, то
`Box[Fruit]` — подтип `Box[Apple]`.

В Python эти концепции важны при работе с типизацией (type hints), особенно при использовании обобщённых типов вроде
`List[T]`, `Callable[[T], R]`.

[Содержание](/CONTENTS.md#содержание)

## **Middle Level**

В контексте Python и статической типизации:

1. **Ковариантность (covariant)**:
    - Обозначается `+T` в определении типа
    - Если `A` — подтип `B`, то `Generic[A]` — подтип `Generic[B]`
    - Пример: `typing.Sequence` ковариантен по типу элемента

2. **Контравариантность (contravariant)**:
    - Обозначается `-T`
    - Если `A` — подтип `B`, то `Generic[B]` — подтип `Generic[A]` (обратное отношение)
    - Пример: `typing.Callable` контравариантен по типам аргументов

3. **Инвариантность (invariant)**:
    - Наиболее распространённый случай по умолчанию
    - Никаких отношений между `Generic[A]` и `Generic[B]`, даже если `A` и `B` связаны
    - Пример: `List[T]` инвариантен (по соображениям безопасности)

4. **Проблема изменяемости**:
    - Ковариантность безопасна для "read-only" типов (`Sequence`, `Iterable`)
    - Изменяемые типы (`List`, `Dict`) должны быть инвариантны для безопасности типов
    - Иначе можно было бы добавить апельсин в список яблок

5. **Практическое применение в Python**:
   ```python
   from typing import TypeVar, Generic, List, Sequence
   
   class Fruit: pass
   class Apple(Fruit): pass
   
   # Ковариантный тип
   T_co = TypeVar('T_co', covariant=True)
   class ReadOnlyBox(Generic[T_co]): pass
   
   # Контравариантный тип  
   T_contra = TypeVar('T_contra', contravariant=True)
   class Consumer(Generic[T_contra]): pass
   ```

6. **Проверка типов (mypy, pyright)**:
    - Статические анализаторы используют информацию о вариативности для проверки безопасности типов
    - Ошибки вариативности — частые причины ошибок типизации

[Содержание](/CONTENTS.md#содержание)

## **Senior Level**

В CPython 3.9+ **инвариантность/ковариантность** реализованы в **generic alias** (`list[T]`) через
`PyGenericAliasObject` (`Objects/genericaliasobject.c`), где **variance** хранится в `ga_variance` поля TypeVar (
`covariant=True`). **Runtime** — только `__class_getitem__`, **статическая проверка** — в type checker'ах (
mypy). `Objects/genericaliasobject.c`,`Include/cpython/typeobject.h`

## 1. Generic alias: list[T] как PyGenericAliasObject

```c
// Objects/genericaliasobject.c — list[int] = PyGenericAliasObject
typedef struct {
    PyObject_HEAD
    PyObject *ga_origin;           // list (оригинальный тип)
    PyObject *ga_params;           // tuple(int,) параметры
    int ga_flags;                  // флаги
    Py_hash_t ga_hash;             // кэш хэша
    unsigned char ga_variance;     // ← VARIANCE! 0=invariant, 1=covariant, 2=contravariant
} PyGenericAliasObject;
```

`list[int]` — **НЕ** класс, а **специальный объект** `PyGenericAliasObject` с полями: `ga_origin=list`,
`ga_params=(int,)`, `ga_variance=1` (covariant). **Runtime** не проверяет типы — только хранит!

## 2. Создание generic alias: __class_getitem__

```c
// Objects/typeobject.c — List.__class_getitem__(int)
static PyObject *
type_subscript(PyTypeObject *type, PyObject *item)
{
    // type=list, item=int → list[int]
    if (PyTuple_Check(item)) {
        // list[int, str]
        return PyGenericAlias_New(type, item, NULL);
    }
    
    // list[int]
    PyObject *params = PyTuple_New(1);
    PyTuple_SET_ITEM(params, 0, Py_NewRef(item));
    return PyGenericAlias_New(type, params, NULL);
}

// Objects/genericaliasobject.c
PyObject *
PyGenericAlias_New(PyObject *origin, PyObject *params, PyObject *context)
{
    PyGenericAliasObject *ga = PyObject_GC_New(PyGenericAliasObject, &PyGenericAlias_Type);
    if (ga == NULL)
        return NULL;
    
    Py_INCREF(origin);
    ga->ga_origin = origin;        // list
    
    Py_INCREF(params);
    ga->ga_params = params;        // (int,)
    
    ga->ga_flags = 0;
    ga->ga_variance = 0;           // по умолчанию invariant
    
    PyObject_GC_Track(ga);
    return (PyObject *)ga;
}
```

`list[int]` → `list.__class_getitem__(int)` → `PyGenericAlias_New(list, (int,), NULL)` → **новый объект** с
`ga_origin=list`, `ga_params=(int,)`. **НЕ** создаёт новый класс!

## 3. TypeVar с variance (typing.TypeVar)

```c
// Lib/typing.py (упрощённо) → компилируется в C-объекты
class TypeVar:
    def __init__(self, name, *constraints, covariant=False, contravariant=False):
        self.__name__ = name
        self.__covariant__ = covariant      # True для covariant
        self.__contravariant__ = contravariant  # True для contravariant
        
# T_co = TypeVar('T_co', covariant=True)
# → T_co.__covariant__ = True
```

```python
from typing import TypeVar

T_co = TypeVar('T_co', covariant=True)  # variance=1
T_inv = TypeVar('T_inv')  # variance=0 (по умолчанию)
```

В CPython это **PyTypeVarObject** (внутреннее представление):

```c
typedef struct {
    PyObject_HEAD
    PyObject *tv_name;             // 'T_co'
    int tv_variance;               // 1=covariant, 0=invariant, -1=contravariant
    PyObject *tv_bound;            // ограничение типа
} PyTypeVarObject;
```

`TypeVar('T', covariant=True)` создаёт объект с флагом `tv_variance=1`. При подстановке в `list[T_co]` этот флаг
копируется в `ga_variance`.

## 4. Подстановка параметров: list[T_co][Animal] → list[Cat]

```c
// Objects/genericaliasobject.c — list[T_co][Animal]
PyObject *
generic_alias_subscript(PyGenericAliasObject *ga, PyObject *item)
{
    // ga = list[T_co], item = Animal
    PyObject *params = ga->ga_params;  // (T_co,)
    PyObject *tv = PyTuple_GET_ITEM(params, 0);  // T_co
    
    // подставляем T_co = Animal
    PyObject *sub = PyGenericAlias_Substitute(ga->ga_origin, params, item);
    return sub;  // list[Animal]
}
```

`list[T_co][Animal]` **НЕ** `list[Animal]` напрямую. Сначала `T_co` замещается на `Animal`, **с учётом variance**.
Runtime **не проверяет** `Cat <: Animal`.

## 5. Variance в isinstance() и issubclass() (ограниченная проверка)

```c
// Objects/abstract.c — isinstance(obj, list[int])
int
PyObject_IsInstance(PyObject *inst, PyObject *cls)
{
    // если cls — generic alias (list[int])
    if (PyGenericAlias_CheckExact(cls)) {
        PyGenericAliasObject *ga = (PyGenericAliasObject*)cls;
        PyObject *origin = ga->ga_origin;  // list
        
        // рекурсивно: isinstance(obj, list) И params совместимы
        int res = PyObject_IsInstance(inst, origin);
        if (res == 0)
            return 0;
            
        // ПРОВЕРКА VARIANCE ТОЛЬКО ДЛЯ BUILTIN!
        return generic_alias_isinstance(inst, ga);
    }
}
```

`isinstance(x, list[int])` работает **только для builtin** (`list`, `tuple`). Для пользовательских классов **игнорирует
** параметры. **НЕ** проверяет `Cat <: Animal`!

## 6. Проверка variance в generic_alias_isinstance()

```c
// Objects/genericaliasobject.c (упрощённо)
static int
generic_alias_isinstance(PyObject *inst, PyGenericAliasObject *ga)
{
    PyObject *origin = ga->ga_origin;      // list
    PyObject *inst_type = Py_TYPE(inst);   // type(x)
    
    // 1. x должен быть list
    if (!PyObject_IsInstance(inst, origin))
        return 0;
    
    // 2. для list[T_co] где T_co covariant:
    //    Cat <: Animal → list[Cat] OK для list[Animal]
    PyObject *params = ga->ga_params;      // (Animal,)
    unsigned char variance = ga->ga_variance;  // 1=covariant
    
    if (variance == 1) {  // covariant
        // проверим подтипность параметров
        return check_covariant_params(inst_type, params);
    }
    
    // invariant: точное совпадение
    return check_exact_params(inst_type, params);
}
```

**Runtime проверка** работает **только для covariant builtin** типа `list[int]`. `list[Cat](cat)` **проходит**
`isinstance(cat, list[Animal])` **если** `Cat <: Animal`. **НЕ работает** для пользовательских классов!

## 7. __class_getitem__ для пользовательских generic (PEP 585, 3.9+)

```c
// Objects/typeobject.c — MyGeneric[int]
static PyObject *
type_subscript_user_generic(PyTypeObject *type, PyObject *item)
{
    // пользовательский класс с Generic[T]
    if (has_generic_base(type)) {
        // возвращаем GenericAlias с origin=type
        return PyGenericAlias_New((PyObject*)type, PyTuple_New(1, item), NULL);
    }
}
```

```python
from typing import Generic, TypeVar

T = TypeVar('T', covariant=True)


class Box(Generic[T]):  # Box[int]
    pass
```

Пользовательский `Box[int]` тоже `PyGenericAliasObject` с `ga_origin=Box`. **Но** `isinstance(x, Box[int])` **НЕ
РАБОТАЕТ** — только статическая проверка в mypy!

## 8. Байткод работы с generic alias

```python
from typing import List

x: List[int] = [1, 2, 3]
```

```
# Байткод (аннотации):
  0 LOAD_GLOBAL          0 (List)
  2 LOAD_GLOBAL          1 (int)
  4 CALL_FUNCTION        1          # List[int] ← __class_getitem__
  6 LOAD_CONST           0 ([])     # дефолт []
  8 STORE_FAST           0 (x)
```

`List[int]` в байткоде — **обычный вызов** `List.__class_getitem__(int)` → `PyGenericAliasObject`. **Runtime** не
проверяет типы в списке!

## 9. Variance в type checker (mypy, НЕ CPython!)

```python
T_co = TypeVar('T_co', covariant=True)


class Box(Generic[T_co]): pass


def accept_box(box: Box[Animal]): ...


b: Box[Cat] = Box()  # Cat <: Animal
accept_box(b)  # OK! covariant
```

**Mypy логика (псевдокод):**

```
if Box.ga_variance == COVARIANT and issubclass(Cat, Animal):
    OK: Box[Cat] <: Box[Animal]
else:
    Error!
```

**CPython runtime** хранит `ga_variance`, **НО** проверку **НЕ делает**. `isinstance()` работает только для **builtin
covariant** коллекций. Полная проверка **только в type checker** (mypy, pyright).

## Итог инвариантности/ковариантности в CPython 3.9+

1. **Хранение**: `PyGenericAliasObject.ga_variance` из `TypeVar(covariant=True)`.
2. **Создание**: `list[int]` → `__class_getitem__` → `PyGenericAlias_New()`.
3. **Runtime**: `isinstance()` **только для builtin** (`list`, `tuple`), **игнорирует** пользовательские классы.
4. **Type checking**: **mypy** анализирует `ga_variance` + подтипность параметров **статически**.

**НЕ путай**: variance — это **информация для статических анализаторов**, **НЕ runtime проверка типов**!

- [Содержание](/CONTENTS.md#содержание)