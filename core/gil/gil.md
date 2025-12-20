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