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

[Содержание](/CONTENTS.md#содержание)

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

[Содержание](/CONTENTS.md#содержание)

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

| Тип связности          | Реализация                | Характеристика                   |
|------------------------|---------------------------|----------------------------------|
| **Высокая (cohesion)** | `PyTypeObject` монолит    | 60+ слотов в одном объекте       |
| **Слабая (loose)**     | `Py_DECREF` независимо    | refcnt==0 → немедленное удаление |
| **Циклическая**        | `gc_refs` в GC            | `a→b→a` → цикл-детекция          |
| **Модульная**          | `MD_STATE_IMPORTING`      | `import a→b→a` → ImportError     |
| **Структурная**        | `__slots__` vs `__dict__` | фиксированные поля vs хеш        |
| **Наследственная**     | `tp_mro` C3               | монотонный порядок               |
| **Слотовая**           | `inherit_slots()`         | копирование указателей           |

**CPython** балансирует: **высокая связность внутри типов**, **слабая между объектами**, **циклы** — GC, **модули** —
import-граф!

- [Содержание](/CONTENTS.md#содержание)