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