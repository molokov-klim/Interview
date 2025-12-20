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
