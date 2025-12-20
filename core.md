[Содержание](/CONTENTS.md#содержание)
---

# **Декораторы и замыкания**

[Содержание](/CONTENTS.md#содержание)

## **Junior Level**

Представьте, что у вас есть функция — допустим, она просто складывает два числа. И вдруг вам понадобилось каждый раз,
когда её вызывают, записывать в лог, что произошел вызов.
Можно, конечно, пойти и добавить в саму функцию строку с `print`, но что если таких функций много? Или если вы хотите то
включать логирование, то выключать?

Вот здесь и появляются декораторы. По сути, декоратор — это функция, которая принимает вашу функцию, оборачивает её в
дополнительную логику и возвращает новую, улучшенную версию. А синтаксис с собачкой (`@`) — это просто красивый способ
сказать Python: "Эй, примени этот декоратор к следующей функции".

А замыкания — это то, что делает декораторы возможными. Если очень просто: замыкание — это функция внутри функции,
которая помнит переменные из внешней функции даже после того, как та завершилась. Как будто у неё есть память. Например,
можно создать функцию-счетчик, которая будет помнить, сколько раз её вызвали.

[Содержание](/CONTENTS.md#содержание)

## **Middle Level**

Декоратор — это по сути паттерн проектирования, который позволяет добавлять поведение к функциям или методам,
не изменяя их исходный код. Это мощный инструмент для разделения ответственности.

Если декоратору нужно передать параметры — например, указать уровень логирования или количество повторов — структура
усложняется. Мы получаем как бы "фабрику декораторов": функция принимает параметры и возвращает уже сам декоратор,
который будет применен к целевой функции.

Когда вы видите несколько декораторов над одной функцией, важно помнить, что они применяются снизу вверх. Тот декоратор,
что ближе к функции, сработает первым, затем следующий и так далее. Это похоже на матрешку — каждый декоратор добавляет
свой слой обертки.

Один из самых важных моментов, который часто упускают — это сохранение метаданных функции. Когда декоратор создает новую
функцию-обертку, она теряет оригинальное имя, документацию и другие атрибуты. Именно поэтому мы всегда используем
`@wraps` из модуля `functools` — он копирует эти метаданные, что критически важно для отладки и работы многих
инструментов.

Декоратором может быть не только функция, но и класс. Для этого класс должен реализовать метод `__call__`, который
делает экземпляр вызываемым. Классы-декораторы особенно удобны, когда нужно сохранять какое-то состояние между
вызовами — например, вести счетчик или кэшировать результаты.

Теперь о замыканиях и областях видимости. Python использует правило LEGB: сначала ищет переменную локально, затем в
замыкающих функциях, потом глобально и наконец во встроенных именах. Замыкания работают благодаря доступу к внешним
областям. Но если вы хотите изменить переменную из замыкания, а не просто прочитать её, нужно использовать ключевое
слово `nonlocal`. Без него Python создаст новую локальную переменную, что обычно приводит к ошибкам.

На практике декораторы и замыкания находят огромное применение. В тестировании, например, `@pytest.mark.parametrize`
позволяет запускать один тест с разными данными, а `@unittest.mock.patch` временно подменяет объекты для изоляции
тестов. Встроенные декораторы Python вроде `@property`, `@classmethod` и `@staticmethod` активно используются в ООП.

Но важно помнить и об ограничениях. Декораторы добавляют накладные расходы — каждый вызов проходит через дополнительную
функцию. В критически важных по производительности местах это может иметь значение. Также цепочки декораторов могут
усложнять отладку, делая стек вызовов очень глубоким.

По сути, декораторы и замыкания — это инструменты, которые помогают нам писать более чистый, модульный и
переиспользуемый код. Они позволяют отделять сквозную функциональность вроде логирования, кэширования или проверки прав
от основной бизнес-логики, что соответствует принципам хорошего проектирования.

[Содержание](/CONTENTS.md#содержание)

## **Senior Level**

В CPython 3.9+ **декораторы** — это **PyFunctionObject** с `func_closure` (tuple of PyCellObject), **замыкания** —
`co_cellvars`/`co_freevars` в PyCodeObject + байткоды `LOAD_CLOSURE`/`LOAD_DEREF`/
`STORE_DEREF`. `Objects/funcobject.c`,`Python/compile.c`,`Python/ceval.c`

## 1. PyFunctionObject с замыканием (3.9+)

```c
typedef struct {
    PyObject_HEAD                  // Стандартный заголовок
    PyObject *func_globals;        // globals() при создании
    PyObject *func_builtins;       // builtins при создании  
    PyObject *func_code;           // PyCodeObject с co_freevars
    PyObject **func_closure;       // NULL или tuple PyCellObject*
    Py_ssize_t func_nfreevars;     // Длина closure (co_freevars)
    PyObject *func_defaults;       // (a=1, b=2)
    PyObject *func_kwdefaults;     // {c: 3}
    PyObject *func_doc;            // __doc__
    PyObject *func_name;           // __name__
    PyObject *func_qualname;       // __qualname__
    PyObject *func_dict;           // __dict__
    PyObject *func_weakreflist;    // Слабые ссылки
    vectorcallfunc vectorcall;     // Быстрый вызов
} PyFunctionObject;
```

Функция с замыканием содержит `func_closure` — массив **ячеек** (PyCellObject), ссылающихся на
переменные внешней функции. `func_nfreevars` говорит, сколько их.

## 2. PyCellObject - "ячейка" замыкания

```c
typedef struct {
    PyObject_HEAD                 // refcnt + type=&PyCell_Type
    PyObject *ob_ref;             // Указатель на значение переменной
} PyCellObject;
```

**Ячейка** — это обёртка над PyObject*. Внешняя функция пишет `x=42` →
`PyCell_SET(cell, PyLong(42))`. Внутренняя читает `PyCell_GET(cell)`.

## 3. PyCodeObject: cellvars/freevars (компиляция)

```c
typedef struct _PyCodeObject {
    // ...
    PyObject *co_cellvars;         // ('x',) - локальные внешней функции
    PyObject *co_freevars;         // ('x',) - свободные внутренней
    // ...
} PyCodeObject;
```

Компилятор помечает переменные: `co_cellvars` — в **внешней** функции (нужны для вложенных),
`co_freevars` — в **внутренней** (ссылается на внешние).

**Пример компиляции:**

```python
def outer():
    x = 1

    def inner():  # co_cellvars=['x']
        return x  # co_freevars=['x']

    return inner
```

```
outer.__code__.co_cellvars     # ('x',)
inner.__code__.co_freevars     # ('x',)
inner.__closure__              # (<cell at 0x...: int object at 0x...>,)
```

## 4. Байткод: LOAD_CLOSURE/STORE_DEREF (ceval.c)

**Внешняя функция (outer):**

```
# Байткод outer():
  0 LOAD_CONST           0 (1)         # x = 1
  2 STORE_DEREF          0 (x)         # Сохраняем в ячейку #0
  4 LOAD_CLOSURE         0 (x)         # Берём ячейку #0
  6 BUILD_TUPLE          1             # (cell,)
  8 LOAD_CONST           1 (<code>)    # PyCodeObject inner
 10 MAKE_CLOSURE         1             # PyFunctionObject(closure=(cell,))
 12 RETURN_VALUE                    # return inner
```

**Внутренняя функция (inner):**

```
# Байткод inner():
  0 LOAD_DEREF           0 (x)         # Читаем из ячейки #0
  2 RETURN_VALUE                    # return x
```

`STORE_DEREF 0` пишет в ячейку #0. `LOAD_CLOSURE 0` берёт **саму ячейку**,
`BUILD_TUPLE/MAKE_CLOSURE` создаёт функцию с `func_closure=(cell0,)`. `LOAD_DEREF 0` внутри читает из той же ячейки.

## 5. LOAD_CLOSURE байткод (ceval.c)

```c
case LOAD_CLOSURE: {
    PyObject *cell = PyGenObject_GET_CLOSURE(frame, oparg);  // Ячейка из co_freevars[oparg]
    if (cell == NULL) {
        goto unbound_error;            // Unbound closure variable
    }
    
    Py_INCREF(cell);                   // +refcnt на PyCellObject
    STACK_GROW(1);
    PEEK(0) = cell;                    // Ячейка на стек
    DISPATCH();
}
```

`LOAD_CLOSURE 0` берёт ячейку из массива замыкания функции (`func_closure[0]`), кладёт *
*ячейку** на стек (не значение!).

## 6. STORE_DEREF / LOAD_DEREF (ceval.c)

```c
case STORE_DEREF: {
    PyObject *v = POP();               // Значение со стека
    PyObject *cell = PyGenObject_GET_CLOSURE(frame, oparg);  // Ячейка
    if (cell == NULL) {
        goto unbound_error;
    }
    
    PyCell_SET(cell, v);               // Записываем в ob_ref ячейки
    Py_DECREF(v);                      // Освобождаем значение
    DISPATCH();
}

case LOAD_DEREF: {
    PyObject *cell = PyGenObject_GET_CLOSURE(frame, oparg);
    if (cell == NULL) {
        goto unbound_error;
    }
    
    PyObject *value = PyCell_GET(cell);  // Читаем ob_ref из ячейки
    if (value == NULL) {
        goto unbound_error;
    }
    
    Py_INCREF(value);                  // +refcnt значения
    PUSH(value);                       // Значение на стек
    DISPATCH();
}
```

`STORE_DEREF` берёт значение со стека → `PyCell_SET(cell, value)` (меняет `cell->ob_ref`).
`LOAD_DEREF` берёт `PyCell_GET(cell)` → значение на стек.

## 7. MAKE_CLOSURE байткод (ceval.c)

```c
case MAKE_CLOSURE: {
    Py_ssize_t free_n = POP();         // Количество свободных переменных
    PyCodeObject *code = POP();        // PyCodeObject
    PyObject *closure_tuple = POP();   // (cell1, cell2, ...)
    
    assert(PyTuple_Check(closure_tuple));
    assert(PyTuple_GET_SIZE(closure_tuple) == free_n);
    
    // Создаём PyFunctionObject
    PyFunctionObject *func = PyFunction_New(code, frame->f_globals);
    if (func == NULL) {
        goto error;
    }
    
    // Прикрепляем замыкание
    func->func_closure = closure_tuple;  // Сохраняем tuple ячеек
    Py_INCREF(closure_tuple);            // +refcnt
    func->func_nfreevars = free_n;
    
    PUSH((PyObject *)func);            // Функция на стек
    DISPATCH();
}
```

`MAKE_CLOSURE 1` берёт `(cell0,)`, PyCodeObject → создаёт PyFunctionObject →
`func_closure=(cell0,)`, `func_nfreevars=1`.

## 8. PyCellObject операции

```c
void PyCell_Set(PyObject *cell, PyObject *value) {
    if (!PyCell_Check(cell)) {
        PyErr_BadInternalCall();
        return;
    }
    
    Py_XSETREF(((PyCellObject *)cell)->ob_ref, Py_NewRef(value));
}

PyObject *PyCell_Get(PyObject *cell) {
    if (!PyCell_Check(cell)) {
        PyErr_BadInternalCall();
        return NULL;
    }
    
    PyObject *value = ((PyCellObject *)cell)->ob_ref;
    if (value == NULL) {
        PyErr_SetString(PyExc_UnboundLocalError, "unbound variable");
        return NULL;
    }
    
    return Py_NewRef(value);
}
```

`PyCell_Set(cell, 42)` → `cell->ob_ref = PyLong(42)`. `PyCell_Get(cell)` → возвращает
`cell->ob_ref` или UnboundLocalError.

## 9. Компиляция декоратора (compile.c)

```c
// В Python/compile.c при парсинге def inner():
if (symtable_lookup(st, name) >= 0 && 
    symtable_lookup(st, name) != LOCAL) {
    // Переменная из внешней области -> cellvar
    symtable->st_cellvars[num_cellvars++] = name;
    code->co_cellvars = PyTuple_New(num_cellvars);
}

// Для внутренней функции:
if (name in enclosing_cellvars) {
    // Свободная переменная
    code->co_freevars[num_freevars++] = name;
}
```

Компилятор сканирует AST: если `inner()` читает `x` из `outer()` → `x` становится **cellvar**
во внешней, **freevar** во внутренней.

## 10. Декоратор: @decorator(f)

```python
def decorator(func):
    def wrapper(*args, **kwargs):
        print("before")
        result = func(*args, **kwargs)
        print("after")
        return result

    return wrapper


@decorator
def f(): pass
```

**Байткод компилируется как:**

```python
# Эквивалентно:
f = decorator(f)
```

```
LOAD_GLOBAL       decorator
LOAD_NAME         f
CALL_FUNCTION     1
STORE_NAME        f
```

Декоратор — это **функция**, возвращающая **другую функцию** с замыканием на оригинальную
`func`. `@decorator` → `decorator(f)` во время выполнения модуля.

**Декораторы/замыкания** в CPython 3.9+ — **PyCellObject** (`ob_ref`), `co_cellvars`/`co_freevars`, байткоды
`LOAD_CLOSURE`/`STORE_DEREF`/`MAKE_CLOSURE`, `func_closure` tuple в PyFunctionObject.

- [Содержание](/CONTENTS.md#содержание)

---

# **GIL (Global Interpreter Lock)**

[Содержание](/CONTENTS.md#содержание)

## **Junior Level**

Представьте, что у вас есть большой офис с несколькими сотрудниками (это потоки) и всего один принтер (это интерпретатор
Python). Все сотрудники могут готовить документы одновременно, но печатать они могут только по очереди — когда принтер
свободен. GIL — это как очередь к этому единственному принтеру.

Технически GIL — это глобальная блокировка в CPython (стандартной реализации Python), которая позволяет выполнять только
одному потоку Python-кода за раз, даже если у вас многоядерный процессор. Это значит, что для задач, которые сильно
нагружают процессор (например, сложные вычисления), многопоточность в Python не даст ускорения. Все потоки будут по
очереди работать на одном ядре.

Но есть важный нюанс. Во время операций ввода-вывода — когда программа ждет ответа от сети, читает файл или общается с
базой данных — поток освобождает GIL, позволяя другим потокам работать. Поэтому для I/O-задач (например, веб-серверов)
многопоточность всё равно полезна.

[Содержание](/CONTENTS.md#содержание)

## **Middle Level**

GIL (Global Interpreter Lock) — это мьютекс (mutex), блокирующий одновременное выполнение нескольких потоков в CPython.
Только один поток может владеть GIL в каждый момент времени, что делает любую многопоточную Python-программу фактически
однопоточной при выполнении байткода.

GIL был введён для упрощения управления памятью CPython и обеспечения интеграции с C-расширениями. Система подсчёта
ссылок (reference counting) в CPython не является потокобезопасной: без защиты несколько потоков могут одновременно
изменять счётчик ссылок объекта, что приводит к утечкам памяти или преждевременному удалению объектов.

Альтернативой GIL могла бы быть установка блокировок на каждый объект Python, но это привело бы к:

- Значительному overhead на множественные блокировки/разблокировки при каждой операции
- Высокому риску взаимоблокировок (deadlocks)
- Снижению производительности однопоточных программ

Вместо этого CPython использует единую глобальную блокировку интерпретатора, что исключает взаимоблокировки и
минимизирует влияние на производительность однопоточного кода.

Механизм работы GIL довольно интересен. Поток удерживает GIL не бесконечно — есть несколько условий, при которых
происходит переключение:

1. **Через определённое количество тиков байт-кода** (обычно каждые 100 инструкций)
2. **При операциях ввода-вывода** — когда поток уходит в ожидание
3. **При явном освобождении** в C-расширениях

Это создаёт некоторые проблемы. Например, если у вас есть поток, который выполняет долгие вычисления без операций
ввода-вывода, он может долго не отдавать GIL, и другие потоки будут простаивать. Это называется "голоданием" потоков.

На практике это означает, что для CPU-интенсивных задач нужно использовать другие подходы. Самый распространённый — *
*многопроцессорность** через модуль `multiprocessing`. Каждый процесс получает свой интерпретатор Python со своим GIL, и
они действительно могут работать параллельно на разных ядрах.

Есть и другие способы обойти ограничения GIL. **Асинхронное программирование** с `asyncio` позволяет эффективно работать
с I/O-задачами в одном потоке. **C-расширения** могут временно освобождать GIL во время вычислительных операций — так
работают библиотеки типа NumPy и SciPy. И существуют альтернативные реализации Python, такие как Jython или IronPython,
где GIL вообще отсутствует, но они имеют свои ограничения.

Важно понимать, что GIL — это особенность именно CPython, и у него есть свои причины для существования. Он упрощает
реализацию интерпретатора и делает более предсказуемой работу с памятью. Для многих реальных задач — веб-серверов,
скриптов автоматизации, работы с базами данных — GIL не является узким местом. Проблемы возникают в основном в научных
вычислениях и высоконагруженных вычислительных задачах, где как раз и используются специализированные библиотеки и
подходы.

Понимание GIL помогает выбрать правильную архитектуру для приложения. Если задача
CPU-интенсивная — смотрим в сторону многопроцессорности или выноса вычислений в C-расширения. Если I/O-интенсивная —
можно использовать потоки, асинхронное программирование или комбинацию подходов. GIL — это не приговор, а особенность,
которую нужно учитывать при проектировании.

[Содержание](/CONTENTS.md#содержание)

## **Senior Level**

В CPython 3.9+ **GIL** (Global Interpreter Lock) — это **мьютекс** `gil->mutex` + **счётчик** `gil->recursion_count` + *
*состояние потока** `tstate->holds_gil` в `_gil_runtime_state`. **PEP 684** (3.12) добавил **per-interpreter GIL**. *
*Free-threaded** (3.13) — скомпилировано с `Py_GIL_DISABLED`. `Python/ceval_gil.c`,`Python/pythonrun.c`

## 1. _gil_runtime_state (Python/ceval_gil.c)

```c
struct _gil_runtime_state {
    _Py_atomic_int locked;             // 1=GIL занят, 0=свободен (атомарно)
    _Py_atomic_int recursion_count;    // Счётчик вложенных захватов
    PyThread_type_lock mutex;          // pthread_mutex_t или Windows CRITICAL_SECTION
    PyThread_cond_t cond;              // pthread_cond_t для ожидания
    PyThread_t owner;                  // ID потока-владельца
    int switch_interval;               // Интервал принудительного drop_gil
    uint64_t last_switch_time;         // Время последнего drop_gil
#ifdef Py_GIL_DISABLED
    int enabled;                       // 0=GIL отключен полностью
#endif
};
```

GIL — это **один глобальный мьютекс** + **счётчик рекурсии** (один поток может захватить много
раз). `owner` — ID потока, который держит GIL.

## 2. PyEval_AcquireLock / take_gil() — захват GIL

```c
void _PyEval_AcquireLock(PyThreadState *tstate) {
    _Py_EnsureTstateNotNULL(tstate);   // tstate не NULL
    take_gil(tstate);                  // Захватываем GIL
}

static void take_gil(PyThreadState *tstate) {
    struct _gil_runtime_state *gil = &tstate->interp->ceval.gil;
    
    // Атомарно проверяем, свободен ли GIL
    if (_Py_atomic_load_int_relaxed(&gil->locked)) {
        // GIL занят — ждём
        MUTEX_LOCK(gil->mutex);
        while (_Py_atomic_load_int_relaxed(&gil->locked)) {
            COND_WAIT(gil->cond, gil->mutex);  // pthread_cond_wait
        }
        MUTEX_UNLOCK(gil->mutex);
    }
    
    // Атомарно захватываем GIL
    _Py_atomic_store_int_relaxed(&gil->locked, 1);
    
    // Устанавливаем владельца
    gil->owner = PyThread_get_thread_ident();
    gil->recursion_count = 1;
    
    // Отмечаем поток как держателя GIL
    tstate->holds_gil = 1;
    
    // Обновляем eval_breaker (прерывания)
    update_eval_breaker_for_thread(tstate->interp, tstate);
}
```

Поток проверяет `gil->locked` атомарно. Если 1 — **ждёт** на condition variable. Захватывает →
`locked=1`, `owner=мой_ID`, `tstate->holds_gil=1`.

## 3. PyEval_ReleaseLock / drop_gil() — освобождение GIL

```c
void _PyEval_ReleaseLock(PyThreadState *tstate) {
    drop_gil(tstate->interp, tstate, 0);  // 0=не финальное освобождение
}

static void drop_gil(PyInterpreterState *interp, PyThreadState *tstate, int final) {
    struct _gil_runtime_state *gil = &interp->ceval.gil;
    
#ifdef Py_GIL_DISABLED
    if (!gil->enabled) {
        return;                        // GIL отключен — выходим
    }
#endif
    
    // Проверяем, что мы владелец
    if (!_Py_atomic_load_int_relaxed(&gil->locked) || 
        gil->owner != PyThread_get_thread_ident()) {
        Py_FatalError("drop_gil: GIL is not locked");
    }
    
    // Уменьшаем счётчик рекурсии
    if (--gil->recursion_count > 0) {
        return;                        // Ещё вложенные захваты
    }
    
    // Сбрасываем флаги потока
    tstate->holds_gil = 0;
    
    // Освобождаем GIL атомарно
    _Py_atomic_store_int_release(&gil->locked, 0);
    
    // Разбудить ждущие потоки
    MUTEX_LOCK(gil->mutex);
    PyThread_cond_broadcast(gil->cond);  // pthread_cond_broadcast
    MUTEX_UNLOCK(gil->mutex);
}
```

`--recursion_count`. Если >0 — остаёмся владельцем. Иначе `holds_gil=0`, `locked=0`, **будим
ВСЕ** ждущие потоки (`broadcast`).

## 4. Автоматический drop_gil в ceval.c (каждые N инструкций)

```c
#define INSTRUCTION_COUNTER() \
    if (--tstate->cframe->instr_counter == 0) { \
        tstate->cframe->instr_counter = INSTR_COUNTER_STEP; \
        _PyEval_SignalAsyncioEventLoop(tstate); \
    }

#define PyEval_EvalFrameDefault _PyEval_EvalFrameDefault

static inline void
frame_insn_counter(PyThreadState *tstate, _PyInterpreterFrame *frame) {
    if (_Py_atomic_load_relaxed(&tstate->gilstate_counter) == 0) {
        // GIL счётчик истёк — пробуем освободить
        _PyEval_ReleaseLock(tstate);
        _PyEval_AcquireLock(tstate);
    }
}
```

Каждые ~1000 инструкций байткода проверяется `gilstate_counter`. Если 0 — **drop_gil() +
take_gil()** (шанс другому потоку).

## 5. PyGILState_Ensure/Release — C API

```c
PyGILState_STATE PyGILState_Ensure(void) {
    PyThreadState *tstate = PyThreadState_Get();  // Текущий поток
    
    if (tstate == NULL) {
        tstate = _PyThreadState_GetUnattached();  // Создаём новый
        if (tstate == NULL) {
            Py_FatalError("PyGILState_Ensure: no thread state");
        }
    }
    
    // Атомарно увеличиваем счётчик
    int gilstate_counter = _Py_atomic_fetch_add_int_relaxed(
        &tstate->interp->ceval.gil.gilstate_counter, 1);
    
    if (gilstate_counter == -1) {
        // Первый захват — берём GIL
        _PyEval_AcquireLock(tstate);
        return PyGILState_LOCKED;
    }
    
    return PyGILState_UNLOCKED;
}

void PyGILState_Release(PyGILState_STATE oldstate) {
    PyThreadState *tstate = PyThreadState_Get();
    
    // Атомарно уменьшаем счётчик
    int gilstate_counter = _Py_atomic_fetch_sub_int_relaxed(
        &tstate->interp->ceval.gil.gilstate_counter, 1);
    
    if (gilstate_counter == 0) {
        // Последний — освобождаем GIL
        _PyEval_ReleaseLock(tstate);
    }
}
```

C-расширения вызывают `PyGILState_Ensure()` → атомарно `++gilstate_counter`. Если был -1 →
захват GIL. `Release()` → `--counter`, если 0 → drop_gil.

## 6. Per-interpreter GIL (PEP 684, 3.12+)

```c
// Каждый PyInterpreterState имеет свой GIL
struct _ceval_state {
    struct _gil_runtime_state gil;     // GIL состояния интерпретатора
    int own_gil;                       // Этот интерпретатор владеет GIL
    // ...
};

PyInterpreterState *PyInterpreterState_New(void) {
    PyInterpreterState *interp = PyMem_Calloc(1, sizeof(*interp));
    init_own_gil(interp, &interp->ceval.gil);  // Создаём GIL для интерпретатора
    return interp;
}
```

**PEP 684**: каждый subinterpreter имеет **свой GIL**. `Py_NewInterpreter()` создаёт отдельный
`interp->ceval.gil`.

## 7. Free-threaded CPython (PEP 703, 3.13+)

```c
#ifdef Py_GIL_DISABLED
static inline void take_gil(PyThreadState *tstate) {
    struct _gil_runtime_state *gil = &tstate->interp->ceval.gil;
    if (!gil->enabled) {               // GIL отключен
        tstate->holds_gil = 1;
        return;
    }
    // Обычная логика захвата
}
#endif
```

`--disable-gil` компиляция: `gil->enabled=0`. Захват/освобождение — **no-op**. Потоки работают
**параллельно**.

## 8. Eval breaker integration

```c
void _PyEval_SignalAsyncioEventLoop(PyThreadState *tstate) {
    struct _ceval_state *ceval = &tstate->interp->ceval;
    
    if (ceval->gil.enabled && tstate->holds_gil) {
        // Устанавливаем бит "drop GIL request"
        _Py_atomic_store_relaxed(&ceval->eval_breaker, _PY_EVAL_BREAKER_DROP_GIL);
    }
}
```

`asyncio`/`signal` сигнализируют через `eval_breaker`. Если держим GIL — бит
`_PY_EVAL_BREAKER_DROP_GIL` → следующий `drop_gil()` прерывается.

## 9. Байткод без GIL (3.13 free-threaded)

```c
case LOAD_GLOBAL: {
#ifdef Py_GIL_DISABLED
    if (!tstate->holds_gil) {
        // Без GIL — атомарный поиск
        res = _PyDict_LookupWithCache(global_dict, name, hash);
    } else {
        // С GIL — обычный поиск
        res = PyDict_GetItemWithCache(global_dict, name, hash);
    }
#else
    // GIL версия
#endif
}
```

Free-threaded использует **атомарные** `PyDict_LookupWithCache`. GIL версия — обычный поиск.

**GIL** в CPython 3.9+ — **_gil_runtime_state** (`mutex + recursion_count`), **take_gil/drop_gil**, **PyGILState_Ensure
** (C API), **per-interp GIL** (3.12), **free-threaded** (`Py_GIL_DISABLED`, 3.13).

- [Содержание](/CONTENTS.md#содержание)

---

# **Изменение коллекции во время итерации**

[Содержание](/CONTENTS.md#содержание)

## **Junior Level**

Изменять коллекцию во время перебора её элементов — это как перестраивать комнату, пока вы в ней находитесь. Вы можете
споткнуться о перемещённую мебель или вовсе оказаться в совершенно другом пространстве. В программировании эта операция
нарушает внутреннюю логику работы итератора и ведёт к непредсказуемым последствиям: пропуску элементов, их двойной
обработке, бесконечным циклам или ошибкам выполнения.

Представьте, что вы экскурсовод, ведущий группу по постоянно меняющемуся музею. Если залы начинают исчезать или
появляться во время экскурсии, ваш маршрут и рассказ мгновенно теряют смысл. Так же и итератор, который хранит текущую
позицию в коллекции, перестаёт корректно работать, когда основание под ним сдвигается.

Это правило касается всех изменяемых коллекций — списков, словарей, множеств. Для безопасной модификации нужно либо
итерироваться по копии коллекции, либо сначала собрать все необходимые изменения, а затем применить их к оригиналу.

[Содержание](/CONTENTS.md#содержание)

## **Middle Level**

Когда вы создаёте цикл `for item in collection:`, Python создаёт объект-итератор, который становится проводником по
вашей коллекции. Этот проводник запоминает текущее положение и следует определённому маршруту. Изменение коллекции во
время такого «путешествия» сбивает все ориентиры.

Механизм итерации устроен так, что итератор хранит внутреннее состояние — текущую позицию. Для списков это индекс
элемента, для словарей и множеств — более сложные структуры, отслеживающие хеш-таблицы. При удалении или добавлении
элементов исходная коллекция меняет свою организацию, но итератор продолжает следовать старому плану, что приводит к
логическим противоречиям.

Интересно, что разные коллекции в Python реагируют на такие изменения по-разному. Словари и множества, начиная с Python
3.7, при обнаружении изменения размера во время итерации вызывают явное исключение `RuntimeError`, предупреждая
программиста о проблеме. Списки же, в силу своей индексной природы, позволяют это делать, но последствия могут быть
особенно коварными — код может работать с ошибками, которые сложно воспроизвести и отладить.

Для разных коллекций существуют свои безопасные паттерны. Со списками часто работает итерация по копии, созданной через
`list()` или срез `[:]`. Для словарей можно итерироваться по списку ключей, предварительно полученных через
`list(dict.keys())`. Множества также требуют создания копии перед модификацией во время итерации.

Универсальный подход — разделить фазу анализа коллекции и фазу её изменения. Сначала соберите всю необходимую
информацию (например, какие элементы нужно удалить или добавить), сохранив её во временной структуре, а затем отдельным
действием примените все изменения к исходной коллекции. Этот метод не только безопасен, но и делает код более понятным,
поскольку чётко разделяет ответственность между этапами обработки данных.

[Содержание](/CONTENTS.md#содержание)

## **Senior Level**

В CPython 3.9+ **изменение коллекции во время итерации** детектируется через **version tag** (`ma_version_tag` в
PyDictObject, `ob_version` в PyListObject) + **итераторное состояние** (`it_version`/`it_index`). **RuntimeError** при
несоответствии версий. `Objects/listobject.c`,`Objects/dictobject.c`,`Objects/iterobject.c`

## 1. PyListIterObject с версией (list_iter)

```c
typedef struct {
    PyObject_HEAD
    Py_ssize_t it_index;           // Текущий индекс
    PyListObject *it_seq;          // Ссылка на список
    uint32_t it_version;           // Копия ob_version списка при создании
} PyListIterObject;
```

Итератор списка копирует `list->ob_version` при создании. Каждый `next()` проверяет версии —
если список изменился → **RuntimeError**.

## 2. list_iter_next() - проверка версии

```c
static PyObject *list_iter_next(PyListIterObject *it) {
    PyListObject *list = it->it_seq;   // Ссылка на список
    Py_ssize_t index = it->it_index;   // Текущий индекс
    
    // КРИТИЧЕСКАЯ ПРОВЕРКА ВЕРСИИ
    if (it->it_version != list->ob_version) {
        PyErr_SetString(PyExc_RuntimeError, 
            "list changed during iteration");
        return NULL;                   // RuntimeError!
    }
    
    if (index >= Py_SIZE(list)) {
        return NULL;                   // Конец -> StopIteration
    }
    
    PyObject *item = Py_NewRef(list->ob_item[index]);
    it->it_index++;                    // ++ индекс
    return item;
}
```

Перед чтением элемента сравниваем `it_version` (копия при создании) с `list->ob_version` (
текущее). Изменился → **мгновенный RuntimeError**.

## 3. list_ob_version - счётчик изменений PyListObject

```c
uint32_t list_ob_version(PyListObject *self) {
    return self->ob_version;           // 32-битный счётчик изменений
}

static Py_ssize_t list_length(PyListObject *self) {
    return Py_SIZE(self);
}

static int list_ass_slice(PyListObject *self, Py_ssize_t low, Py_ssize_t high, PyObject *v) {
    // ... логика присваивания ...
    
    // КРИТИЧЕСКИ: увеличиваем версию после изменения
    self->ob_version++;                // ++ общий счётчик изменений
    return 0;
}

static int list_append(PyListObject *self, PyObject *v) {
    // ... логика добавления ...
    
    self->ob_version++;                // ++ после append/pop/insert/...
    return 0;
}
```

**Любое** изменение списка (`append`, `pop`, `del lst[i]`, `lst[i]=x`) → `self->ob_version++`.
Итератор видит несоответствие → **RuntimeError**.

## 4. PyDictObject ma_version_tag (3.9+ split table)

```c
typedef struct {
    Py_ssize_t ma_used;            // Количество пар
    uint64_t ma_version_tag;       // 64-битный счётчик изменений (атомарный!)
    PyDictKeysObject *ma_keys;     // Ключи
    PyObject **ma_values;          // Значения
} PyDictObject;
```

Словарь имеет **64-битный атомарный** `ma_version_tag`. Изменение (set/del) →
`DICT_NEXT_VERSION()` (вероятно `++`).

## 5. dict_iter_next() - проверка версии словаря

```c
static PyObject *dict_iter_next(PyDictIterObject *iter) {
    PyDictObject *dict = iter->di_dict;    // Ссылка на словарь
    
    // КРИТИЧЕСКАЯ ПРОВЕРКА ВЕРСИИ
    if (iter->di_version_tag != dict->ma_version_tag) {
        PyErr_SetString(PyExc_RuntimeError,
            "dictionary changed size during iteration");
        return NULL;
    }
    
    // Получаем следующий элемент по iter->di_used
    PyDictKeyEntry *entry = get_entry(dict, iter->di_used);
    if (entry->me_key == NULL) {
        return NULL;                       // Конец
    }
    
    iter->di_used++;                       // Следующий
    return PyDictItem_KeyValue(entry);     // (key, value)
}
```

`for k in d:` создаёт PyDictIterObject с копией `d->ma_version_tag`. Каждый `next()` проверяет
версии → **"dictionary changed size"**.

## 6. DICT_NEXT_VERSION() - обновление версии

```c
#define DICT_NEXT_VERSION() \
    (_Py_atomic_fetch_add_uint64(&_PyRuntime.dict_version, 1) + 1)

static int insertdict(PyDictObject *mp, PyObject *key, Py_ssize_t hash, PyObject *value) {
    // ... логика вставки ...
    
    // Обновляем версию АТОМАРНО после изменения
    mp->ma_version_tag = DICT_NEXT_VERSION();
    mp->ma_used++;
    
    return 0;
}
```

**Любое** изменение словаря (`d[k]=v`, `del d[k]`, `d.clear()`) → атомарное
`ma_version_tag = global_dict_version++`. Итератор видит → **RuntimeError**.

## 7. PySetObject версия (аналогично dict)

```c
typedef struct {
    Py_ssize_t used;               // Количество элементов
    uint64_t version_tag;          // Счётчик изменений
    PyDictKeysObject *keys;        // Shared keys
} PySetObject;

static PyObject *set_iter_next(PySetIterObject *setiter) {
    PySetObject *set = setiter->it_set;
    
    if (setiter->it_version_tag != set->version_tag) {
        PyErr_SetString(PyExc_RuntimeError,
            "set changed size during iteration");
        return NULL;
    }
    // ... остальная логика
}
```

Множества работают **идентично** словарям: `version_tag`, проверка в итераторе → **"set
changed size during iteration"**.

## 8. Tuple/String итераторы (без проверок)

```c
typedef struct {
    PyObject_HEAD
    Py_ssize_t it_index;
    PyTupleObject *it_seq;         // НЕИЗМЕНЯЕМЫЕ!
} PyTupleIterObject;

static PyObject *tupleiter_next(PyTupleIterObject *it) {
    // НЕТ проверки версии - tuple неизменяемы!
    Py_ssize_t index = it->it_index;
    PyTupleObject *tuple = it->it_seq;
    
    if (index < Py_SIZE(tuple)) {
        return Py_NewRef(tuple->ob_item[index++]);
    }
    return NULL;
}
```

**Неизменяемые** типы (tuple, str, bytes) **НЕ** проверяют версию — они **физически** не могут
измениться.

## 9. Создание итератора: PyObject_GetIter()

```c
PyObject *PyObject_GetIter(PyObject *obj) {
    PyTypeObject *tp = Py_TYPE(obj);
    
    // Быстрый путь для списков
    if (PyList_CheckExact(obj)) {
        PyListIterObject *it = PyObject_GC_New(PyListIterObject, &PyListIter_Type);
        it->it_seq = (PyListObject *)Py_NewRef(obj);
        it->it_index = 0;
        it->it_version = ((PyListObject *)obj)->ob_version;  // <- КОПИРУЕМ ВЕРСИЮ!
        _PyObject_GC_TRACK(it);
        return (PyObject *)it;
    }
    
    // Общий путь через tp_iter
    unaryfunc iter = tp->tp_iter;
    if (iter != NULL) {
        return (*iter)(obj);
    }
    
    // __iter__()
    return PyObject_CallMethodObjArgs(obj, &_Py_ID(__iter__), NULL);
}
```

`iter(lst)` → PyListIterObject → `it_version = lst->ob_version`. **Любое** изменение списка →
`lst->ob_version++` → итератор **ломается**.

## 10. Байткод: GET_ITER → FOR_ITER

```python
# for x in lst:
```

```
  0 GET_ITER                 # iter(lst) -> PyListIterObject
  2 FOR_ITER       10 (to 14) # next(it) -> x или JUMP
  4 STORE_FAST      0 (x)     # x в локальную
  6 <BODY>
 10 JUMP_ABSOLUTE   0         # Следующая итерация
14 <END>
```

**FOR_ITER в ceval.c:**

```c
case FOR_ITER: {
    PyObject *iter = TOP();            // Итератор
    PyObject *next = (*Py_TYPE(iter)->tp_iternext)(iter);  // next(it)
    
    if (next != NULL) {
        STACKADJ(-1);
        PEEK(0) = next;                // Значение на стек
        JUMPBY(oparg);                 // К STORE_FAST x
    } else {
        Py_DECREF(iter);               // Конец -> StopIteration
        if (!_PyErr_Occurred(tstate) || 
            !_PyErr_ExceptionMatches(tstate, PyExc_StopIteration)) {
            // RuntimeError от итератора!
            goto error;
        }
        _PyErr_Clear(tstate);          // Очищаем StopIteration
        STACKADJ(-1);                  // Убираем iter
        JUMPBY(oparg + 1);             // К концу цикла
    }
    DISPATCH();
}
```

`FOR_ITER` вызывает `it->tp_iternext()` → проверка версии → RuntimeError или значение.
StopIteration → **нормальный** конец цикла.

## 11. Пользовательский итератор с __iter__/__next__

```c
static PyObject *myiter_next(MyIterObject *self) {
    if (self->version != self->seq->ob_version) {
        PyErr_SetString(PyExc_RuntimeError,
            "container modified during iteration");
        return NULL;
    }
    // ... логика
}
```

Пользовательские классы **могут** проверять версии в `__next__()`.

**Изменение коллекции** в CPython 3.9+ детектируется **version tag** (`ob_version`/`ma_version_tag`) в
PyListObject/PyDictObject + проверкой в `tp_iternext()` итераторов → **RuntimeError** при несоответствии.

- [Содержание](/CONTENTS.md#содержание)

---

### **Области видимости**

[Содержание](/CONTENTS.md#содержание)

## **Junior Level**

Представьте себе область видимости в Python как систему комнат в доме. Переменные, созданные в одной комнате (например,
в функции), не всегда видны в другой. Это фундаментальное правило помогает организовывать код и избегать конфликтов
имён.

Когда вы обращаетесь к переменной, Python ищет её, последовательно проверяя четыре «комнаты» или уровня. Это правило
удобно запомнить как акроним **LEGB**:

* **Local (Локальная):** Сначала интерпретатор смотрит внутри текущей функции — это её внутренняя, локальная комната.
* **Enclosing (Охватывающая):** Если функция вложена в другую, Python проверяет «комнаты» внешних функций.
* **Global (Глобальная):** Затем поиск переходит на уровень всего модуля (вашего файла с кодом).
* **Built-in (Встроенная):** В самом конце проверяются встроенные функции и типы языка, такие как `print` или `len`.

Ключевой момент: простое использование (чтение) переменной из внешней области обычно работает, а вот её **изменение** —
нет. Если внутри функции вы попытаетесь присвоить новое значение переменной с внешнего уровня, Python по умолчанию
создаст новую локальную переменную с тем же именем, оставив внешнюю неизменной. Чтобы явно сказать интерпретатору, что
вы хотите работать с уже существующей внешней переменной, используются специальные инструкции:

* `global` — позволяет изменять переменную, объявленную на уровне модуля (глобально).
* `nonlocal` — используется во вложенных функциях и указывает, что переменная принадлежит области видимости ближайшей
  внешней функции (но не глобальной).

[Содержание](/CONTENTS.md#содержание)

## **Middle Level**

Углубляясь, стоит понимать, что каждая область видимости связана со своим **пространством имён**. По сути, это словарь,
который связывает имя объекта (например, переменной) с самим объектом в памяти. Время жизни этих пространств разное:
локальное пространство функции рождается при её вызове и обычно исчезает после завершения, в то время как глобальное
пространство модуля существует всё время его загрузки.

Инструкции `global` и `nonlocal` — это инструменты для управления связью с этими пространствами имён. Важно, что
`nonlocal` появилось в Python 3 именно для решения задач с вложенными областями, позволяя изменять переменные не
локально и не глобально, а именно в охватывающей функции.

Эта механика лежит в основе **замыканий** — мощного приёма, когда внутренняя функция «запоминает» (сохраняет ссылку на)
окружение, в котором она была создана, даже после того, как внешняя функция завершила работу. Благодаря этому,
переменные из внешней области могут жить дольше, чем сама функция, их создавшая.

Современный Python (начиная с версии 3) также изолирует области видимости внутри **генераторных выражений, списковых
включений (comprehensions)** и аналогичных конструкций. Это означает, что переменная, объявленная внутри такого
выражения, не «просачивается» наружу, предотвращая неожиданные изменения в вашем коде.

Когда речь заходит о **классах**, их тело во время создания формирует своё временное пространство имён, которое затем
превращается в атрибут `__dict__` класса. Методы экземпляра получают доступ к данным через первый аргумент `self`,
который ссылается на конкретный экземпляр, обеспечивая чёткое разделение между атрибутами класса и атрибутами его
объектов.

Наконец, важно помнить о контексте выполнения модуля. При запуске скрипта напрямую его глобальное пространство имён
получает специальное имя `__main__`. Каждый импортированный модуль живёт в своём собственном изолированном глобальном
пространстве, что предотвращает коллизии имён между разными частями программы и способствует созданию чистой, модульной
архитектуры.

[Содержание](/CONTENTS.md#содержание)

## **Senior Level**

В CPython 3.9+ **области видимости** определяются **symtable** (Python/symtable.c) во время компиляции → флаги
`DEF_LOCAL`/`DEF_GLOBAL_IMPLICIT`/`DEF_FREE` → выбор байткода `LOAD_FAST`/`LOAD_NAME`/`LOAD_GLOBAL`/`LOAD_DEREF`. Поиск
в `f_localsplus`/`f_globals`/`f_builtins`. `Python/symtable.c`,`Python/compile.c`,`Python/ceval.c`

## 1. symtable_lookup() - анализ области видимости (symtable.c)

```c
long symtable_lookup(struct symtable *st, PyObject *name) {
    PyObject *o = PyDict_GetItem(st->st_symbols, name);  // Ищем имя в таблице символов
    if (!o) {
        return -1;                         // Символа нет
    }
    
    long flags = PyLong_AsLong(o);         // Получаем флаги символа
    
    if (flags & DEF_LOCAL) {
        return LOCAL;                      // Локальная переменная (LOAD_FAST)
    }
    if (flags & DEF_GLOBAL) {
        return GLOBAL_EXPLICIT;            // global x (LOAD_GLOBAL)
    }
    if (flags & DEF_NONLOCAL) {
        return CELL;                       // nonlocal x (LOAD_DEREF)
    }
    if (flags & DEF_FREE) {
        return FREE;                       // Замыкание (LOAD_DEREF)
    }
    if (flags & DEF_GLOBAL_IMPLICIT) {
        return GLOBAL_IMPLICIT;            // Не присвоена в функции (LOAD_NAME/LOAD_GLOBAL)
    }
    
    return -1;                             // Неизвестно
}
```

Компилятор создаёт таблицу символов для каждой функции/класса/модуля. Для каждого имени `x`
вычисляет флаги: LOCAL (в `f_localsplus`), GLOBAL, FREE (замыкание), GLOBAL_IMPLICIT (глобал по умолчанию).

## 2. compiler_symbol_table() - генерация флагов (compile.c)

```c
static void
compiler_symbol_table(struct compiler *c, stmt_ty s) {
    struct symtable *st = c->u->u_ste;
    
    // Для каждой инструкции AST
    VISIT(c, symtable, s);             // Рекурсивно анализируем тело
    
    // После полного анализа
    symtable_analyze(st);              // Финальный анализ областей
    
    // Для каждой переменной
    PyObject *name;
    Py_ssize_t pos = 0;
    while (PyDict_Next(st->st_symbols, &pos, &name, NULL)) {
        long flags = symtable_lookup(st, name);
        
        if (flags == LOCAL && !st->st_varargs && !st->st_varkw) {
            // Локальная -> добавляем в co_varnames
            PyList_Append(c->u->u_varnames, name);
        }
        
        if (flags == FREE || flags == CELL) {
            // Замыкание -> co_cellvars/co_freevars
            PyList_Append(c->u->u_cellvars, name);
        }
    }
}
```

Компилятор проходит по AST, собирает все `x=1`, `for x in y`, `def f(x):`. Затем
`symtable_analyze()` решает: LOCAL (в locals), FREE (замыкание), GLOBAL.

## 3. LOAD_FAST - самый быстрый (ceval.c)

```c
case LOAD_FAST: {
    PyObject *value = GETLOCAL(oparg);     // f_localsplus[oparg] (прямой доступ!)
    
    if (value == NULL) {
        format_exc_unbound(tstate, PyExc_UnboundLocalError,
            UNBOUNDLOCAL_ERROR_MSG,
            PyTuple_GET_ITEM(co->co_varnames, oparg));
        goto error;                        // UnboundLocalError
    }
    
    Py_INCREF(value);                      // +refcnt
    PUSH(value);                           // На стек
    FAST_DISPATCH();                       // Быстрый переход (без switch)
}
```

`LOAD_FAST 3` = `f_localsplus[3]` (массив указателей в фрейме). **Мгновенно** — без
хеш-таблиц/поиска/MRO. Только проверка NULL.

## 4. LOAD_NAME - локал/глобал/встроенный (ceval.c)

```c
case LOAD_NAME: {
    PyObject *name = GETITEM(names, oparg);     // co_names[oparg] = "x"
    PyObject *localsplus = frame->f_localsplus;
    Py_ssize_t i = name_hint;                   // Кеш из co_namei
    
    // Кеш работает?
    if (i != INDEX_NONE && 
        PyTuple_GET_ITEM(co->co_varnames, i) == name) {
        PyObject *value = localsplus[i];
        if (value != NULL) {
            Py_INCREF(value);
            PUSH(value);
            DISPATCH();
        }
        i = INDEX_NONE;                        // Кеш промах
    }
    
    // 1. Локальные (f_locals)
    if (PyDict_CheckExact(f_locals)) {
        value = PyDict_GetItemWithCache(f_locals, name, hash);
        if (value != NULL) {
            Py_INCREF(value);
            PUSH(value);
            DISPATCH();
        }
    }
    
    // 2. Глобальные (f_globals)
    value = PyDict_GetItemWithCache(f_globals, name, hash);
    if (value != NULL) {
        Py_INCREF(value);
        PUSH(value);
        DISPATCH();
    }
    
    // 3. Встроенные (f_builtins)
    value = PyDict_GetItemWithCache(f_builtins, name, hash);
    if (value != NULL) {
        Py_INCREF(value);
        PUSH(value);
        DISPATCH();
    }
    
    // NameError
    format_exc_check_arg(tstate, PyExc_NameError, NAME_ERROR_MSG, name);
    goto error;
}
```

`LOAD_NAME "x"` → **3 поиска**: locals dict → globals dict → builtins dict. **Кеш** `co_namei`
ускоряет (опарг → индекс в `co_varnames`).

## 5. LOAD_GLOBAL - только глобал/встроенный (ceval.c)

```c
case LOAD_GLOBAL: {
    PyObject *name = GETITEM(names, oparg>>1);      // co_names[oparg>>1]
    PyObject *res = NULL;
    uint32_t hints = oparg & 0xFFFF;               // Кеш: global_index + builtin_index
    
    // Кеш глобальной переменной
    Py_ssize_t global_index = hints >> 8;
    if (global_index != INDEX_NONE && 
        PyTuple_GET_ITEM(co->co_names, global_index) == name) {
        res = _PyDict_LookupWithCache(frame->f_globals, name, hints>>8);
    }
    
    if (res == NULL) {
        // Кеш встроенной
        Py_ssize_t builtin_index = hints & 0xFF;
        if (builtin_index != INDEX_NONE && 
            PyTuple_GET_ITEM(co->co_names, builtin_index) == name) {
            res = _PyDict_LookupWithCache(frame->f_builtins, name, hints&0xFF);
        }
    }
    
    if (res != NULL) {
        Py_INCREF(res);
        PUSH(res);
        DISPATCH();
    }
    
    // Полный поиск globals -> builtins
    res = PyDict_GetItemWithCache(frame->f_globals, name, hash);
    if (res == NULL) {
        res = PyDict_GetItemWithCache(frame->f_builtins, name, hash);
    }
    
    if (res == NULL) {
        format_exc_check_arg(PyExc_NameError, NAME_ERROR_MSG, name);
        goto error;
    }
    
    Py_INCREF(res);
    PUSH(res);
    DISPATCH();
}
```

`LOAD_GLOBAL "print"` → **только** globals + builtins (НЕ locals). **Двойной кеш** (16 бит): 8
бит глобал + 8 бит builtin индекс.

## 6. LOAD_DEREF - замыкание/ nonlocal (ceval.c)

```c
case LOAD_DEREF: {
    PyObject *cell = GETCLOSURE(oparg);     // func_closure[oparg]
    
    if (cell == NULL) {
        goto unbound_error;                // Unbound closure variable
    }
    
    PyObject *value = PyCell_Get(cell);    // cell->ob_ref
    if (value == NULL) {
        goto unbound_error;
    }
    
    Py_INCREF(value);
    PUSH(value);
    DISPATCH();
}

case STORE_DEREF: {
    PyObject *v = POP();                   // Значение со стека
    PyObject *cell = GETCLOSURE(oparg);    // func_closure[oparg]
    
    if (cell == NULL) {
        goto unbound_error;
    }
    
    PyCell_Set(cell, v);                   // cell->ob_ref = v
    Py_DECREF(v);
    DISPATCH();
}
```

`LOAD_DEREF 0` → `func_closure[0]->ob_ref` (значение ячейки). `STORE_DEREF 0` →
`func_closure[0]->ob_ref = value`.

## 7. compiler_lookup_name() - выбор опкода (compile.c)

```c
static int
compiler_lookup_name(struct compiler *c, location loc, identifier name,
                     expr_context_ty ctx, int error_ok) {
    struct symtable *st = c->u->u_ste;
    long flags = symtable_lookup(st, name);     // Получаем флаги
    
    switch (flags) {
    case LOCAL:
        if (ctx == Load) {
            ADDINSTR(c, loc, LOAD_FAST, i);    // x -> LOAD_FAST i
        } else {
            ADDINSTR(c, loc, STORE_FAST, i);   // x = 1 -> STORE_FAST i
        }
        break;
        
    case FREE:
    case CELL:
        if (ctx == Load) {
            ADDINSTR(c, loc, LOAD_DEREF, i);   // nonlocal x -> LOAD_DEREF
        } else {
            ADDINSTR(c, loc, STORE_DEREF, i);
        }
        break;
        
    case GLOBAL_EXPLICIT:
    case GLOBAL_IMPLICIT:
        if (ctx == Load) {
            ADDINSTR(c, loc, LOAD_GLOBAL, i);  // global x -> LOAD_GLOBAL
        } else {
            ADDINSTR(c, loc, STORE_GLOBAL, i);
        }
        break;
        
    default:
        if (ctx == Load) {
            ADDINSTR(c, loc, LOAD_NAME, i);    // Неопределённая -> LOAD_NAME
        } else {
            ADDINSTR(c, loc, STORE_NAME, i);
        }
    }
}
```

Компилятор по флагам символа генерирует **правильный** опкод: LOCAL→FAST, FREE→DEREF,
GLOBAL→GLOBAL, остальное→NAME.

## 8. f_localsplus - универсальный массив (3.11+)

```c
struct _PyInterpreterFrame {
    // ...
    PyObject **localsplus;             // Массив: varnames + cellvars + freevars
    Py_ssize_t nlocalsplus;            // Размер массива
    // ...
};

#define GETLOCAL(i) (frame->localsplus[i])
#define GETCLOSURE(i) (frame->localsplus[frame->f_lasti + (i)])
```

**Единый массив** `localsplus[]`: сначала параметры/локальные (`co_varnames`), потом ячейки (
`co_cellvars`), потом свободные (`co_freevars`). `LOAD_FAST 0` = `localsplus[0]`.

## 9. Кеш LOAD_NAME/LOAD_GLOBAL (co_opcache)

```c
typedef struct _PyCodeObject {
    // ...
    uint16_t *co_opcache;              // Кеш для LOAD_NAME/LOAD_GLOBAL
    // ...
} PyCodeObject;
```

`co_opcache[oparg]` хранит **индекс** в `co_varnames` или хеш для globals/builtins. Промах →
полный поиск.

**Области видимости** в CPython 3.9+ — **symtable** (флаги DEF_*), компилятор → `LOAD_FAST`/`LOAD_NAME`/`LOAD_GLOBAL`/
`LOAD_DEREF`, поиск localsplus→locals→globals→builtins, кеш `co_opcache`.

- [Содержание](/CONTENTS.md#содержание)

---

# **Lambda-функции**

[Содержание](/CONTENTS.md#содержание)

## **Junior Level**

Лямбда-функции в Python — это компактные анонимные инструменты, создаваемые «на лету» для выполнения простых одношаговых
операций. Если обычную функцию, объявленную через `def`, можно сравнить с многофункциональным кухонным комбайном, то
лямбда — это как карманный ножик: простой, специализированный и удобный в ситуациях, когда не хочется или не нужно
использовать тяжёлую технику.

Они идеально подходят для случаев, когда функцию нужно передать как аргумент другой функции. Типичные примеры —
сортировка данных по нестандартному ключу (`sorted(items, key=lambda x: x['price'])`), фильтрация коллекции (
`filter(lambda x: x > 0, values)`) или преобразование элементов (`map(lambda x: x * 2, numbers)`). Синтаксис лямбды
элементарен: после ключевого слова `lambda` указываются аргументы, затем двоеточие и единственное выражение, результат
которого автоматически возвращается.

Основное ограничение вытекает из их предназначения: лямбда может содержать только одно выражение, но не операторы. Это
делает их непригодными для сложной логики с условиями `if-elif-else` (хотя можно использовать тернарный оператор),
циклами или присваиваниями. Если логика перестаёт помещаться в одну строку или требует пояснений, это верный признак,
что пора использовать обычную функцию.

[Содержание](/CONTENTS.md#содержание)

## **Middle Level**

Под капотом лямбда создаёт полноценный объект функции, который, однако, лишён имени (в системных атрибутах он фигурирует
как `'<lambda>'`). Несмотря на минималистичный синтаксис, лямбда-функции обладают почти всей мощью обычных функций: они
поддерживают замыкания (захват переменных из окружающей области видимости), аргументы по умолчанию и произвольное
количество позиционных и именованных аргументов.

Именно способность к замыканиям делает лямбды особенно выразительными в функциональном стиле программирования, позволяя
создавать фабрики поведения прямо в коде. Однако здесь кроется и классическая ловушка: переменные из внешней области
захватываются по ссылке, а не по значению на момент создания. Из-за этого цикл, создающий несколько лямбд с зависимостью
от переменной-счётчика, часто приводит к неожиданному результату — все функции будут использовать одно и то же (
финальное) значение переменной.

Другая тонкость связана с пространствами имён и временем вычисления. Значения аргументов по умолчанию в лямбде, как и в
обычной функции, вычисляются один раз — в момент её определения, а не каждого вызова. Это поведение важно учитывать при
использовании изменяемых объектов, таких как списки или словари, в качестве значений по умолчанию.

Хотя лямбды часто ассоциируются с функциями `map()`, `filter()` и `sorted()`, в современном Python для первых двух часто
более читаемой альтернативой являются списковые включения (list comprehensions) и выражения-генераторы. Лямбда же
остаётся незаменимой там, где требуется передать простой ключ или правило преобразования, особенно в методах,
принимающие аргумент `key`, как в `sorted()`, `min()`, `max()` или группировке `itertools.groupby()`. Их сила — в
лаконичности и возможности быть определёнными именно там, где они используются, что уменьшает разрыв между объявлением и
применением.

[Содержание](/CONTENTS.md#содержание)

## **Senior Level**

В CPython 3.9+ **lambda-функции** компилируются в **отдельный PyCodeObject** через `compiler_lambda()` (
Python/compile.c), создаются через `MAKE_FUNCTION 0` байткод (без defaults/closure), выполняются как обычные
PyFunctionObject. `Python/compile.c`,`Python/ceval.c`,`Objects/funcobject.c`

## 1. compiler_lambda() - компиляция lambda (compile.c)

```c
static int
compiler_lambda(struct compiler *c, location loc, expr_ty e) {
    PyCodeObject *co;
    PyObject *qualname;
    stmt_ty s = e->v.Lambda.body;      // Тело lambda: x + y
    arguments_ty a = e->v.Lambda.args; // Аргументы: (x, y)
    
    // Создаём уникальное имя <lambda N>
    qualname = _PyCompiler_QualifiedName(c, "<lambda>", loc);
    if (qualname == NULL) {
        return -1;
    }
    
    // Компилируем тело lambda как expr
    if (!compiler_enter_scope(c, qualname, COMPILER_SCOPE_LAMBDA, 
                              a, loc)) {
        goto error;
    }
    
    VISIT(c, expr, s);                 // Компилируем x + y -> байткод
    
    // Генерируем PyCodeObject
    co = compiler_make_closure(c, 0, 0, qualname);  // 0 defaults, 0 closure
    compiler_exit_scope(c);
    
    if (co == NULL) {
        goto error;
    }
    
    ADDINSTR(c, loc, MAKE_FUNCTION, 0);  // MAKE_FUNCTION 0 (lambda code)
    ADDINSTR(c, loc, LOAD_CONST, add(co));  // Константа PyCodeObject
    FREE(qualname);
    return 0;
    
error:
    Py_XDECREF(qualname);
    return -1;
}
```

`lambda x: x+1` → компилятор создаёт **отдельный scope** с именем `<lambda>`, компилирует тело
`x+1` в PyCodeObject, генерирует `MAKE_FUNCTION 0 + LOAD_CONST(code)`.

## 2. Байткод lambda x: x + 1

```
# Эквивалентный байткод:
  0 LOAD_FAST           0 (x)      # x на стек
  2 LOAD_CONST           1 (1)     # 1 на стек
  4 BINARY_ADD                 # x + 1
  6 RETURN_VALUE              # Возврат результата
```

**Полный байткод вызова:**

```python
lambda_func = lambda x: x + 1
result = lambda_func(42)
```

```
# Создание lambda:
  0 LOAD_CONST           0 (<code object <lambda> at 0x...>)
  2 MAKE_FUNCTION        0          # PyFunctionObject(code)
  4 STORE_FAST           0 (lambda_func)

# Вызов:
  6 LOAD_FAST            0 (lambda_func)
  8 LOAD_CONST           1 (42)
 10 CALL                 1          # lambda_func(42)
```

Lambda — **обычная функция** с **автогенерированным** PyCodeObject `<lambda>`.
`MAKE_FUNCTION 0` = без параметров/замыканий.

## 3. MAKE_FUNCTION 0 байткод (ceval.c)

```c
case MAKE_FUNCTION: {
    Py_ssize_t flags = POP();          // oparg (0 для lambda)
    PyCodeObject *code = POP();        // PyCodeObject lambda
    PyObject *qualname = POP();        // "<lambda>"
    
    // Создаём PyFunctionObject
    PyFunctionObject *func = PyFunction_New(code, frame->f_globals);
    if (func == NULL) {
        goto error;
    }
    
    // flags == 0: чистая lambda (без defaults/kwdefaults/closure)
    if (flags & 0xFF) {                // defaults
        func->func_defaults = POP();
    }
    if (flags & 0xFF00) {              // kwdefaults
        func->func_kwdefaults = POP();
    }
    if (flags & 0xFF0000) {            // closure
        func->func_closure = POP();
    }
    
    func->func_name = qualname;        // __name__ = "<lambda>"
    Py_INCREF(qualname);
    
    PUSH((PyObject *)func);            // Lambda на стек
    DISPATCH();
}
```

`MAKE_FUNCTION 0` берёт PyCodeObject → создаёт PyFunctionObject → `func_name="<lambda>"`,
`func_code=code`. Без closure/defaults (flags=0).

## 4. PyFunction_New - создание PyFunctionObject

```c
PyObject *PyFunction_New(PyCodeObject *code, PyObject *globals) {
    PyFunctionObject *op;
    
    // Выделяем PyFunctionObject
    op = PyObject_GC_New(PyFunctionObject, &PyFunction_Type);
    if (op == NULL) {
        return NULL;
    }
    
    // Заполняем базовые поля
    Py_INCREF(code);
    op->func_code = code;              // Сохраняем PyCodeObject
    
    Py_XINCREF(globals);
    op->func_globals = globals;        // globals() на момент создания
    
    // builtins из globals или интерпретатора
    PyObject *builtins = _PyDict_GetItemIdWithCache(globals, &PyId___builtins__);
    if (builtins == NULL) {
        builtins = PyEval_GetBuiltins();
        Py_INCREF(builtins);
    }
    op->func_builtins = builtins;
    
    // Пустые значения по умолчанию
    op->func_defaults = NULL;
    op->func_kwdefaults = NULL;
    op->func_closure = NULL;
    op->func_doc = NULL;
    op->func_name = NULL;
    op->func_qualname = NULL;
    op->func_dict = NULL;
    op->func_weakreflist = NULL;
    
    _PyObject_GC_TRACK(op);            // Регистрируем в GC
    return (PyObject *)op;
}
```

Lambda = PyFunctionObject с `func_code=<lambda bytecode>`, `func_globals=текущие globals`,
`func_builtins=builtins`. Остальное NULL.

## 5. Вызов lambda: CALL_FUNCTION (ceval.c)

```c
case CALL_FUNCTION: {
    PyObject *callable = PEEK(oparg);      // lambda_func
    Py_ssize_t na = oparg;                 // Количество аргументов
    
    // Быстрый vectorcall путь
    if (_PyObject_HasVectorcall(callable)) {
        PyObject *const *args = (PyObject **)PyMem_Malloc(na * sizeof(PyObject *));
        for (Py_ssize_t i = 0; i < na; i++) {
            args[i] = PEEK(oparg - i);     // Аргументы на стек
        }
        
        PyObject *result = _PyObject_Vectorcall(callable, args, na, NULL);
        PyMem_Free(args);
        
        if (result == NULL) {
            goto error;
        }
        
        STACKADJ(-(oparg + 1));            // Убираем lambda + args
        PUSH(result);                      # Результат на стек
        DISPATCH();
    }
    
    // Медленный путь PyObject_Call
    // ...
}
```

`lambda_func(42)` → `CALL_FUNCTION 1` → `_PyObject_Vectorcall(lambda, [42], 1, NULL)` →
выполнение lambda bytecode.

## 6. Выполнение lambda bytecode: _PyEval_EvalFrameDefault

```c
PyObject *_PyEval_EvalFrameDefault(PyThreadState *tstate, PyFrameObject *f) {
    // ...
    while (1) {
        // Читаем следующую инструкцию
        uint16_t opcode = NEXT_BYTE;       # LOAD_FAST 0
        uint16_t oparg = NEXT_UINT16;      # Индекс x
        
        switch (opcode) {
        case LOAD_FAST: {
            PyObject *value = GETLOCAL(oparg);  # f_localsplus[0] = x
            if (value == NULL) {
                unboundlocal_error();          # UnboundLocalError
            }
            Py_INCREF(value);
            PUSH(value);
            FAST_DISPATCH();                   # Быстрый переход
        }
        
        case BINARY_ADD: {
            PyObject *b = POP();               # 1
            PyObject *a = POP();               # x
            PyObject *result = PyNumber_Add(a, b);
            Py_DECREF(a);
            Py_DECREF(b);
            if (result == NULL) {
                goto error;
            }
            PUSH(result);
            DISPATCH();
        }
        
        case RETURN_VALUE: {
            PyObject *retval = POP();          # x + 1
            Py_DECREF(f);                      # Освобождаем фрейм
            return retval;                     # Возвращаем результат
        }
        }
    }
}
```

Lambda выполняется в **новом фрейме** с `f_localsplus[0]=x=42`. `LOAD_FAST 0` → `42`,
`LOAD_CONST 1 1` → `1`, `BINARY_ADD` → `43`, `RETURN_VALUE 43`.

## 7. Lambda с замыканием

```python
def outer(y):
    return lambda x: x + y  # Замыкание на y
```

```
# Байткод outer():
  0 LOAD_CLOSURE        0 (y)        # Берём ячейку y
  2 BUILD_TUPLE         1            # (cell_y,)
  4 LOAD_CONST          1 (<code>)
  6 MAKE_CLOSURE        1            # closure=(cell_y,)
  8 RETURN_VALUE

# Байткод lambda:
  0 LOAD_FAST           0 (x)        # x
  2 LOAD_DEREF          0 (y)        # cell_y->ob_ref
  4 BINARY_ADD                # x + y
  6 RETURN_VALUE
```

`lambda x: x + y` захватывает `y` в **ячейку**. `outer(10)` → lambda с
`func_closure=(cell_y=10)`. Вызов → `LOAD_DEREF 0` читает `cell_y->ob_ref=10`.

## 8. Различия lambda vs def (компилятор)

```c
// compiler_function() для def f():
COMPILER_SCOPE_FUNCTION  // co_flags |= CO_NEWLOCALS

// compiler_lambda() для lambda:
COMPILER_SCOPE_LAMBDA    // co_flags без CO_NEWLOCALS (использует globals)
```

**def** создаёт **локальные** переменные (`f_localsplus`). **lambda** — **выражение**,
использует **globals** внешней функции (без CO_NEWLOCALS).

**Lambda** в CPython 3.9+ — **PyCodeObject** из `compiler_lambda()`, `MAKE_FUNCTION 0`, **отдельный PyFrameObject** при
вызове, **без локальных** (CO_NEWLOCALS=0), поддержка замыканий через `LOAD_DEREF`.

- [Содержание](/CONTENTS.md#содержание)

---

# *Comprehensions и генераторные выражения*

[Содержание](/CONTENTS.md#содержание)

## **Junior Level**

Comprehensions — это лаконичный синтаксис для создания коллекций (списков, множеств, словарей) на основе итераций с
возможностью фильтрации. Генераторные выражения, оформленные в круглые скобки, создают итераторы, которые вычисляют
элементы «лениво», экономя память. Обе конструкции выполняются в собственной области видимости и обычно работают быстрее
эквивалентных циклов за счёт внутренних оптимизаций CPython.

List comprehension (списковое включение) создаёт новый список: `[x*2 for x in range(5)]` даст `[0, 2, 4, 6, 8]`. Set
comprehension создаёт множество, dict comprehension — словарь. Генераторное выражение выглядит похоже, но в круглых
скобках: `(x*2 for x in range(5))`. Разница в том, что генераторное выражение не создаёт коллекцию сразу, а возвращает
итератор, который вычисляет элементы «лениво», по одному, что экономит память при работе с большими объёмами данных.

Их удобно использовать для фильтрации (добавив `if`) и для преобразования элементов. Это делает код чище и часто
быстрее, чем аналогичные циклы.

[Содержание](/CONTENTS.md#содержание)

## **Middle Level**

1. **Типы comprehensions**:

- List comprehension: `[выражение for элемент in итератор]`
- Set comprehension: `{выражение for элемент in итератор}`
- Dict comprehension: `{ключ: значение for элемент in итератор}`
- Generator expression: `(выражение for элемент in итератор)`

2. **Синтаксические возможности**:

- Могут содержать несколько циклов `for`: `[x+y for x in list1 for y in list2]`
- Поддерживают условия фильтрации `if`: `[x for x in range(10) if x % 2 == 0]`
- Условия могут быть вложенными и комбинированными

3. **Область видимости**: Начиная с Python 3, comprehensions и генераторные выражения выполняются в собственной области
   видимости. Переменные, созданные внутри (например, переменная цикла), не «просачиваются» наружу, что предотвращает
   случайные перезаписи.

4. **Производительность**: List comprehensions обычно выполняются быстрее эквивалентных циклов `for`, потому что они
   оптимизированы на уровне байткода и выполняют операции добавления элементов напрямую, минуя вызовы методов.
   Генераторные выражения экономят память, но имеют небольшие накладные расходы на каждый вызов `next()`.

5. **Ленивые вычисления**: Генераторные выражения вычисляют элементы только когда они запрашиваются (например, в цикле
   `for` или при вызове `next()`). Это позволяет работать с бесконечными последовательностями и потоками данных.

6. **Отличия от функций-генераторов**: Генераторные выражения — это синтаксический сахар для создания анонимных
   генераторов. Они не могут содержать сложную логику с несколькими `yield` или `return`, в отличие от
   функций-генераторов.

[Содержание](/CONTENTS.md#содержание)

## **Senior Level**

В CPython 3.9+ **comprehensions** компилируются в **отдельный PyCodeObject** через `compiler_listcomp()`/
`compiler_dictcomp()`, **генераторные выражения** — через `compiler_genexp()` с `MAKE_FUNCTION + GET_ITER + FOR_ITER`. *
*PEP 709** (3.12+) **inlines** comprehensions напрямую в байткод без функции. `Python/compile.c`,`Python/ceval.c`

## 1. До PEP 709 (3.9-3.11): отдельная функция

```python
lst = [x ** 2 for x in range(10)]
```

```
# Байткод (Python 3.11):
  0 LOAD_GLOBAL           0 (range)
  2 LOAD_CONST            1 (10)
  4 CALL                  1
  6 LIST_COMP            1           # Вызов <listcomp> функции
  8 STORE_FAST            0 (lst)
```

**`<listcomp>` функция:**

```
# Байткод <listcomp>:
  0 BUILD_LIST            0          # []
  2 LOAD_FAST             0 (.0)     # range iterator
>> 4 FOR_ITER             12 (to 18)
  6 STORE_FAST            1 (x)      # x = next(range)
  8 LOAD_FAST             1 (x)
 10 LOAD_CONST            0 (2)
 12 BINARY_POWER                # x**2
 14 LIST_APPEND           2         # lst.append(x**2)
 16 JUMP_ABSOLUTE         4         # <- цикл
>>18 RETURN_VALUE               # return lst
```

`[x**2 for x in range(10)]` → **отдельная функция** `<listcomp>` с **своим фреймом**.
`LIST_APPEND 2` добавляет в список **фиксированной позиции** (`.0` на стеке).

## 2. PEP 709 Inline comprehensions (Python 3.12+)

```
# Байткод Python 3.12 (inlined!):
  0 LOAD_GLOBAL           1 (range)
  2 LOAD_CONST            1 (10)
  4 PRECALL               1
  6 CALL                  1
  8 GET_ITER                   # iter(range)
 10 LOAD_FAST_AND_CLEAR   1 (.0)  # Сохраняем .0 = iter(range)
 12 SWAP                  2          # iter на вершину
 14 BUILD_LIST            0       # []
 16 SWAP                  2          # list на .0
>>18 FOR_ITER             8 (to 34)  # Цикл
 20 STORE_FAST            2 (x)   # x = next(iter)
 22 LOAD_FAST             2 (x)
 24 LOAD_CONST            0 (2)
 26 PRECALL               0
 28 BINARY_OP_SUBSCRIPT   8 (**)
 30 LIST_APPEND           2       # lst.append(x**2)
 32 JUMP_BACKWARD         12 (to 18)
>>34 SWAP                  2          # Восстанавливаем .0
 36 STORE_FAST            1 (.0)  # Восстанавливаем .0
 38 RETURN_VALUE
```

**3.12**: comprehension **встроен** в байткод. `LOAD_FAST_AND_CLEAR 1 (.0)` сохраняет
`iter(range)` в слот `.0`. `FOR_ITER` использует его напрямую. **Быстрее** — без function call overhead.

## 3. compiler_listcomp() - компиляция (compile.c)

```c
static int
compiler_listcomp(struct compiler *c, location loc, expr_ty e) {
    compiler_enter_scope(c, "_<listcomp>", COMPILER_SCOPE_COMPREHENSION, 
                         e->v.ListComp.generators[0].target, loc);
    
    NEW_JUMP_TARGET_BLOCK(c, listcomp_inlined);  // Метка для inline (3.12+)
    
    VISIT(c, expr, e->v.ListComp.elt);         // x**2
    
    // Создаём список
    if (compiler_ok_to_inline(c)) {
        ADDINSTR(c, loc, BUILD_LIST, 0);       // [] inline
    } else {
        VISIT_SEQ(c, expr, e->v.ListComp.generators);  // for x in range(10)
        ADDINSTR(c, loc, LIST_COMP, 1);        // До PEP 709
    }
    
    compiler_exit_scope(c);
    return 0;
}
```

Компилятор создаёт **scope COMPILER_SCOPE_COMPREHENSION** (без CO_NEWLOCALS), компилирует
`x**2` + `for x in ...`, генерирует `BUILD_LIST 0` или `LIST_COMP 1`.

## 4. LIST_APPEND байткод (ceval.c) - сердце comprehension

```c
case LIST_APPEND: {
    PyObject *v = PEEK(oparg + 1);     // Значение (x**2)
    PyObject *list = PEEK(oparg + 2);  // Список (.0)
    
    if (PyList_CheckExact(list)) {
        // Быстрый путь для PyListObject
        if (PyList_Append(list, v) < 0) {
            goto error;
        }
    } else {
        // Общий путь PyObject.call(method='append', args=(v,))
        PyObject *meth = PyObject_GetAttr(list, &_Py_ID(append));
        if (meth == NULL) {
            goto error;
        }
        PyObject *res = _PyObject_Vectorcall(meth, &v, 1, NULL);
        Py_DECREF(meth);
        if (res == NULL) {
            goto error;
        }
        Py_DECREF(res);
    }
    
    Py_DECREF(v);                          // Освобождаем x**2
    STACKADJ(-1);                          // Убираем со стека
    DISPATCH_SAME_OPARG(1);
}
```

`LIST_APPEND 2` берёт **список из слота .0** (не со стека!), значение `x**2`, вызывает
`list.append(x**2)`. **Экономит стек** — список в фиксированном слоте.

## 5. Генераторное выражение: (x**2 for x in range(10))

```
# Байткод:
  0 LOAD_GLOBAL           0 (range)
  2 LOAD_CONST            1 (10)
  4 CALL                  1
  6 GET_AWAITABLE              # iter(range)
  8 LOAD_GENEXPR          1          # <genexpr> функция
```

**`<genexpr>` функция:**

```
  0 LOAD_CONST            0 ((None,))
  2 COPY                   1          # Сохраняем None
  4 LOAD_FAST             0 (.0)     # range iterator (.0)
>> 6 FOR_ITER             12 (to 24)
  8 STORE_FAST            1 (x)
 10 LOAD_FAST             1 (x)
 12 LOAD_CONST            1 (2)
 14 BINARY_POWER
 16 RETURN_GENERATOR           # yield x**2
```

Генераторное выражение — **функция-генератор** с `yield x**2`. `LOAD_GENEXPR` создаёт
PyGenObject.

## 6. compiler_genexp() - компиляция genexp

```c
static int
compiler_genexp(struct compiler *c, location loc, expr_ty e) {
    compiler_enter_scope(c, "_<genexpr>", COMPILER_SCOPE_GENERATOR, 
                         e->v.GenExp.generators[0].target, loc);
    
    // Создаём helper функцию с yield
    VISIT(c, expr, e->v.GenExp.elt);       // x**2
    
    // Генераторы for/if
    VISIT_SEQ(c, expr, e->v.GenExp.generators);
    
    // co_flags |= CO_GENERATOR
    c->u->u_ste->ste_flags |= CO_GENERATOR;
    
    PyCodeObject *co = compiler_make_closure(c, 0, 0, qualname);
    compiler_exit_scope(c);
    
    ADDINSTR(c, loc, LOAD_CONST, add(co));     # PyCodeObject
    ADDINSTR(c, loc, MAKE_FUNCTION, 0);        # PyFunctionObject
    ADDINSTR(c, loc, LOAD_GENEXPR, 1);         # Генератор
}
```

Генераторное выражение компилируется как **функция с CO_GENERATOR** флагом + `yield`.
`LOAD_GENEXPR` создаёт PyGenObject.

## 7. Inline dict/set comprehensions (PEP 709, 3.12+)

```python
# {x: x**2 for x in range(10)} -> BUILD_MAP_UNPACK_WITH_CALL? Нет!
# Python 3.12 использует DICT_MERGE + специальные опкоды
```

```
# Байткод dictcomp (до 3.12):
DICT_COMP        1

# 3.12+: inline
BUILD_MAP        0
LOAD_FAST        (.0)
FOR_ITER
STORE_FAST       (x)
LOAD_FAST        (x)
LOAD_FAST        (x)
LOAD_CONST       (2)
BINARY_POWER
MAP_ADD          3      # dict[key] = value
JUMP_BACKWARD
```

`MAP_ADD 3` добавляет `key=value` в словарь **фиксированной позиции** (аналог LIST_APPEND).

## 8. LIST_APPEND оптимизация (ceval.c)

```c
case LIST_APPEND: {
    PyObject *v = PEEK(oparg + 1);         # Значение x**2
    PyObject *list = PEEK(oparg + 2);      # Список .0
    
    // Супер-быстрый путь PyListObject
    if (PyList_CheckExact(list)) {
        PyListObject *l = (PyListObject *)list;
        Py_ssize_t n = Py_SIZE(l);
        
        if (n < l->allocated) {            # Есть место (over-allocation)
            PyObject *item = Py_NewRef(v);
            l->ob_item[n] = item;          # Копируем указатель
            Py_SET_SIZE(l, n + 1);         # ++ ob_size
        } else {
            PyList_Append(list, v);        # Медленный resize
        }
    }
    
    Py_DECREF(v);
    STACKADJ(-1);
    DISPATCH_SAME_OPARG(1);
}
```

Comprehension использует **over-allocation** списка (allocated > ob_size). `LIST_APPEND` → *
*прямое** присваивание `ob_item[n++]` **без realloc**.

## 9. Async comprehensions (3.5+, улучшено в 3.9+)

```python
# [x async for x in agen()] -> ASYNC_COMPREHENSION_GENERATOR
```

```
# Байткод:
GET_AITER              # aiter(agen)
LOAD_CONST             (<asyncgen>)
ASYNC_COMPREHENSION_GENERATOR 1
```

Async genexp → **async generator** с `async for`/`await` в байткоде.

**Comprehensions** в CPython 3.9+ — **отдельные PyCodeObject** (`COMPILER_SCOPE_COMPREHENSION`), **LIST_APPEND**/*
*MAP_ADD** с фиксированным списком/словарём в `.0`, **PEP 709 inline** (3.12+ без function call), **over-allocation**
оптимизация.

- [Содержание](/CONTENTS.md#содержание)

---

# **copy() и deepcopy()**

[Содержание](/CONTENTS.md#содержание)

## **Junior Level**

Когда вам нужно создать независимую копию объекта в Python, на помощь приходят функции `copy()` и `deepcopy()` из модуля
`copy`. Разница между ними напоминает разницу между созданием ярлыка на папку или её копирование.

`copy()` создаёт поверхностную (shallow) копию — новый "контейнер", который, однако, продолжает ссылаться на те же
внутренние объекты, что и оригинал. Представьте, что вы сделали копию папки с документами: сама папка новая, но лежащие
в ней файлы — те же самые. Если вы измените содержимое файла в копии папки, изменения отразятся и в оригинале, потому
что это один и тот же физический файл.

`deepcopy()` выполняет глубокое копирование — она рекурсивно создаёт полностью независимые копии всех объектов, включая
вложенные структуры. Это похоже на то, как если бы вы не только создали новую папку, но и перепечатали каждый документ
внутри неё. После такой операции изменения в копии никак не затронут оригинал, и наоборот.

Выбор между этими двумя функциями зависит от ситуации. Если вы работаете с неизменяемыми объектами (числа, строки,
кортежи) или уверены, что не будете менять вложенные элементы, достаточно `copy()`. Когда же требуется полная изоляция —
например, для тестовых данных, конфигураций или сложных изменяемых структур — без `deepcopy()` не обойтись.

[Содержание](/CONTENTS.md#содержание)

## **Middle Level**

Под поверхностью этих функций скрывается довольно интересная механика. Поверхностное копирование создаёт новый объект
того же типа, но не углубляется в его содержимое — все вложенные элементы остаются общими с оригиналом. Это происходит
потому, что `copy()` копирует только ссылки, а не сами объекты. Такой подход эффективен по памяти и времени, но требует
осторожности: изменение вложенного изменяемого объекта в копии повлияет на оригинал.

Глубокое копирование устроено сложнее. Функция `deepcopy()` рекурсивно обходит всю структуру объекта, создавая новые
экземпляры для каждого встреченного изменяемого объекта. Она умеет корректно обрабатывать даже циклические ссылки (когда
объекты ссылаются друг на друга), используя внутренний словарь для отслеживания уже скопированных объектов и
предотвращения бесконечной рекурсии.

Обе функции уважают специальные методы объектов: если класс определяет `__copy__()` или `__deepcopy__()`, используются
эти реализации, что позволяет контролировать процесс копирования. Это особенно полезно для объектов со сложным
внутренним состоянием, например, открытых файлов или сетевых соединений, которые не могут быть просто продублированы.

Стоит помнить о некоторых особенностях. Некоторые объекты (модули, классы, функции) не копируются, а возвращаются как
есть, поскольку обычно в этом нет смысла. Также `deepcopy()` может быть довольно медленной для больших и сложных
структур из-за рекурсивного обхода и создания множества новых объектов.

На практике глубокое копирование незаменимо при работе с многомерными структурами данных, графами или любыми объектами,
где важна полная независимость копии. Поверхностное же копирование часто используется для создания "снимков" состояния
объекта в определённый момент, когда достаточно скопировать только верхний уровень структуры.

[Содержание](/CONTENTS.md#содержание)

## **Senior Level**

`copy.copy()` и `copy.deepcopy()` — это чистый Python‑код в `Lib/copy.py`, который реализует общий протокол копирования
через поиск специальных методов, тип‑специфические таблицы и рекурсивный обход структуры с мемоизацией. Байткода как
такового там немного (обычные вызовы функций и атрибутов), вся «магия» — в логике диспетчеризации.

## copy.copy(): выбор стратегии

В CPython 3.9+ **copy.copy()** использует **tp_traverse** + `__copy__()` + **shallow copy** (`PyObject_Malloc`), *
*copy.deepcopy()** — **рекурсивный _deepcopy_dispatch** с **memo dict** (id→copy), `_reconstruct()` для классов, защита
от циклов. `Lib/copy.py`,`Objects/object.c`

## 1. copy.copy() - Lib/copy.py (Python C API)

```python
def copy(x):
    """Shallow copy operation on arbitrary Python objects."""
    cls = type(x)

    # 1. __copy__() метод класса
    copier = _copy_dispatch.get(cls)
    if copier is not None:
        return copier(x)

    try:
        # 2. Пробуем __copy__()
        cpy = x.__copy__()
        if hasattr(cpy, '__dict__'):
            cpy.__dict__.update(x.__dict__)
        return cpy
    except AttributeError:
        pass

    # 3. Встроенные типы
    if isinstance(x, dict):
        return dict(x)
    if isinstance(x, list):
        return list(x)
    if isinstance(x, set):
        return set(x)
    # ...

    # 4. Fallback: PyObject_CallMethod("__getnewargs__")
    newargs = _copy_call_getnewargs(x)
    return _reconstruct(x, len(newargs), newargs, ())
```

`copy(lst)` → `list(lst)` (shallow), `copy(obj)` → `obj.__copy__()` или
`type(obj)(*obj.__getnewargs__())`. **Не рекурсивно** — копирует только верхний уровень.

## 2. copy.deepcopy() - рекурсивный dispatch (Lib/copy.py)

```python
def deepcopy(x, memo=None):
    """Deep copy operation on arbitrary Python objects."""
    if memo is None:
        memo = {}

    d = id(x)
    y = memo.get(d, None)  # Уже копировали?
    if y is not None:
        return y  # Цикл! Возвращаем копию

    cls = type(x)

    # 1. __deepcopy__()
    copier = _deepcopy_dispatch.get(cls)
    if copier is not None:
        y = copier(x, memo)

    # 2. Встроенные типы
    elif cls in _deepcopy_dispatch:
        y = _deepcopy_dispatch[cls](x, memo)

    # 3. Классы с __reduce__
    elif hasattr(x, '__reduce_ex__'):
        args = x.__reduce_ex__(2)[1]
        y = _reconstruct(x, 0, args, (), None, None, memo)

    else:
        # 4. Fallback: __getnewargs__ + __dict__ + __slots__
        reductor = getattr(x, "__reduce_ex__", None)
        if reductor is not None:
            args = reductor(4)[1]
        else:
            args = _copy_call_getnewargs(x)
        y = _reconstruct(x, len(args), args, (), None, None, memo)

    memo[d] = y  # Записываем в memo
    return y
```

`deepcopy(obj)` рекурсивно копирует **все вложенные объекты**. `memo[id(obj)]=copy`
предотвращает **бесконечный цикл** при `l.append(l)`.

## 3. _deepcopy_dispatch таблица (Lib/copy.py)

```python
_deepcopy_dispatch = d = {}
d[dict] = _deepcopy_dict
d[list] = _deepcopy_list
d[set] = _deepcopy_set
d[tuple] = _deepcopy_tuple
d[frozenset] = _deepcopy_frozenset


def _deepcopy_dict(x, memo):
    y = {}
    memo[id(x)] = y
    for key, value in x.items():
        y[deepcopy(key, memo)] = deepcopy(value, memo)  # Рекурсия!
    return y


def _deepcopy_list(x, memo):
    y = []
    memo[id(x)] = y
    append = y.append  # Быстрый append
    for item in x:
        append(deepcopy(item, memo))  # Рекурсия!
    return y
```

`_deepcopy_dict({a: [1]})` → `{copy(a): copy([1])}` → рекурсивно копирует **каждый**
ключ/значение. `memo` сохраняет уже сделанные копии.

## 4. PyList_Type.tp_richcompare для list.copy()

```c
static PyObject *list_copy(PyListObject *self) {
    Py_ssize_t len = Py_SIZE(self);
    PyObject **src = self->ob_item;
    PyObject **dest;
    
    // Создаём новый список той же ёмкости
    PyListObject *newlist = _PyList_New(len);  # Выделяем ob_item[len+overalloc]
    if (newlist == NULL)
        return NULL;
    
    dest = newlist->ob_item;
    
    // Копируем указатели (shallow!)
    for (Py_ssize_t i = 0; i < len; i++) {
        PyObject *v = src[i];
        Py_INCREF(v);              // +refcnt на каждый элемент
        dest[i] = v;
    }
    
    return (PyObject *)newlist;
}
```

`lst.copy()` → `_PyList_New(len)` → копирует **указатели** `ob_item[]` с `Py_INCREF()`. *
*Shallow** — вложенные списки **не копируются**.

## 5. _PyDict_NewPresized() для dict.copy() (3.9+)

```c
PyObject *_PyDict_NewPresized(Py_ssize_t expected_size) {
    PyDictObject *mp;
    Py_ssize_t table_size = _PyDict_NewPresizedTableSize(expected_size);
    
    mp = PyObject_GC_New(PyDictObject, &PyDict_Type);
    if (mp == NULL)
        return NULL;
    
    mp->ma_used = 0;
    mp->ma_version_tag = DICT_NEXT_VERSION();
    mp->ma_table = PyMem_Calloc(table_size, sizeof(PyDictUnicodeEntry));
    mp->ma_keys = NULL;  // Создадим позже
    mp->ma_values = NULL;
    
    // Выделяем ma_keys той же ёмкости
    if (dictkeys_new(mp, table_size) < 0) {
        PyDictObject_Clear(mp);
        Py_DECREF(mp);
        return NULL;
    }
    
    _PyObject_GC_TRACK(mp);
    return (PyObject *)mp;
}
```

`dict.copy()` → новый PyDictObject с **пустой** `ma_keys` + `ma_values`, затем
`dict_update(new, old)` копирует все пары **shallow**.

## 6. _reconstruct() для пользовательских классов

```python
def _reconstruct(x, len_args, args, state, listitems, dictitems, memo):
    # 1. Создаём новый экземпляр
    cls = type(x)
    newobj = object.__new__(cls)

    # 2. Восстанавливаем состояние
    if state is not None:
        state = deepcopy(state, memo)
        newobj.__dict__ = state
    elif hasattr(newobj, '__slots__'):
        # __slots__ копируем вручную
        for key, value in state.items():
            setattr(newobj, key, deepcopy(value, memo))

    # 3. Восстанавливаем mutable атрибуты
    if len_args > 0:
        newobj.__init__(*deepcopy(args, memo))

    return newobj
```

`deepcopy(MyClass())` → `MyClass.__new__()` → `deepcopy(state)` → `__dict__` →
`__init__(*args)`. **Полная копия состояния**.

## 7. Защита от рекурсии: memo[id(obj)]

```python
l = [1, 2]
l.append(l)  # Цикл!
deep_l = deepcopy(l, memo={})

# Первый вызов: memo={} пустой
y = _deepcopy_list(l, memo)
memo[id(l)] = y  # Записали!

# Рекурсивный вызов l[3] = l:
item = deepcopy(l[3], memo)  # l[3] == l
d = id(l[3])  # == id(l)
y = memo.get(d)  # НАЙДЕН! Возвращаем копию
```

`memo[id(original)]=copy` **предотвращает** копирование уже обработанных объектов.
`[1, [1, [1, ...]]]` копируется **один раз**.

## 8. C-level: PyObject_Malloc для shallow copy

```c
static PyObject *generic_copy(PyObject *self) {
    PyObject *newobj;
    
    // 1. Выделяем память той же структуры
    newobj = PyType_GenericAlloc(Py_TYPE(self), 1);
    if (newobj == NULL)
        return NULL;
    
    // 2. Копируем PyObject_HEAD
    Py_INCREF(Py_TYPE(self));
    newobj->ob_refcnt = 1;
    newobj->ob_type = Py_TYPE(self);
    
    // 3. Копируем __dict__ shallow
    if (Py_TYPE(self)->tp_dictoffset != 0) {
        PyObject **dictptr = _PyObject_GetDictPtr(self);
        if (dictptr != NULL && *dictptr != NULL) {
            PyObject *dict = PyDict_Copy(*dictptr);  # Shallow dict
            if (dict == NULL) {
                Py_DECREF(newobj);
                return NULL;
            }
            _PyObject_SetDict(newobj, dict);
        }
    }
    
    return newobj;
}
```

`__copy__()` → `PyType_GenericAlloc()` → копирует `ob_type` + `__dict__.copy()` (shallow). *
*Не трогает атрибуты**.

## 9. Не копируемые типы (deepcopy игнорирует)

```python
# Модули, функции, файлы, сокеты НЕ копируются
def _deepcopy_func(x, memo):
    return x  # Возвращаем оригинал!


def _deepcopy_module(x, memo):
    return x


_deepcopy_dispatch[type(open('file.txt'))] = _deepcopy_file_like
```

`deepcopy(open())` → **оригинал** (нельзя клонировать дескриптор). Функции/классы — **shallow
** (refcnt++).

**copy.copy()/deepcopy()** в CPython 3.9+ — **Lib/copy.py** с `_deepcopy_dispatch`, **memo[id→copy]** против циклов,
`PyList_New()`/`PyDict_NewPresized()` + `Py_INCREF()` (shallow), `_reconstruct()` для классов, `__copy__()`/
`__deepcopy__()` хуки.

- [Содержание](/CONTENTS.md#содержание)

---

# **Асинхронность**

[Содержание](/CONTENTS.md#содержание)

## **Junior Level**

Асинхронность в Python — это модель программирования, позволяющая эффективно управлять множеством операций, которые
много ждут, но мало вычисляют. Представьте, что вы — единственный водитель, который развозит нескольких пассажиров по
одному маршруту. Вместо того чтобы ждать у дверей каждого, пока он соберётся (блокирующее ожидание), вы отвозите
первого, а пока он выходит, едете за следующим. Так работает асинхронный код: он не блокируется на одной задаче, а
переключается на другие, пока первая ожидает, например, ответа от сервера или чтения файла. (???)

Ключевыми инструментами здесь являются `async` и `await`. `async def` определяет асинхронную функцию (корутину) —
специальную функцию, которая умеет ставить себя на паузу. Ключевое слово `await` как раз и ставит корутину на паузу,
говоря: "Я буду ждать результата этой операции, а пока можешь заняться чем-то другим". Всей этой каруселью управляет
диспетчер — **цикл событий (event loop)**, который решает, какую корутину запустить или возобновить дальше.

Этот подход идеален для задач, связанных с вводом-выводом (I/O): сетевые запросы, работа с базами данных, чтение файлов.
Он позволяет обрабатывать тысячи одновременных соединений в одном потоке, экономя ресурсы. Важно помнить, что
асинхронность не ускоряет вычисления — для сложной математики лучше подходят другие инструменты, например,
многопроцессорность.

[Содержание](/CONTENTS.md#содержание)

## **Middle Level**

За внешней простотой асинхронного кода скрывается продуманная архитектура, основанная на нескольких ключевых понятиях. В
её сердце находятся **корутины** — функции, умеющие приостанавливать выполнение, сохраняя своё состояние, а затем
возобновляться с того же места. Когда корутина встречает выражение `await`, она не блокирует поток, а возвращает
управление циклу событий, позволяя другим задачам продвинуться вперёд. Этот механизм работает благодаря специальным *
*awaitable-объектам**, к которым относятся сами корутины, **Задачи (Tasks)** и низкоуровневые **Фьючерсы (Futures)**.

Задача — это обёртка, которая планирует выполнение корутины в цикле событий. Создавая задачу через
`asyncio.create_task()`, вы запускаете фоновое выполнение, которое будет конкурентно с другими задачами. Фьючерс же
представляет собой отложенный результат асинхронной операции и служит строительным блоком для более высокоуровневых
абстракций.

Управляет всем этим хозяйством **цикл событий**, который непрерывно опрашивает готовность операций ввода-вывода (
например, проверяет, пришли ли данные в сокет) и возобновляет корутины, ожидающие эти операции. Для координации между
множеством конкурентных задач `asyncio` предоставляет асинхронные аналоги классических примитивов синхронизации.

**Блокировка (Lock)** гарантирует, что только одна корутина в данный момент может работать с защищаемым ресурсом,
например, изменять общую структуру данных или записывать в файл. Когда корутина захватывает блокировку, все остальные,
пытающиеся сделать то же самое, будут ждать её освобождения.

**Семафор (Semaphore)** расширяет эту идею, позволяя ограничить количество корутин, одновременно работающих с ресурсом.
Например, семафор со значением 3 разрешит трём корутинам выполнять операцию, а четвёртая будет ждать, пока одна из
первых не завершится. Это полезно для ограничения количества одновременных сетевых запросов или подключений к базе
данных.

**Событие (Event)** позволяет корутинам ждать какого-либо однократного события. Пока флаг события не установлен, все
ожидающие его корутины приостановлены. Как только событие происходит (вызывается `event.set()`), все ждущие корутины
просыпаются и продолжают работу. Так можно координировать начало выполнения или ждать инициализации системы.

**Условная переменная (Condition)** — наиболее гибкий примитив, позволяющий корутинам ждать не просто события, а
выполнения определённого условия. Она часто используется в паттерне «производитель-потребитель», где потребители ждут
появления данных в очереди, а производитель уведомляет их, когда данные готовы. Условная переменная внутренне содержит
блокировку, поэтому работает в связке с `async with`.

Помимо примитивов синхронизации, асинхронность в Python включает специальные конструкции для работы с ресурсами:
`async with` для асинхронных контекстных менеджеров (например, для подключений к базе данных) и `async for` для итерации
по асинхронным генераторам.

При работе с асинхронным кодом важно помнить несколько принципов. Используйте примитивы синхронизации только когда это
действительно необходимо — часто можно обойтись очередями (`asyncio.Queue`). Избегайте длительного удержания блокировок,
особенно с вызовами `await` внутри, чтобы не создавать взаимные блокировки. Помните, что ожидание примитива может быть
прервано отменой задачи, и корректно обрабатывайте это. И наконец, тщательно тестируйте код на наличие состояний гонки,
которые могут возникнуть даже в одном потоке из-за переключения между корутинами.

Асинхронность в Python — это мощный инструмент для создания высокопроизводительных приложений, работающих с множеством
одновременных операций ввода-вывода. При грамотном использовании она позволяет писать чистый, структурированный и
эффективный код, в котором конкурентность управляется явно и предсказуемо.

[Содержание](/CONTENTS.md#содержание)

## **Senior Level**

В CPython 3.9+ **асинхронность** — **PyCoroObject**/**PyAsyncGenObject** (`CO_COROUTINE`/`CO_ASYNC_GENERATOR` флаги),
байткоды `GET_AWAITABLE`/`GET_AITER`/`GET_ANEXT`, **event loop** в `Modules/_asynciomodule.c` с **TaskObj** + **Future
**. `Objects/genobject.c`,`Python/ceval.c`,`Modules/_asynciomodule.c`

## 1. PyCoroObject структура (Objects/genobject.c)

```c
typedef struct {
    PyGenObject_HEAD              // Наследует от PyGenObject
    PyObject *cr_origin;          // "<async def coro>" строка происхождения
    int cr_frame_origin_depth;    // Глубина фрейма для отладки
} PyCoroObject;

typedef struct {
    PyGenObject_HEAD
    PyObject *ag_frame;           // Async generator frame
    PyObject *ag_running;         // Lock
    PyObject *ag_finalizer;       // aclose() финализатор
} PyAsyncGenObject;
```

`async def` → **PyCoroObject** (замороженный фрейм с `await`), `async def gen(): yield` → *
*PyAsyncGenObject**. `cr_origin` для traceback.

## 2. Байткод async/await (Python 3.9+)

```python
async def coro():
    await asyncio.sleep(1)  # GET_AWAITABLE
    return 42
```

```
# Байткод:
  0 GET_AWAITABLE         0    # sleep_obj.__await__()
  2 LOAD_CONST            0 (None)
  4 YIELD_FROM                  # Вызываем awaitable.send(None)
  6 POP_TOP                       # Результат sleep
  8 LOAD_CONST            1 (42)
 10 RETURN_VALUE
```

`await obj` → `GET_AWAITABLE` (obj.__await__() → iterator) → `YIELD_FROM` (iterator.send(
None) → приостанавливаем корутину).

## 3. GET_AWAITABLE байткод (ceval.c)

```c
case GET_AWAITABLE: {
    PyObject *iter = TOP();            // obj (sleep())
    
    // 1. Уже корутина? Возвращаем как есть
    if (PyCoro_CheckExact(iter) || 
        (PyGen_CheckExact(iter) && 
         ((PyCodeObject*)PyGen_GET_CODE(iter))->co_flags & CO_ITERABLE_COROUTINE)) {
        Py_INCREF(iter);
        DISPATCH();
    }
    
    // 2. Ищем __await__()
    PyObject *awaitable = PyObject_CallMethodObjArgs(
        iter, &_Py_ID(__await__), NULL);
    
    if (awaitable == NULL) {
        goto error;
    }
    
    // 3. __await__ должен вернуть iterator
    if (!PyIter_Check(awaitable)) {
        PyErr_Format(PyExc_TypeError,
            "'%.200s.__await__()' must return an iterator",
            Py_TYPE(iter)->tp_name);
        Py_DECREF(awaitable);
        goto error;
    }
    
    Py_DECREF(iter);                   // Заменяем obj → awaitable iterator
    PEEK(0) = awaitable;
    DISPATCH();
}
```

`GET_AWAITABLE sleep()` → `sleep.__await__()` → **iterator** на стек. `YIELD_FROM` вызывает
`iterator.send(None)` → **приостанавливает** корутину.

## 4. YIELD_FROM для await (ceval.c)

```c
case YIELD_FROM: {
    PyObject *iter = TOP();            // __await__() iterator
    PyObject *sub_iter = NULL;
    
    // Получаем send arg (None для первого await)
    PyObject *send_arg = PEEK(1);
    
    // Вызываем iterator.send(None)
    PyObject *result = PyIter_Send(iter, send_arg, &sub_iter);
    Py_DECREF(send_arg);
    
    if (result == NULL) {
        // StopIteration(result.value) → возобновляем корутину
        PyObject *exc = PyErr_Occurred();
        if (PyExceptionInstance_Check(exc) && 
            PyExceptionInstance_Class(exc) == (PyObject*)&PyExc_StopIteration) {
            PyObject *value = PyObject_GetAttr(exc, &_Py_ID(value));
            Py_DECREF(exc);
            if (value == NULL) {
                goto error;
            }
            Py_DECREF(iter);
            PEEK(0) = value;           // Результат await на стек
            DISPATCH();
        }
        goto error;
    }
    
    // Возвращаем управление event loop
    Py_DECREF(result);
    Py_DECREF(iter);
    goto yield_from_suspend;           // Корутина приостанавливается
}
```

**Объяснение людей:** `YIELD_FROM` → `awaitable.send(None)` → **StopIteration(result)** → **возобновляем** корутину с
результатом. Event loop берёт управление.

## 5. _Py_MakeCoro() - создание корутины (genobject.c)

```c
PyObject *_Py_MakeCoro(PyFunctionObject *func) {
    PyCodeObject *code = (PyCodeObject*)func->func_code;
    int coro_flags = code->co_flags;
    
    assert(coro_flags & (CO_COROUTINE | CO_ASYNC_GENERATOR));
    
    if (coro_flags == CO_COROUTINE) {
        PyCoroObject *coro = (PyCoroObject*)make_gen(&PyCoro_Type, func);
        if (coro == NULL) return NULL;
        
        // Заполняем cr_origin для traceback
        coro->cr_origin = compute_cr_origin(0, NULL);
        return (PyObject*)coro;
    }
    
    if (coro_flags == CO_ASYNC_GENERATOR) {
        PyAsyncGenObject *asyncgen = (PyAsyncGenObject*)make_gen(&PyAsyncGen_Type, func);
        // ...
        return (PyObject*)asyncgen;
    }
}
```

`async def f():` → `PyFunctionObject` с `CO_COROUTINE` → `f()` → `_Py_MakeCoro()` → *
*PyCoroObject** с `gi_frame`.

## 6. asyncio TaskObj (Modules/_asynciomodule.c)

```c
typedef struct {
    PyObject_HEAD
    PyObject *task_future;         // Связанный Future
    PyObject *task_coro;           // Корутина
    PyObject *task_context;        // Task context
    PyObject *task_loop;           // Event loop
    PyObject *task_handle;         // Callback в loop
    int task_must_cancel;          // Отмена?
    int task_log_destroy_pending;  // Лог?
    int task_canceled;             // Отменена
    int task_state;                // TASK_PENDING/TASK_FINISHED
} TaskObj;
```

`asyncio.create_task(coro())` → **TaskObj** с `task_coro=PyCoroObject`, `task_future=Future`.
Event loop вызывает `task_coro.send()`.

## 7. task_step() - выполнение Task (asynciomodule.c)

```c
static PyObject *task_step(TaskObj *task) {
    PyObject *res;
    PyObject *coro = task->task_coro;
    
    // Выполняем корутину до следующего await
    res = PyCoro_Send(coro, Py_None);  // coro.send(None)
    
    if (res == NULL) {
        PyObject *exc = PyErr_Occurred();
        if (PyErr_GivenExceptionMatches(exc, PyExc_StopIteration)) {
            // Корутина завершилась
            PyErr_Clear();
            task->task_state = TASK_FINISHED;
            return task_set_result(task, Py_None);
        }
        // Ошибка → set_exception
        return task_set_exception(task, exc);
    }
    
    // Получили awaitable → планируем следующий шаг
    task->task_handle = create_future_callback(task_loop, task_wakeup, task);
    Py_INCREF(res);
    return res;  // Вернём управление event loop
}
```

Event loop → `task_step()` → `coro.send(None)` → **awaitable** → **callback** `task_wakeup` в
loop. **Корутина приостанавливается**.

## 8. Async for/await (GET_AITER/GET_ANEXT)

```python
async for x in agen():  # agen.__aiter__()
```

```
# Байткод:
GET_AITER                 # aiter(agen)
SETUP_ASYNC_WITH          # Сохраняем aiter
GET_ANEXT                 # anext(awaitable)
GET_AWAITABLE             # anext.__await__()
YIELD_FROM                # await anext()
END_ASYNC_FOR             # Обработка StopAsyncIteration
```

`async for` → `GET_AITER` (`agen.__aiter__()`) → `GET_ANEXT` (`aiter.__anext__()`) → `await` →
`x = result`.

## 9. Event loop: BaseEventLoop (Lib/asyncio/events.py)

```python
class BaseEventLoop:
    def _run_once(self):
        """Run one full iteration of the event loop."""
        # 1. Выполняем scheduled callbacks
        self._run_ready_callbacks()

        # 2. Poll I/O selectors (epoll/select)
        timeout = self._calculate_timeout()
        event_list = self._selector.select(timeout)

        # 3. Выполняем I/O callbacks
        for key, mask in event_list:
            self._process_events(key, mask)

        # 4. Обновляем timeouts
        self._update_timers()
```

Event loop **круг**: callbacks → I/O poll (epoll/kqueue) → I/O callbacks → timers. *
*Однопоточный**, **кооперативный**.

## 10. coro.send(None) - возобновление (genobject.c)

```c
PyObject *PyCoro_Send(PyObject *coro, PyObject *arg) {
    PyCoroObject *co = (PyCoroObject*)coro;
    PyFrameObject *f = co->gi_frame;
    
    if (f == NULL || f->frame_flags == FRAME_CLOSED) {
        PyErr_SetNone(PyExc_StopIteration);
        return NULL;
    }
    
    // Кладём arg на стек фрейма
    *f->f_stacktop++ = Py_NewRef(arg ? arg : Py_None);
    
    // Запускаем до следующего yield/await
    PyObject *retval = _PyEval_EvalFrame(f);
    
    if (retval == (PyObject*)co) {
        retval = NULL;  // Возвращаемся event loop
    }
    
    return retval;
}
```

`awaitable.send(None)` → `f_stacktop[0]=None` → выполнение до `YIELD_FROM` → **возврат
корутины** event loop'у.

**Асинхронность** в CPython 3.9+ — **PyCoroObject** (`CO_COROUTINE`), `GET_AWAITABLE`/`YIELD_FROM`, **TaskObj** в
`_asynciomodule.c`, **event loop** (epoll + callbacks), **кооперативное** переключение на `await`.

- [Содержание](/CONTENTS.md#содержание)

---

# **Многопоточность**

[Содержание](/CONTENTS.md#содержание)

## **Junior Level**

Многопоточность в Python позволяет нескольким потокам выполняться в рамках одного процесса, подобно сотрудникам,
работающим над разными задачами в общем офисе. Каждый поток — это независимая последовательность инструкций, которая
может выполняться параллельно с другими, используя общую память и ресурсы программы.

Главная цель многопоточности — повысить отзывчивость приложений, особенно когда они сталкиваются с операциями
ввода-вывода (I/O), такими как сетевые запросы или чтение файлов. Пока один поток ждёт ответа от сервера, другие могут
продолжать работу, что создаёт иллюзию одновременного выполнения.

Однако у многопоточности в Python есть особенность — **Global Interpreter Lock (GIL)**, который позволяет только одному
потоку в любой момент времени выполнять байт-код Python. Это делает многопоточность неэффективной для задач, интенсивно
использующих процессор (например, сложных математических вычислений), но вполне подходящей для операций, связанных с
ожиданием, так как во время ожидания I/O поток освобождает GIL, давая возможность работать другим.

Для создания и управления потоками используется модуль `threading`, который предоставляет класс `Thread` для создания
потоков и различные примитивы синхронизации, такие как блокировки (`Lock`), семафоры (`Semaphore`) и события (`Event`),
помогающие координировать доступ к общим ресурсам.

[Содержание](/CONTENTS.md#содержание)

## **Middle Level**

В основе многопоточности Python лежит **Global Interpreter Lock (GIL)** — механизм, который защищает память
интерпретатора CPython, поскольку его система управления памятью не является потокобезопасной. GIL гарантирует, что
только один поток в любой момент времени выполняет байт-код Python, что упрощает работу с объектами, но ограничивает
истинный параллелизм. Тем не менее, во время операций ввода-вывода или при вызове внешнего кода (например, функций из
библиотек, написанных на C, таких как `numpy`), GIL может быть отпущен, позволяя другим потокам активироваться.

Поток в течение своего жизненного цикла проходит через состояния: создание, готовность к выполнению, запуск,
блокировка (при ожидании I/O или освобождения блокировки) и завершение. Управление этими состояниями осуществляется
планировщиком операционной системы, который распределяет время процессора между потоками.

Для координации потоков модуль `threading` предлагает набор примитивов синхронизации.

- **Блокировка (`Lock`)** обеспечивает взаимное исключение, разрешая доступ к общему ресурсу только одному потоку.
- **Реентерабельная блокировка (`RLock`)** позволяет одному потоку захватывать её несколько раз, что полезно в
  рекурсивных функциях.
- **Семафор (`Semaphore`)** ограничивает количество потоков, одновременно получающих доступ к ресурсу.
- **Событие (`Event`)** позволяет потокам ждать сигнала от других потоков
- **Условная переменная (`Condition`)** — ждать выполнения определённого условия, связанного с общим ресурсом.

Потоки могут быть демоническими (`daemon=True`), что означает их автоматическое завершение при завершении основного
потока программы. Не-демонические потоки продолжают выполнение, даже если основной поток завершился. Для хранения
данных, уникальных для каждого потока, используется `threading.local()`, что удобно для хранения состояния, специфичного
для потока.

Для удобного управления множеством задач применяется `ThreadPoolExecutor` из модуля `concurrent.futures`,
предоставляющий высокоуровневый интерфейс для пула потоков. Это упрощает параллельное выполнение задач, автоматически
управляя созданием и завершением потоков.

Понимание работы GIL и правильное использование примитивов синхронизации критически важны для написания безопасного и
эффективного многопоточного кода. Многопоточность в Python — мощный инструмент для I/O-задач, но для CPU-интенсивных
вычислений стоит рассмотреть многопроцессорность или альтернативные реализации Python, такие как PyPy или использование
библиотек, отпускающих GIL.

[Содержание](/CONTENTS.md#содержание)

## **Senior Level**

В CPython 3.9+ **многопоточность** — **PyThreadState** на поток (`tstate->thread_id`), **GIL** (`take_gil/drop_gil`), **
PyThreadState_Swap()`, **PyThread_start_new_thread()` + **_PyRuntime.threads.list**. **PEP 684** (3.12) *
*per-interpreter** потоки. `Python/pylifecycle.c`,`Python/ceval_gil.c`,`Include/pystate.h`

## 1. PyThreadState структура (Include/pystate.h)

```c
typedef struct _ts {
    struct _ts *next;              // Список потоков интерпретатора
    PyInterpreterState *interp;    // Принадлежит интерпретатору
    PyThread_id thread_id;         // pthread_self() или GetCurrentThreadId()
    int gilstate_counter;          // Счётчик PyGILState_Ensure()
    int recursion_depth;           // Глубина рекурсии
    int tracing;                   // sys.settrace()
    int use_tracing;               // Активен ли tracing
    PyObject *dict;                // thread-local storage
    PyObject *async_exc;           // Текущее исключение
    PyFrameObject *frame;          // Текущий фрейм (для inspect)
    PyObject *frame_obj;           // PyFrameObject
    uint64_t coroutines;           // Счётчик корутин
    // GIL состояние
    int holds_gil;                 // Держит ли GIL
    PyObject *py_thread_state;     // Python threading._get_thread_ident()
} PyThreadState;
```

Каждый поток имеет **свой** PyThreadState с `thread_id`, `holds_gil`, `frame`. **GIL**
принадлежит **только одному** tstate.

## 2. Py_NewInterpreter() / PyThreadState_New (pylifecycle.c)

```c
PyThreadState *PyThreadState_New(PyInterpreterState *interp) {
    PyThreadState *tstate;
    
    tstate = (PyThreadState*)PyObject_MALLOC(sizeof(PyThreadState));
    if (tstate == NULL)
        return NULL;
    
    tstate->interp = interp;           // Связываем с интерпретатором
    tstate->thread_id = PyThread_get_thread_ident();  // pthread_self()
    tstate->frame = NULL;
    tstate->recursion_depth = 0;
    tstate->tracing = 0;
    tstate->use_tracing = 0;
    tstate->holds_gil = 0;
    tstate->gilstate_counter = 0;
    tstate->dict = NULL;
    
    // Добавляем в список потоков интерпретатора
    tstate->next = interp->tstate_head;
    interp->tstate_head = tstate;
    
    _PyObject_INIT((PyObject*)tstate, &PyThreadState_Type);
    return tstate;
}
```

Новый поток → `PyThreadState_New(main_interp)` → `tstate->thread_id=my_tid` → добавляем в
`interp->tstate_head` список.

## 3. PyThread_start_new_thread() - запуск потока

```c
long PyThread_start_new_thread(PyThread_start_func_t func, void *arg) {
    PyThreadState *tstate = PyThreadState_Get();  // Текущий поток
    
    // Создаём PyThreadState для нового потока
    PyThreadState *new_tstate = PyThreadState_New(tstate->interp);
    if (new_tstate == NULL) {
        return -1;
    }
    
    // C-thread функция
    void *thread_arg = PyMem_Malloc(sizeof(struct thread_arg));
    ((struct thread_arg*)thread_arg)->interp = tstate->interp;
    ((struct thread_arg*)thread_arg)->tstate = new_tstate;
    ((struct thread_arg*)thread_arg)->start_func = func;
    ((struct thread_arg*)thread_arg)->start_arg = arg;
    
    // Запускаем pthread_create()
    int err = pthread_create(&tid, NULL, pythread_run, thread_arg);
    
    if (err != 0) {
        PyThreadState_Clear(new_tstate);
        PyMem_Free(thread_arg);
        PyThreadState_DeleteCurrent();
        return -1;
    }
    
    return 0;
}
```

`threading.Thread().start()` → `PyThread_start_new_thread()` →
`pthread_create(pythread_run, arg)` → **новый C-поток**.

## 4. pythread_run() - входная точка потока

```c
static void *pythread_run(void *arg_) {
    struct thread_arg *arg = (struct thread_arg*)arg_;
    
    PyThreadState_Swap(arg->tstate);   // Активируем PyThreadState
    _PyRuntime.threads.list_append(arg->tstate);  // В глобальный список
    
    // Захватываем GIL
    PyEval_AcquireLock(arg->tstate);
    
    // Вызываем Python функцию
    PyObject *res = arg->start_func(arg->start_arg);
    
    // Освобождаем ресурсы
    PyEval_ReleaseLock(arg->tstate);
    _PyRuntime.threads.list_remove(arg->tstate);
    
    PyThreadState_Swap(NULL);          // Деактивируем tstate
    PyThreadState_Clear(arg->tstate);
    PyThreadState_DeleteCurrent();
    
    PyMem_Free(arg);
    return (void*)res;
}
```

C-поток → `PyThreadState_Swap(tstate)` → **GIL захват** → `target_func()` → **GIL освобождение
** → `PyThreadState_Clear()`.

## 5. PyThreadState_Swap() - переключение контекста

```c
PyThreadState *PyThreadState_Swap(PyThreadState *new_tstate) {
    PyThreadState *old_tstate = _PyThreadState_UncheckedGet();  // TLS
    
    if (old_tstate == new_tstate) {
        return old_tstate;             // Уже активен
    }
    
    // Сохраняем старый контекст
    if (old_tstate != NULL) {
        // Сохраняем фрейм, исключения, GIL
        old_tstate->frame = _PyThreadState_GetFrame(old_tstate);
        old_tstate->recursion_depth = PyThreadState_Get()->recursion_depth;
    }
    
    // Активируем новый
    _PyThreadState_Set(new_tstate);    // Thread Local Storage (TLS)
    
    if (new_tstate != NULL) {
        // Восстанавливаем контекст
        PyEval_RestoreThread(new_tstate);
    }
    
    return old_tstate;
}
```

**TLS** (Thread Local Storage) хранит **активный** PyThreadState. `Swap(NULL)` → деактивируем
Python. `Swap(tstate)` → активируем поток.

## 6. threading.Thread C API (Lib/threading.py → Modules/_threadmodule.c)

```c
static PyObject *thread_PyThread_start_new_thread(PyObject *self, PyObject *args) {
    PyObject *func, *arg;
    
    if (!PyArg_ParseTuple(args, "OO:start_new_thread", &func, &arg))
        return NULL;
    
    // Вызываем PyThread_start_new_thread()
    long retval = PyThread_start_new_thread(
        (PyThread_start_func_t)pythread_wrapper, func);
    
    if (retval < 0) {
        PyErr_SetFromErrno(PyExc_RuntimeError);
        return NULL;
    }
    
    Py_RETURN_NONE;
}
```

`Thread(target=f).start()` → `_thread.start_new_thread(f, ())` →
`PyThread_start_new_thread(pythread_wrapper, f)`.

## 7. Per-interpreter потоки (PEP 684, 3.12+)

```c
PyStatus PyInterpreterState_New(PyThreadState *tstate) {
    PyInterpreterState *interp = PyInterpreterState_New();
    
    // Каждый интерпретатор имеет свой список потоков
    interp->tstate_head = NULL;
    interp->threads = NULL;           // PyThreadState.list
    interp->threads_lock = PyThread_create_lock();
    
    // Создаём главный поток для интерпретатора
    PyThreadState *interp_tstate = PyThreadState_New(interp);
    interp->tstate_head = interp_tstate;
    
    return PyStatus_OK();
}
```

**3.12+**: `subinterp = Py_NewInterpreter()` → **отдельный** список `tstate_head`. Потоки **не
делят** tstate между интерпретаторами.

## 8. _PyRuntime.threads - глобальный реестр (3.13+)

```c
struct _PyRuntimeState {
    PyThreadStateList threads;     // Все активные потоки
    PyThread_type_lock threads_lock;
};

void _PyRuntime_ThreadsList_Init(struct _PyRuntimeState *runtime) {
    runtime->threads_lock = PyThread_create_lock();
    runtime->threads.head = NULL;
}

void _PyRuntime_ThreadsList_Append(PyThreadState *tstate) {
    PyThread_acquire_lock(_PyRuntime.threads_lock, 1);
    tstate->threads_next = _PyRuntime.threads.head;
    _PyRuntime.threads.head = tstate;
    PyThread_release_lock(_PyRuntime.threads_lock);
}
```

**Глобальный** список **всех** PyThreadState. `sys._current_frames()` перебирает
`threads.head`.

## 9. GIL + многопоточность взаимодействие

```c
// Каждый байткод под GIL
case LOAD_FAST: {
    if (!tstate->holds_gil) {
        Py_FatalError("bytecode without GIL");
    }
    // ...
}

// Авто drop_gil каждые 1000 инструкций
if (--tstate->gilstate_counter <= 0) {
    _PyEval_ReleaseLock(tstate);
    _PyEval_AcquireLock(tstate);   // Другой поток может захватить
}
```

**Все** байткоды требуют `tstate->holds_gil=1`. Каждые ~1000 инструкций — **шанс** другому
потоку.

## 10. Free-threaded (PEP 703, 3.13+ --disable-gil)

```c
#ifdef Py_GIL_DISABLED
case LOAD_GLOBAL: {
    PyObject *name = GETITEM(names, oparg>>1);
    
    // Атомарный поиск без GIL!
    res = _PyDict_LookupWithCacheAtomic(
        frame->f_globals, name, hints);
        
    if (res == NULL) {
        res = _PyDict_LookupWithCacheAtomic(
            frame->f_builtins, name, hints);
    }
}
#endif
```

**3.13 free-threaded**: **атомарные** операции словарей/списков. **Настоящий** параллелизм
байткода.

**Многопоточность** в CPython 3.9+ — **PyThreadState/thread_id**, `PyThreadState_Swap()` + **TLS**,
`PyThread_start_new_thread()` → `pthread_create()`, **per-interpreter** списки (3.12), **GIL** (`holds_gil`), *
*free-threaded** атомарные операции (3.13).

- [Содержание](/CONTENTS.md#содержание)

---

# **Мультипроцессинг**

[Содержание](/CONTENTS.md#содержание)

## **Junior Level**

Мультипроцессинг в Python — это подход к параллельным вычислениям, использующий несколько независимых процессов вместо
потоков в рамках одной программы. Представьте, что вместо одного офиса с сотрудниками (потоками), работающими в общем
пространстве, вы создаёте несколько полностью отдельных офисов, каждый со своим собственным помещением, оборудованием и
набором сотрудников.

Каждый процесс работает в своём собственном пространстве памяти и имеет отдельный экземпляр интерпретатора Python с
собственным Global Interpreter Lock (GIL). Это позволяет полностью обойти ограничения GIL и задействовать все доступные
ядра процессора для выполнения задач, требующих интенсивных вычислений — таких как обработка изображений, сложные
математические расчёты или анализ больших данных.

Полная изоляция процессов обеспечивает большую стабильность: если один процесс завершится с ошибкой, остальные продолжат
работу. Однако эта же изоляция усложняет обмен данными между процессами — они не могут просто обращаться к общей памяти,
как это делают потоки. Для взаимодействия приходится использовать специальные механизмы межпроцессного взаимодействия (
IPC). Для работы с процессами в Python используется модуль `multiprocessing`, который предоставляет API, во многом
похожий на `threading`, но предназначенный для процессов.

[Содержание](/CONTENTS.md#содержание)

## **Middle Level**

Архитектурно каждый процесс в Python представляет собой отдельный экземпляр интерпретатора со своим адресным
пространством, кучей и стеком. Это обеспечивает настоящий параллелизм на уровне операционной системы — процессы могут
выполняться одновременно на разных ядрах процессора, что делает мультипроцессинг эффективным решением для
CPU-интенсивных задач.

Создание процессов может происходить тремя основными способами, каждый со своими особенностями. **Fork** (доступен в
Unix-системах) быстро копирует память родительского процесса, но при этом наследует все открытые файловые дескрипторы и
блокировки, что иногда приводит к неожиданным проблемам. **Spawn** (используется по умолчанию начиная с Python 3.8)
запускает новый интерпретатор Python, который импортирует модуль и выполняет целевую функцию — это безопаснее, но
требует больше времени. **Forkserver** предлагает компромисс: создаётся серверный процесс, от которого затем порождаются
все остальные процессы, что позволяет избежать многократного копирования ненужных ресурсов.

Для обмена данными между изолированными процессами модуль `multiprocessing` предоставляет несколько механизмов
межпроцессного взаимодействия (IPC).

- **Очереди (Queue)** представляют собой потокобезопасные FIFO-структуры,
  реализованные через системные каналы (pipes).
- **Каналы (Pipe)** обеспечивают двунаправленную связь между двумя
  процессами.
- **Разделяемая память (Shared Memory)** через `multiprocessing.Value` и `multiprocessing.Array` позволяет
  создавать переменные, доступные из разных процессов.
- **Менеджеры (Managers)** предлагают высокоуровневый API для
  создания разделяемых объектов (словарей, списков) через прокси-объекты, абстрагируя сложности IPC.

Синхронизация процессов осуществляется с помощью примитивов, аналогичных тем, что используются в многопоточности (Lock,
Semaphore, Event, Condition), но работающих через механизмы операционной системы или общую память. Для управления
множеством задач часто применяются **пулы процессов (Process Pool)** через `multiprocessing.Pool` или
`concurrent.futures.ProcessPoolExecutor`, которые автоматически распределяют задачи между фиксированным количеством
рабочих процессов, предоставляя удобные методы вроде `map()` и `apply_async()`.

Производительность мультипроцессинга определяется балансом между выгодами от истинного параллелизма и накладными
расходами на создание процессов и IPC. Процессы эффективны для задач, требующих значительных вычислений, но могут быть
избыточны для простых операций из-за высокой стоимости создания. Кроме того, при использовании fork необходимо
учитывать, что дочерний процесс получает копию всей памяти родителя, что может привести к неожиданному потреблению
памяти.

Мультипроцессинг в Python — это мощный инструмент для преодоления ограничений GIL и использования всех вычислительных
ресурсов системы, особенно когда задачи могут быть эффективно распараллелены и требуют минимального обмена данными между
процессами.

[Содержание](/CONTENTS.md#содержание)

## **Senior Level**

В CPython 3.9+ **мультипроцессинг** — **отдельные процессы** через `fork()`/`spawn()`/`forkserver()` в
`Lib/multiprocessing/spawn.py`, **IPC** через `Pipe`/`Queue` (`_multiprocessing.so`), **новый PyInterpreterState** в
каждом процессе, **semaphore/shm** для синхронизации. `Lib/multiprocessing/`,`Modules/_multiprocessing.c`

## 1. multiprocessing.set_start_method() - выбор метода (Lib/multiprocessing/context.py)

```python
def set_start_method(method, force=False):
    if method == 'fork':
        _fork_posix()
    elif method == 'spawn':
        _spawn_posix()
    elif method == 'forkserver':
        _forkserver_posix()
    else:
        raise ValueError(f"unknown start method {method}")

    _current_context._set_start_method(method, force)
```

Когда ты пишешь `mp.set_start_method('spawn')`, Python **выбирает способ** создания
нового процесса. `'fork'` — **быстрое копирование** текущего процесса (только Linux), `'spawn'` — **с нуля** (
Windows/macOS/Linux), `'forkserver'` — **сервер копий**. Это **критично** влияет на производительность и безопасность.

## 2. Process._bootstrap() - входная точка процесса (Lib/multiprocessing/process.py)

```python
def _bootstrap():
    # 1. Восстанавливаем аргументы из командной строки
    sys.argv = _args_from_interpreter_flags()

    # 2. Импортируем main модуль заново
    code, filename, main_path = _args_from_interpreter_flags()
    assert main_path is not None
    _run_module_as_main(main_path, code)
```

Новый процесс запускается с **аргументами**
`python -c "import main; main.worker()"`. **Главное** — `if __name__ == '__main__':` **обязательно**, иначе *
*бесконечный импорт** main модуля в дочернем процессе.

## 3. spawn._main() - spawn метод (Lib/multiprocessing/spawn.py)

```python
def _main(fd):
    # 1. Подключаемся к родительскому процессу через Unix pipe
    parent_r, parent_w = os.pipe()
    code, filename = _read_signed(fd)  # Читаем код main модуля

    # 2. Создаём новый Python интерпретатор
    interp = PyInterpreterState_New()
    tstate = PyThreadState_New(interp)
    PyThreadState_Swap(tstate)

    # 3. Выполняем код worker'а
    exec(code, {'__file__': filename})
```

**spawn** = **новый процесс** → **pipe** с родителем → **чтение** `main.py`
байткода → **PyInterpreterState_New()** → **отдельный интерпретатор** → `exec(main.worker())`. **Ничего** от родителя не
наследуется!

## 4. _multiprocessing.Connection - IPC Pipe (Lib/multiprocessing/connection.py)

```python
class Connection:
    def __init__(self, handle, readable=True):
        self._handle = handle  # file descriptor (Unix pipe/socket)
        self._readable = readable

    def send(self, obj):
        # Сериализуем объект pickle → пишем в pipe
        buf = pickle.dumps(obj)
        self._send_bytes(_header(buf) + buf)

    def recv(self):
        # Читаем из pipe → десериализуем
        buf = self._recv_bytes()
        return pickle.loads(buf)
```

`Queue.put(123)` → `pickle.dumps(123)` → **запись** в Unix pipe → другой процесс
`pipe.read()` → `pickle.loads()`. **Единственный способ** передачи данных между процессами.

## 5. _multiprocessing.SemLock - семафоры (Modules/_multiprocessing/semaphore.c)

```c
typedef struct {
    PyObject_HEAD
    volatile int state;            // 0=locked, 1=unlocked
    HANDLE sem;                    // Windows: HANDLE, Unix: sem_t*
    int wait_flag;                 // Для acquire()
    int release_flag;
} SemLockObject;

static PyObject *semlock_acquire(SemLockObject *self) {
    if (self->state == 1) {
        self->state = 0;           // Быстрое захватывание
        Py_RETURN_TRUE;
    }
    
    // Блокируемся на семафоре
    int success = WaitForSingleObject(self->sem, INFINITE);
    if (success == WAIT_OBJECT_0) {
        self->state = 0;
        Py_RETURN_TRUE;
    }
    Py_RETURN_FALSE;
}
```

`Lock.acquire()` → **system semaphore** (Unix `sem_t`, Windows `HANDLE`). **Один**
процесс захватывает, **другие ждут**. **Не Python lock** — **ОС-level** синхронизация.

## 6. multiprocessing.Pool - пул процессов (Lib/multiprocessing/pool.py)

```python
class Pool:
    def __init__(self, processes=None, initializer=None):
        self._processes = processes
        self._pool = []  # Список Process

        # Создаём worker процессы
        for i in range(self._processes):
            p = self.Process(target=worker)
            p.start()
            self._pool.append(p)

    def map(self, func, iterable):
        # Разбиваем задачу → отправляем в Queue
        # Ждём результаты из result_queue
        pass
```

`Pool(4)` → **4 постоянных** процесса-worker'ов → `map(f, lst)` → **разбивает**
`lst` на куски → **отправляет** в Queue каждому worker'у → **собирает** результаты.

## 7. fork() системный вызов (только Unix, spawn контекст)

```c
// В Lib/multiprocessing/forking.py (C вызов)
pid_t pid = fork();
if (pid == 0) {
    // Дочерний процесс
    close(parent_fd);
    _bootstrap_child();        // Входная точка
} else {
    // Родительский
    close(child_fd);
    return pid;
}
```

`fork()` — **клонирует** **весь** процесс **мгновенно** (копирует **page table**
памяти). **Оба** процесса видят **одинаковую** память до **первой записи** (Copy-on-Write). **Опасно** с GIL/threads!

## 8. PyInterpreterState_New() в дочернем процессе

```c
PyStatus PyInterpreterState_New(PyThreadState *tstate) {
    PyInterpreterState *interp;
    
    interp = PyMem_RawCalloc(1, sizeof(PyInterpreterState));
    if (interp == NULL) {
        return PyStatus_NoMemory();
    }
    
    // Инициализируем **новый** интерпретатор
    interp->tstate_head = NULL;
    interp->modules = NULL;
    interp->modules_reloading = 0;
    interp->sysdict = NULL;
    
    // Создаём PyThreadState для этого процесса
    PyThreadState *new_tstate = PyThreadState_New(interp);
    PyThreadState_Swap(new_tstate);
    
    // Инициализация интерпретатора
    if (_PyInterpreterState_Init(interp) < 0) {
        PyInterpreterState_Delete(interp);
        return PyStatus_Err();
    }
    
    return PyStatus_OK();
}
```

Каждый процесс имеет **свой** `PyInterpreterState` + `PyThreadState`. **Никаких**
общих globals/modules/builtins. **Полная изоляция**.

## 9. Shared Memory (multiprocessing.shared_memory)

```c
// Modules/_multiprocessing/shm_posix.c
int shm_open(const char *name, int oflag, mode_t mode) {
    // Создаёт /dev/shm/name сегмент памяти
    return syscall(SYS_shm_open, name, oflag, mode);
}

void *mmap(void *addr, size_t length, int prot, int flags, int fd, off_t offset) {
    // Отображает shm в адресное пространство процесса
}
```

`SharedMemory('name', size=1e6)` → **/dev/shm/name** файл → `mmap()` → **общая
физическая память**. **Байтовое** копирование между процессами.

## 10. multiprocessing.Manager() - proxy объекты

```python
class Server:
    def serve_forever(self):
        # Запускает сокет сервер
        while True:
            conn = self.listener.accept()
            t = threading.Thread(target=Dispatcher(conn))
            t.start()
```

`Manager().dict()` → **отдельный процесс-сервер** → **Unix socket** → **pickle
запросы** → **Python RPC**. **Автоматическая** сериализация.

**Мультипроцессинг** в CPython 3.9+ — **spawn/fork/forkserver**, **новый PyInterpreterState** в каждом процессе, **Pipe
** (`pickle` + Unix sockets), **sem_t/HANDLE** семафоры, **Pool** worker'ы, **Copy-on-Write** (`fork`), **SharedMemory
** (`mmap`).

- [Содержание](/CONTENTS.md#содержание)

---

# **Dataclass**

[Содержание](/CONTENTS.md#содержание)

## **Junior Level**

Dataclass — это декоратор из модуля `dataclasses`, который автоматически добавляет в класс стандартные методы, избавляя
разработчика от написания шаблонного кода. Если обычный класс, предназначенный в основном для хранения данных, требует
ручного определения методов `__init__`, `__repr__` и `__eq__`, то dataclass делает это за вас одной строкой.

Представьте, что вам нужно создать простой класс для представления точки с координатами. Без dataclass пришлось бы
писать несколько методов, которые часто выглядят одинаково для разных классов. С dataclass достаточно объявить поля с
аннотациями типов, и всё необходимое появится автоматически. Это делает код чище, уменьшает вероятность ошибок и
упрощает поддержку.

Например, класс `Point` можно определить всего в две строки, и он сразу будет иметь конструктор, читаемое строковое
представление и корректное сравнение экземпляров по значению полей. Dataclass также предлагает гибкие настройки: можно
создавать неизменяемые классы, управлять участием полей в сравнении, задавать значения по умолчанию и даже определять
поля, которые вычисляются динамически.

[Содержание](/CONTENTS.md#содержание)

## **Middle Level**

Под капотом dataclass анализирует аннотации полей класса и на их основе генерирует необходимые методы. Помимо очевидных
`__init__`, `__repr__` и `__eq__`, он может создавать и другие специальные методы в зависимости от параметров
декоратора. Например, при `order=True` генерируются методы сравнения (`__lt__`, `__le__`, `__gt__`, `__ge__`), что
позволяет упорядочивать экземпляры. При `frozen=True` экземпляры становятся неизменяемыми, и любая попытка изменить поле
вызывает исключение `FrozenInstanceError`. Это полезно для создания объектов, которые должны оставаться постоянными,
например, для использования в качестве ключей словаря.

Функция `field()` предоставляет тонкий контроль над каждым полем. Через неё можно указать, должно ли поле участвовать в
конструкторе, строковом представлении, сравнении или вычислении хеша. Особенно важны параметры `default` для значений по
умолчанию и `default_factory` для изменяемых объектов — например, чтобы каждый экземпляр получал свой собственный
список, а не ссылался на один и тот же. Параметр `metadata` позволяет прикреплять к полям произвольные метаданные,
которые не влияют на поведение dataclass, но могут быть использованы сторонними инструментами для валидации или
генерации документации.

Dataclass поддерживает наследование, собирая поля от всех родительских классов в порядке MRO (Method Resolution Order).
Однако при наследовании требуется аккуратность с значениями по умолчанию, чтобы избежать проблем с порядком аргументов в
конструкторе. Для дополнительной инициализации или валидации можно определить метод `__post_init__()`, который
вызывается автоматически после основного конструктора.

Начиная с Python 3.10, dataclass поддерживает паттерн-матчинг через параметр `match_args=True`, что делает экземпляры
класса удобными для использования в конструкциях `match/case`. Также при использовании `slots=True` dataclass может
генерировать классы с оптимизацией памяти, аналогичные классам с `__slots__`, что особенно полезно при создании большого
количества экземпляров.

В целом, dataclass — это не просто удобное сокращение кода, а полноценный инструмент для создания надёжных и эффективных
классов данных, который сочетает простоту использования с широкими возможностями настройки под конкретные задачи.

1. **Автоматически генерируемые методы**:
    - `__init__()`: Инициализатор с параметрами для всех полей
    - `__repr__()`: Читаемое строковое представление
    - `__eq__()`: Сравнение по значениям всех полей
    - `__ne__()`: Автоматически на основе `__eq__`
    - Опционально (через параметры декоратора):
        - `__lt__()`, `__le__()`, `__gt__()`, `__ge__()`: при `order=True`
        - `__hash__()`: при `frozen=True` или явном указании

2. **Параметры декоратора `@dataclass`**:
    - `init=True`: Генерация `__init__`
    - `repr=True`: Генерация `__repr__`
    - `eq=True`: Генерация `__eq__`
    - `order=False`: Генерация методов сравнения
    - `frozen=False`: Запрет изменения полей после создания
    - `unsafe_hash=False`: Принудительная генерация `__hash__`
    - `match_args=True`: Поддержка паттерн-матчинга (Python 3.10+)

3. **Поле `field()`**: Функция для тонкой настройки полей:
    - `default`: Значение по умолчанию
    - `default_factory`: Фабрика для изменяемых значений по умолчанию
    - `init`: Включать ли поле в `__init__`
    - `repr`: Включать ли поле в `__repr__`
    - `compare`: Участвует ли поле в сравнении
    - `hash`: Участвует ли поле в вычислении хеша
    - `metadata`: Произвольные метаданные

[Содержание](/CONTENTS.md#содержание)

## **Senior Level**

В CPython 3.9+ **dataclass** — **Python декоратор** `Lib/dataclasses.py` с `_process_class()`, генерирует **специальные
методы** (`__init__`, `__repr__`, `__eq__`) через `_create_fn()` + `exec()`, использует **metaclass** `DataclassType`. *
*PEP 557**. `Lib/dataclasses.py`

## 1. @dataclass декоратор (Lib/dataclasses.py)

```python
def dataclass(cls=None, *, init=True, repr=True, eq=True, order=False,
              unsafe_hash=False, frozen=False, match_args=True,
              kw_only=False, slots=False, weakref_slot=False):
    def wrap(cls):
        # 1. Собираем поля класса
        fields = fields(cls)

        # 2. Генерируем специальные методы
        ns = dict(cls.__dict__)

        # __init__
        if init:
            __init__ = _init_fn(fields, locals())
            ns['__init__'] = __init__

        # __repr__
        if repr:
            __repr__ = _repr_fn(fields, globals())
            ns['__repr__'] = __repr__

        # __eq__
        if eq:
            __eq__ = _eq_fn(fields, globals())
            ns['__eq__'] = __eq__

        # __hash__ / __setattr__
        if frozen:
            ns['__setattr__'] = _frozen_setattr()
            ns['__delattr__'] = _frozen_delattr()

        # 3. Создаём новый класс
        return type(cls)(cls.__name__, cls.__bases__, ns)

    return wrap if cls is None else wrap(cls)
```

**Объяснение для тупого человека (очень подробно):** Представь, что у тебя класс `Person` с полями `name: str` и
`age: int`. Когда ты пишешь `@dataclass`, Python **не меняет** твой исходный класс. Вместо этого он **создаёт новый
класс** с **такими же** полями, но **добавляет** готовые методы `__init__`, `__repr__`, `__eq__`. Это как **фабрика**,
которая берёт твой чертеж дома и **достраивает** коммуникации, двери, окна. Ты написал только стены — Python добавил
остальное. `_process_class()` — это **главная фабрика**, которая анализирует аннотации `: str`, `: int` и понимает, что
это **поля dataclass'а**.

## 2. fields() - сбор полей класса

```python
def fields(cls):
    flds = []
    bases = list(cls.__bases__)

    # Ищем __dataclass_fields__
    ann = getattr(cls, '__annotations__', {})
    for name, type_ in ann.items():
        if isinstance(type_, Field):
            flds.append(type_)
        else:
            # Обычная аннотация -> Field(name, type_)
            flds.append(Field(name, type_, default=MISSING))

    # Наследование
    for b in bases[::-1]:  # MRO
        flds = fields(b) + flds

    return tuple(flds)
```

Python **сканирует** `__annotations__` = `{'name': str, 'age': int}`. Каждое имя *
*становится Field**. Если написал `name: str = "Bob"` — это **default значение**. Если класс **наследуется** от другого
dataclass — **берёт** поля родителя **первым**. Представь, что собираешь конструктор LEGO: сначала **база** от
родителей, потом **твои** кубики сверху.

## 3. _init_fn() - генерация __init__

```python
def _init_fn(fields, locals):
    # Сигнатура: def __init__(self, name: str, age: int = 0):
    sig_lines = ['def __init__(self, /, ']

    # Параметры из полей
    for idx, field in enumerate(fields):
        if field.init_arg is False:
            continue
        arg = field.name
        if field.default is not MISSING:
            arg += '=' + repr(field.default)
        sig_lines.append(arg)

    sig = ', '.join(sig_lines) + '):'

    # Тело: self.name = name
    body_lines = []
    for field in fields:
        if field.init_arg:
            line = f'self.{field.name} = {field.name}'
            body_lines.append(line)

    # exec() создаёт функцию
    fn = _create_fn('__init__', ('self',),
                    [sig] + body_lines,
                    locals=locals)
    return fn
```

Для `@dataclass class Person: name: str; age: int = 0` создаётся **строка кода**:

```
def __init__(self, /, name: str, age: int = 0):
    self.name = name
    self.age = 0
```

Затем `exec()` **компилирует** эту строку в **настоящую** Python функцию и **вставляет** в класс. Это **магия**: твой
класс получает **готовый** `__init__`, который принимает **именно те параметры**, что ты объявил.

## 4. _create_fn() - компиляция кода в функцию

```python
def _create_fn(name, args, body_lines, globals=None, locals=None,
               return_type=None):
    # Формируем полный код функции
    code = ['def ' + name + '(' + ', '.join(args) + '):']
    code.extend('    ' + line for line in body_lines)

    # Компилируем в PyCodeObject
    src = '\n'.join(code)
    co = compile(src, '<dataclasses>', 'exec')

    # Создаём функцию
    fn = types.FunctionType(co, globals, name, (), closure=None)
    return fn
```

`_create_fn()` — **мини-компилятор**. Берёт строки
`def __init__(self, name): self.name=name`, **склеивает** в одну большую строку, вызывает `compile()` → **PyCodeObject
**, затем `types.FunctionType()` → **готовую функцию**. Это **точно так же**, как если бы ты написал эту функцию *
*руками** — никакого отличия в скорости!

## 5. __repr__ генерация

```python
def _repr_fn(fields, globals):
    # 'Person(name=\'Bob\', age=30)'
    body_lines = []
    body_lines.append('r = self.__class__.__name__ + \'(\'')

    for field in fields:
        if field.repr:
            line = f"r += '{field.name}=' + repr({field.name}) + ', '"
            body_lines.append(line)

    # Убираем последнюю запятую
    body_lines.append('return r.rstrip(\", \") + \')\'')

    return _create_fn('__repr__', ('self',), body_lines, globals)
```

`print(Person)` → `'Person(name=\'Bob\', age=30)'`. Python **проходит** по всем
полям, для **каждого** делает `name='Bob'`, **склеивает** через запятую. **Точно** как `str(dict)`, но **для твоего
класса**. Если поле `repr=False` — **пропускает** его.

## 6. DataclassType метакласс

```python
class DataclassType(type):
    def __new__(cls, name, bases, ns):
        # Автоматическая обработка @dataclass
        if hasattr(ns, '__dataclass_transform__'):
            return super().__new__(cls, name, bases, ns)

        # Обрабатываем поля
        fields = []
        for base in bases[::-1]:
            fields.extend(getattr(base, '__dataclass_fields__', {}))

        # Добавляем __dataclass_fields__
        ns['__dataclass_fields__'] = dict(fields)

        return super().__new__(cls, name, bases, ns)
```

**Метакласс** — это **фабрика классов**. Когда Python видит `class Person:`, он *
*спрашивает** метакласс: "как создать этот класс?". `DataclassType` **добавляет** `__dataclass_fields__` =
`{'name': Field(...), 'age': Field(...)}`. Это **словарь полей** для наследования.

## 7. Field() - дескриптор полей

```python
class Field:
    __slots__ = ('name', 'type', 'default', 'default_factory',
                 'init', 'repr', 'eq', 'order', 'hash',
                 'compare', 'metadata', 'init_arg')

    def __init__(self, default, *, default_factory, init, repr, eq,
                 order, unsafe_hash, frozen, compare, metadata):
        self.name = None
        self.default = default
        self.default_factory = default_factory
        self.init = init
        self.repr = repr
        self.eq = eq
        self.order = order
        self.hash = unsafe_hash
        self.compare = compare
        self.metadata = metadata
```

`name: str = field(default_factory=list)` → **Field объект** с флагами `repr=True`,
`init=True`, `default_factory=<class 'list'>`. Когда `__init__` видит `default_factory` — вызывает `list()` вместо
копирования значения.

## 8. __post_init__() поддержка

```python
# Автоматический вызов после __init__
def _init_fn(fields, locals):
    # ... обычный __init__ ...

    # Добавляем __post_init__ если есть
    if '__post_init__' in locals:
        body_lines.append('self.__post_init__()')
```

Dataclass **всегда** вызывает `self.__post_init__()` **после** заполнения полей.
Полезно для **валидации**: `def __post_init__(self): assert self.age >= 0`.

## 9. frozen=True - неизменяемый dataclass

```python
def _frozen_setattr():
    return _create_fn('__setattr__', ('self', 'name', 'value'),
                      ['if _is_dataclass_field(self, name):',
                       '    raise FrozenInstanceError(f"cannot assign to {name}")',
                       'object.__setattr__(self, name, value)'])
```

`@dataclass(frozen=True)` → `__setattr__` **блокирует** изменение полей.
`p.name = "new"` → **FrozenInstanceError**. **Только** `object.__setattr__` для служебных полей.

## 10. Наследование dataclass

```python
@dataclass
class Person:
    name: str


@dataclass
class Employee(Person):
    salary: int
```

**Результат:** `__init__(self, name: str, salary: int)`. **Поля родителей** идут **первыми**.

**Dataclass** в CPython 3.9+ — **Lib/dataclasses.py** декоратор, **_process_class()** анализ `__annotations__`, **
_create_fn() + exec()** генерация `__init__`/`__repr__`, **Field дескрипторы**, **DataclassType метакласс**, `frozen`/
`__post_init__` поддержка.

- [Содержание](/CONTENTS.md#содержание)

---

# **Enum**

[Содержание](/CONTENTS.md#содержание)

## **Junior Level**

Enum (перечисление) в Python — это специальный тип данных, позволяющий создавать набор именованных констант, что делает
код более читаемым и защищённым от опечаток. Члены Enum являются иммутабельными синглтонами и поддерживают итерацию,
сравнение и доступ по имени или значению. Python предоставляет несколько вариантов: базовый `Enum`, `IntEnum` (
наследующий `int`), `Flag` для битовых операций и `StrEnum` (с Python 3.11) для строковых констант.

Например, вместо использования чисел 0, 1, 2 для статусов задачи можно создать Enum:

```python
from enum import Enum


class TaskStatus(Enum):
    PENDING = 0
    RUNNING = 1
    COMPLETED = 2
    FAILED = 3
```

Теперь в коде можно использовать `TaskStatus.RUNNING` вместо просто `1`. Enum предоставляет итерацию, сравнение и доступ
к членам по имени или значению. Особенно полезны они для ограничения допустимых значений параметров функции и замены
строковых констант.

[Содержание](/CONTENTS.md#содержание)

## **Middle Level**

1. **Типы Enum**:
    - `Enum`: Базовый класс для создания перечислений
    - `IntEnum`: Члены являются подклассами `int`, могут использоваться везде, где ожидается целое число
    - `Flag`, `IntFlag`: Для битовых флагов, поддерживают побитовые операции
    - `StrEnum` (Python 3.11+): Члены являются подклассами `str`

2. **Свойства членов Enum**:
    - `name`: Имя члена (строка)
    - `value`: Значение члена (может быть любого типа)
    - `__members__`: Словарь всех членов {имя: член}

3. **Автоматические значения**:
    - `auto()`: Автоматически присваивает уникальные значения (начиная с 1)
    - `@enum.unique`: Декоратор, гарантирующий уникальность значений

4. **Методы и операции**:
    - Итерация: `for status in TaskStatus:`
    - Доступ по значению: `TaskStatus(1)`
    - Доступ по имени: `TaskStatus['RUNNING']`
    - Проверка принадлежности: `isinstance(value, TaskStatus)`

5. **Расширенные возможности**:
    - Методы класса: можно добавлять методы в класс Enum
    - Свойства (property): вычисляемые атрибуты
    - Наследование: Enum может наследоваться от других классов (кроме другого Enum)

6. **Иммутабельность**: Члены Enum — синглтоны. Нельзя изменить их значение после создания.

[Содержание](/CONTENTS.md#содержание)

## **Senior Level**

В CPython 3.9+ **Enum** — **EnumType метакласс** (`Lib/enum.py`), **EnumDict** (отслеживает порядок `_member_names_`), *
*`_member_map_`** (name→member), **`__new__`/`__init__`** для создания **EnumMember** объектов, *
*`_generate_next_value_`** + `auto()`. **Singleton-подобные** immutable объекты. `Lib/enum.py`

## 1. EnumType.__new__() - метакласс создания (Lib/enum.py)

```python
class EnumType(type):
    def __new__(metacls, cls, bases, classdict):
        # 1. EnumDict уже обработал поля
        enum_dict = classdict

        # 2. Определяем тип значений (_member_type_)
        member_type, first_enum = metacls._get_mixins_(cls, bases)

        # 3. Создаём enum класс
        enum_class = super().__new__(metacls, cls, bases, classdict)

        # 4. Инициализируем структуры
        enum_class._member_names_ = []  # ['RED', 'GREEN']
        enum_class._member_map_ = {}  # {'RED': <RED>, 'GREEN': <GREEN>}
        enum_class._member_type_ = member_type
        enum_class._value_ = None  # Для StrEnum/IntEnum

        # 5. Создаём члены enum
        for member_name in enum_dict.member_names:
            value = enum_dict[member_name]
            member = metacls._create_(cls, member_name, value)
            enum_class._member_names_.append(member_name)
            enum_class._member_map_[member_name] = member

        return enum_class
```

**Объяснение для тупого человека (очень подробно):** Представь, что пишешь `class Color(Enum): RED = 1; GREEN = 2`.
Python **НЕ** создаёт обычный класс. Вместо этого **метакласс** `EnumType` берёт твой словарь `{'RED': 1, 'GREEN': 2}` и
**превращает** его в **специальные структуры**: `_member_names_ = ['RED', 'GREEN']` (порядок объявления) и
`_member_map_ = {'RED': <Color.RED>, 'GREEN': <Color.GREEN>}` (словарь для быстрого поиска). Каждый член — **отдельный
объект** `Color.RED`, а **НЕ** число `1`. Это как **фабрика**, которая берёт твои константы и делает из них **умные
объекты** с методами.

## 2. EnumDict - специальный словарь (Lib/enum.py)

```python
class EnumDict(dict):
    def __init__(self):
        super().__init__()
        self._member_names = {}  # Имена членов в порядке объявления
        self._last_values = []  # Для auto()

    def __setitem__(self, key, value):
        """Запрещаем дубликаты имён"""
        if key in self._member_names:
            raise TypeError(f'Attempted to reuse member {key!r}')
        if _is_dunder(key):
            super().__setitem__(key, value)
        else:
            self._member_names[key] = None  # Запоминаем порядок
            super().__setitem__(key, value)

    @property
    def member_names(self):
        return list(self._member_names)
```

Обычный `dict` **позволит** `class Color: RED=1; RED=2` (перезапишет). `EnumDict` *
*запрещает** дубликаты: `RED` второй раз → **TypeError**. `_member_names` хранит **порядок** объявления (
`['RED', 'GREEN']`), **НЕ** порядок хеш-таблицы. Это **очень важно** для `list(Color)` → `Color.RED, Color.GREEN`.

## 3. _create_() - создание EnumMember (EnumType)

```python
def _create_(cls, member_name, value):
    # 1. Создаём EnumMember объект
    enum_member = object.__new__(cls)

    # 2. Устанавливаем свойства
    enum_member._name_ = member_name
    enum_member._value_ = value
    enum_member.__objclass__ = cls
    enum_member.__init__(value)

    # 3. Алиасы (дубликаты значений)
    for name, canonical_member in cls._member_map_.items():
        if canonical_member._value_ == value and canonical_member is not enum_member:
            enum_member._missing_(name)  # Алиас

    return enum_member
```

`Color.RED = 1` → **новый объект** `Color.RED` с `_name_='RED'`, `_value_=1`. Если
позже `ALIEN = 1` → `Color.ALIEN` **указывает** на тот же **самый** объект `Color.RED` (алиас).
`Color.RED is Color.ALIEN` = `True`. Это **экономит память** и гарантирует **единственность**.

## 4. Enum.__new__() / __init__() членов

```python
class Enum:
    def __new__(cls, value):
        # Ищем существующий член с таким value
        for member in cls._member_map_.values():
            if member._value_ == value:
                return member

        # Новый член (только через класс!)
        member = object.__new__(cls)
        member._value_ = value
        return member

    def __init__(self, value):
        self._value_ = value

    def __repr__(self):
        return f"<{self.__class__.__name__}.{self._name_}>"
```

`Color(1)` → ищет в `_member_map_` член с `value=1` → **возвращает** `Color.RED` (
существующий объект). `Color.RED` → `'Color.RED'`. **НЕ** создаёт новые экземпляры — возвращает **сигнатоны** (единичные
объекты).

## 5. auto() + _generate_next_value_ (Lib/enum.py)

```python
def auto():
    """Генерирует значение автоматически"""
    value = _next_value_
    _next_value_ += 1
    return value


class Color(Enum):
    def _generate_next_value_(name, start, count, last_values):
        # Автоинкремент
        return count

    RED = auto()  # 1
    GREEN = auto()  # 2
    BLUE = auto()  # 3
```

`RED = auto()` → вызывает `_generate_next_value_('RED', 1, 0, [])` → возвращает `0`.
`GREEN = auto()` → `_generate_next_value_('GREEN', 1, 1, [0])` → `1`. **Глобальный счётчик** `count` (номер члена).
Можно **переопределить**: `return 3**count` → `RED=1, GREEN=3, BLUE=9`.

## 6. Enum.__iter__() / __len__()

```python
def __iter__(cls):
    return (cls._member_map_[name] for name in cls._member_names_)


def __len__(cls):
    return len(cls._member_names_)


def __getitem__(cls, name):
    return cls._member_map_[name]
```

`list(Color)` → `[Color.RED, Color.GREEN]` **в порядке объявления**. `len(Color)` →
`3`. `Color['RED']` → `Color.RED`. **`_member_names_`** гарантирует **порядок** и **быстрый поиск**.

## 7. IntEnum / StrEnum наследование

```python
class IntEnum(int, Enum):
    pass


class StrEnum(str, Enum):
    pass


class Color(IntEnum):
    RED = 1
    GREEN = 2
```

`Color.RED + 1` → `2` (потому что `IntEnum` наследует `int`). `Color.GREEN == '2'` →
`False` (потому что `_value_=2`, а **НЕ** строка). `Color.RED.value` → `1` (сырое значение).

## 8. _simple_enum() - оптимизация (3.11+)

```python
def _simple_enum(etype, seq, value=1):
    """Быстрое создание простого enum"""
    members = dict()
    for i, name in enumerate(seq):
        val = etype._generate_next_value_(name, 1, i, [])
        member = object.__new__(etype)
        member._value_ = val
        member._name_ = name
        member.__objclass__ = etype
        members[name] = member

    # Создаём класс
    cls = type(seq.__class__.__name__, (etype,), members)
    cls._member_names_ = list(members)
    cls._member_map_ = members
    return cls
```

`Enum('RED GREEN BLUE')` → **оптимизированный** путь без метакласса. **Быстрее** для
простых случаев.

## 9. Flag / IntFlag - битовые флаги

```python
class Perm(Flag):
    R = 4
    W = 2
    X = 1

    RWX = R | W | X
```

`Perm.RWX` → **новый** объект с `_value_=7`. `Perm.RWX & Perm.R` → `Perm.R`.
`Perm.RWX | Perm.W` → `Perm.RWX`. **Битовые операции** работают с `_value_`.

## 10. pickle поддержка

```python
def __reduce_ex__(self, proto):
    return (self.__class__, (self._value_,))
```

`pickle.dumps(Color.RED)` → `(Color, (1,))` → при восстановлении `Color(1)` → **тот
же** `Color.RED` объект. **НЕ** новый экземпляр!

**Enum** в CPython 3.9+ — **EnumType метакласс**, **EnumDict** (порядок + уникальность), *
*`_member_map_/ _member_names_`**, **сигнатоны** через `__new__`, **`auto()`** + `_generate_next_value_`, *
*IntEnum/StrEnum** наследование, **Flag** битовые операции.

- [Содержание](/CONTENTS.md#содержание)

---

# **Garbage Collector (Сборщик мусора)**

[Содержание](/CONTENTS.md#содержание)

## **Junior Level**

Сборщик мусора в Python — это автоматический механизм управления памятью, который освобождает память от объектов,
которые больше не используются программой. Он использует комбинированный подход: **подсчет ссылок (reference counting)**
для немедленного освобождения памяти и **циклический сборщик мусора (cycle collector)** для обнаружения и удаления
циклических ссылок, которые не может обработать первый механизм.

Представьте, что каждый объект имеет счётчик (`ob_refcnt`), который увеличивается при создании новой ссылки на объект и
уменьшается при её удалении. Когда счётчик достигает нуля, память объекта немедленно освобождается. Однако, если два или
более объекта ссылаются друг на друга (образуя цикл), их счётчики никогда не станут нулевыми, даже если они уже
недоступны из программы. Для таких случаев существует циклический сборщик.

Для оптимизации процесса объекты делятся на три поколения (0, 1, 2), основываясь на наблюдении, что большинство объектов
живут недолго. Сборка мусора чаще всего происходит в самом молодом поколении (0). Модуль `gc` позволяет управлять этим
процессом, получать статистику и настраивать поведение сборщика.

[Содержание](/CONTENTS.md#содержание)

## **Middle Level**

1. **Два механизма управления памятью**:
    * **Подсчет ссылок**: Быстрый и детерминированный механизм. Каждая операция присваивания или удаления ссылки
      изменяет счётчик. При достижении нуля сразу вызывается деструктор (`__del__`, если он есть) и освобождается
      память.
    * **Циклический сборщик**: Специальный алгоритм (использующий трёхцветную маркировку) для обнаружения и удаления
      недостижимых циклических ссылок между объектами. Работает периодически.

2. **Поколения объектов (Generational GC)**:
    * Все объекты делятся на три поколения. Новые объекты попадают в поколение 0.
    * Частота сборки зависит от поколения: поколение 0 сканируется чаще всего, поколение 1 — реже, поколение 2 — ещё
      реже. Это повышает эффективность, так как сосредотачивает усилия на «молодых» объектах, где скапливается больше
      всего мусора.

3. **Пороги сборки**:
    * Параметры `gc.get_threshold()` возвращают кортеж `(threshold0, threshold1, threshold2)`.
    * Сборка в поколении 0 запускается, когда количество созданных объектов Python с момента последней сборки в этом
      поколении минус количество удалённых превышает `threshold0`.
    * После `threshold0` сборок в поколении 0 происходит одна сборка в поколении 1. После `threshold1` сборок в
      поколении 1 — одна сборка в поколении 2.

4. **Модуль `gc`**:
    * `gc.enable()` / `gc.disable()`: Включение/выключение циклического сборщика.
    * `gc.collect(generation=None)`: Принудительный запуск сборки для указанного поколения (по умолчанию для всех).
    * `gc.get_referents(obj)`: Возвращает объекты, на которые ссылается `obj`.
    * `gc.get_referrers(obj)`: Возвращает объекты, которые ссылаются на `obj`.
    * `gc.set_debug(flags)`: Установка флагов отладки для логирования процесса сборки.

5. **Проблемные случаи**:
    * **Циклические ссылки с `__del__`**: Если объекты в цикле имеют определенный метод `__del__`, сборщик может не
      определить порядок их удаления и оставить такие объекты в памяти (в списке `gc.garbage`), чтобы избежать
      непредсказуемого поведения.
    * **Слабые ссылки (`weakref`)**: Позволяют ссылаться на объект, не увеличивая его счётчик ссылок. Они полезны для
      создания кэшей или наблюдателей, которые не должны мешать удалению основного объекта.

6. **Производительность**:
    * Подсчёт ссылок — это операция с низкими накладными расходами, выполняемая при каждой манипуляции со ссылками.
    * Запуск циклического сборщика (особенно для старших поколений) может вызывать заметные паузы (stop-the-world), так
      как требует обхода всех проверяемых объектов.

[Содержание](/CONTENTS.md#содержание)

## **Senior Level**

В CPython 3.9+ **GC** — **двухфазный**: **refcount** (`Py_INCREF`/`Py_DECREF`) + **generational cycle detection** (
`Modules/gcmodule.c`). **PyGC_Head** (16 байт) в начале каждого GC-объекта, **3 поколения** (threshold 700/10/10), **DFS
** для циклов. `Modules/gcmodule.c`,`Include/objimpl.h`

## 1. PyGC_Head структура (Include/objimpl.h)

```c
typedef union _gc_head {
    struct {
        union _gc_head *gc_next;   // Следующий в списке поколения
        union _gc_head *gc_prev;   // Предыдущий в списке поколения
        PyGC_Head *gc_gc_head;     // Указатель на себя (для GC)
        Py_ssize_t gc_refs;        // Количество ссылок (отдельно от ob_refcnt)
    } gc;
    double dummy;                          // Выравнивание 16 байт
} PyGC_Head;
```

**Объяснение для тупого человека (очень подробно):** Представь, что каждый объект в Python — это **коробка** с
игрушками. Обычный `refcnt` (`ob_refcnt`) считает **сколько коробок ссылается** на эту игрушку. Но если две игрушки *
*указывают друг на друга** (`a.x = b; b.y = a`), refcnt **НЕ упадёт до 0** — **цикл**! GC добавляет **специальный
заголовок** `PyGC_Head` **перед** каждым объектом (list, dict, set, custom с `tp_traverse`). Это **16 байт** со *
*списком** (`gc_next/gc_prev`) для быстрого прохода по всем объектам поколения. `gc_refs` считает **только GC-ссылки** (
отдельно от обычных).

## 2. PyObject + PyGC_Head layout (реальная структура)

```c
// PyListObject в памяти:
// [PyGC_Head 16 байт] + [PyObject_HEAD 16 байт] + [PyListObject поля]
typedef struct {
    PyGC_Head gc;                      // ПЕРВЫЕ 16 байт!
    PyObject_HEAD                       // ob_refcnt + ob_type
    Py_ssize_t ob_size;                // Длина списка
    PyObject **ob_item;                // Массив указателей
} PyListObject;
```

**ВСЯ** память объекта:
`[gc.gc_next(8)][gc.gc_prev(8)][gc.gc_refs(8)][PyObject ob_refcnt(8)][ob_type(8)][ob_size(8)][ob_item(8)]`.
`Py_REFCNT(obj)` → `obj->gc.gc_refs` для GC-объектов. **Overhead** 16 байт на **каждый** list/dict!

## 3. PyObject_GC_New / PyObject_GC_Track (gcmodule.c)

```c
PyObject *_PyObject_GC_New(PyTypeObject *tp) {
    PyObject *op = _PyObject_New(tp);  // Обычный malloc
    if (op != NULL) {
        _PyObject_GC_Link(op);         // Добавляем в GC список
    }
    return op;
}

void _PyObject_GC_Link(PyObject *op) {
    GCState *gcstate = get_gc_state();
    PyGC_Head *gchead = GC_HEAD(op);   // &op->gc
    
    // Добавляем в конец поколения 0 (молодые объекты)
    PyGC_Head *gen0 = &gcstate->generations[0].head;
    gchead->gc.gc_next = gen0->gc_next;
    gchead->gc.gc_prev = gen0;
    gen0->gc_next->gc.gc_prev = gchead;
    gen0->gc_next = gchead;
    
    // Увеличиваем счётчики поколений
    gcstate->generations[0].count++;   // +1 в gen0
    for (int i = 1; i <= NUM_GENERATIONS; i++) {
        gcstate->generations[i].count++;  // +1 во всех старших
    }
    
    gchead->gc.gc_refs = Py_REFCNT(op);  // Копируем refcnt
}
```

`lst = []` → `_PyObject_GC_New(&PyList_Type)` → **malloc** → `_PyObject_GC_Link()` →
**вставляем** `lst.gc` в **двусвязный список** поколения 0. **Все поколения** получают `count++`. **Поколение 0** — *
*самое частое** (каждые 700 объектов).

## 4. Py_DECREF + GC check (Objects/object.c)

```c
Py_ssize_t _Py_DecRef(PyObject *op) {
    Py_ssize_t refcnt = Py_REFCNT(op) - 1;
    
    if (refcnt == 0) {
        // Refcnt=0 → tp_dealloc
        op->ob_type->tp_dealloc(op);
        return 0;
    }
    
    Py_REFCNT(op) = refcnt;
    
    // GC объект? Проверяем gc_refs
    if (PyObject_IS_GC(op) && refcnt < _PyGC_Threshold && 
        Py_REFCNT(op) == 0) {
        _PyObject_GC_Unlink(op);       // Удаляем из списка
        _Py_Dealloc(op);               // Освобождаем
    }
    
    return refcnt;
}
```

`del lst` → `Py_DECREF(lst)` → `ob_refcnt--`. Если `refcnt=0` → **освобождаем**. Для
GC-объектов: если `gc.gc_refs < threshold` (700) **и** `refcnt=0` → `_PyObject_GC_Unlink()` (удаляем из списка) → *
*malloc free**.

## 5. _PyGC_CollectNoFail() - триггер GC (gcmodule.c)

```c
Py_ssize_t _PyGC_CollectNoFail(void) {
    GCState *gcstate = get_gc_state();
    
    // Проверяем все поколения
    for (int gen = NUM_GENERATIONS - 1; gen >= 0; gen--) {
        Py_ssize_t count = gcstate->generations[gen].count;
        Py_ssize_t threshold = gcstate->threshold[gen];
        
        if (count > threshold) {
            // ТРИГГЕР! Собираем это поколение
            return gc_collect(gcstate, gen);
        }
    }
    
    return 0;
}
```

**Каждое** выделение памяти → `_PyGC_CollectNoFail()`. Проверяет **сначала старшее
поколение** (gen2), потом gen1, gen0. Если `gen0.count > 700` → **собираем gen0**. `gen1.count > 10` → собираем *
*gen0+gen1**. **Автоматически**!

## 6. gc_collect() - основной цикл сборки (gcmodule.c)

```c
static Py_ssize_t gc_collect(GCState *gcstate, int gen) {
    Py_ssize_t n = 0;
    
    // Собираем поколения младше gen
    for (int i = 0; i <= gen; i++) {
        n += collect_generation(gcstate, i);
    }
    
    return n;
}

static Py_ssize_t collect_generation(GCState *gcstate, int gen) {
    PyGC_Head *head = &gcstate->generations[gen].head;
    PyGC_Head *next, *cur;
    
    // Проходим по всему поколению
    for (cur = head->gc.gc_next; cur != head; cur = next) {
        next = cur->gc.gc_next;
        
        // Проверяем refcnt
        if (cur->gc.gc_refs == 0) {
            // Мусор! Удаляем
            _PyObject_GC_Unlink((PyObject*)cur);
            PyObject_GC_Del((PyObject*)cur);
            gcstate->collected++;
        }
    }
    
    return gcstate->collected;
}
```

GC **проходит** по **двусвязному списку** поколения:
`head → obj1 → obj2 → ... → head`. Для **каждого** проверяет `gc.gc_refs`. `0` → **мусор** → `_PyObject_GC_Unlink()` +
`free()`. **Живые** остаются.

## 7. Цикловая детекция: _PyGC_traverse() (gcmodule.c)

```c
static int _PyGC_traverse(PyObject *op) {
    Py_ssize_t refs;
    PyTypeObject *tp = Py_TYPE(op);
    
    // tp_traverse для контейнеров
    if (tp->tp_traverse != NULL) {
        refs = tp->tp_traverse(op, (visitproc)_PyGC_traverse);
        if (refs < 0) {
            return refs;
        }
    }
    
    // Считаем живые ссылки
    gc_refs = gc_refs + refs;
    
    return gc_refs;
}
```

**Фаза 2** (циклы): для **каждого** живого объекта вызываем
`tp_traverse(list_traverse)` → **рекурсивно** считаем ссылки. `list_traverse(lst)` проходит `lst->ob_item[]`, вызывает
`_PyGC_traverse(item)`. **Нулируем** `gc.gc_refs` для unreachable.

## 8. PyList_Type.tp_traverse для списков

```c
int list_traverse(PyListObject *op, visitproc visit, void *arg) {
    Py_ssize_t i, len = Py_SIZE(op);
    PyObject **p;
    
    // Проходим все элементы списка
    for (i = 0, p = op->ob_item; i < len; i++, p++) {
        Py_VISIT(*p);              // Вызываем visit(*p)
    }
    
    return 0;
}
```

`list_traverse([a,b,c])` → `Py_VISIT(a)` → `Py_VISIT(b)` → `Py_VISIT(c)`.
`Py_VISIT(obj)` → если `obj` живой → `obj->gc.gc_refs++`. **Цепочка** ссылок!

## 9. Поколения и продвижение (gcmodule.c)

```c
static void move_objects_to_new_gen(GCState *gcstate, int gen0, int gen1) {
    PyGC_Head *gen0_head = &gcstate->generations[gen0].head;
    PyGC_Head *gen1_head = &gcstate->generations[gen1].head;
    
    // Перемещаем выжившие из gen0 → gen1
    for (PyGC_Head *cur = gen0_head->gc.gc_next; cur != gen0_head; ) {
        PyGC_Head *next = cur->gc.gc_next;
        if (cur->gc.gc_refs > 0) {     // Живой!
            _PyObject_GC_Unlink((PyObject*)cur);
            _PyObject_GC_LinkTo((PyObject*)cur, gen1);  // В старшее поколение
        }
        cur = next;
    }
}
```

**Выжившие** из gen0 (700 объектов) **переезжают** в gen1 (собирается реже). Gen1
выжившие → gen2 (самое старшее). **Старые** проверяются **реже** — **оптимизация**!

## 10. gc.collect() C API (gcmodule.c)

```c
static PyObject *gc_collect(PyObject *self, PyObject *args) {
    int n;
    GCState *gcstate = get_gc_state();
    
    if (!PyArg_ParseTuple(args, "|i:collect", &n)) {
        return NULL;
    }
    
    if (n == 0) {
        n = NUM_GENERATIONS - 1;       // Полная сборка gen2
    }
    
    Py_ssize_t collected = _PyGC_CollectNoFail();
    
    return PyLong_FromSsize_t(collected);
}
```

`gc.collect()` → `_PyGC_CollectNoFail()` → собирает **самое старшее** поколение (
gen2 + все младшие). Возвращает **количество** освобождённых объектов.

**GC** в CPython 3.9+ — **PyGC_Head** (16 байт), **3 поколения** (700/10/10), **refcount** + **tp_traverse DFS**, *
*двусвязные списки** поколений, **продвижение** выживших, **триггер** при `count > threshold`.

- [Содержание](/CONTENTS.md#содержание)

---

# **Сложность кода (Asymptotic, Cyclomatic, Coupling, Maintainability)**

[Содержание](/CONTENTS.md#содержание)

## **Junior Level**

Сложность кода — это набор характеристик, определяющих, насколько программа понятна, эффективна и удобна для работы.
Выделяют четыре ключевых типа.

1. **Асимптотическая сложность (Asymptotic Complexity)** — показывает, как время выполнения или использование памяти
   программой зависит от объема входных данных (например, количества элементов в списке). Выражается в "О-нотации": O(
    1) — время постоянно, O(n) — растет линейно, O(n²) — растет квадратично (намного быстрее).

2. **Цикломатическая сложность (Cyclomatic Complexity)** — это мера количества независимых путей выполнения в коде (
   например, в функции). Чем больше операторов ветвления (`if/else`, `switch`) и циклов (`for`, `while`), тем она выше.
   Высокая цикломатическая сложность делает код трудным для понимания и тестирования.

3. **Сложность связей (Coupling Complexity)** — определяет, насколько сильно разные модули, классы или функции зависят
   друг от друга. Сильно связанный код сложно изменять, тестировать изолированно и повторно использовать.

4. **Сложность поддержки (Maintainability Complexity)** — общая оценка того, насколько легко анализировать, изменять и
   расширять код. Зависит от его читаемости, документированности, структуры и соответствия стандартам.

**Практическое значение для разработчика**:

- **Высокая асимптотическая сложность** → проверка производительности на больших данных.
- **Высокая цикломатическая сложность** → необходимость в большем количестве тестов для покрытия всех путей выполнения.
- **Высокая связанность** → сложности с изоляцией модулей для модульного тестирования.
- **Низкая поддерживаемость** → больше времени тратится на понимание кода перед внесением изменений или написанием
  тестов.

[Содержание](/CONTENTS.md#содержание)

## **Middle Level**

### **1. Асимптотическая сложность (Asymptotic Complexity)**

Это математическая оценка роста ресурсоемкости алгоритма.

- **Верхняя граница (O-нотация)**: `O(f(n))` означает, что время выполнения не превысит `c * f(n)` для всех достаточно
  больших `n`. Это наиболее часто используемая оценка.
- **Нижняя граница (Ω-нотация)**: `Ω(f(n))` означает, что время выполнения не меньше `c * f(n)`.
- **Точная граница (Θ-нотация)**: `Θ(f(n))` означает, что алгоритм является одновременно и `O(f(n))`, и `Ω(f(n))`.
- **Амортизированный анализ**: оценивает среднюю производительность операции в худшем случае за всю последовательность
  вызовов, что полезно для анализа структур данных.
- **Космическая сложность**: оценивает рост потребления памяти аналогично временной сложности.

**Практика для разработки и тестирования**:

- **Тестирование на разных объемах данных**: малые (пограничные случаи), средние, большие (нагрузочное тестирование).
- **Профилирование**: использование профилировщиков (cProfile, memory_profiler) для поиска узких мест.
- **Анализ в CI/CD**: интеграция проверки сложности в конвейер сборки с помощью инструментов статического анализа.

### **2. Цикломатическая сложность (Cyclomatic Complexity)**

Количественная метрика, основанная на графе потока управления программы.

- **Расчет**: `M = E - N + 2P`, где `E` — число рёбер в графе, `N` — число узлов, `P` — число компонентов связности (
  обычно 1). На практике рассчитывается автоматически.
- **Практические рекомендации** по значениям для функции/метода:
    - **1-10**: простая, низкий риск.
    - **11-20**: умеренная сложность.
    - **21-50**: высокая сложность, требует рассмотрения рефакторинга.
    - **51+**: очень высокая, тестирование затруднено, высок риск ошибок.
- **Инструменты**: radon, mccabe, pylint, а также комплексные платформы (SonarQube, Codacy), которые включают этот
  анализ.

**Связь с тестированием**:

- Минимальное количество тестов для покрытия всех путей должно быть не меньше цикломатической сложности.
- Помогает выявить функции с излишней логикой ветвления, которые являются кандидатами на упрощение.

### **3. Сложность связей (Coupling Complexity)**

Оценивает степень взаимозависимости между модулями. Цель — достичь слабой связанности.

**Типы связности (от худшего к лучшему)**:

- **Content coupling**: один модуль напрямую изменяет внутренние данные другого.
- **Common coupling**: модули используют общие глобальные данные.
- **Control coupling**: один модуль передает другому флаг или данные, управляющие его логикой.
- **Stamp coupling**: модулям передается большая структура данных, хотя нужна лишь ее часть.
- **Data coupling**: модули обмениваются только необходимыми данными через параметры (идеал).

**Метрики**:

- **CBO (Coupling Between Objects)**: количество классов, с которыми непосредственно связан данный класс.
- **Ca (Afferent Coupling)**: количество классов, которые зависят от данного (входящие связи).
- **Ce (Efferent Coupling)**: количество классов, от которых зависит данный (исходящие связи).

**Влияние на разработку**:

- Высокая связанность затрудняет модульное тестирование, требуя создания множества mock-объектов.
- Повышает риск каскадных изменений: правка в одном модуле ведет к необходимости правок во многих других.
- Современные инструменты анализа кода помогают визуализировать зависимости и выявлять проблемные узлы.

### **4. Сложность поддержки (Maintainability Complexity)**

Составная характеристика, прогнозирующая усилия по сопровождению кода.

**Основные компоненты**:

- **Анализируемость**: легкость понимания кода, отладки и локализации дефектов.
- **Изменяемость**: легкость внесения изменений и минимальный риск побочных эффектов.
- **Стабильность**: устойчивость к распространению ошибок при изменениях.
- **Тестируемость**: степень, в которой код облегчает создание и проведение тестов.

**Ключевые метрики**:

- **Индекс поддерживаемости (MI)**: комплексный показатель, рассчитываемый на основе цикломатической сложности,
  количества строк кода (LOC) и объема комментариев.
- **Соотношение комментарии/код**: показатель документированности.
- **Нарушения стандартов кода**: количество отступлений от соглашений (PEP8, Google Style и др.).
- **Дублирование кода**: процент строк, повторяющихся в кодовой базе.

**Инструменты**: SonarQube, CodeClimate, Codacy. Эти платформы агрегируют метрики, устанавливают пороговые значения ("
Quality Gate") и предоставляют сводные отчеты о здоровье кода, помогая командам проактивно работать над улучшением
сопровождаемости.

Надеюсь, это объяснение было полезным и полным. Если у вас есть вопросы по какому-то из видов сложности или инструментам
для их анализа — обращайтесь.

[Содержание](/CONTENTS.md#содержание)

## **Senior Level**

### **1. Асимптотическая сложность (Asymptotic Complexity) (Big O only)**

## **O(1) - Константная сложность**

```python
def get_first_element(arr):
    return arr[0]  # Всегда одна операция


def is_even(n):
    return n % 2 == 0  # Одна операция
```

## **O(log n) - Логарифмическая сложность**

```python
def binary_search(arr, target):
    left, right = 0, len(arr) - 1
    while left <= right:
        mid = (left + right) // 2
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    return -1
```

## **O(n) - Линейная сложность**

```python
def linear_search(arr, target):
    for i in range(len(arr)):  # Проход по всем элементам
        if arr[i] == target:
            return i
    return -1


def sum_array(arr):
    total = 0
    for num in arr:  # Один цикл по всем элементам
        total += num
    return total
```

## **O(n log n) - Линейно-логарифмическая сложность**

```python
def merge_sort(arr):
    if len(arr) <= 1:
        return arr

    mid = len(arr) // 2
    left = merge_sort(arr[:mid])
    right = merge_sort(arr[mid:])

    return merge(left, right)


def merge(left, right):
    result = []
    i = j = 0
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            result.append(left[i])
            i += 1
        else:
            result.append(right[j])
            j += 1
    result.extend(left[i:])
    result.extend(right[j:])
    return result
```

## **O(n²) - Квадратичная сложность**

```python
def bubble_sort(arr):
    n = len(arr)
    for i in range(n):  # Внешний цикл
        for j in range(0, n - i - 1):  # Внутренний цикл
            if arr[j] > arr[j + 1]:
                arr[j], arr[j + 1] = arr[j + 1], arr[j]
    return arr


def find_duplicates(arr):
    duplicates = []
    for i in range(len(arr)):  # Двойной цикл
        for j in range(i + 1, len(arr)):
            if arr[i] == arr[j]:
                duplicates.append(arr[i])
    return duplicates
```

## **O(n³) - Кубическая сложность**

```python
def find_triplets(arr):
    n = len(arr)
    triplets = []
    for i in range(n):  # Три вложенных цикла
        for j in range(i + 1, n):
            for k in range(j + 1, n):
                if arr[i] + arr[j] + arr[k] == 0:
                    triplets.append((arr[i], arr[j], arr[k]))
    return triplets
```

## **O(2ⁿ) - Экспоненциальная сложность**

```python
def fibonacci_recursive(n):
    if n <= 1:
        return n
    return fibonacci_recursive(n - 1) + fibonacci_recursive(n - 2)


def generate_subsets(arr):
    def backtrack(start, current):
        subsets.append(current.copy())
        for i in range(start, len(arr)):
            current.append(arr[i])
            backtrack(i + 1, current)
            current.pop()

    subsets = []
    backtrack(0, [])
    return subsets
```

## **O(n!) - Факториальная сложность**

```python
def generate_permutations(arr):
    def backtrack(path):
        if len(path) == len(arr):
            permutations.append(path.copy())
            return
        for num in arr:
            if num not in path:
                path.append(num)
                backtrack(path)
                path.pop()

    permutations = []
    backtrack([])
    return permutations

# Пример: для 3 элементов будет 3! = 6 перестановок
```

## **O(√n) - Сложность квадратного корня**

```python
def is_prime(n):
    if n < 2:
        return False
    for i in range(2, int(n ** 0.5) + 1):  # До квадратного корня
        if n % i == 0:
            return False
    return True
```

**Ключевые выводы:**

1. **O(1)** и **O(log n)** - самые эффективные
2. **O(n)** и **O(n log n)** - приемлемы для больших данных
3. **O(n²)**, **O(2ⁿ)**, **O(n!)** - становятся непрактичными при росте n
4. При выборе алгоритма важно оценивать ожидаемый размер входных данных

### **2. Цикломатическая сложность (Cyclomatic Complexity)**

### **V(G) = 1** - Простая функция

```python
def simple_greeting(name):
    """Цикломатическая сложность = 1"""
    return f"Hello, {name}!"

# Граф: один путь выполнения
```

### **V(G) = 2** - Одно условие

```python
def check_age(age):
    """Цикломатическая сложность = 2 (1 условие + 1)"""
    if age >= 18:
        return "Adult"
    else:
        return "Minor"

# Условия: 1
# Пути: 1) age >= 18, 2) age < 18
```

### **V(G) = 3** - Два условия (или if-elif)

```python
def grade_score(score):
    """Цикломатическая сложность = 3"""
    if score >= 90:
        return "A"
    elif score >= 70:
        return "B"
    else:
        return "C"

# Условия: 2 (score >= 90 и score >= 70)
# Пути: 3 возможных пути
```

### **V(G) = 4** - Несколько условий и цикл

```python
def categorize_temperature(temp):
    """Цикломатическая сложность = 4"""
    if temp < 0:
        category = "Freezing"
    elif temp < 15:
        category = "Cold"
    elif temp < 25:
        category = "Warm"
    else:
        category = "Hot"

    # Добавляем дополнительный путь
    if "Cold" in category or "Freezing" in category:
        return f"{category} - Wear jacket"
    return f"{category} - Light clothes"

# Условия: 4
# Пути: множество комбинаций
```

### **V(G) = 6** - Вложенные условия

```python
def check_access(user_role, subscription_type, age):
    """Цикломатическая сложность = 6"""
    if user_role == "admin":
        if subscription_type == "premium":
            return "Full access"
        else:
            return "Admin basic access"
    elif user_role == "user":
        if age >= 18:
            if subscription_type == "premium":
                return "Premium user access"
            else:
                return "Basic user access"
        else:
            return "Restricted access (minor)"
    else:
        return "No access"

# Граф:
# - 3 внешних условия (admin/user/else)
# - 2 вложенных условия в admin
# - 3 вложенных условия в user
```

### **V(G) = 8** - Сложная логика с циклами

```python
def process_items(items, threshold, max_attempts):
    """Цикломатическая сложность = 8"""
    result = []

    for item in items:  # Цикл добавляет 1 к сложности
        if item is None:
            continue  # Дополнительный путь

        attempts = 0
        success = False

        while attempts < max_attempts and not success:  # Еще +1
            if item["value"] > threshold:
                if item.get("valid", False):
                    success = True
                    result.append(item["value"] * 2)
                else:
                    result.append(item["value"])
            elif item["value"] > 0:
                result.append(item["value"] // 2)
            else:
                break  # Еще один путь

            attempts += 1

        if not success:
            result.append(-1)

    return result

# Составляющие сложности:
# - for loop: +1
# - while loop: +1  
# - if item is None: +1
# - if item["value"] > threshold: +1
# - if item.get("valid", False): +1
# - elif item["value"] > 0: +1
# - else в цикле while: +1
# - if not success: +1
# Итого: 8
```

### **V(G) = 15+** - "Плохой" код с высокой сложностью

```python
def monster_function(a, b, c, d, e, f):
    """Цикломатическая сложность > 15 - ТРЕБУЕТ РЕФАКТОРИНГА!"""
    result = 0

    if a > 0:
        if b < 10:
            result += 1
        elif b < 20:
            result += 2
        else:
            if c == "type1":
                result += 3
            elif c == "type2":
                result += 4
            else:
                result += 5
    elif a == 0:
        if d:
            for i in range(e):  # +1
                if i % 2 == 0:  # +1
                    result += i
                else:
                    if f:  # +1
                        result -= i
                    else:
                        result += i * 2
        else:
            result = -1
    else:
        if e > 100:
            result = e * 2
        elif e > 50:
            result = e
        else:
            if f:
                result = e // 2
            else:
                result = e // 4

    # Дополнительные вложенные условия
    if result > 1000:
        if a > 50:
            result *= 1.1
        elif a > 20:
            result *= 1.05

    return result

# Эта функция имеет очень высокую цикломатическую сложность
# из-за глубокой вложенности и множества условий
```

# **typing**

[Содержание](/CONTENTS.md#содержание)

## **Junior Level**

`typing` модуль в Python предоставляет инструменты для добавления подсказок типов (type hints) в код. Это не меняет
поведение программы во время выполнения, но помогает разработчикам, IDE и статическим анализаторам понимать, какие типы
данных ожидаются.

**Optional[X]** означает, что значение может быть либо типа `X`, либо `None`. Это удобный способ сказать "этот аргумент
может быть передан, а может и нет".

**Union[X, Y, ...]** означает, что значение может быть одного из перечисленных типов. Например, `Union[int, str]` — это
либо целое число, либо строка.

**TypeVar** используется для создания обобщенных (generic) типов. Например, если у вас есть функция, которая работает со
списками любого типа, вы можете использовать `TypeVar` чтобы показать, что тип элементов входного списка и выходного
значения одинаков.

**Generic** — это способ создавать классы или функции, которые могут работать с разными типами, но сохранять информацию
о конкретном типе. Например, `List[int]` — это список целых чисел, а `List[str]` — список строк. `Generic` позволяет вам
создавать свои собственные классы, которые могут быть параметризованы типами.

[Содержание](/CONTENTS.md#содержание)

## **Middle Level**

Технически, эти конструкции являются частью системы типизации, которая реализована в модуле `typing` и поддерживается
статическими анализаторами (mypy, pyright, PyCharm).

1. **Optional и Union:**
    - `Optional[X]` это просто сокращение для `Union[X, None]`.
    - `Union` может использоваться для любого количества типов. В Python 3.10 появился синтаксис `X | Y` как
      альтернатива `Union[X, Y]`.
    - Важно: `Optional` и `Union` не выполняют проверку типов во время выполнения. Они только для статического анализа.
    - Для проверки в runtime можно использовать `isinstance()` с кортежем типов, но это не связано напрямую с
      аннотациями.

2. **TypeVar:**
    - Создается вызовом `TypeVar(name, *constraints, bound=None, covariant=False, contravariant=False)`.
    - **Ограничения (constraints):** TypeVar может быть ограничен конкретными типами. Тогда он может представлять только
      один из них.
    - **Связывание (bound):** TypeVar может быть связан с базовым классом или протоколом. Тогда он представляет любой
      подтип этого базового класса.
    - **Ковариантность/контравариантность:** Позволяют выразить отношения между производными типами. Например, если `C`
      ковариантен по `T`, то `C[Dog]` является подтипом `C[Animal]` (при условии, что `Dog` подтип `Animal`). Это важно
      для корректного определения подтипов в обобщенных классах.

3. **Generic:**
    - Класс, наследующийся от `Generic[T]`, где `T` — это TypeVar, становится обобщенным.
    - Можно использовать несколько TypeVar: `Generic[T, U]`.
    - Внутри класса можно использовать `T` как обычный тип в аннотациях методов и атрибутов.
    - При наследовании от обобщенного класса можно либо конкретизировать тип (`ChildClass[int]`), либо передать TypeVar
      дальше.

4. **Для AQA:**
    - Type hints помогают документировать интерфейсы тестовых утилит и фикстур.
    - `Optional` часто используется для необязательных аргументов в функциях-фикстурах.
    - `Union` полезен, когда функция может возвращать разные типы данных в зависимости от условий (например,
      `Union[WebElement, None]` при поиске элемента).
    - `Generic` и `TypeVar` позволяют создавать гибкие, переиспользуемые компоненты тестовых фреймворков, например,
      абстрактный репозиторий для тестовых данных `Repository[T]`, где `T` — тип модели.

[Содержание](/CONTENTS.md#содержание)

## **Senior Level**

## Хранение аннотаций в CPython

Аннотации типов в CPython (Python 3.9+) хранятся в специальном атрибуте `__annotations__` объектов (функций, классов,
модулей). Этот атрибут представляет собой словарь `dict[str, object]`, где ключи — имена переменных/параметров, а
значения — сами аннотации (типы или выражения). CPython не выполняет аннотации во время выполнения (runtime) — они
остаются "ленивыми" объектами для статических анализаторов вроде mypy.

В режиме `from __future__ import annotations` (по умолчанию с Python 3.10, рекомендуется в 3.9+), аннотации сохраняются
как строки без вычисления, что предотвращает ошибки вроде `NameError` при ссылке на неопределённые имена. Это
реализовано в парсере AST (Abstract Syntax Tree) CPython: во время компиляции в байткод аннотации парсятся как
`ast.AnnAssign` и сериализуются в строки.

```python
# Байткод примера: def func(x: str) -> int:
# dis.dis(func) покажет LOAD_BUILD_CLASS и MAKE_FUNCTION с флагом CO_NESTED
# Аннотации попадают в co_varnames и co_cellvars фрейма функции
import dis


def func(x: str) -> int:
    return x


print(func.__annotations__)  # {'x': <class 'str'>, 'return': <class 'int'>}
```

Представь, что CPython — это почтальон. Когда ты пишешь `x: str`, он не проверяет посылку (str) сразу, а просто
записывает адрес "x ожидает str" в специальную тетрадку `__annotations__`. Позже статический анализатор (mypy) открывает
тетрадку и проверяет, всё ли правильно. Если включишь `from __future__ import annotations`, он даже не смотрит на типы —
просто записывает как текст "str", чтоб не было ошибок, если имя типа ещё не определено.

## Реализация в модуле Lib/typing.py

Модуль `typing.py` (CPython sources: `Lib/typing.py`) — это чистый Python-код (~3000 строк в 3.12+), который создаёт
подклассы и прокси для типов. Он не трогает C-уровень CPython напрямую, но использует служебные классы вроде
`_GenericAlias`, `_SpecialForm`. Ключевые структуры: `_TypingEmpty`, `_TypingEllipsis` для специальных случаев (
Tuple[...], Callable).

Основная магия — в `__getitem__()` generic-типов. Когда пишешь `List[int]`, CPython вызывает `list.__getitem__(int)`, но
typing перехватывает через подкласс `_GenericAlias`:

```python
# Из CPython Lib/typing.py (Python 3.12+ fragment, упрощённо)
class _GenericAlias:
    def __getitem__(self, params):
        if not isinstance(params, tuple):
            params = (params,)
        # Проверяем, что параметры — валидные типы
        msg = "Parameters to generic types must be types."
        params = tuple(_type_check(p, msg) for p in params)
        # Собираем type variables для подстановки
        self.__parameters__ = _collect_type_vars(params)
        # Создаём новый alias с подстановкой
        return _subs_tvars(self, self.__parameters__, params)


# Пример работы:
List = _GenericAlias('list', inst=actual_list)  # actual_list из builtins
List[int]  # -> _GenericAlias(list, (int,), name='list[int]')
```

`_GenericAlias` — как фабрика по производству "типовых коробок". `List[int]` говорит: "Возьми коробку list и пометь её,
что внутри int". CPython проверяет, что `int` — настоящий тип (не число или строка), собирает переменные типов (
TypeVar), подставляет их и выдаёт новую "метку" вида list[int]. Это не настоящий list, а прокси, который mypy понимает.
На runtime ничего не проверяется — просто метка болтается в памяти.

## C-уровень: Обработка __annotations__ в Objects

На C-уровне (Objects/descrobject.c, Objects/functionobject.c) `__annotations__` — это дескриптор (data descriptor).
Доступ через `PyObject_GenericGetAttr` → проверка `__dict__` → fallback на слоты типа `tp_getattro`.

Для функций: при создании `PyFunctionObject` (Objects/funcobject.c) байткод компилятора (Python/compile.c) заполняет
`func_annotations` через `STORE_ANNOTATION` opcode (с Python 3.5+). В 3.9+ добавлена оптимизация: аннотации кэшируются в
`PyDictObject` и не пересчитываются.

```c
// Из CPython Objects/funcobject.c (Python 3.12, упрощённо)
// PyFunctionObject структура:
typedef struct {
    PyObject_HEAD
    PyObject *func_code;      /* код байткода */
    PyObject *func_globals;   /* глобальное пространство */
    PyObject *func_annotations; /* <-- НАШЕ dict[str, annotation] */
    // ... другие слоты
} PyFunctionObject;

// В func_new():
if (annotations != NULL) {
    // Копируем dict аннотаций из co_annotations код-объекта
    func->func_annotations = Py_NewRef(annotations);
    // PyDict_SetItem копирует пары key:value без вычисления
}
```

C-код CPython — это "склад товаров". Каждый объект (функция) имеет полку `func_annotations` — большой ящик-словарь.
Когда компилятор видит `: str` в коде, он кладёт ярлык "{'x': str}" в этот ящик, не открывая посылки. При
`func.__annotations__` C-функция просто достаёт ящик и отдаёт. Нет проверок типов — только хранение. В 3.9+ ящик
сделан "ленивым": если `from __future__ import annotations`, ярлыки — сырые строки вроде "<class 'str'>", чтоб не
ломалось при импорте.

## Байткод и компиляция аннотаций (Python 3.9+)

Компилятор (Python/compile.c → Python/symtable.c) парсит AST-узлы `AnnAssign`/`arg.annotation`. В 3.9+ добавлена
поддержка built-in generics (`list[int]` вместо `typing.List[int]`) через `symtable.c` анализ forward refs.

Ключевой opcode: `STORE_ANNOTATION` (opcode 95 в 3.12). Он сохраняет аннотацию в `co_annotations` код-объекта.

```python
# dis.dis для def func(a: int): pass
import dis


def func(a: int): pass


dis.dis(func)
# Вывод (Python 3.12):
#   2           0 RESUME                   0
#               1 LOAD_CONST               1 (<code object func>)
#               ...
# В co_annotations: {'a': <class 'int'>}
```

Байткод — инструкции для виртуальной машины CPython. `STORE_ANNOTATION` — команда "положи метку типа в специальный сейф
код-объекта". При запуске функции VM берёт сейф и копирует в `func_annotations`. В 3.9+ парсер стал умнее: распознаёт
`list[int]` на лету, без импорта typing, и сохраняет как `_GenericAlias`. Но на выполнении байткода типы игнорируются —
VM просто пропускает STORE_ANNOTATION.

## TypeVar и Generic: Внутренняя подстановка

`TypeVar` — `_Final` subclass с `__parameters__`. Generic использует `_collect_type_vars()` для рекурсивного сбора
переменных и `_subs_tvars()` для подстановки (из typing.py).

```python
# Из Lib/typing.py
def _subs_tvars(original, parameters, args):
    """Подстановка TypeVar в Generic"""
    new_args = []
    for arg in original.__args__:
        if isinstance(arg, _TypeVarLike):
            idx = parameters.index(arg)
            new_args.append(args[idx])
        else:
            new_args.append(arg)
    return type(original)(original.__origin__, tuple(new_args))
```

TypeVar — как пустая коробка с именем "T". Generic — шаблон "Коробка[T]". При `MyGeneric[str]` CPython находит все "T" в
шаблоне, заменяет на "str" и выдаёт новую коробку "Коробка[str]". Это рекурсивно: если внутри T есть другие переменные,
они тоже подставляются. Всё в Python-коде typing.py, без C, но быстро благодаря `@_tp_cache` (lru_cache).

Эти механизмы делают typing эффективным: ~0 overhead на runtime, полная поддержка в IDE/mypy. Для собеседования
акцентируй: нет enforcement в CPython, только storage/parsing.

- [Содержание](/CONTENTS.md#содержание)

---

### **3. Сложность связей (Coupling Complexity)**

#### **Content Coupling (Худший вид)**

Один модуль напрямую изменяет внутренние данные или логику другого.

```python
# ПЛОХО: Модуль A манипулирует внутренним состоянием модуля B
class UserDatabase:
    def __init__(self):
        self._users = []  # Приватное поле


class AdminPanel:
    def clear_database(self, db):
        db._users.clear()  # Нарушение инкапсуляции!


# Использование:
db = UserDatabase()
admin = AdminPanel()
admin.clear_database(db)  # Прямой доступ к приватному полю
```

#### **Common/Global Coupling**

Модули используют общие глобальные данные.

```python
# ПЛОХО: Множество компонентов зависят от глобальной переменной
CONFIG = {"theme": "dark", "timeout": 30}  # Глобальное состояние


class UserService:
    def get_timeout(self):
        return CONFIG["timeout"]  # Прямая зависимость


class ApiClient:
    def request(self):
        timeout = CONFIG["timeout"]  # Та же зависимость
        # ... использование timeout
```

#### **Control Coupling**

Один модуль передает флаги или данные, управляющие логикой другого.

```python
# ПЛОХО: Флаг управляет внутренней логикой метода
def process_data(data, format_type):
    """format_type: 'json', 'xml', 'csv'"""
    if format_type == "json":
        return json.dumps(data)
    elif format_type == "xml":
        return to_xml(data)  # Предположим, есть такая функция
    elif format_type == "csv":
        return to_csv(data)
    else:
        raise ValueError("Unknown format")


# Вызывающая сторона управляет внутренним поведением
result = process_data(my_data, "json")
```

#### **Stamp Coupling**

Передача больших структур данных, когда нужна только их часть.

```python
# ПЛОХО: Передается весь объект пользователя, хотя нужен только email
def send_email(user):
    """Требуется только user.email, но получает целый объект"""
    email = user.email
    name = user.name  # Не используется в отправке
    # ... логика отправки


# Более четкий интерфейс:
def send_email_to_address(email_address, message):
    # Использует только необходимые данные
    pass
```

#### **Data Coupling (Идеал)**

Модули обмениваются только необходимыми данными через параметры.

```python
# ХОРОШО: Слабая связанность через четкие интерфейсы и зависимости
class EmailSender:
    def send(self, to_address: str, subject: str, body: str) -> bool:
        # Логика отправки
        return True


class UserNotifier:
    def __init__(self, email_sender: EmailSender):  # Внедрение зависимости
        self._email_sender = email_sender

    def notify_password_change(self, user_email: str):
        # Использует только необходимые данные и явную зависимость
        self._email_sender.send(
            to_address=user_email,
            subject="Password Changed",
            body="Your password was successfully changed."
        )


# Использование с Dependency Injection
sender = EmailSender()
notifier = UserNotifier(sender)
notifier.notify_password_change("user@example.com")
```

### **4. Сложность поддержки (Maintainability Complexity)**

**Низкая поддерживаемость (Technical Debt):**

```python
# ПЛОХО: Множество проблем в одной функции
def proc(d, t, s, f, m, c):  # Непонятные параметры
    r = 0
    # Магические числа
    if d > 100 and t == "A" or t == "B" and s:  # Сложное условие
        for i in range(len(f)):  # C-стиль итерации
            if f[i] and m.get(c, False):  # Смесь логики
                try:
                    r += complex_operation(d, f[i])  # Неочевидная функция
                except:
                    r = -1  # Подавление исключений
    elif d < 0:  # Не покрывает edge-case d == 0
        r = None
    # Отсутствует обработка других случаев
    return r  # Может вернуть число, None или -1
```

**Высокая поддерживаемость:**

```python
# ХОРОШО: Чистый, самодокументирующийся код
from typing import Optional, List
from enum import Enum


class DeviceType(Enum):
    TYPE_A = "A"
    TYPE_B = "B"
    UNKNOWN = "UNKNOWN"


class ProcessingResult:
    def __init__(self, value: float, status: str):
        self.value = value
        self.status = status


def process_device_data(
        device_id: int,
        device_type: DeviceType,
        is_active: bool,
        sensor_readings: List[float],
        calibration_map: dict,
        calibration_key: str
) -> Optional[ProcessingResult]:
    """
    Обрабатывает данные устройства и возвращает результат калибровки.
    
    Args:
        device_id: Уникальный идентификатор устройства
        device_type: Тип устройства из перечисления DeviceType
        is_active: Флаг активности устройства
        sensor_readings: Показания датчиков устройства
        calibration_map: Карта калибровочных коэффициентов
        calibration_key: Ключ для поиска коэффициента калибровки
    
    Returns:
        ProcessingResult с значением и статусом или None при ошибке
    
    Raises:
        ValueError: При некорректных входных данных
    """
    VALID_DEVICE_THRESHOLD = 100

    # Валидация входных данных
    if not sensor_readings:
        raise ValueError("Sensor readings cannot be empty")

    # Проверка условий обработки
    should_process = (
            device_id > VALID_DEVICE_THRESHOLD and
            device_type in (DeviceType.TYPE_A, DeviceType.TYPE_B) and
            is_active
    )

    if not should_process:
        return None

    # Обработка показаний датчиков
    calibration_factor = calibration_map.get(calibration_key)
    if not calibration_factor:
        return ProcessingResult(
            value=0.0,
            status="CALIBRATION_MISSING"
        )

    try:
        total = sum(
            reading * calibration_factor
            for reading in sensor_readings
            if reading is not None
        )
        average = total / len(sensor_readings)

        return ProcessingResult(
            value=average,
            status="SUCCESS"
        )

    except (ZeroDivisionError, TypeError) as e:
        logger.error(f"Processing failed for device {device_id}: {e}")
        return ProcessingResult(
            value=0.0,
            status="PROCESSING_ERROR"
        )
```

- [Содержание](/CONTENTS.md#содержание)

---

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

---

# Сравнение объектов**

[Содержание](/CONTENTS.md#содержание)

## **Junior Level**

[Содержание](/CONTENTS.md#содержание)

## **Middle Level**

[Содержание](/CONTENTS.md#содержание)

## **Senior Level**

- [Содержание](/CONTENTS.md#содержание)

---

# Управление памятью

[Содержание](/CONTENTS.md#содержание)

## **Junior Level**

[Содержание](/CONTENTS.md#содержание)

## **Middle Level**

[Содержание](/CONTENTS.md#содержание)

## **Senior Level**

- [Содержание](/CONTENTS.md#содержание)

---

# Исключения и обработка ошибок

[Содержание](/CONTENTS.md#содержание)

## **Junior Level**

[Содержание](/CONTENTS.md#содержание)

## **Middle Level**

[Содержание](/CONTENTS.md#содержание)

## **Senior Level**

- [Содержание](/CONTENTS.md#содержание)

---

# Сериализация/десериализация

[Содержание](/CONTENTS.md#содержание)

## **Junior Level**

[Содержание](/CONTENTS.md#содержание)

## **Middle Level**

[Содержание](/CONTENTS.md#содержание)

## **Senior Level**

- [Содержание](/CONTENTS.md#содержание)

---

# Работа с файлами и путями

[Содержание](/CONTENTS.md#содержание)

## **Junior Level**

[Содержание](/CONTENTS.md#содержание)

## **Middle Level**

[Содержание](/CONTENTS.md#содержание)

## **Senior Level**

- [Содержание](/CONTENTS.md#содержание)

---

# Системные вызовы

[Содержание](/CONTENTS.md#содержание)

## **Junior Level**

[Содержание](/CONTENTS.md#содержание)

## **Middle Level**

[Содержание](/CONTENTS.md#содержание)

## **Senior Level**

- [Содержание](/CONTENTS.md#содержание)

---

# Метаклассы

[Содержание](/CONTENTS.md#содержание)

## **Junior Level**

[Содержание](/CONTENTS.md#содержание)

## **Middle Level**

[Содержание](/CONTENTS.md#содержание)

## **Senior Level**

- [Содержание](/CONTENTS.md#содержание)

---

# Дескрипторы

[Содержание](/CONTENTS.md#содержание)

## **Junior Level**

[Содержание](/CONTENTS.md#содержание)

## **Middle Level**

[Содержание](/CONTENTS.md#содержание)

## **Senior Level**

- [Содержание](/CONTENTS.md#содержание)

---

# Слоты

[Содержание](/CONTENTS.md#содержание)

## **Junior Level**

[Содержание](/CONTENTS.md#содержание)

## **Middle Level**

[Содержание](/CONTENTS.md#содержание)

## **Senior Level**

- [Содержание](/CONTENTS.md#содержание)

---

# Weak references

[Содержание](/CONTENTS.md#содержание)

## **Junior Level**

[Содержание](/CONTENTS.md#содержание)

## **Middle Level**

[Содержание](/CONTENTS.md#содержание)

## **Senior Level**

- [Содержание](/CONTENTS.md#содержание)

---

# Атрибуты объектов

[Содержание](/CONTENTS.md#содержание)

## **Junior Level**

[Содержание](/CONTENTS.md#содержание)

## **Middle Level**

[Содержание](/CONTENTS.md#содержание)

## **Senior Level**

- [Содержание](/CONTENTS.md#содержание)

---

# Система импорта

[Содержание](/CONTENTS.md#содержание)

## **Junior Level**

[Содержание](/CONTENTS.md#содержание)

## **Middle Level**

[Содержание](/CONTENTS.md#содержание)

## **Senior Level**

- [Содержание](/CONTENTS.md#содержание)

---

# Пул потоков/процессов

[Содержание](/CONTENTS.md#содержание)

## **Junior Level**

[Содержание](/CONTENTS.md#содержание)

## **Middle Level**

[Содержание](/CONTENTS.md#содержание)

## **Senior Level**

- [Содержание](/CONTENTS.md#содержание)

---

# Очереди

[Содержание](/CONTENTS.md#содержание)

## **Junior Level**

[Содержание](/CONTENTS.md#содержание)

## **Middle Level**

[Содержание](/CONTENTS.md#содержание)

## **Senior Level**

- [Содержание](/CONTENTS.md#содержание)

---

# Синхронизация

[Содержание](/CONTENTS.md#содержание)

## **Junior Level**

[Содержание](/CONTENTS.md#содержание)

## **Middle Level**

[Содержание](/CONTENTS.md#содержание)

## **Senior Level**

- [Содержание](/CONTENTS.md#содержание)
