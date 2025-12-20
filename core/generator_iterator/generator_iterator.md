# **Генераторы и итераторы**

[Содержание](/CONTENTS.md#содержание)

## **Junior Level**

Чтобы разобраться с генераторами, сначала нужно понять два ключевых понятия: **итерируемые объекты** и **итераторы**.

**Итерируемый объект (Iterable)** — это любой объект, по которому можно пройтись в цикле `for`. Например, списки,
строки, словари. У такого объекта есть метод `__iter__()`, который возвращает специальный объект — итератор.

**Итератор (Iterator)** — это "обходчик" или "курсор". Его задача — помнить, какой элемент будет следующим, и выдавать
его по запросу. У каждого итератора есть метод `__next__()`. Когда вы вызываете его (обычно это делает цикл `for`), он
возвращает следующий элемент. Когда элементы заканчиваются, он сигнализирует об этом, вызывая исключение
`StopIteration`.

Цикл `for` работает по такой схеме:

1. Берёт итерируемый объект (например, список).
2. Вызывает у него `iter()`, чтобы получить итератор.
3. Внутри цикла многократно вызывает у этого итератора `next()`, пока не получит `StopIteration`.

**Генератор (Generator)** — это особый, "ленивый" итератор, который создаётся не для готовой коллекции, а по мере
необходимости.

Представьте, что вам нужно прочитать огромную книгу. Обычный итератор — это как если бы вы скопировали всю книгу в
оперативную память, прежде чем начать читать. Генератор — это как если бы вы читали её по одной странице, и следующая
страница подгружалась только когда вы к ней переходите. Это и есть **ленивое вычисление (lazy evaluation)**.

Создаётся генератор с помощью функции, где вместо `return` используется ключевое слово `yield`:

* При вызове такая функция возвращает не результат, а объект-генератор.
* При первом вызове `next()` с ним функция выполняется до первого `yield`, возвращает значение и **замораживает** своё
  состояние (все локальные переменные, место в коде).
* При следующем вызове `next()` выполнение возобновляется с того же места и длится до следующего `yield`.

Главные преимущества генераторов:

1. **Экономия памяти:** Не нужно хранить всю последовательность данных сразу.
2. **Работа с бесконечными потоками:** Можно создать генератор, который, например, выдаёт новые случайные числа
   бесконечно.
3. **Гибкость:** С генератором можно общаться "в обе стороны" — не только получать значения, но и посылать ему данные
   или исключения с помощью методов `.send()` и `.throw()`.

[Содержание](/CONTENTS.md#содержание)

## **Middle Level**

На этом уровне важно понимать не только как этим пользоваться, но и как всё устроено изнутри, а также знать продвинутые
приёмы.

### 1. Детали протокола итераторов

* Итерируемый объект реализует метод `__iter__()`. Его задача — вернуть новый итератор. У самого итератора метод
  `__iter__()` обычно возвращает `self` (самого себя).
* Для обратной совместимости объект без `__iter__()`, но с корректно реализованным методом `__getitem__(index)` (который
  работает для индексов, начиная с 0), также считается итерируемым. Цикл `for` будет вызывать его, пока не получит
  `IndexError`.

### 2. Генераторы: под капотом и на практике

* **Генератор — это состояние.** Когда выполнение приостанавливается на `yield`, генератор сохраняет в памяти весь свой
  контекст: локальные переменные, указатель инструкций и стек вызовов. Это делает его мощным инструментом для создания *
  *сопрограмм** (простых кооперативных задач).
* **`return` в генераторе:** В генераторной функции можно использовать `return`. Когда поток выполнения доходит до него,
  возникает исключение `StopIteration`, а значение из `return` становится атрибутом `value` этого исключения. Его можно
  перехватить, но в цикле `for` оно будет проигнорировано.

### 3. Генераторные выражения

Это компактный синтаксис для создания простых генераторов "на лету".

```python
# List comprehension — создаёт список в памяти сразу
squares_list = [x ** 2 for x in range(1000000)]

# Generator expression — создаёт генератор, который будет вычислять значения по одному
squares_gen = (x ** 2 for x in range(1000000))
```

Генераторные выражения идеально подходят для передачи в функции, которые работают с итераторами (`sum()`, `max()`,
`min()`, `join()`), позволяя не создавать промежуточные списки.

### 4. Продвинутые методы генераторов

Генераторы поддерживают **двусторонний обмен**, что отличает их от простых итераторов.

* **`.send(value)`:** Позволяет отправить значение *внутрь* генератора. Это значение становится результатом выражения
  `yield`, на котором генератор был приостановлен. Это основа для более сложных паттернов (например, корутин).
* **`.throw(exception)`:** Позволяет инициировать исключение *внутри* генератора в точке приостановки. Это даёт внешнему
  коду возможность управлять поведением генератора, сообщая об ошибках или условиях завершения.
* **`.close()`:** Корректно останавливает генератор, вызывая внутри него исключение `GeneratorExit`. Генератор может его
  перехватить, чтобы выполнить финальные действия (например, закрыть файл).

### 5. `yield from` — делегирование генераторов (Python 3.3+)

Этот оператор решает проблему композиции генераторов. Вместо того чтобы вручную перебирать элементы вложенного итератора
в цикле:

```python
def chain_old(*iterables):
    for it in iterables:
        for item in it:
            yield item
```

Можно просто делегировать выполнение:

```python
def chain(*iterables):
    for it in iterables:
        yield from it  # Делегируем генерацию под-итератору `it`
```

**`yield from`** — это не просто синтаксический сахар:

* Он автоматически передаёт значения из вложенного генератора напрямую внешнему.
* Он также **прозрачно передаёт** вызовы `.send()` и `.throw()` во вложенный генератор, что критически важно для
  сохранения семантики двусторонней связи в цепочке генераторов. Это делает код с вложенными генераторами чистым и
  предсказуемым.

[Содержание](/CONTENTS.md#содержание)

## **Senior Level**

В CPython 3.9+ **генераторы** используют **_PyInterpreterFrame** с состоянием `FRAME_GENERATOR_SUSPENDED`, **итераторы
** — протокол `tp_iternext`/`tp_iter`. Опкоды `YIELD_VALUE`/`ASYNC_GEN_WRAP`/
`GET_AWAITABLE`. `Objects/genobject.c`, Python/ceval.c`

## 1. PyGenObject структура (3.9+)

```c
typedef struct {
    PyObject_HEAD                 // Стандартный заголовок
    PyFrameObject *gi_frame;       // Текущий фрейм выполнения
    PyCodeObject *gi_code;         // Исходный код функции
    PyObject *gi_name;             // __name__ генератора
    PyObject *gi_qualname;         // __qualname__
    PyObject *gi_running;          // Lock объект (если выполняется)
    PyObject *gi_weakreflist;      // Список слабых ссылок
    uint16_t gi_frame_state;       // FRAME_SUSPENDED, FRAME_CLOSED
    char gi_needs_finalizing;      // Нужен ли __del__
    char gi_is_running;            // Выполняется ли сейчас
} PyGenObject;
```

Генератор — это **замороженный фрейм** выполнения. `gi_frame` указывает, где остановились (
после `yield`). `gi_running` — мьютекс, чтобы не запускать параллельно.

## 2. Создание генератора: MAKE_FUNCTION + CALL_FUNCTION

```
def gen():
    yield 1
    yield 2

g = gen()  # Создаётся PyGenObject
```

```
# Байткод gen():
  0 RESUME               0    # Восстанавливаем состояние
  2 LOAD_CONST           1 (1)
  4 YIELD_VALUE          0    # yield 1 -> приостанавливаем
  6 POP_TOP                     # Убираем результат yield
  8 LOAD_CONST           2 (2)
 10 YIELD_VALUE          0    # yield 2
 12 LOAD_CONST           0 (None)
 14 RETURN_GENERATOR         # Завершаем генератор
```

`CALL_FUNCTION gen()` создаёт PyGenObject с `FRAME_CREATED`. Первый `next(g)` → `RESUME 0`
запускает до первого `YIELD_VALUE`.

## 3. YIELD_VALUE байткод (ceval.c)

```c
case YIELD_VALUE: {
    PyObject *retval = POP();          // Значение после yield
    PyFrameObject *f = frame;          // Текущий фрейм
    
    if (_PyFrame_IsGenerator(f)) {
        // Сохраняем значение в gi_frame->f_stacktop
        *f->f_stacktop++ = retval;     // retval остаётся в фрейме
        retval = Py_NewRef((PyObject *)gen);  // Возвращаем генератор
        
        // Меняем состояние фрейма
        f->frame_obj = NULL;           // Отсоединяем от PyGenObject
        _PyInterpreterFrame_MARK_FRAME_SUSPENDED(f);
        
        // Устанавливаем исключение StopIteration
        _PyErr_SetNone(tstate, PyExc_StopIteration);
        _PyErr_SetRaisedException(tstate, retval);
        
        // Возвращаемся к вызывающему коду
        JUMP_TO_INSTRUCTION(frame, next_instr);
        DISPATCH_SAME_OPARG(1);
    }
    break;
}
```

`YIELD_VALUE 1` берёт `1` со стека, сохраняет в фрейме, возвращает **сам генератор**
вызывающему. Устанавливает **внутреннее** StopIteration с значением `1`.

## 4. next(gen) → gen_send(NULL) → RESUME

```c
PyObject *PyGen_NewFrame(PyGenObject *gen) {
    PyFrameObject *f = _PyFrame_New(gen->gi_code, NULL, NULL, NULL);
    if (f == NULL) {
        return NULL;
    }
    
    // Связываем фрейм с генератором
    f->frame_obj = (PyObject *)gen;
    gen->gi_frame = f;
    
    // Устанавливаем начальное состояние
    _PyInterpreterFrame_MARK_FRAME_CREATED(f);
    return (PyObject *)f;
}

static PyObject *gen_send_ex(PyGenObject *gen, PyObject *arg, int exc) {
    PyFrameObject *f = gen->gi_frame;
    
    if (gen->gi_running != NULL) {
        PyErr_SetString(PyExc_ValueError, "generator already executing");
        return NULL;
    }
    
    // Блокируем выполнение
    Py_XINCREF(gen->gi_running);
    Py_XSETREF(gen->gi_running, Py_NewRef(Py_True));
    
    PyObject *retval;
    
    if (f == NULL || f->frame_flags == FRAME_CLOSED) {
        PyErr_SetNone(PyExc_StopIteration);
        retval = NULL;
    } else {
        // Восстанавливаем фрейм
        _PyInterpreterFrame_MARK_FRAME_READY(f);
        
        // Устанавливаем arg как следующее значение yield
        *f->f_stacktop++ = Py_NewRef(arg ? arg : Py_None);
        
        // Запускаем интерпретатор
        retval = _PyEvaluator_RunFrame(tstate, f);
        
        // Состояние после yield/return
        if (f->frame_flags & FRAME_SUSPENDED) {
            _PyInterpreterFrame_MARK_FRAME_SUSPENDED(f);
        }
    }
    
    // Разблокируем
    Py_CLEAR(gen->gi_running);
    
    return retval;
}
```

`next(g)` → `gen_send_ex(g, Py_None, 0)`: берёт фрейм из `gi_frame`, кладёт `Py_None` на
стек (как "результат предыдущего yield"), запускает `_PyEvaluator_RunFrame` до следующего `YIELD_VALUE`.

## 5. _PyInterpreterFrame состояния (3.11+)

```c
enum _PyFrame_Status {
    FRAME_CREATED,     // Только создали (RESUME 0)
    FRAME_SUSPENDED,   // После YIELD_VALUE (RESUME oparg)
    FRAME_CLOSED,      // После RETURN_GENERATOR
    FRAME_ERROR,       // Ошибка выполнения
};

#define _PyInterpreterFrame_MARK_FRAME_SUSPENDED(f) \
    ((f)->frame_flags = FRAME_SUSPENDED)

#define _PyInterpreterFrame_MARK_FRAME_READY(f) \
    ((f)->frame_flags = FRAME_READY)
```

Фрейм имеет **4 состояния**. После `yield` → `FRAME_SUSPENDED`. `RESUME oparg` восстанавливает
выполнение с правильного PC (program counter).

## 6. RESUME байткод - восстановление генератора

```c
case RESUME: {
    PyFrameObject *f = frame;
    
    if (_PyFrame_IsGenerator(f)) {
        PyGenObject *gen = _PyFrame_GetGenerator(f);
        
        // Проверяем блокировку
        if (gen->gi_running != Py_None) {
            PyErr_SetString(PyExc_ValueError, 
                "generator already executing");
            goto error;
        }
        
        // Восстанавливаем состояние по oparg
        switch (oparg) {
        case 0:                        // Первый запуск
            _PyInterpreterFrame_MARK_FRAME_EXECUTING(f);
            break;
        case 1:                        // После yield
            _PyInterpreterFrame_MARK_FRAME_SUSPENDED(f);
            break;
        default:
            Py_UNREACHABLE();
        }
        
        // Возвращаем управление интерпретатору
        DISPATCH_SAME_OPARG(1);
    }
    break;
}
```

`RESUME 0` — первый запуск (до первого yield). `RESUME 1` — возобновление после yield.
Проверяет `gi_running != Py_None` (защита от race condition).

## 7. Итераторы: tp_iter/tp_iternext протокол

```c
// PyListIterObject
typedef struct {
    PyObject_HEAD
    Py_ssize_t it_index;           // Текущий индекс
    PyListObject *it_seq;          // Ссылка на список
} PyListIterObject;

static PyObject *list_iter_next(PyListIterObject *it) {
    PyListObject *list = it->it_seq;
    Py_ssize_t index = it->it_index;
    
    if (index >= Py_SIZE(list)) {
        // Конец итерации
        PyErr_SetNone(PyExc_StopIteration);
        return NULL;
    }
    
    PyObject *item = Py_NewRef(list->ob_item[index]);
    it->it_index++;                    // ++ индекс
    return item;
}
```

`for x in lst:` → `lst.__iter__()` → PyListIterObject с `it_index=0`. Каждый `tp_iternext`
возвращает `lst.ob_item[it_index++]` до конца.

## 8. PyObject_GetIter - универсальный итератор

```c
PyObject *PyObject_GetIter(PyObject *obj) {
    PyTypeObject *t = Py_TYPE(obj);
    
    if (PyDict_CheckExact(obj)) {
        return PyDictIter(obj);        // Специальный dict итератор
    }
    
    if (PySequence_Check(obj)) {
        // Пробуем tp_iter
        unaryfunc iter = t->tp_iter;
        if (iter != NULL) {
            PyObject *res = (*iter)(obj);
            if (res != NULL && !PyIter_Check(res)) {
                PyErr_Format(PyExc_TypeError,
                    "'%.200s' object is not an iterator",
                    Py_TYPE(res)->tp_name);
                Py_DECREF(res);
                return NULL;
            }
            return res;
        }
    }
    
    // Fallback на __iter__
    PyObject *iterobj = PyObject_CallMethodObjArgs(obj, &_Py_ID(__iter__), NULL);
    if (iterobj != NULL && !PyIter_Check(iterobj)) {
        PyErr_Format(PyExc_TypeError, "'%.200s' returned non-iterator",
            Py_TYPE(iterobj)->tp_name);
        Py_DECREF(iterobj);
        iterobj = NULL;
    }
    return iterobj;
}
```

`iter(lst)` → `PyList_Type.tp_iter(lst)` → PyListIterObject. Если нет `tp_iter` — вызывает
`obj.__iter__()`.

**Генераторы/итераторы** в CPython 3.9+ — **PyGenObject** с **замороженным _PyInterpreterFrame**, состояния
`FRAME_SUSPENDED`, байткоды `YIELD_VALUE`/`RESUME`, протокол `tp_iter`/`tp_iternext`, `PyObject_GetIter` диспетчер.

- [Содержание](/CONTENTS.md#содержание)
