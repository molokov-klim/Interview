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