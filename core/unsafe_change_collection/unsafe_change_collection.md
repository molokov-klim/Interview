# **Изменение коллекции во время итерации**

[Содержание](/CONTENTS.md#содержание)

## **Junior Level**

Изменять коллекцию во время перебора её элементов — это как перестраивать комнату, пока вы в ней находитесь. Вы можете
споткнуться о перемещённую мебель или вовсе оказаться в совершенно другом пространстве. В программировании эта операция
нарушает внутреннюю логику работы итератора и ведёт к непредсказуемым последствиям: пропуску элементов, их двойной
обработке, бесконечным циклам или ошибкам выполнения.

Представьте, что вы экскурсовод, ведущий группу по постоянно меняющемуся музею. Если залы начинают исчезать или
появляться во время экскурсии, ваш маршрут и рассказ мгновенно теряют смысл. Так же и итератор, который хранит текущую
позицию в коллекции, перестаёт корректно работать, когда основание под ним сдвигается.

Это правило касается всех изменяемых коллекций — списков, словарей, множеств. Для безопасной модификации нужно либо
итерироваться по копии коллекции, либо сначала собрать все необходимые изменения, а затем применить их к оригиналу.

[Содержание](/CONTENTS.md#содержание)

## **Middle Level**

Когда вы создаёте цикл `for item in collection:`, Python создаёт объект-итератор, который становится проводником по
вашей коллекции. Этот проводник запоминает текущее положение и следует определённому маршруту. Изменение коллекции во
время такого «путешествия» сбивает все ориентиры.

Механизм итерации устроен так, что итератор хранит внутреннее состояние — текущую позицию. Для списков это индекс
элемента, для словарей и множеств — более сложные структуры, отслеживающие хеш-таблицы. При удалении или добавлении
элементов исходная коллекция меняет свою организацию, но итератор продолжает следовать старому плану, что приводит к
логическим противоречиям.

Интересно, что разные коллекции в Python реагируют на такие изменения по-разному. Словари и множества, начиная с Python
3.7, при обнаружении изменения размера во время итерации вызывают явное исключение `RuntimeError`, предупреждая
программиста о проблеме. Списки же, в силу своей индексной природы, позволяют это делать, но последствия могут быть
особенно коварными — код может работать с ошибками, которые сложно воспроизвести и отладить.

Для разных коллекций существуют свои безопасные паттерны. Со списками часто работает итерация по копии, созданной через
`list()` или срез `[:]`. Для словарей можно итерироваться по списку ключей, предварительно полученных через
`list(dict.keys())`. Множества также требуют создания копии перед модификацией во время итерации.

Универсальный подход — разделить фазу анализа коллекции и фазу её изменения. Сначала соберите всю необходимую
информацию (например, какие элементы нужно удалить или добавить), сохранив её во временной структуре, а затем отдельным
действием примените все изменения к исходной коллекции. Этот метод не только безопасен, но и делает код более понятным,
поскольку чётко разделяет ответственность между этапами обработки данных.

[Содержание](/CONTENTS.md#содержание)

## **Senior Level**

В CPython 3.9+ **изменение коллекции во время итерации** детектируется через **version tag** (`ma_version_tag` в
PyDictObject, `ob_version` в PyListObject) + **итераторное состояние** (`it_version`/`it_index`). **RuntimeError** при
несоответствии версий. `Objects/listobject.c`,`Objects/dictobject.c`,`Objects/iterobject.c`

## 1. PyListIterObject с версией (list_iter)

```c
typedef struct {
    PyObject_HEAD
    Py_ssize_t it_index;           // Текущий индекс
    PyListObject *it_seq;          // Ссылка на список
    uint32_t it_version;           // Копия ob_version списка при создании
} PyListIterObject;
```

Итератор списка копирует `list->ob_version` при создании. Каждый `next()` проверяет версии —
если список изменился → **RuntimeError**.

## 2. list_iter_next() - проверка версии

```c
static PyObject *list_iter_next(PyListIterObject *it) {
    PyListObject *list = it->it_seq;   // Ссылка на список
    Py_ssize_t index = it->it_index;   // Текущий индекс
    
    // КРИТИЧЕСКАЯ ПРОВЕРКА ВЕРСИИ
    if (it->it_version != list->ob_version) {
        PyErr_SetString(PyExc_RuntimeError, 
            "list changed during iteration");
        return NULL;                   // RuntimeError!
    }
    
    if (index >= Py_SIZE(list)) {
        return NULL;                   // Конец -> StopIteration
    }
    
    PyObject *item = Py_NewRef(list->ob_item[index]);
    it->it_index++;                    // ++ индекс
    return item;
}
```

Перед чтением элемента сравниваем `it_version` (копия при создании) с `list->ob_version` (
текущее). Изменился → **мгновенный RuntimeError**.

## 3. list_ob_version - счётчик изменений PyListObject

```c
uint32_t list_ob_version(PyListObject *self) {
    return self->ob_version;           // 32-битный счётчик изменений
}

static Py_ssize_t list_length(PyListObject *self) {
    return Py_SIZE(self);
}

static int list_ass_slice(PyListObject *self, Py_ssize_t low, Py_ssize_t high, PyObject *v) {
    // ... логика присваивания ...
    
    // КРИТИЧЕСКИ: увеличиваем версию после изменения
    self->ob_version++;                // ++ общий счётчик изменений
    return 0;
}

static int list_append(PyListObject *self, PyObject *v) {
    // ... логика добавления ...
    
    self->ob_version++;                // ++ после append/pop/insert/...
    return 0;
}
```

**Любое** изменение списка (`append`, `pop`, `del lst[i]`, `lst[i]=x`) → `self->ob_version++`.
Итератор видит несоответствие → **RuntimeError**.

## 4. PyDictObject ma_version_tag (3.9+ split table)

```c
typedef struct {
    Py_ssize_t ma_used;            // Количество пар
    uint64_t ma_version_tag;       // 64-битный счётчик изменений (атомарный!)
    PyDictKeysObject *ma_keys;     // Ключи
    PyObject **ma_values;          // Значения
} PyDictObject;
```

Словарь имеет **64-битный атомарный** `ma_version_tag`. Изменение (set/del) →
`DICT_NEXT_VERSION()` (вероятно `++`).

## 5. dict_iter_next() - проверка версии словаря

```c
static PyObject *dict_iter_next(PyDictIterObject *iter) {
    PyDictObject *dict = iter->di_dict;    // Ссылка на словарь
    
    // КРИТИЧЕСКАЯ ПРОВЕРКА ВЕРСИИ
    if (iter->di_version_tag != dict->ma_version_tag) {
        PyErr_SetString(PyExc_RuntimeError,
            "dictionary changed size during iteration");
        return NULL;
    }
    
    // Получаем следующий элемент по iter->di_used
    PyDictKeyEntry *entry = get_entry(dict, iter->di_used);
    if (entry->me_key == NULL) {
        return NULL;                       // Конец
    }
    
    iter->di_used++;                       // Следующий
    return PyDictItem_KeyValue(entry);     // (key, value)
}
```

`for k in d:` создаёт PyDictIterObject с копией `d->ma_version_tag`. Каждый `next()` проверяет
версии → **"dictionary changed size"**.

## 6. DICT_NEXT_VERSION() - обновление версии

```c
#define DICT_NEXT_VERSION() \
    (_Py_atomic_fetch_add_uint64(&_PyRuntime.dict_version, 1) + 1)

static int insertdict(PyDictObject *mp, PyObject *key, Py_ssize_t hash, PyObject *value) {
    // ... логика вставки ...
    
    // Обновляем версию АТОМАРНО после изменения
    mp->ma_version_tag = DICT_NEXT_VERSION();
    mp->ma_used++;
    
    return 0;
}
```

**Любое** изменение словаря (`d[k]=v`, `del d[k]`, `d.clear()`) → атомарное
`ma_version_tag = global_dict_version++`. Итератор видит → **RuntimeError**.

## 7. PySetObject версия (аналогично dict)

```c
typedef struct {
    Py_ssize_t used;               // Количество элементов
    uint64_t version_tag;          // Счётчик изменений
    PyDictKeysObject *keys;        // Shared keys
} PySetObject;

static PyObject *set_iter_next(PySetIterObject *setiter) {
    PySetObject *set = setiter->it_set;
    
    if (setiter->it_version_tag != set->version_tag) {
        PyErr_SetString(PyExc_RuntimeError,
            "set changed size during iteration");
        return NULL;
    }
    // ... остальная логика
}
```

Множества работают **идентично** словарям: `version_tag`, проверка в итераторе → **"set
changed size during iteration"**.

## 8. Tuple/String итераторы (без проверок)

```c
typedef struct {
    PyObject_HEAD
    Py_ssize_t it_index;
    PyTupleObject *it_seq;         // НЕИЗМЕНЯЕМЫЕ!
} PyTupleIterObject;

static PyObject *tupleiter_next(PyTupleIterObject *it) {
    // НЕТ проверки версии - tuple неизменяемы!
    Py_ssize_t index = it->it_index;
    PyTupleObject *tuple = it->it_seq;
    
    if (index < Py_SIZE(tuple)) {
        return Py_NewRef(tuple->ob_item[index++]);
    }
    return NULL;
}
```

**Неизменяемые** типы (tuple, str, bytes) **НЕ** проверяют версию — они **физически** не могут
измениться.

## 9. Создание итератора: PyObject_GetIter()

```c
PyObject *PyObject_GetIter(PyObject *obj) {
    PyTypeObject *tp = Py_TYPE(obj);
    
    // Быстрый путь для списков
    if (PyList_CheckExact(obj)) {
        PyListIterObject *it = PyObject_GC_New(PyListIterObject, &PyListIter_Type);
        it->it_seq = (PyListObject *)Py_NewRef(obj);
        it->it_index = 0;
        it->it_version = ((PyListObject *)obj)->ob_version;  // <- КОПИРУЕМ ВЕРСИЮ!
        _PyObject_GC_TRACK(it);
        return (PyObject *)it;
    }
    
    // Общий путь через tp_iter
    unaryfunc iter = tp->tp_iter;
    if (iter != NULL) {
        return (*iter)(obj);
    }
    
    // __iter__()
    return PyObject_CallMethodObjArgs(obj, &_Py_ID(__iter__), NULL);
}
```

`iter(lst)` → PyListIterObject → `it_version = lst->ob_version`. **Любое** изменение списка →
`lst->ob_version++` → итератор **ломается**.

## 10. Байткод: GET_ITER → FOR_ITER

```python
# for x in lst:
```

```
  0 GET_ITER                 # iter(lst) -> PyListIterObject
  2 FOR_ITER       10 (to 14) # next(it) -> x или JUMP
  4 STORE_FAST      0 (x)     # x в локальную
  6 <BODY>
 10 JUMP_ABSOLUTE   0         # Следующая итерация
14 <END>
```

**FOR_ITER в ceval.c:**

```c
case FOR_ITER: {
    PyObject *iter = TOP();            // Итератор
    PyObject *next = (*Py_TYPE(iter)->tp_iternext)(iter);  // next(it)
    
    if (next != NULL) {
        STACKADJ(-1);
        PEEK(0) = next;                // Значение на стек
        JUMPBY(oparg);                 // К STORE_FAST x
    } else {
        Py_DECREF(iter);               // Конец -> StopIteration
        if (!_PyErr_Occurred(tstate) || 
            !_PyErr_ExceptionMatches(tstate, PyExc_StopIteration)) {
            // RuntimeError от итератора!
            goto error;
        }
        _PyErr_Clear(tstate);          // Очищаем StopIteration
        STACKADJ(-1);                  // Убираем iter
        JUMPBY(oparg + 1);             // К концу цикла
    }
    DISPATCH();
}
```

`FOR_ITER` вызывает `it->tp_iternext()` → проверка версии → RuntimeError или значение.
StopIteration → **нормальный** конец цикла.

## 11. Пользовательский итератор с __iter__/__next__

```c
static PyObject *myiter_next(MyIterObject *self) {
    if (self->version != self->seq->ob_version) {
        PyErr_SetString(PyExc_RuntimeError,
            "container modified during iteration");
        return NULL;
    }
    // ... логика
}
```

Пользовательские классы **могут** проверять версии в `__next__()`.

**Изменение коллекции** в CPython 3.9+ детектируется **version tag** (`ob_version`/`ma_version_tag`) в
PyListObject/PyDictObject + проверкой в `tp_iternext()` итераторов → **RuntimeError** при несоответствии.

- [Содержание](/CONTENTS.md#содержание)