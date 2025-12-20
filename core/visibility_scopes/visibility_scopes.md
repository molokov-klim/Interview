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