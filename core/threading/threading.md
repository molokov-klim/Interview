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