# **Diamond Problem**

[Содержание](/CONTENTS.md#содержание)

## **Junior Level**

Проблема ромбовидного наследования (Diamond Problem) — это классическая проблема в объектно-ориентированном
программировании, возникающая при множественном наследовании.
Представьте себе ромб (алмаз), где вершина — это базовый класс A, от него наследуются
два класса B и C, а от обоих наследуется класс D. Если в классе A есть метод `some_method()`, и классы B и C
переопределяют его по-разному, то возникает вопрос: какую реализацию унаследует класс D — от B или от C?

Эта проблема создает неоднозначность в поведении программы. В Python эта проблема решается с помощью четко определенного
**порядка разрешения методов (MRO - Method Resolution Order)**, который определяет, в каком порядке интерпретатор будет
искать метод в иерархии наследования.

[Содержание](/CONTENTS.md#содержание)

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

[Содержание](/CONTENTS.md#содержание)

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

- [Содержание](/CONTENTS.md#содержание)