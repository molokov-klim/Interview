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