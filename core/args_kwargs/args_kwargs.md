# **args и kwargs**

[Содержание](/CONTENTS.md#содержание)

## **Junior Level**

`*args` и `**kwargs` — это специальные синтаксические конструкции в Python, позволяющие функциям принимать произвольное
количество аргументов.

**`*args`** (от слова "arguments") собирает все **позиционные аргументы**, переданные функции сверх явно объявленных, в
**кортеж**. Это полезно, когда количество передаваемых аргументов заранее неизвестно.

**`**kwargs`** (от "keyword arguments") собирает все **именованные аргументы** (ключ=значение), которые не были явно
перечислены в параметрах функции, в **словарь**.

Также символы `*` и `**` используются при **вызове** функции для распаковки коллекций:

- `*` распаковывает итерируемый объект (список, кортеж) в позиционные аргументы
- `**` распаковывает словарь в именованные аргументы

Этот механизм — основа для создания гибких API, декораторов и функций-обёрток.

[Содержание](/CONTENTS.md#содержание)

## **Middle Level**

1. **Строгий порядок параметров в определении функции**:
   ```
   def f(a, b, *args, c=None, d=None, **kwargs)
   ```
   Порядок следования:
    - Позиционные параметры (a, b)
    - `*args` — собирает избыточные позиционные аргументы
    - Keyword-only аргументы (c, d) — после `*args` все параметры требуют явного указания имени
    - `**kwargs` — собирает избыточные именованные аргументы

2. **Внутреннее представление и особенности**:
    - При передаче словаря в `**kwargs` ключи **должны быть строками**
    - Дублирование имен аргументов при распаковке вызывает `TypeError`
    - `**kwargs` сохраняет порядок аргументов (с Python 3.6)
    - Метод `__getitem__` объекта используется при распаковке через `**`, что позволяет распаковывать любые
      mapping-объекты

3. **Принцип работы распаковки**:
   Когда вызывается `func(*[1, 2, 3])`, интерпретатор:
    - Создаёт итерируемый объект
    - Распаковывает его элементы в отдельные позиционные аргументы
    - Внутри функции эти аргументы доступны через кортеж `args`

[Содержание](/CONTENTS.md#содержание)

## **Senior Level**

В CPython `*args` и `**kwargs` реализуются через специальные флаги в `PyCodeObject`, локальные переменные в фрейме
`_PyInterpreterFrame` и C-логику в `ceval.c` для упаковки/распаковки аргументов при вызове
функций. `Python/ceval.c`, `Objects/call.c`

## 1. PyCodeObject: флаги аргументов

```c
typedef struct _PyCodeObject {
    PyObject_HEAD
    int co_argcount;           // Количество позиционных аргументов
    int co_posonlyargcount;    // Только позиционные аргументы
    int co_kwonlyargcount;     // Только именованные аргументы
    int co_nlocals;            // Общее кол-во локальных переменных
    int co_stacksize;          // Размер стека
    int co_flags;              // Флаги (CO_VARARGS, CO_VARKEYWORDS)
    PyObject *co_code;         // Байткод
    PyObject *co_consts;       // Константы
    PyObject *co_names;        // Имена (аргументы, глобальные)
    PyObject *co_varnames;     // Имена локальных переменных
    PyObject *co_freevars;     // Free переменные (nonlocal)
    PyObject *co_cellvars;     // Cell переменные (nonlocal)
    // ...
} PyCodeObject;
```

`def f(a, *args, b=1, **kwargs):` → `co_argcount=1` (a), `CO_VARARGS=1`, `CO_VARKEYWORDS=1`.
`co_varnames=["a", "args", "b", "kwargs"]`.

**Флаги CO_* в co_flags:**

```c
#define CO_OPTIMIZED             0x0001  // Локальные оптимизированы
#define CO_NEWLOCALS             0x0002  // Новый locals dict
#define CO_VARARGS               0x0004  // *args присутствует
#define CO_VARKEYWORDS           0x0008  // **kwargs присутствует
#define CO_NESTED                0x0010  // Вложенная функция
#define CO_GENERATOR             0x0020  // yield присутствует
```

[Содержание](/CONTENTS.md#содержание)

## 2. CALL_FUNCTION в ceval.c: распаковка аргументов

```c
case CALL_FUNCTION: {
    PyObject **args;           // Массив аргументов
    PyObject *callable;        // Вызываемая функция
    Py_ssize_t na = oparg;     // Количество аргументов
    Py_ssize_t nargs = na;     // Фактическое количество
    
    // Снимаем аргументы со стека
    args = (PyObject **)PyMem_Malloc((na + 1) * sizeof(PyObject *));
    if (args == NULL) {
        PyErr_NoMemory();
        goto error;
    }
    
    // Копируем аргументы в обратном порядке (справа налево)
    for (Py_ssize_t i = 0; i < na; i++) {
        args[i] = POP();       // Берём со стека
    }
    args[na] = NULL;           // Завершающий NULL
    
    // Берём вызываемую функцию
    callable = PEEK(oparg);
    
    // Вызываем с распаковкой
    PyObject *result = _PyObject_Vectorcall(callable, args, nargs, NULL);
    PyMem_Free(args);
    
    Py_DECREF(callable);
    if (result == NULL) {
        goto error;
    }
    
    PUSH(result);              // Результат на стек
    DISPATCH();
}
```

Интерпретатор снимает N аргументов со стека, кладёт их в массив `args[]`, вызывает
`_PyObject_Vectorcall(func, args, N, NULL)`. Массив освобождается после вызова.

[Содержание](/CONTENTS.md#содержание)

## 3. _PyObject_Vectorcall: универсальный вызов

```c
PyObject *
_PyObject_Vectorcall(PyObject *callable, PyObject *const *args,
                     size_t nargsf, PyObject *kwnames) {
    Py_ssize_t nargs = PyVectorcall_NARGS(nargsf);
    
    // Проверяем, поддерживает ли объект vectorcall (быстрый путь)
    if (_PyObject_HasVectorcall(callable)) {
        vectorcallfunc func = _PyObject_GetVectorcall(callable);
        return func(callable, args, nargsf, kwnames);
    }
    
    // Медленный путь: PyObject_Call
    PyObject *argstuple = PyTuple_FromArray(args, nargs);
    if (argstuple == NULL) {
        return NULL;
    }
    
    PyObject *result;
    if (kwnames == NULL) {
        result = PyObject_Call(callable, argstuple, NULL);
    } else {
        result = PyObject_Call(callable, argstuple, kwnames);
    }
    
    Py_DECREF(argstuple);
    return result;
}
```

Новые функции используют **vectorcall** (C API с массивом аргументов). Старые — через
`PyTuple_FromArray` (медленнее).

[Содержание](/CONTENTS.md#содержание)

## 4. Функция с *args/**kwargs: распаковка в MAKE_FUNCTION

**Python:** `def f(a, *args, b=1, **kwargs):`

```
# Байткод в начале функции:
0
MAKE_FUNCTION
15(code_object)  # Создаём PyFunctionObject
2
LOAD_FAST 0(.0)  # Берём self/аргументы
```

**ceval.c MAKE_FUNCTION распаковывает *args/**kwargs:**

```c
case MAKE_FUNCTION: {
    PyObject *code = POP();        // PyCodeObject
    PyObject *defaults = POP();    // (a=1, b=2)
    PyObject *kwdefaults = POP();  // {c: 3}
    PyObject *closure = POP();     // Замыкания
    PyObject *annotations = POP(); // Аннотации
    PyObject *qualname = POP();    // __qualname__
    
    PyFunctionObject *func = PyFunction_New(code, globals);
    if (func == NULL) {
        goto error;
    }
    
    // Устанавливаем *args/**kwargs флаги
    if (PyCode_GET_FLAGS(code) & CO_VARARGS) {
        // args находится в co_varnames[co_argcount]
        Py_ssize_t argcount = PyCode_GETARGCOUNT(code);
        PyObject *args_name = PyTuple_GET_ITEM(code->co_varnames, argcount);
        func->func_args = args_name;  // Сохраняем имя *args
    }
    
    // Аналогично для **kwargs
    if (PyCode_GET_FLAGS(code) & CO_VARKEYWORDS) {
        Py_ssize_t kwonlyargcount = PyCode_GETKWONLYARGCOUNT(code);
        Py_ssize_t varargcount = PyCode_GETVARARGS(code) ? 1 : 0;
        Py_ssize_t nlocals = PyCode_GETNLOCALS(code);
        PyObject *kwargs_name = PyTuple_GET_ITEM(code->co_varnames, 
                                                 nlocals - kwonlyargcount - varargcount - 1);
        func->func_kwargs = kwargs_name;
    }
    
    PUSH((PyObject *)func);
}
```

При создании функции интерпретатор читает флаги CO_VARARGS/CO_VARKEYWORDS и находит имена
`*args`/`**kwargs` в `co_varnames`.

[Содержание](/CONTENTS.md#содержание)

## 5. Распаковка *args/**kwargs в фрейме выполнения

**При входе в функцию CALL_FUNCTION распаковывает:**

```c
static PyObject *fast_function(PyFunctionObject *func, PyObject ***pp_stack,
                               Py_ssize_t nargs, PyObject *kwnames) {
    PyCodeObject *co = func->func_code;
    Py_ssize_t nposargs = co->co_argcount + co->co_kwonlyargcount;
    
    // Позиционные аргументы
    for (Py_ssize_t i = 0; i < co->co_argcount; i++) {
        PyObject *arg = args[i];
        PyFrame_SetLocal(frame, i, Py_NewRef(arg));  // args[0] -> locals[0]
    }
    
    // *args (если есть)
    if (PyCode_GET_FLAGS(co) & CO_VARARGS) {
        Py_ssize_t nargs_var = nargs - co->co_argcount;
        PyObject *args_tuple = PyTuple_New(nargs_var);
        for (Py_ssize_t i = 0; i < nargs_var; i++) {
            PyTuple_SET_ITEM(args_tuple, i, Py_NewRef(args[co->co_argcount + i]));
        }
        PyFrame_SetLocal(frame, co->co_argcount, args_tuple);
    }
    
    // **kwargs (если есть)
    if (PyCode_GET_FLAGS(co) & CO_VARKEYWORDS) {
        PyObject *kwargs_dict = PyDict_New();
        // Распаковываем kwnames в kwargs_dict
        PyFrame_SetLocal(frame, func->func_kwargs_slot, kwargs_dict);
    }
    
    return NULL;  // Успех
}
```

При входе в `def f(a, *args, **kwargs):` интерпретатор кладёт `a=args[0]`,
`*args=tuple(args[1:])`, `**kwargs=dict(kw)` в локальные переменные фрейма.

[Содержание](/CONTENTS.md#содержание)

## 6. PyFrameObject: хранение args/kwargs

```c
typedef struct _PyFrameObject {
    PyObject_HEAD
    struct _PyInterpreterFrame *f_frame;  // Текущий фрейм
    PyCodeObject *f_code;                 // PyCodeObject
    PyObject *f_globals;                  // globals()
    PyObject *f_builtins;                 // builtins()
    PyObject *f_locals;                   // locals() или NULL
    PyObject **f_localsplus;              // Локальные + fast locals
    // ...
} PyFrameObject;
```

`f_localsplus[]` — массив всех локальных переменных. `f_localsplus[0]="a"`,
`f_localsplus[1]="args"`, `f_localsplus[2]="b"`, `f_localsplus[3]="kwargs"`.

[Содержание](/CONTENTS.md#содержание)

## 7. Вызов с распаковкой: f(*args, **kwargs)

**Python:** `f(*lst, **dct)`

```assembler
# Байткод:
LOAD_GLOBAL f
LOAD_FAST lst
UNPACK_EX 0  # Распаковка *args
LOAD_FAST dct
UNPACK_EX 1  # Распаковка **kwargs (1 kwarg)
CALL_FUNCTION_KW 3 1  # 3 позиционных + 1 именованный
```

**CALL_FUNCTION_KW обрабатывает kwnames:**

```c
case CALL_FUNCTION_KW: {
    PyObject *kwnames = POP();     // Кортеж именованных аргументов
    Py_ssize_t na = oparg >> 1;    // Позиционные аргументы
    Py_ssize_t nk = oparg & 0xff;  // Именованные аргументы
    
    // Снимаем аргументы
    PyObject **args = PyMem_Malloc((na + 1) * sizeof(PyObject *));
    for (Py_ssize_t i = 0; i < na; i++) {
        args[i] = POP();
    }
    args[na] = NULL;
    
    // Вызываем с именованными аргументами
    PyObject *result = _PyObject_Vectorcall(callable, args, na, kwnames);
    PyMem_Free(args);
    Py_DECREF(kwnames);
    PUSH(result);
}
```

`*lst` распаковывается в позиционные аргументы, `**dct` → кортеж имен (
`kwnames=("key1", "key2")`) + значения. Передаются в vectorcall.

`*args`/**kwargs** в CPython — это флаги `CO_VARARGS`/`CO_VARKEYWORDS` в `PyCodeObject`, распаковка в `f_localsplus[]`
фрейма через `fast_function`, поддержка через байткоды `CALL_FUNCTION_KW` и vectorcall протокол.

[Содержание](/CONTENTS.md#содержание)