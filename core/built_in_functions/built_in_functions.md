# *Встроенные функции*

[Содержание](/CONTENTS.md#содержание)

## **Junior Level**

Встроенные функции — это базовый набор функций Python, доступных без импорта, так как они находятся в автоматически
загружаемом модуле `builtins`. Они охватывают основные операции языка: преобразование типов, математические вычисления,
работу с коллекциями, ввод-вывод и интроспекцию. Будучи частью ядра языка, эти функции реализованы максимально
эффективно и имеют стандартизированное поведение.

Эти функции охватывают основные операции: работу с типами данных (`str()`, `int()`, `list()`), математические
вычисления (`abs()`, `round()`, `sum()`), преобразования (`len()`, `sorted()`, `reversed()`), ввод-вывод (`print()`,
`input()`), итерации (`range()`, `enumerate()`, `zip()`), проверки (`isinstance()`, `hasattr()`), и другие
фундаментальные операции.

Важно понимать, что это не просто функции, а часть ядра языка. Они реализованы максимально эффективно и их поведение
стандартизировано.

[Built in functions how to use](how_to_use.md#built-in-functions-how-to-use)

[Содержание](/CONTENTS.md#содержание)

## **Middle Level**

1. **Пространство имен `builtins`**: Все встроенные функции находятся в модуле `builtins`, который автоматически
   импортируется при запуске интерпретатора. Можно получить прямой доступ через `import builtins`. Переопределение
   функций в этом модуле (что крайне не рекомендуется) повлияет на всю программу.

2. **Категории встроенных функций**:

**Конструкторы типов (приведение и создание объектов):**

* `int()` (создает целое число из числа или строки)
* `float()` (создает число с плавающей точкой)
* `complex()` (создает комплексное число)
* `str()` (создает строковое представление объекта)
* `bytes()` (создает неизменяемую байтовую последовательность)
* `bytearray()` (создает изменяемую байтовую последовательность)
* `memoryview()` (создает "представление памяти" объекта для эффективного доступа без копирования)
* `bool()` (возвращает логическое значение объекта)
* `list()` (создает список)
* `tuple()` (создает кортеж)
* `range()` (создает неизменяемую последовательность чисел)
* `dict()` (создает словарь)
* `set()` (создает изменяемое множество)
* `frozenset()` (создает неизменяемое множество)
* `object()` (создает новый базовый объект — корень иерархии классов)

**Математические операции и числа:**

* `abs()` (возвращает абсолютное значение числа)
* `pow(x, y)` (возводит x в степень y, эквивалентно `x**y`)
* `divmod(a, b)` (возвращает частное и остаток от деления a на b как кортеж)
* `round()` (округляет число до заданной точности)
* `sum()` (суммирует элементы итерируемого объекта)
* `min()` (находит наименьший элемент)
* `max()` (находит наибольший элемент)
* `hex()` (преобразует целое число в шестнадцатеричную строку)
* `oct()` (преобразует целое число в восьмеричную строку)
* `bin()` (преобразует целое число в двоичную строку)

**Преобразования и проверки типов:**

* `ascii()` (возвращает строку, содержащую только ASCII-символы, не-ASCII экранируются)
* `repr()` (возвращает официальное строковое представление объекта, часто пригодное для `eval()`)
* `format(value, spec)` (форматирует значение по спецификации)
* `ord()` (возвращает код Unicode для заданного символа)
* `chr()` (возвращает символ Unicode по его коду)
* `hash()` (возвращает хеш-значение объекта)
* `type()` (возвращает тип объекта или создает новый класс)
* `isinstance()` (проверяет, является ли объект экземпляром класса или кортежа классов)
* `issubclass()` (проверяет, является ли класс подклассом другого класса)
* `callable()` (проверяет, можно ли вызвать объект как функцию)
* `len()` (возвращает длину (количество элементов) объекта)

**Работа с последовательностями и итерируемыми объектами:**

* `sorted()` (возвращает новый отсортированный список из итерируемого объекта)
* `reversed()` (возвращает обратный итератор)
* `enumerate()` (возвращает итератор, генерирующий пары (индекс, элемент))
* `zip()` (комбинирует элементы нескольких итераций в кортежи)
* `filter(func, iterable)` (фильтрует элементы, оставляя только те, для которых `func` возвращает `True`)
* `map(func, iterable)` (применяет функцию к каждому элементу итерируемого объекта)
* `all()` (возвращает `True`, если все элементы итерируемого объекта истинны)
* `any()` (возвращает `True`, если хотя бы один элемент итерируемого объекта истинен)
* `slice()` (создает объект среза для извлечения части последовательности)

**Итераторы и генераторы:**

* `iter()` (возвращает итератор для объекта)
* `next()` (возвращает следующий элемент итератора)

**Ввод-вывод и операции с файлами:**

* `print()` (выводит объекты в текстовый поток, обычно на экран)
* `input()` (считывает строку из стандартного ввода)
* `open()` (открывает файл и возвращает файловый объект)

**Компиляция и выполнение кода:**

* `eval()` (выполняет строку с кодом Python и возвращает результат)
* `exec()` (выполняет динамически созданный код Python)
* `compile()` (компилирует исходный код в объект кода или AST)

**Интроспекция и работа с атрибутами (Reflection):**

* `dir()` (возвращает список имен в текущей локальной области видимости или атрибутов объекта)
* `vars()` (возвращает словарь `__dict__` объекта или локальной области)
* `globals()` (возвращает словарь текущей глобальной области видимости)
* `locals()` (возвращает словарь текущей локальной области видимости)
* `getattr()` (возвращает значение атрибута объекта по его имени)
* `setattr()` (устанавливает значение атрибута объекта)
* `delattr()` (удаляет атрибут объекта)
* `hasattr()` (проверяет наличие атрибута у объекта)
* `id()` (возвращает "идентификатор" объекта — его уникальный адрес в памяти)

**Работа с классами и объектно-ориентированное программирование:**

* `property()` (создает свойство (property) — управляемый атрибут с геттером/сеттером)
* `classmethod()` (преобразует метод в метод класса (принимает `cls` вместо `self`))
* `staticmethod()` (преобразует метод в статический метод (не принимает `self` или `cls`))
* `super()` (возвращает прокси-объект, который делегирует вызовы методов родительскому классу)

**Разное и системные функции:**

* `__import__()` (низкоуровневая функция, которая реализует оператор `import`)
* `breakpoint()` (вызывает отладчик (по умолчанию pdb) в месте вызова)
* `help()` (запускает встроенную интерактивную справочную систему)
* `memoryview()` (см. конструкторы)
* `hash()` (см. преобразования и проверки)

3. **Особенности поведения**:

* `sorted()` всегда возвращает новый список, тогда как метод `list.sort()` изменяет список на месте
* `reversed()` возвращает итератор, а не список
* `map()` и `filter()` в Python 3 возвращают итераторы, а не списки (как было в Python 2)
* `range()` тоже возвращает специальный объект, а не список
* `open()` является фабрикой, возвращающей файловый объект с разным поведением в зависимости от режима

4. **Функции высшего порядка**: `map()`, `filter()`, `sorted()` принимают функции в качестве аргументов. Это делает их
   мощным инструментом для функционального программирования.

[Содержание](/CONTENTS.md#содержание)

## **Senior Level**

В CPython 3.9+ **встроенные функции** реализуются через **PyCFunctionObject**/**PyCMethodObject** в модуле
`bltinmodule.c`, регистрируемые в `__builtins__` через `builtin_functions[]`. Поддерживают **vectorcall** (
METH_FASTCALL) и классические **METH_VARARGS**. `Python/bltinmodule.c`,`Objects/methodobject.c`

[Содержание](/CONTENTS.md#содержание)

## 1. PyCFunctionObject - структура встроенной функции

```c
typedef struct {
    PyObject_HEAD              // Стандартный заголовок (refcnt + type)
    PyMethodDef *m_ml;         // Указатель на PyMethodDef (имя + C-функция)
    PyObject *m_self;          // self (NULL для функций, класс для методов)
    PyObject *m_module;        // Модуль (__module__)
    vectorcallfunc vectorcall; // Быстрый вызов (3.9+)
    PyObject *m_weakreflist;   // Список слабых ссылок
} PyCFunctionObject;
```

Встроенная функция `len()` в памяти — это 64-байт структура: заголовок + указатель на
C-функцию `len_func` + `__module__="builtins"`. `vectorcall` — быстрый путь вызова без tuple.

[Содержание](/CONTENTS.md#содержание)

## 2. PyMethodDef - таблица встроенных функций

```c
static PyMethodDef builtin_functions[] = {
    {"abs",         builtin_abs,        METH_VARARGS, abs_doc},
    {"aiter",       builtin_aiter,      METH_O, aiter_doc},
    {"all",         builtin_all,        METH_O, all_doc},
    {"any",         builtin_any,        METH_O, any_doc},
    {"ascii",       builtin_ascii,      METH_O, ascii_doc},
    {"bin",         builtin_bin,        METH_O, bin_doc},
    {"bool",        builtin_bool,       METH_VARARGS, bool_doc},
    {"bytearray",   builtin_bytearray,  METH_VARARGS|METH_KEYWORDS, bytearray_doc},
    {"bytes",       builtin_bytes,      METH_VARARGS|METH_KEYWORDS, bytes_doc},
    {"callable",    builtin_callable,   METH_O, callable_doc},
    {"chr",         builtin_chr,        METH_O, chr_doc},
    {"classmethod", builtin_classmethod,METH_O, classmethod_doc},
    {"compile",     builtin_compile,    METH_VARARGS|METH_KEYWORDS, compile_doc},
    {"complex",     builtin_complex,    METH_VARARGS|METH_KEYWORDS, complex_doc},
    {"delattr",     builtin_delattr,    METH_VARARGS, delattr_doc},
    {"dict",        builtin_dict,       METH_VARARGS|METH_KEYWORDS, dict_doc},
    {"dir",         builtin_dir,        METH_VARARGS, dir_doc},
    {"divmod",      builtin_divmod,     METH_VARARGS, divmod_doc},
    {"enumerate",   builtin_enumerate,  METH_VARARGS|METH_KEYWORDS, enumerate_doc},
    {"eval",        builtin_eval,       METH_VARARGS|METH_KEYWORDS, eval_doc},
    {"exec",        builtin_exec,       METH_VARARGS|METH_KEYWORDS, exec_doc},
    {"filter",      builtin_filter,     METH_VARARGS, filter_doc},
    {"float",       builtin_float,      METH_VARARGS|METH_KEYWORDS, float_doc},
    {"format",      builtin_format,     METH_VARARGS, format_doc},
    {"frozenset",   builtin_frozenset,  METH_VARARGS|METH_KEYWORDS, frozenset_doc},
    {"getattr",     builtin_getattr,    METH_VARARGS, getattr_doc},
    {"globals",     builtin_globals,    METH_NOARGS, globals_doc},
    {"hasattr",     builtin_hasattr,    METH_VARARGS, hasattr_doc},
    {"hash",        builtin_hash,       METH_O, hash_doc},
    {"hex",         builtin_hex,        METH_O, hex_doc},
    {"id",          builtin_id,         METH_O, id_doc},
    {"input",       builtin_input,      METH_VARARGS, input_doc},
    {"int",         builtin_int,        METH_VARARGS|METH_KEYWORDS, int_doc},
    {"isinstance",  builtin_isinstance, METH_VARARGS, isinstance_doc},
    {"issubclass",  builtin_issubclass, METH_VARARGS, issubclass_doc},
    {"iter",        builtin_iter,       METH_VARARGS, iter_doc},
    {"len",         builtin_len,        METH_O, len_doc},
    {"list",        builtin_list,       METH_VARARGS, list_doc},
    {"locals",      builtin_locals,     METH_NOARGS, locals_doc},
    {"map",         builtin_map,        METH_VARARGS, map_doc},
    {"max",         builtin_max,        METH_VARARGS|METH_KEYWORDS, max_doc},
    {"memoryview",  builtin_memoryview, METH_O, memoryview_doc},
    {"min",         builtin_min,        METH_VARARGS|METH_KEYWORDS, min_doc},
    {"next",        builtin_next,       METH_VARARGS|METH_KEYWORDS, next_doc},
    {"object",      builtin_object,     METH_VARARGS|METH_KEYWORDS, object_doc},
    {"oct",         builtin_oct,        METH_O, oct_doc},
    {"open",        builtin_open,       METH_VARARGS|METH_KEYWORDS, open_doc},
    {"ord",         builtin_ord,        METH_O, ord_doc},
    {"pow",         builtin_pow,        METH_VARARGS|METH_KEYWORDS, pow_doc},
    {"print",       builtin_print,      METH_VARARGS|METH_KEYWORDS, print_doc},
    {"property",    builtin_property,   METH_VARARGS|METH_KEYWORDS, property_doc},
    {"range",       builtin_range,      METH_VARARGS|METH_KEYWORDS, range_doc},
    {"repr",        builtin_repr,       METH_O, repr_doc},
    {"reversed",    builtin_reversed,   METH_O, reversed_doc},
    {"round",       builtin_round,      METH_VARARGS|METH_KEYWORDS, round_doc},
    {"set",         builtin_set,        METH_VARARGS|METH_KEYWORDS, set_doc},
    {"setattr",     builtin_setattr,    METH_VARARGS, setattr_doc},
    {"slice",       builtin_slice,      METH_VARARGS|METH_KEYWORDS, slice_doc},
    {"sorted",      builtin_sorted,     METH_VARARGS|METH_KEYWORDS, sorted_doc},
    {"staticmethod",builtin_staticmethod,METH_O, staticmethod_doc},
    {"str",         builtin_str,        METH_VARARGS, str_doc},
    {"sum",         builtin_sum,        METH_VARARGS|METH_KEYWORDS, sum_doc},
    {"super",       builtin_super,      METH_VARARGS|METH_KEYWORDS, super_doc},
    {"tuple",       builtin_tuple,      METH_VARARGS, tuple_doc},
    {"type",        builtin_type,       METH_VARARGS|METH_KEYWORDS, type_doc},
    {"vars",        builtin_vars,       METH_VARARGS, vars_doc},
    {"zip",         builtin_zip,        METH_VARARGS, zip_doc},
    {NULL,          NULL}              /* Sentinel */
};
```

Это **таблица методов** — массив структур `{имя, C_функция, флаги, docstring}`. `builtin_len`
принимает METH_O (1 аргумент). Регистрируется в `__builtins__`.

[Содержание](/CONTENTS.md#содержание)

## 3. Регистрация в bltinmodule.c: builtinmodule_exec

```c
static struct PyModuleDef builtindef = {
    PyModuleDef_HEAD_INIT,
    .m_name = "builtins",
    .m_doc = builtin_module_doc,
    .m_size = 0,
    .m_methods = builtin_functions,  // <- Таблица выше
};

PyMODINIT_FUNC
PyInit_builtins(void) {
    PyObject *m, *d;
    
    // Создаём модуль builtins
    m = PyModule_Create(&builtindef);
    if (m == NULL)
        return NULL;
    
    d = PyModule_GetDict(m);           // __dict__ модуля
    
    // Добавляем все builtin функции
    if (builtin_add_funcs(d, builtin_functions) < 0) {
        Py_DECREF(m);
        return NULL;
    }
    
    // Добавляем константы
    if (builtin_add_constants(d) < 0) {
        Py_DECREF(m);
        return NULL;
    }
    
    return m;
}

static int builtin_add_funcs(PyObject *mod_dict, PyMethodDef *functions) {
    PyMethodDef *ml;
    
    for (ml = functions; ml->ml_name != NULL; ml++) {
        PyObject *descr;
        descr = PyCFunction_NewEx(ml, NULL, mod_dict);  // Создаём PyCFunctionObject
        if (descr == NULL) {
            return -1;
        }
        if (PyDict_SetItemString(mod_dict, ml->ml_name, descr) < 0) {
            Py_DECREF(descr);
            return -1;
        }
        Py_DECREF(descr);
    }
    return 0;
}
```

`PyInit_builtins()` создаёт модуль `builtins`, проходит по таблице `builtin_functions[]`,
вызывает `PyCFunction_NewEx` (создаёт PyCFunctionObject для каждой функции), кладёт в `__dict__` модуля как `len`,
`print`, `abs`.

[Содержание](/CONTENTS.md#содержание)

## 4. PyCFunction_NewEx - создание PyCFunctionObject

```c
PyObject *PyCFunction_NewEx(PyMethodDef *ml, PyObject *self, PyObject *module) {
    PyCFunctionObject *op;
    
    // Выделяем память
    op = PyObject_GC_New(PyCFunctionObject, &PyCFunction_Type);
    if (op == NULL)
        return NULL;
    
    // Заполняем поля
    op->m_ml = ml;                    // Ссылка на PyMethodDef
    Py_XSETREF(op->m_self, Py_XNewRef(self));  // self (NULL для функций)
    Py_XSETREF(op->m_module, Py_XNewRef(module));  // "builtins"
    
    // Выбираем vectorcall функцию по флагам
    op->vectorcall = _PyCFunction_VectorcallByFlags(ml->ml_flags);
    
    _PyObject_GC_TRACK(op);           // Регистрируем в GC
    return (PyObject *)op;
}
```

**Объяснение людей:** Для `len` создаётся PyCFunctionObject: `m_ml=&builtin_len_def`, `m_self=NULL`,
`m_module="builtins"`, `vectorcall=cfunction_vectorcall_FASTCALL`.

[Содержание](/CONTENTS.md#содержание)

## 5. Вызов len(obj): vectorcall путь (3.9+)

```c
static PyObject *cfunction_vectorcall_FASTCALL(
    PyObject *func, PyObject *const *args, size_t nargsf, PyObject *kwnames) {
    PyCFunctionObject *self = _PyCFunctionObject_CAST(func);
    PyMethodDef *ml = self->m_ml;
    Py_ssize_t nargs = PyVectorcall_NARGS(nargsf);
    
    assert(nargs == 1);                // METH_O: 1 аргумент
    assert(kwnames == NULL);           // Без kwargs
    
    // Вызываем C-функцию напрямую
    PyCFunction meth = PyCFunction_GET_FUNCTION(self);
    PyObject *self_or_module = PyCFunction_GET_SELF(self);
    
    PyObject *result = meth(self_or_module, args[0]);  // len(NULL, obj)
    
    if (result != NULL && !(ml->ml_flags & METH_COEXIST)) {
        /* steal reference to result */
        Py_DECREF(result);
        PyErr_Format(PyExc_TypeError,
                     "%.200s() takes no arguments (1 given)",
                     ml->ml_name);
        return NULL;
    }
    
    return result;
}
```

`len(obj)` → `cfunction_vectorcall_FASTCALL` → `builtin_len(NULL, obj)` → C-функция получает
`self=NULL`, `arg=obj`. Без создания tuple/list — **максимальная скорость**.

[Содержание](/CONTENTS.md#содержание)

## 6. builtin_len - реализация len()

```c
static PyObject *builtin_len(PyObject *self, PyObject *obj) {
    Py_ssize_t res;
    
    // Пытаемся взять tp_as_sequence->sq_length
    res = PyObject_Length(obj);
    if (res < 0 && PyErr_Occurred()) {
        return NULL;
    }
    
    return PyLong_FromSsize_t(res);
}

Py_ssize_t PyObject_Length(PyObject *o) {
    PySequenceMethods *m;
    
    if (o == NULL) {
        PyErr_BadInternalCall();
        return -1;
    }
    
    m = o->ob_type->tp_as_sequence;
    if (m && m->sq_length) {
        Py_ssize_t len = m->sq_length(o);  // Вызываем sq_length (list_length)
        if (len < 0 && !_PyErr_ExceptionMatches(PyExc_TypeError)) {
            return len;
        }
        // Если TypeError - пробуем mapping
        if (len < 0) {
            PyObject *r = PyObject_CallMethod(o, "__len__", NULL);
            if (r == NULL)
                return -1;
            len = PyLong_AsSsize_t(r);
            Py_DECREF(r);
        }
        return len;
    }
    
    return PyMapping_Size(o);          // dict.__len__
}
```

`len(lst)` → `PyObject_Length(lst)` → `PyList_Type.tp_as_sequence->sq_length(lst)` →
`list_length()` возвращает `self->ob_size`. Если нет `sq_length` — пробует `__len__()`.

[Содержание](/CONTENTS.md#содержание)

## 7. list_length - sq_length для PyListObject

```c
static Py_ssize_t list_length(PyListObject *self) {
    return Py_SIZE(self);              // Просто ob_size из PyVarObject
}
```

`len([1,2,3])` возвращает `ob_size=3` из заголовка PyVarObject. **Мгновенно** — поле в самом
объекте.

[Содержание](/CONTENTS.md#содержание)

## 8. Байткод: LOAD_GLOBAL -> builtins.len

```
# len(lst)
  0 LOAD_GLOBAL         0 (len)       # Ищем len в globals -> builtins
  2 LOAD_FAST           0 (lst)       # lst на стек
  4 CALL                1             # len(lst)
  6 RETURN_VALUE                    # Возвращаем результат
```

**LOAD_GLOBAL в ceval.c (3.9+):**

```c
case LOAD_GLOBAL: {
    PyObject *name = PyCode_GET_GLOBAL(frame->f_code, oparg>>1);
    
    // Поиск: locals -> globals -> builtins
    PyObject *res = _PyDict_GetItemWithCache(
        frame->f_globals, name, (hash_t)arg & 0xfffff);
    
    if (res == NULL) {
        // Не в globals -> ищем в builtins
        res = _PyDict_GetItemWithCache(
            frame->f_builtins, name, (hash_t)arg & 0xfffff);
    }
    
    if (res == NULL) {
        // NameError
        format_exc_check_arg(PyExc_NameError,
            MODULE_FUNC_STR "name '%U' is not defined", name);
        goto error;
    }
    
    Py_INCREF(res);                    // +refcnt
    STACK_GROW(1);                     // Увеличиваем стек
    PEEK(0) = res;                     // len() на вершину стека
    DISPATCH();
}
```

`LOAD_GLOBAL len` ищет имя `"len"` сначала в `globals()`, потом в `builtins`. Находит
PyCFunctionObject, увеличивает refcnt, кладёт на стек.

[Содержание](/CONTENTS.md#содержание)

## 9. Vectorcall диспетчеризация (3.9+)

```c
PyObject *_PyObject_Vectorcall(PyObject *callable,
                               PyObject *const *args,
                               size_t nargsf, PyObject *kwnames) {
    if (PyCFunction_Check(callable)) {
        // Быстрый путь для PyCFunctionObject
        return _PyCFunction_Vectorcall(callable, args, nargsf, kwnames);
    }
    
    // Медленный путь через tp_call
    vectorcallfunc func = _PyObject_GetVectorcall(callable);
    if (func != NULL) {
        return func(callable, args, nargsf, kwnames);
    }
    
    // Fallback на PyObject_Call
    return PyObject_Call(callable, args[0], NULL);
}
```

`CALL 1` → `_PyObject_Vectorcall(len_obj, [lst], 1, NULL)` → `_PyCFunction_Vectorcall` →
`builtin_len(NULL, lst)`. **Без tuple создания**.

**Встроенные функции** в CPython 3.9+ — **PyCFunctionObject** из `bltinmodule.c`, **vectorcall** (METH_FASTCALL) для
скорости, **PyMethodDef таблица**, поиск через `LOAD_GLOBAL` → `globals/builtins`, диспетч `PyObject_Length` →
`sq_length`.

- [Содержание](/CONTENTS.md#содержание)