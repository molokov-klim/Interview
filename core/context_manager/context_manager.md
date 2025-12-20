# *Контекстные менеджеры*

[Содержание](/CONTENTS.md#содержание)

## **Junior Level**

Контекстные менеджеры в Python — это специальные объекты, которые позволяют управлять ресурсами и выполнять настройку и
очистку до и после выполнения блока кода. Они используются с оператором `with`, который гарантирует правильное
приобретение и освобождение ресурсов, даже если в блоке кода произошла ошибка. Это делает их идеальными для работы с
файлами, сетевыми соединениями, транзакциями баз данных и блокировками.
Основная цель контекстного менеджера — безопасное управление ресурсами, требующими явного закрытия или очистки.
Классический пример — работа с файлами:

```python
with open('file.txt') as f:
    data = f.read()
# Файл гарантированно закрыт, даже если при чтении возникло исключение
```

Без использования `with` пришлось бы оборачивать операции в `try...finally`:

```python
f = open('file.txt')
try:
    data = f.read()
finally:
    f.close()
```

Контекстный менеджер инкапсулирует эту логику, делая код чище и безопаснее.

[Содержание](/CONTENTS.md#содержание)

## **Middle Level**

### 1. **Протокол контекстного менеджера**

Любой объект становится контекстным менеджером, если реализует два специальных метода:

- `__enter__(self)` — вызывается при входе в блок `with`. Возвращаемое значение присваивается переменной после `as`.
- `__exit__(self, exc_type, exc_value, traceback)` — вызывается при выходе из блока `with`. Получает информацию об
  исключении (аргументы будут `None`, если исключения не было). Если метод возвращает `True`, исключение считается
  обработанным и не пробрасывается дальше.

### 2. **Способы создания**

**Через класс:**

```python
class MyContextManager:
    def __enter__(self):
        print("Выделение ресурса")
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        print("Освобождение ресурса")
        # Если вернуть True, исключение будет подавлено
        return False


with MyContextManager() as cm:
    print("Работа внутри контекста")
```

**С помощью `contextlib.contextmanager` и генератора:**

```python
from contextlib import contextmanager


@contextmanager
def my_context():
    print("Выделение ресурса")
    yield "ресурс"  # значение для as
    print("Освобождение ресурса")


with my_context() as value:
    print(f"Работа с {value}")
```

Генератор должен содержать ровно один `yield`. Код до `yield` выполняется как `__enter__`, после — как `__exit__`.

### 3. **Готовые менеджеры из `contextlib`**

- `closing(thing)` — гарантирует вызов `thing.close()`.
- `suppress(*exceptions)` — подавляет указанные исключения в блоке.
- `nullcontext(enter_result)` — полезен для подстановки заглушки в тестах.
- `ExitStack` — для управления динамическим набором контекстов.

### 4. **Вложенность и группировка**

Контекстные менеджеры можно использовать группами:

```python
with open('a.txt') as f1, open('b.txt', 'w') as f2:
    f2.write(f1.read())
```

Порядок выхода из контекстов обратен порядку входа (LIFO). Исключение в `__enter__` приведет к тому, что `__exit__` не
будет вызван для этого менеджера, но уже вошедшие менеджеры получат вызов `__exit__`.

### 5. **Обработка исключений — детали**

Метод `__exit__` получает три аргумента об исключении:

- `exc_type`: класс исключения.
- `exc_value`: экземпляр исключения.
- `traceback`: объект трассировки.
  Если исключения не было, все они равны `None`. Возврат `True` подавляет исключение. Исключение, возникшее *внутри*
  `__exit__`, заменяет исходное (если оно было).

[Содержание](/CONTENTS.md#содержание)

## **Senior Level**

В CPython 3.9+ **контекстные менеджеры** реализуются через байткод `WITH_EXCEPT_START`/`GET_AITER`/`GET_ANEXT`/
`BEFORE_WITH` и специальные структуры `_PyWithCtxManager`/`_PyWithCtxEnterStar`. Вызов `__enter__`/`__exit__` через
`_Py_SpecialMethods[SPECIAL___ENTER__]`. [Python/ceval.c][Include/internal/pycore_ceval.h]

## 1. Байткод WITH_EXCEPT_START (Python 3.11+)

```python
# with ctx as var:
#     BODY
```

```
# Байткод (Python 3.12):
  0 LOAD_NAME          0 (ctx)         # ctx на стек
  2 BEFORE_WITH       0                # Подготавливаем with
  4 STORE_FAST        0 (var)          # var = ctx.__enter__()
  6 SETUP_WITH        0                # SETUP_FINALLY для __exit__
  8 <BODY инструкции>
 12 PUSH_EXC_INFO                   # Сохраняем исключение
 14 WITH_EXCEPT_START         16     # Вызываем __exit__(exc_info)
 16 LOAD_FAST        1 (__exit__)     # __exit__ метод
 18 CALL_FUNCTION    3                # __exit__(exc_type, exc_val, tb)
 20 POP_TOP                        # Результат __exit__
 22 POP_EXCEPT                     # Восстанавливаем стек
 24 RETURN_VALUE
```

`BEFORE_WITH` вызывает `__enter__`, `STORE_FAST` сохраняет результат в `var`.
`WITH_EXCEPT_START` вызывает `__exit__(exc_type, exc_val, tb)` при выходе/исключении.

[Содержание](/CONTENTS.md#содержание)

## 2. BEFORE_WITH байткод (ceval.c)

```c
case BEFORE_WITH: {
    PyObject *a = TOP();               // ctx объект
    PyObject *enter = _PyObject_LookupSpecial(a, &_Py_ID(__enter__));  // Ищем __enter__
    
    if (enter == NULL) {
        if (!_PyErr_Occurred(tstate)) {
            _PyErr_Format(tstate, PyExc_AttributeError,
                "'%.50s' object has no attribute '__enter__'",
                Py_TYPE(a)->tp_name);
        }
        Py_DECREF(a);
        goto error;
    }
    
    PyObject *res = _PyObject_CallNoArgs(enter);  // Вызываем __enter__()
    Py_DECREF(enter);
    
    if (res == NULL) {
        Py_DECREF(a);
        goto error;
    }
    
    STACKADJ(-1);                      // Убираем ctx
    PEEK(0) = res;                     // Результат __enter__ на вершину
    JUMPTO(oparg);                     // Переходим к STORE_FAST var
    DISPATCH_SAME_OPARG(1);
}
```

`BEFORE_WITH` ищет `ctx.__enter__()` через `_PyObject_LookupSpecial` (быстрый поиск по MRO),
вызывает без аргументов, кладёт результат на стек вместо `ctx`.

[Содержание](/CONTENTS.md#содержание)

## 3. WITH_EXCEPT_START - вызов __exit__

```c
case WITH_EXCEPT_START: {
    PyObject *exc_info = POP();        // (exc_type, exc_val, tb) tuple
    PyObject *exit = TOP();            // __exit__ метод
    
    // Вызываем __exit__(type, value, traceback)
    PyObject *res = PyObject_Call(exit, exc_info, NULL);
    Py_DECREF(exit);
    Py_DECREF(exc_info);
    
    if (res == NULL) {
        goto error;
    }
    
    // __exit__ возвращает True? Подавляем исключение
    int suppress = PyObject_IsTrue(res);
    Py_DECREF(res);
    if (suppress < 0) {
        goto error;
    }
    
    if (suppress) {
        // Очищаем исключение
        _PyErr_Clear(tstate);
    }
    
    JUMPTO(oparg);                     // Продолжаем после PUSH_EXC_INFO
    DISPATCH_SAME_OPARG(1);
}
```

При исключении/выходе из `with` берём сохранённое `(exc_type, exc_val, tb)`, вызываем
`__exit__(exc_info)`, если возвращает `True` — **подавляем** исключение.

[Содержание](/CONTENTS.md#содержание)

## 4. SETUP_WITH (SETUP_FINALLY + PUSH_EXC_INFO)

```c
case SETUP_WITH: {
    PyObject *enter = PEEK(0);         // Результат __enter__
    PyObject *exc = PyTuple_New(3);    // (None, None, None)
    
    if (exc == NULL) {
        Py_DECREF(enter);
        goto error;
    }
    
    PyTuple_SET_ITEM(exc, 0, Py_NewRef(Py_None));      // exc_type=None
    PyTuple_SET_ITEM(exc, 1, Py_NewRef(Py_None));      // exc_val=None  
    PyTuple_SET_ITEM(exc, 2, Py_NewRef(Py_None));      // traceback=None
    
    // Ищем __exit__
    PyObject *exit_meth = _PyObject_LookupSpecial(enter, &_Py_ID(__exit__));
    if (exit_meth == NULL) {
        Py_DECREF(exc);
        Py_DECREF(enter);
        goto error;
    }
    
    STACK_GROW(2);                     // +2 слота на стеке
    PEEK(2) = exc;                     // exc_info tuple
    PEEK(1) = exit_meth;               // __exit__ метод
    JUMPTO(oparg);                     // К BODY
    DISPATCH_SAME_OPARG(1);
}
```

Создаёт пустой tuple `(None,None,None)` для будущего исключения, ищет `__exit__`, кладёт оба
на стек. `oparg` указывает offset до `PUSH_EXC_INFO`.

## 5. PUSH_EXC_INFO - захват исключения

```c
case PUSH_EXC_INFO: {
    PyObject *tb = NULL;
    PyObject *exc = _PyErr_GetRaisedException(tstate);  // Текущее исключение
    PyObject *exc_tuple = PyTuple_New(3);
    
    if (exc_tuple == NULL) {
        goto error;
    }
    
    // Заполняем tuple текущим исключением
    PyTuple_SET_ITEM(exc_tuple, 0, (PyObject *)Py_TYPE(exc));  // exc_type
    PyTuple_SET_ITEM(exc_tuple, 1, Py_NewRef(exc));            // exc_val
    PyTuple_SET_ITEM(exc_tuple, 2, Py_NewRef(tb));             // traceback
    
    _PyErr_SetRaisedException(tstate, NULL);  // Очищаем глобальное исключение
    
    STACK_GROW(1);
    PEEK(0) = exc_tuple;               // exc_info на стек
    JUMPTO(oparg);                     // К WITH_EXCEPT_START
}
```

**Объяснение людей:** При исключении в `with BODY` сохраняет `(TypeError, "msg", traceback)` в tuple, **очищает**
глобальное исключение, передаёт tuple в `__exit__`.

## 6. _PyObject_LookupSpecial - быстрый поиск __enter__/__exit__

```c
PyObject *
_PyObject_LookupSpecial(PyObject *obj, struct _Py_Identifier *id) {
    PyTypeObject *tp = Py_TYPE(obj);
    
    // Быстрый путь: кешированные слоты
    if (tp->tp_vectorcall_offset != 0) {
        // Уже проверяли
    }
    
    // Поиск по MRO через tp_getattro
    return PyObject_GenericGetAttr(obj, id->string);
}
```

Ищет `__enter__`/`__exit__` **только в типе**, не в `__dict__` экземпляра (быстрее).
Использует MRO + дескрипторный протокол.

## 7. contextlib.contextmanager (генераторный декоратор)

**Lib/contextlib.py использует yield:**

```python
@contextmanager
def managed_file(name):
    f = open(name, 'w')  # __enter__
    try:
        yield f  # with as var:
    finally:
        f.close()  # __exit__
```

**Преобразуется в:**

```python
class _GeneratorContextManager:
    def __init__(self, gen, args, kwds):
        self.gen = gen  # Генератор
        self.args = args
        self.kwds = kwds
        self._reinit()  # Состояние

    def __enter__(self):
        self._push_cm_exit(self.gen)  # Регистрируем __exit__
        return next(self.gen)  # yield значение

    def __exit__(self, *exc_info):
        self.gen.close()  # Генераторный cleanup
```

`@contextmanager` оборачивает генератор в класс с `__enter__` (next(gen)) и `__exit__` (
gen.close()). `yield f` становится значением `as var`.

## 8. file.__enter__/__exit__ (пример)

```c
static PyObject *
io_FileIO___enter__(PyFileIOObject *self) {
    if (self->fd < 0) {
        Py_RETURN_FALSE;           // Уже закрыт
    }
    Py_INCREF(self);
    return (PyObject *)self;       // Возвращаем self
}

static PyObject *
io_FileIO___exit__(PyFileIOObject *self, PyObject *args) {
    // args = (exc_type, exc_val, tb)
    (void)args;                    // Игнорируем исключение
    
    PyObject *closed = PyObject_CallMethodObjArgs(
        (PyObject *)self, &_Py_ID(close), NULL);
    
    if (closed == NULL) {
        return NULL;
    }
    Py_DECREF(closed);
    Py_RETURN_FALSE;               // Не подавляем исключение
}
```

`open('file.txt')` возвращает PyFileIOObject. `__enter__` просто +refcnt, `__exit__` вызывает
`f.close()`.

**Контекстные менеджеры** в CPython 3.9+ — байткоды `BEFORE_WITH` (вызов `__enter__`), `WITH_EXCEPT_START` (вызов
`__exit__(exc_info)`), структуры для сохранения `exc_info` tuple, быстрый поиск через `_PyObject_LookupSpecial`.

- [Содержание](/CONTENTS.md#содержание)
