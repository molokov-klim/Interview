# Собеседование Python AQA

# Содержание

## Core

- [Типы данных](#типы-данных)
- [Срезы](#срезы)
- [*args и **kwargs](#args-и-kwargs)
- [Хеш-таблица](#хеш-таблица)
- [Встроенные функции](#встроенные-функции)
- [Контекстные менеджеры (with)](#контекстные-менеджеры)
- [Генераторы и итераторы](#генераторы-и-итераторы)
- [Декораторы и замыкания](#декораторы-и-замыкания)
- [GIL (Global Interpreter Lock)](#gil-global-interpreter-lock)
- [Изменение списка во время итерации](#изменение-коллекции-во-время-итерации)
- [Области видимости](#области-видимости)
- [Lambda-функции](#lambda-функции)
- [Comprehensions и генераторные выражения](#comprehensions-и-генераторные-выражения)
- [copy() и deepcopy()](#copy-и-deepcopy)
- [Асинхронность](#асинхронность)
- [Многопоточность](#многопоточность)
- [Мультипроцессинг](#мультипроцессинг)
- [Dataclass](#dataclass)
- [Enum](#enum)
- [Garbage Collector (сборщик мусора)](#garbage-collector-сборщик-мусора)
- [Сложность кода (Asymptotic, Cyclomatic, Coupling, Maintainability)](#сложность-кода-asymptotic-cyclomatic-coupling-maintainability)

## РАЗДЕЛ ООП

- [ООП](#ооп)
- [Абстракция (ООП)](#абстракция)
- [Инкапсуляция (ООП)](#инкапсуляция)
- [Наследование (ООП)](#наследование)
- [Полиморфизм (ООП)](#полиморфизм)
- [Diamond Problem](#diamond-problem)
- [Магические методы](#магические-методы)
- [Инвариантность и ковариантность](#инвариантность-и-ковариантность)
- [Декораторы классов и методов](#декораторы-классов-и-методов)
- [Множественное наследование и MRO](#множественное-наследование-и-mro)
- [ABC (Abstract Base Classes)](#abc)
- [Протокол (Protocol)](#протокол-protocol)
- [Паттерны проектирования](#паттерны-проектирования)
- [Наследование vs композиция](#наследование-vs-композиция)
- [Композиция и агрегация](#композиция-и-агрегация)
- [Связность и связанность](#связность-и-связанность)
- [SOLID](#solid)
- [Метапрограммирование](#метапрограммирование)
- [Миксины](#миксины)

## Типизация

1. [typing: Optional, Union, TypeVar, Generic](#typing)
2. [Literal, TypedDict, Protocol](#literal-typeddict-protocol)
3. [Ковариантность, контравариантность](#ковариантность-контравариантность)

## Инструменты

1. [pytest](#pytest)
2. [pytest hooks](#pytest-hooks)
3. [Kubernetes](#kubernetes)

## Теория тестирования

1. [Пирамида тестирования](#пирамида-тестирования)
2. [Виды тестирования](#виды-тестирования)
3. [Метрики тестирования](#метрики-тестирования)
4. [Техники тест дизайна](#техники-тест-дизайна)

---

# **Типы данных**

## **Junior Level**

Типы данных в Python делятся на **изменяемые** (mutable) и **неизменяемые** (immutable).

**Простые типы:**

- Числа: `int`, `float`, `complex`
- Строки: `str`
- Логические значения: `bool`
- Специальный тип: `NoneType` (единственное значение `None`)

**Коллекционные типы:**

- `list` — изменяемая упорядоченная коллекция
- `tuple` — неизменяемая упорядоченная коллекция
- `dict` — изменяемая коллекция пар «ключ-значение» (упорядоченная с Python 3.7)
- `set` / `frozenset` — изменяемое и неизменяемое множества уникальных элементов
- `bytes` / `bytearray` — неизменяемая и изменяемая последовательности байтов

## **Middle Level**

1. **Ключевые различия мутабельности:**
    - **Изменяемые:** `list`, `dict`, `set`, `bytearray`, пользовательские классы. Можно модифицировать после создания.
    - **Неизменяемые:** `int`, `float`, `str`, `bytes`, `tuple`, `frozenset`, `bool`, `NoneType`. Любая операция создаёт
      новый объект.

2. **Практические следствия:**
    - Передача изменяемых объектов в функции позволяет модифицировать оригинал
    - Только неизменяемые объекты могут быть ключами словаря (требуется хэшируемость)
    - Мутабельность влияет на потокобезопасность и кэширование

3. **Специфика типов:**
    - `None` — синглтон, обозначающий отсутствие значения
    - `bool` — подкласс `int`, значения `True` и `False` — синглтоны
    - `tuple` — неизменяем, но может содержать изменяемые элементы
    - `set`/`frozenset` — хранят только хэшируемые элементы, реализация аналогична словарям без значений

## **Senior Level**

В CPython **все типы данных** наследуют от базовой структуры `PyObject`, которая содержит refcount и указатель на тип.
Каждый тип описывается массивом **слотов** в `PyTypeObject`. [Include/object.h][Include/cpython/object.h]

## 1. Базовая структура PyObject

```c
typedef struct _object {
    _PyObject_HEAD_EXTRA       // Платформо-зависимые поля для отладки (PyDebug)
    Py_ssize_t ob_refcnt;      // Счётчик ссылок - при 0 вызывается tp_dealloc
    struct _typeobject *ob_type; // Указатель на PyTypeObject описывающий тип
} PyObject;
```

**Объяснение для людей:** Каждый объект в памяти начинается с 16-24 байт заголовка. `ob_refcnt` считает, сколько ссылок
на объект существует. Когда он доходит до 0, объект уничтожается. `ob_type` говорит, какого он типа (int/list/dict).

## 2. PyVarObject для контейнеров

```c
typedef struct {
    PyObject ob_base;          // Встроенный PyObject (refcnt + type)
    Py_ssize_t ob_size;        // Количество элементов (длина строки/списка)
} PyVarObject;
```

**Объяснение для людей:** Контейнеры (списки, строки, словари) имеют дополнительное поле `ob_size` сразу после заголовка
PyObject. Это длина коллекции.

## 3. PyTypeObject - "паспорт" каждого типа

```c
typedef struct _typeobject {
    PyVarObject ob_base;       // Тип сам является объектом (можно наследовать)
    
    const char *tp_name;       // Имя типа ("list", "dict", "int")
    Py_ssize_t tp_basicsize;   // Размер в байтах без переменной части
    Py_ssize_t tp_itemsize;    // Размер одного элемента переменной части
    
    destructor tp_dealloc;     // Функция уничтожения (list_dealloc)
    printfunc tp_print;        // Для print()
    reprfunc tp_repr;          // Для repr()
    
    // Протокол чисел
    PyNumberMethods *tp_as_number;  // nb_add, nb_sub, nb_multiply...
    
    // Протокол последовательностей
    PySequenceMethods *tp_as_sequence; // sq_item, sq_ass_slice...
    
    // Протокол маппингов
    PyMappingMethods *tp_as_mapping;  // mp_subscript, mp_ass_subscript
    
    // Поиск атрибутов
    getattrofunc tp_getattro;      // obj.attr
    setattrofunc tp_setattro;      // obj.attr = value
    
    // Дескрипторы (property, method)
    descrgetfunc tp_descr_get;     // __get__
    descrsetfunc tp_descr_set;     // __set__
    
    Py_ssize_t tp_dictoffset;      // Смещение __dict__ (или -1)
    Py_ssize_t tp_weaklistoffset;  // Смещение weakref списка
    
    PyObject *tp_mro;              // Method Resolution Order (tuple типов)
    PyObject *tp_cache;            // Кеш атрибутов (free-threaded)
    unsigned int tp_subclasses;    // Количество живых подклассов
    
    PyObject *tp_dict;             // __dict__ класса
    // ... 100+ слотов
} PyTypeObject;
```

**Объяснение для людей:** PyTypeObject - это как "техпаспорт" типа. Он говорит интерпретатору размер объекта, как его
уничтожать, как складывать/умножать, как брать по индексу `lst[0]`, как искать атрибуты `obj.attr`. Без этого паспорта
интерпретатор не знает, что делать с объектом.

## 4. PyLongObject (int) - переменной точности

```c
typedef uint32_t digit;        // 30-битная цифра (2^30 = ~1e9)

typedef struct _longobject {
    PyObject_VAR_HEAD          // PyObject + ob_size (кол-во цифр)
    digit ob_digit[1];         // Массив цифр (размер ob_size)
} PyLongObject;
```

**Объяснение для людей:** Целые числа хранятся как массив 30-битных "цифр". Маленькие числа (-5..256) кешируются как
singletons для экономии памяти.

**Создание PyLongObject:**

```c
PyObject *_PyLong_New(Py_ssize_t size) {
    PyLongObject *result;
    
    size = Py_ABS(size);  // Берем модуль размера
    
    // Выделяем память под заголовок + массив цифр
    result = PyObject_MALLOC(sizeof(PyLongObject) + 
                             (size-1) * sizeof(digit));
    if (!result) {
        return PyErr_NoMemory();
    }
    
    // Инициализируем как PyObject
    PyObject_INIT(result, &PyLong_Type);
    Py_SET_SIZE(result, size);  // Устанавливаем ob_size
    result->ob_digit[0] = 0;    // Нулевая цифра
    
    return (PyObject *)result;
}
```

**Объяснение для людей:** Выделяем ровно столько памяти, сколько нужно под заголовок + нужное количество 30-битных цифр.
Например, число 1e18 требует ~6 цифр (6*30=180 бит).

## 5. PyListObject со слотами протоколов

```c
typedef struct {
    PyObject_VAR_HEAD         // PyObject + ob_size (длина)
    PyObject **ob_item;       // Указатели на элементы
    Py_ssize_t allocated;     // Выделенная ёмкость (> ob_size)
} PyListObject;
```

**Объяснение для людей:** Список - это массив указателей на PyObject*. `allocated` больше `ob_size` для оптимизации (
over-allocation ~1.125x).

**Слоты PyList_Type:**

```c
PyTypeObject PyList_Type = {
    PyVarObject_HEAD_INIT(&PyType_Type, 0)  // Наследуем от type
    "list",                    // tp_name
    sizeof(PyListObject),      // tp_basicsize
    0,                         // tp_itemsize (переменная часть в ob_item)
    
    (destructor)list_dealloc,  // tp_dealloc
    0,                         // tp_print
    0,                         // tp_getattr
    0,                         // tp_setattr
    0,                         // tp_reserved
    list_repr,                 // tp_repr -> str(list)
    
    0,                         // tp_as_number
    &list_as_sequence,         // tp_as_sequence <- ВАЖНО!
    0,                         // tp_as_mapping
    (hashfunc)PyObject_HashNotImplemented,  // tp_hash (списки не хешируемы)
};
```

**Объяснение для людей:** `tp_as_sequence` указывает на таблицу со слотами `sq_item` (lst), `sq_slice` (lst[1:3]),
`sq_ass_slice` (lst[1:3]=[]).

**list_as_sequence.sq_item (lst[i]):**

```c
static PyObject *list_item(PyListObject *self, Py_ssize_t i) {
    if (i < 0 || i >= Py_SIZE(self)) {
        PyErr_SetString(PyExc_IndexError, "list index out of range");
        return NULL;
    }
    Py_INCREF(self->ob_item[i]);   // Увеличиваем refcnt
    return self->ob_item[i];       // Возвращаем элемент
}
```

**Объяснение для людей:** Проверяем индекс, увеличиваем refcnt элемента (теперь владелец отвечает за его жизнь),
возвращаем указатель.

## 6. PyDictObject с split table (с 3.6)

```c
typedef struct {
    Py_ssize_t ma_used;        // Кол-во ключей (не слотов!)
    uint64_t ma_version_tag;   // Версия для итераторов
    PyDictKeysObject *ma_keys; // Общие ключи
    PyObject **ma_values;      // Массив значений
} PyDictObject;
```

**Объяснение для людей:** С 3.6 словари компактные: ключи вынесены в отдельную `PyDictKeysObject`, значения в массиве.
`ma_used` считает реальные пары, а не слоты.

**PyDictKeysObject:**

```c
struct _dictkeysobject {
    Py_ssize_t dk_size;        // Размер хеш-таблицы
    enum dict_keys_kind dk_kind; // DICT_KEYS_UNICODE и т.д.
    union {
        PyDictUnicodeEntry *dk_entries;  // Полная таблица key+value
        PyDictKeyEntry *dk_indices;      // Только индексы
    } dk;
    uint64_t dk_version_tag;
};
```

**Объяснение для людей:** Ключи компактно хранятся в `PyDictKeysObject`, который может быть **shared** между словарями (
экономия памяти).

## 7. GC-интеграция для контейнеров

```c
// list_traverse - обход ссылок для GC
static int list_traverse(PyListObject *o, visitproc visit, void *arg) {
    Py_ssize_t i = 0;
    Py_ssize_t len = Py_SIZE(o);
    for (; i < len; i++) {
        Py_VISIT(o->ob_item[i]);  // Отмечаем каждый элемент как живой
    }
    return 0;
}

// list_clear - разрыв циклических ссылок
static int list_clear(PyListObject *o) {
    Py_ssize_t i = 0;
    Py_ssize_t len = Py_SIZE(o);
    for (; i < len; i++) {
        Py_XDECREF(o->ob_item[i]);  // Уменьшаем refcnt элементов
        o->ob_item[i] = NULL;       // NULL'им ссылки
    }
    return 0;
}
```

**Объяснение для людей:** GC вызывает `tp_traverse` для обхода ссылок внутри объекта (чтобы найти живые объекты).
`tp_clear` обнуляет ссылки перед удалением, разрывая циклы.

## 8. Байткод-интеграция: LIST_APPEND

```c
case LIST_APPEND: {
    PyObject *v = TOP();              // Берём значение с вершины стека
    PyObject *list = PEEK(oparg + 1); // Берём список из фиксированной позиции
    Py_ssize_t index = oparg;         // Индекс списка в localsplus
    
    // Вызываем list.append(v)
    int err = PyList_Append(list, v);
    Py_DECREF(v);                     // Освобождаем значение
    
    if (err == 0) {
        STACKADJ(-1);                 // Убираем значение со стека
    } else {
        // Ошибка - прерываем выполнение
        break;
    }
    DISPATCH_SAME_OPARG(1);           // Следующая инструкция
}
```

**Объяснение для людей:** В listcomp `[x for x in lst]` список берётся не с вершины стека (чтобы не мешать вычислениям),
а из фиксированного слота localsplus. Это экономит push/pop операции.

## 9. Инициализация типов: PyType_Ready

```c
int PyType_Ready(PyTypeObject *type) {
    if (type->tp_flags & Py_TPFLAGS_READY)  // Уже инициализирован
        return 0;
    
    // Наследуем слоты от базовых классов
    if (type->tp_bases) {
        Py_ssize_t i, nbase = PyTuple_GET_SIZE(type->tp_bases);
        for (i = 0; i < nbase; i++) {
            PyTypeObject *base = (PyTypeObject *)PyTuple_GET_ITEM(type->tp_bases, i);
            if (PyType_Ready(base) < 0)
                return -1;
            
            // Наследуем слоты (tp_as_number, tp_as_sequence...)
            inherit_special(base, type);
        }
    }
    
    // Вычисляем MRO
    if (mro_internal(type) < 0)
        return -1;
        
    // Инициализируем tp_dict
    if (type->tp_dict == NULL) {
        if (PyType_AllocDict(type) < 0)
            return -1;
    }
    
    type->tp_flags |= Py_TPFLAGS_READY;  // Отмечаем готовым
    return 0;
}
```

**Объяснение для людей:** Перед первым использованием типа вызывается PyType_Ready. Оно наследует слоты от родителей,
вычисляет MRO, создаёт `__dict__` класса. Без этого тип не готов к работе.

Типы данных в CPython — это **PyTypeObject** с 100+ слотами протоколов, живущие в памяти как обычные объекты, с
refcount'ами, GC-интеграцией и наследованием слотов через MRO.

- [Содержание](#содержание)

---

# **Срезы**

## **Junior Level**

Срезы в Python — это удобный способ получить часть последовательности (например, списка, строки или кортежа), не
перебирая элементы вручную. Представьте, что у вас есть книга, и вам нужно вырвать из неё несколько страниц — срез
делает примерно то же самое с данными.

**Базовый синтаксис:** `последовательность[начало:конец:шаг]`

- **Начало (start)** — индекс, с которого начинается срез (включается в результат). Если не указан — считается 0.
- **Конец (stop)** — индекс, на котором срез заканчивается (НЕ включается в результат). Если не указан — считается до
  конца последовательности.
- **Шаг (step)** — через сколько элементов брать следующий. Если не указан — считается 1 (брать все подряд).

**Простые примеры:**

```python
text = "Привет, мир!"
print(text[0:6])  # "Привет" (символы с 0 по 5)
print(text[8:])  # "мир!"   (с 8 до конца)
print(text[:6])  # "Привет" (с начала до 5)
print(text[::2])  # "Пие,мр" (каждый второй символ)
```

**Отрицательные индексы** — отсчитываются с конца:

- `-1` — последний элемент
- `-2` — предпоследний элемент

```python
numbers = [10, 20, 30, 40, 50]
print(numbers[-3:-1])  # [30, 40] (с третьего с конца до предпоследнего)
```

**Отрицательный шаг** позволяет идти в обратном порядке:

```python
word = "питон"
print(word[::-1])  # "нотип" (разворот строки)
print(numbers[3:0:-1])  # [40, 30, 20] (от индекса 3 до 1 в обратном порядке)
```

**Важно:** срезы всегда создают **новый объект**. Если вы сделаете срез списка, получится новый список, а не ссылка на
старый. Это защищает от случайного изменения исходных данных.

## **Middle Level**

### 1. **Как Python интерпретирует срезы "под капотом"**

Когда вы пишете `seq[start:stop:step]`, Python фактически создаёт объект `slice(start, stop, step)` и передаёт его в
метод `__getitem__` вашей последовательности. Этот объект можно сохранить и использовать повторно:

```python
data = list(range(100))
my_slice = slice(10, 50, 3)  # создаём объект-срез
print(data[my_slice])  # равносильно data[10:50:3]
```

Это особенно полезно при работе с многомерными массивами в библиотеках типа NumPy, где срезы могут быть сложными.

### 2. **Особенности работы с разными типами последовательностей**

- **Списки (list)** — срезы создают новые списки (shallow copy). Элементы копируются по ссылкам, а не дублируются:
  ```python
  matrix = [[1, 2], [3, 4]]
  part = matrix[:1]     # part = [[1, 2]]
  part[0][0] = 99       # изменится и matrix[0][0] тоже!
  ```
  Для полного копирования вложенных структур используйте `copy.deepcopy()`.

- **Строки (str) и байты (bytes)** — срезы всегда возвращают новые объекты, поскольку эти типы неизменяемы.

- **Пользовательские классы** — вы можете реализовать поддержку срезов, определив методы `__getitem__`, `__setitem__` и
  `__delitem__`, которые будут обрабатывать объекты `slice`.

### 3. **Расширенное использование с шагом (step)**

Шаг может быть любым целым числом, кроме 0. Особый случай — **отрицательный шаг**:

- При отрицательном шаге индексы `start` и `stop` меняются ролями по умолчанию: если не указать `start`, он становится
  последним элементом; если не указать `stop` — первым.
- Последовательность перебирается в обратном порядке.

```python
seq = [0, 1, 2, 3, 4, 5]

# Что происходит при seq[5:1:-1]?
# 1. Начинаем с индекса 5 (элемент 5)
# 2. Идём в обратном порядке с шагом 1
# 3. Останавливаемся ДО индекса 1 (т.е. на индексе 2)
print(seq[5:1:-1])  # [5, 4, 3, 2]
```

### 4. **Идиомы и продвинутые паттерны**

- **Копирование последовательности:** `seq[:]` или `seq.copy()` для списков
- **Разворот:** `seq[::-1]` — создаёт развёрнутую копию
- **Получение каждого N-го элемента:**
  ```python
  # Каждый третий элемент, начиная со второго
  seq = list(range(20))
  print(seq[1::3])  # [1, 4, 7, 10, 13, 16, 19]
  ```
- **Удаление части последовательности (для изменяемых типов):**
  ```python
  numbers = [0, 1, 2, 3, 4, 5]
  numbers[2:4] = []           # удаляет элементы 2 и 3
  # или del numbers[2:4]
  ```

### 5. **Срезы и производительность**

Срезы работают за O(k), где k — размер среза, а не исходной последовательности. Однако есть нюансы:

- Для списков срез создаёт новую структуру, копируя ссылки на элементы
- При больших срезах это может быть накладно по памяти
- Для строк срезы более эффективны благодаря оптимизациям интерпретатора

### 6. **Где срезы не работают как ожидается**

- **Словари и множества** не поддерживают срезы, поскольку не являются последовательностями (у них нет порядка элементов
  до Python 3.7+ для dict, и даже тогда срезы не определены)
- **Итераторы и генераторы** — тоже не поддерживают срезы напрямую. Вместо этого используйте `itertools.islice()`:
  ```python
  from itertools import islice
  
  def count():
      i = 0
      while True:
          yield i
          i += 1
          
  # Получить элементы с 10 по 20 (не включая 20)
  sliced = islice(count(), 10, 20)
  ```

### 7. **Срезы в присваивании (для изменяемых последовательностей)**

Это мощная особенность Python, позволяющая заменять часть последовательности другой последовательностью произвольной
длины:

```python
numbers = [0, 1, 2, 3, 4, 5]
numbers[2:4] = [20, 30, 40]  # Заменяем 2 элемента на 3
print(numbers)  # [0, 1, 20, 30, 40, 4, 5]

# Можно даже удалять элементы, вставляя пустую последовательность
numbers[1:3] = []  # Удаляем элементы 1 и 2
```

Это работает потому, что для изменяемых последовательностей срезы в левой части присваивания реализуются через метод
`__setitem__`, который может принимать объект slice и произвольную последовательность для вставки.

**Итог:** Срезы — это не просто синтаксический сахар, а полноценный механизм доступа к данным, который при грамотном
использовании делает код чище, выразительнее и иногда даже эффективнее.

## **Senior Level**

В CPython **срезы** реализуются через компактный объект `PySliceObject`, байткоды `BUILD_SLICE`/`BINARY_SUBSCR`/
`STORE_SUBSCR` и унифицированный C-API `PySlice_GetIndicesEx` для нормализации индексов во всех
типах. `Objects/sliceobject.c`, `Include/sliceobject.h`

## 1. Структура PySliceObject

```c
typedef struct {
    PyObject_VAR_HEAD     // PyObject + ob_size=3 (всегда 3 элемента)
    PyObject *start;      // Начальный индекс (или Py_None)
    PyObject *stop;       // Конечный индекс (или Py_None)
    PyObject *step;       // Шаг (или Py_None)
} PySliceObject;
```

**Объяснение для людей:** Срез `lst[1:3:2]` в памяти — это структура из 4 полей: 24-байт заголовок PyObject + 3
указателя (start=1, stop=3, step=2). `Py_None` означает "используй значение по умолчанию".

## 2. Создание среза: BUILD_SLICE байткод

**Python код:** `lst[1:3]`

```
 0 LOAD_FAST          0 (lst)      # Загружает переменную lst на стек
 2 LOAD_CONST         0 (1)        # Загружает константу 1 на стек  
 4 LOAD_CONST         1 (3)        # Загружает константу 3 на стек
 6 BUILD_SLICE        2            # Создает объект среза slice(1, 3, None)
 8 BINARY_SUBSCR                   # Выполняет операцию взятия среза: lst[1:3]
```

**Реализация BUILD_SLICE в ceval.c:**

```c
case BUILD_SLICE: {
    PyObject *slice;           // Будущий PySliceObject
    PyObject *stop, *start;    // Аргументы со стека
    
    // Снимаем аргументы (step, stop, start)
    if (oparg == 3) {
        PyObject *step = POP();    // step (может быть None)
        stop = POP();              // stop
        start = POP();             // start
        slice = PySlice_New(start, stop, step);
        Py_DECREF(step);
    } else {
        // 2 аргумента: slice(start, stop, None)
        stop = POP();
        start = POP();
        slice = PySlice_New(start, stop, NULL);
    }
    
    Py_DECREF(start);              // Освобождаем временные объекты
    Py_DECREF(stop);
    
    if (slice == NULL) {           // Ошибка создания
        return NULL;
    }
    
    PUSH(slice);                   // PySliceObject на вершину стека
    DISPATCH();                    // Следующая инструкция
}
```

**Объяснение для людей:** Интерпретатор снимает 2/3 значения со стека, вызывает PySlice_New (создаёт PySliceObject),
кладёт результат обратно на стек. Всё за 1 инструкцию.

## 3. PySlice_New - фабрика срезов

```c
PyObject *PySlice_New(PyObject *start, PyObject *stop, PyObject *step) {
    PySliceObject *self;
    
    // Проверяем аргументы
    if (step == NULL) {
        step = Py_None;            // По умолчанию step=None
    }
    if (start == NULL) {
        start = Py_None;
    }
    if (stop == NULL) {
        stop = Py_None;
    }
    
    // Выделяем память под PySliceObject (PyObject + 3 PyObject*)
    self = PyObject_GC_New(PySliceObject, &PySlice_Type);
    if (self == NULL) {
        return NULL;
    }
    
    // Заполняем поля (увеличиваем refcnt)
    self->start = Py_NewRef(start);
    self->stop = Py_NewRef(stop);
    self->step = Py_NewRef(step);
    
    _PyObject_GC_TRACK(self);      // Добавляем в GC
    return (PyObject *)self;
}
```

**Объяснение для людей:** PySlice_New создаёт объект из 24 байт заголовка + 3 указателя. Каждый аргумент получает +1 к
refcnt. Объект регистрируется в GC для поиска циклов.

## 4. Нормализация индексов: PySlice_GetIndicesEx

**Ключевой API для всех типов (list/dict/numpy/...):**

```c
int PySlice_GetIndicesEx(
    PySliceObject* s,           // Входной срез
    Py_ssize_t length,          // Длина контейнера
    Py_ssize_t *start,          // [out] нормализованный start
    Py_ssize_t *stop,           // [out] нормализованный stop  
    Py_ssize_t *step,           // [out] нормализованный step
    Py_ssize_t *slicelength     // [out] длина результата среза
) {
    PyObject *start_o = s->start;
    PyObject *stop_o = s->stop;
    PyObject *step_o = s->step;
    
    Py_ssize_t istart = 0, iend = 0, istep = 1;
    
    // Конвертируем start
    if (start_o == Py_None) {
        istart = 0;
    } else if (PySlice_Unpack(start_o, &istart, &iend, &istep) < 0) {
        return -1;                 // Ошибка конвертации
    }
    
    // Аналогично для stop_o
    if (stop_o == Py_None) {
        iend = length;
    } else if (PySlice_Unpack(stop_o, &istart, &iend, &istep) < 0) {
        return -1;
    }
    
    // Аналогично для step_o
    if (step_o == Py_None) {
        istep = 1;
    } else if (PySlice_Unpack(step_o, &istart, &iend, &istep) < 0) {
        return -1;
    }
    
    // Нормализуем отрицательные индексы
    PySlice_AdjustIndices(length, istart, iend, istep, start, stop);
    
    // Вычисляем длину результата
    *slicelength = PySlice_ComputeLength(*start, *stop, *step, length);
    
    *step = istep;
    return 0;
}
```

**Объяснение для людей:** Эта функция превращает `slice(-2:, None, 2)` в конкретные числа: для списка длины 10 даёт
start=8, stop=10, step=2. Вычисляет, сколько элементов будет в результате (2 элемента).

## 5. PySlice_AdjustIndices - обработка отрицательных индексов

```c
void PySlice_AdjustIndices(
    Py_ssize_t length,         // Длина контейнера
    Py_ssize_t istart,         // Сырой start
    Py_ssize_t iend,           // Сырой stop
    Py_ssize_t istep,          // Шаг
    Py_ssize_t *start,         // [out] нормализованный start
    Py_ssize_t *end            // [out] нормализованный stop
) {
    Py_ssize_t i;
    
    if (istep > 0) {               // Положительный шаг
        if (istart < 0) {
            i = length + istart;   // -2 -> len-2
            *start = i < 0 ? 0 : i;
        } else {
            *start = istart;
        }
        
        if (iend < 0) {
            i = length + iend;     // -1 -> len-1
            *end = i < 0 ? 0 : i;
        } else {
            *end = iend;
        }
    } else {                       // Отрицательный шаг
        // Симметричная логика для обратного прохода
        if (istart < 0) {
            i = length + istart;
            *start = i < -1 ? length-1 : i;
        } else {
            *start = istart;
        }
        // Аналогично для end...
    }
}
```

**Объяснение для людей:** `-1` всегда означает "последний элемент", `-2` — предпоследний. Функция переводит
отрицательные индексы в абсолютные, обрезая по границам массива.

## 6. List срез: list_slice()

```c
static PyObject *list_slice(PyListObject *self, Py_ssize_t ilow, Py_ssize_t ihigh) {
    register Py_ssize_t i;
    Py_ssize_t len = ihigh - ilow;     // Длина среза
    
    if (len <= 0) {
        return Py_NewRef(&PyEmptyList);  // Пустой срез -> singleton []
    }
    
    if (len == 1) {
        // Быстрый путь для одного элемента
        Py_INCREF(self->ob_item[ilow]);
        return self->ob_item[ilow];
    }
    
    // Создаём новый список
    PyObject *np = PyList_New(len);
    if (np == NULL)
        return NULL;
    
    // Копируем элементы
    for (i = 0; i < len; i++) {
        PyObject *v = Py_NewRef(self->ob_item[ilow + i]);
        PyList_SET_ITEM(np, i, v);
    }
    
    return np;
}
```

**Объяснение для людей:** Создаём новый список нужной длины, увеличиваем refcnt каждого элемента оригинала (+1), кладём
указатели в новый список. Эффективно благодаря PyList_New (over-allocation).

## 7. Срезовое присваивание: list_ass_slice()

```c
static int list_ass_slice(PyListObject *self, Py_ssize_t low, Py_ssize_t high, PyObject *v) {
    Py_ssize_t n;              // Длина заменяемого среза
    PyObject **src, **dest;
    Py_ssize_t i;
    
    n = high - low;            // Длина удаляемого среза
    
    if (v == NULL) {
        // Удаление: lst[1:3] = []
        return list_ass_slice_delete(self, low, high);
    }
    
    if (!PyList_Check(v)) {
        // Сжатие: lst[1:3] = 42
        return list_ass_item(self, low, v);
    }
    
    Py_ssize_t n_added = Py_SIZE(v);  // Длина нового списка
    
    // Перераспределяем память
    if (_PyList_Resize_Shrink(self, low + n_added) < 0) {
        return -1;
    }
    
    // Копируем элементы
    dest = self->ob_item + low;
    src = ((PyListObject *)v)->ob_item;
    for (i = 0; i < n_added; i++) {
        PyObject *w = Py_NewRef(src[i]);
        PyList_SET_ITEM(self, low + i, w);
    }
    
    return 0;
}
```

**Объяснение для людей:** `lst[1:3] = [10,20]` удаляет 2 элемента, добавляет 2 новых, сдвигает хвост.
`PyList_Resize_Shrink` оптимизирует память (over-allocation).

## 8. Слоты PyList_Type для поддержки срезов

```c
static PySequenceMethods list_as_sequence = {
    // sq_length: len(lst)
    (lenfunc)list_length,
    
    // sq_concat: lst + lst2
    (binaryfunc)list_concat,
    
    // sq_repeat: lst * 3
    (ssizeargfunc)list_repeat,
    
    // sq_item: lst[i]
    (ssizeargfunc)list_item,
    
    // sq_slice: lst[1:3] <- ВАЖНО!
    (ssizessizeargfunc)list_slice,
    
    // sq_ass_item: lst[i] = x
    (ssizeobjargproc)list_ass_item,
    
    // sq_ass_slice: lst[1:3] = ... <- ВАЖНО!
    (ssizeobjargproc)list_ass_slice,
};
```

**Объяснение для людей:** Все типы регистрируют таблицу `PySequenceMethods` со слотами. BINARY_SUBSCR выбирает
`sq_slice`/`mp_subscript` в зависимости от типа.

Срезы в CPython — это **PySliceObject** (3 указателя) + **PySlice_GetIndicesEx** (нормализация) + **тип-специфичные**
`sq_slice`/`sq_ass_slice`, вызываемые через байткоды BUILD_SLICE/BINARY_SUBSCR/STORE_SUBSCR.

---

# **args и kwargs**

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

**Объяснение для людей:** `def f(a, *args, b=1, **kwargs):` → `co_argcount=1` (a), `CO_VARARGS=1`, `CO_VARKEYWORDS=1`.
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

**Объяснение для людей:** Интерпретатор снимает N аргументов со стека, кладёт их в массив `args[]`, вызывает
`_PyObject_Vectorcall(func, args, N, NULL)`. Массив освобождается после вызова.

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

**Объяснение для людей:** Новые функции используют **vectorcall** (C API с массивом аргументов). Старые — через
`PyTuple_FromArray` (медленнее).

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

**Объяснение для людей:** При создании функции интерпретатор читает флаги CO_VARARGS/CO_VARKEYWORDS и находит имена
`*args`/`**kwargs` в `co_varnames`.

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

**Объяснение для людей:** При входе в `def f(a, *args, **kwargs):` интерпретатор кладёт `a=args[0]`,
`*args=tuple(args[1:])`, `**kwargs=dict(kw)` в локальные переменные фрейма.

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

**Объяснение для людей:** `f_localsplus[]` — массив всех локальных переменных. `f_localsplus[0]="a"`,
`f_localsplus[1]="args"`, `f_localsplus[2]="b"`, `f_localsplus[3]="kwargs"`.

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

**Объяснение для людей:** `*lst` распаковывается в позиционные аргументы, `**dct` → кортеж имен (
`kwnames=("key1", "key2")`) + значения. Передаются в vectorcall.

`*args`/**kwargs** в CPython — это флаги `CO_VARARGS`/`CO_VARKEYWORDS` в `PyCodeObject`, распаковка в `f_localsplus[]`
фрейма через `fast_function`, поддержка через байткоды `CALL_FUNCTION_KW` и vectorcall протокол.

- [Содержание](#содержание)

---

# *Хеш-таблица*

## **Junior Level**

Хеш-таблица — это структура данных, обеспечивающая амортизированную сложность O(1) для операций поиска, вставки и
удаления за счёт использования хеш-функции, преобразующей ключ в индекс массива. В Python она лежит в основе словарей (
dict) и множеств (set). Ключевыми особенностями являются требование хешируемости (неизменяемости) ключей, сохранение
порядка вставки и автоматическое увеличение размера при достижении определённого коэффициента заполнения.

Представьте библиотеку, где номер полки вычисляется по названию книги по определённому правилу (например, первая буква).
Это правило — **хеш-функция**. Она преобразует ключ в число-индекс. В идеале вы находите элемент за O(1) время.

**Коллизии** (когда разным ключам соответствует один индекс) решаются разными способами. Например, на «полке» может быть
список пар «ключ-значение», и вы ищете среди них по полному ключу.

В Python ключ словаря или элемент множества должен быть **хешируемым** (неизменяемым) объектом.

## **Middle Level**

В Python хеш-таблицы используют **открытую адресацию** с **квадратичным зондированием** для разрешения коллизий.

**Ключевые аспекты:**

1. **Хешируемость:** Объект хешируем, если:
    * Имеет метод `__hash__`, возвращающий целое число.
    * Имеет метод `__eq__` для сравнения.
    * Выполняется условие: `a == b` ⇒ `hash(a) == hash(b)`.
    * Неизменяемые типы (int, str, tuple, frozenset) хешируемы по умолчанию.
2. **Размер таблицы:** Всегда является степенью двойки, что позволяет вычислять индекс через битовую маску:
   `index = hash(key) & (table_size - 1)`.
3. **Коэффициент загрузки (load factor):** При заполнении ~2/3 таблица увеличивается вдвое (**rehashing**), что является
   амортизированной операцией O(n).
4. **Удаление элементов:** Элемент помечается как **dummy** (удалённый слот), чтобы не разрывать цепочки зондирования.
5. **Сохранение порядка:** Начиная с Python 3.7, порядок вставки в словаре гарантирован. Это достигается отдельным
   массивом записей (ключ-значение), который сохраняет порядок.

## **Senior Level**

В CPython 3.9+ **хеш-таблица** (`PyDictObject`) использует **split table** с **компактным представлением**: общие
`PyDictKeysObject` (shared keys) + массив значений `ma_values[]`. Поиск через `lookdict_*` с **робин-худ хешированием**
и **атомарными индексами**. `Objects/dictobject.c`,`Include/cpython/dictobject.h`

## 1. PyDictObject (Python 3.9+)

```c
typedef struct {
    Py_ssize_t ma_used;            // Количество *реальных* пар ключ-значение
    uint64_t ma_version_tag;       // Версия для итераторов (атомарно)
    PyDictKeysObject *ma_keys;     // Общие ключи (может быть shared)
    PyObject **ma_values;          // Массив значений (split table)
} PyDictObject;
```

**Объяснение для людей:** Словарь = счетчик занятых слотов + версия + общие ключи + массив значений. `ma_used` считает
пары, а не слоты хеш-таблицы.

## 2. PyDictKeysObject - shared keys

```c
typedef struct _dictkeysobject {
    Py_ssize_t dk_size;            // Размер хеш-таблицы (степень двойки)
    enum dict_keys_kind dk_kind;   // DICT_KEYS_UNICODE, DICT_KEYS_SPLIT и т.д.
    union {
        PyDictUnicodeEntry *dk_entries;  // Полная таблица для unicode ключей
        PyDictKeyEntry *dk_indices;      // Только индексы (int8/int16)
    } dk;
    uint64_t dk_version_tag;       // Версия ключей
    Py_ssize_t dk_nentries;        // Количество активных записей
    Py_ssize_t dk_usable;          // Сколько слотов ещё можно занять
    Py_ssize_t dk_refcnt;          // Счётчик ссылок (shared keys)
} PyDictKeysObject;
```

**Объяснение для людей:** `PyDictKeysObject` содержит хеш-таблицу индексов + сами ключи. Может быть **общим** для многих
словарей одной структуры (экономия памяти на классах).

## 3. Индексы в хеш-таблице (dk_indices)

```c
// dk_indices содержит значения:
// DKIX_EMPTY   (0-1)   - пустой слот
// DKIX_DUMMY   (0x8000)- удалённый слот (tombstone)
// DKIX_ACTIVE  (>0)    - индекс в entries/values (1..dk_size-1)
// 0xFFFF       - не используется
```

**Объяснение для людей:** Каждый слот хеш-таблицы — это **8-битный индекс** (int8_t): 0=пусто, 255=удалено,
1-254=указатель на запись с ключом.

## 4. lookdict_unicode - основной поиск (unicode ключи)

```c
static PyDictKeyEntry *lookdict_unicode(PyDictKeysObject *keys,
                                        PyObject *key, Py_hash_t hash) {
    size_t i = (size_t)hash & DK_MASK(keys);  // Начальный индекс (hash % size)
    PyDictKeyEntry *entries = keys->dk_entries;
    PyDictUnicodeEntry *unicode_entries = (PyDictUnicodeEntry *)entries;
    
    // Пробуем точное совпадение хеша + строки
    Py_hash_t mask = DK_MASK(keys);
    Py_ssize_t perturb = hash;
    PyDictKeyEntry *freeslot = NULL;
    PyDictKeyEntry *first_removed = NULL;
    
    // Робин-худ хеширование
    while (1) {
        PyDictKeyEntry *entry = &entries[i];
        if (entry->me_key == NULL) {
            // Пустой слот
            return freeslot ? freeslot : entry;
        }
        
        // Проверяем dummy слоты (удалённые)
        if (entry->me_key == DKIX_DUMMY) {
            if (!first_removed) {
                first_removed = entry;
            }
            i = (i + 1) & mask;
            continue;
        }
        
        // Сравниваем хеши
        if (entry->me_hash == hash) {
            PyDictUnicodeEntry *ue = (PyDictUnicodeEntry *)entry;
            if (_PyUnicode_EqualShared((PyASCIIObject *)ue->me_key, 
                                       (PyASCIIObject *)key)) {
                return entry;  // НАЙДЕН!
            }
        }
        
        // Робин-худ: ищем "бедного" соседа
        perturb >>= PERTURB_SHIFT;
        i = (i * 5 + 1 + perturb) & mask;
        if (!freeslot) {
            freeslot = first_removed;
        }
    }
}
```

**Объяснение для людей:** Вычисляем `hash(key) % размер`. Если слот пустой — останавливаемся. Если занят — проверяем
ключ. Если не наш — делаем **робин-худ шаг** `(i*5 + 1 + perturb) % size`, пока не найдём или не упрёмся в пустой слот.

## 5. Вставка: insertdict()

```c
static int insertdict(PyDictObject *mp, PyObject *key, Py_ssize_t hash, 
                      PyObject *value, PyDictUnicodeEntry *entries) {
    PyDictKeyEntry *ep;
    size_t ix;                 // Индекс в хеш-таблице
    PyDictKeysObject *keys = mp->ma_keys;
    
    // Находим место для вставки
    ep = lookdict_unicode(keys, key, hash);
    
    if (ep->me_key != NULL && ep->me_key != DKIX_DUMMY) {
        // Ключ уже существует - заменяем значение
        if (mp->ma_values) {
            PyDictValues *values = mp->ma_values;
            Py_XSETREF(values->values[ep - entries], Py_NewRef(value));
        } else {
            Py_XSETREF(ep->me_value, Py_NewRef(value));
        }
        return 0;
    }
    
    // Проверяем load factor (2/3)
    if (keys->dk_usable == 0) {
        // Таблица заполнена - увеличиваем в 2 раза
        if (make_keys_object(mp, 2 * DK_SIZE(keys)) < 0) {
            return -1;
        }
        keys = mp->ma_keys;
        ep = lookdict_unicode(keys, key, hash);
    }
    
    // Вставляем новую запись
    PyDictUnicodeEntry *ue = (PyDictUnicodeEntry *)ep;
    ue->me_key = Py_NewRef(key);
    ue->me_hash = hash;
    if (mp->ma_values) {
        mp->ma_values->values[ep-entries] = Py_NewRef(value);
    } else {
        ue->me_value = Py_NewRef(value);
    }
    
    keys->dk_nentries++;       // +1 активная запись
    keys->dk_usable--;         // -1 свободный слот
    mp->ma_used++;
    
    return 0;
}
```

**Объяснение для людей:** Ищем место через lookdict. Если ключ есть — меняем значение. Если таблица заполнена на 2/3 —
удваиваем размер. Копируем ключ/значение (+refcnt).

## 6. Resize: make_keys_object()

```c
static int make_keys_object(PyDictObject *mp, Py_ssize_t newsize) {
    PyDictKeysObject *oldkeys = mp->ma_keys;
    PyDictKeysObject *newkeys;
    
    // Округляем до степени двойки (8, 16, 32...)
    newsize = estimate_new_capacity(mp->ma_used, newsize);
    
    // Создаём новые ключи
    newkeys = raw_make_keys(newsize);
    if (newkeys == NULL) {
        return -1;
    }
    
    // Перехешируем все старые элементы
    PyDictKeyEntry *oldentries = oldkeys->dk_entries;
    for (Py_ssize_t i = 0; i < mp->ma_used; i++) {
        PyDictKeyEntry *old_ep = &oldentries[i];
        if (old_ep->me_key != NULL && old_ep->me_key != DKIX_DUMMY) {
            insertdict(mp, old_ep->me_key, old_ep->me_hash, 
                      old_ep->me_value, newkeys->dk_entries);
        }
    }
    
    // Заменяем ключи атомарно
    _PyDict_SetKeys(mp, newkeys);
    
    // Уменьшаем refcnt старых ключей
    DK_DECREF(oldkeys);
    
    return 0;
}
```

**Объяснение для людей:** При 2/3 заполнении создаём новую таблицу вдвое больше, перехешируем **все** элементы заново,
атомарно меняем указатель `ma_keys`.

## 7. Атомарные операции (Python 3.9+)

```c
// Атомарная загрузка индекса (relaxed memory order)
#define LOAD_INDEX(keys, size, idx) \
    _Py_atomic_load_int##size##_relaxed(&((const int##size##_t*)keys->dk_indices)[idx])

// Атомарное сохранение индекса (release memory order)
#define STORE_INDEX(keys, size, idx, value) \
    _Py_atomic_store_int##size##_release(&((int##size##_t*)keys->dk_indices)[idx], (int##size##_t)value)

// При добавлении записи (thread-safe)
static inline void split_keys_entry_added(PyDictKeysObject *keys) {
    // Атомарно увеличиваем счётчики
    _Py_atomic_fetch_add_ssize_relaxed(&keys->dk_nentries, 1);
    _Py_atomic_fetch_sub_ssize_release(&keys->dk_usable, 1);
}
```

**Объяснение для людей:** В многопоточной среде (GIL released) индексы `dk_indices[]` обновляются атомарно. `release`
гарантирует видимость изменений.

## 8. Shared keys для классов (экономия памяти)

```c
// Классовые словари используют общие PyDictKeysObject
static PyDictKeysObject *class_keys = NULL;

PyObject *PyDict_FromKeys(PyObject *keys, PyObject *values) {
    // Создаём shared keys для классов
    if (PyType_Check(values)) {
        class_keys = intern_keys(keys);
        Py_INCREF(class_keys);
    }
}
```

**Объяснение для людей:** Все экземпляры класса `class C:` делят один `PyDictKeysObject` с одинаковыми именами
атрибутов. Значения хранятся отдельно в `ma_values[]`.

## 9. Байткод: DICT_MERGE (PEP 584, 3.9+)

```
# d1 |= d2  # Байткод:
DICT_MERGE   1     # Объединяем словари
```

```c
case DICT_MERGE: {
    PyObject *update = PEEK(oparg);  // Второй словарь
    PyObject *target = PEEK(oparg + 1);  // Первый словарь
    
    if (!_PyDict_MergeEx(target, update, oparg)) {
        goto error;
    }
    Py_DECREF(update);
    DISPATCH_SAME_OPARG(2);
}
```

**Объяснение для людей:** `d1 |= d2` вызывает `_PyDict_MergeEx` (атомарное объединение с приоритетом правого словаря).

**Хеш-таблица** в CPython 3.9+ — **split table** (`PyDictKeysObject` + `ma_values[]`), **робин-худ хеширование**, *
*атомарные индексы**, **shared keys** для классов, resize при 2/3 load factor, vectorcall поддержка.

- [Содержание](#содержание)

---

# *Встроенные функции*

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

## **Senior Level**

В CPython 3.9+ **встроенные функции** реализуются через **PyCFunctionObject**/**PyCMethodObject** в модуле
`bltinmodule.c`, регистрируемые в `__builtins__` через `builtin_functions[]`. Поддерживают **vectorcall** (
METH_FASTCALL) и классические **METH_VARARGS**. `Python/bltinmodule.c`,`Objects/methodobject.c`

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

**Объяснение для людей:** Встроенная функция `len()` в памяти — это 64-байт структура: заголовок + указатель на
C-функцию `len_func` + `__module__="builtins"`. `vectorcall` — быстрый путь вызова без tuple.

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

**Объяснение для людей:** Это **таблица методов** — массив структур `{имя, C_функция, флаги, docstring}`. `builtin_len`
принимает METH_O (1 аргумент). Регистрируется в `__builtins__`.

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

**Объяснение для людей:** `PyInit_builtins()` создаёт модуль `builtins`, проходит по таблице `builtin_functions[]`,
вызывает `PyCFunction_NewEx` (создаёт PyCFunctionObject для каждой функции), кладёт в `__dict__` модуля как `len`,
`print`, `abs`.

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

**Объяснение для людей:** `len(obj)` → `cfunction_vectorcall_FASTCALL` → `builtin_len(NULL, obj)` → C-функция получает
`self=NULL`, `arg=obj`. Без создания tuple/list — **максимальная скорость**.

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

**Объяснение для людей:** `len(lst)` → `PyObject_Length(lst)` → `PyList_Type.tp_as_sequence->sq_length(lst)` →
`list_length()` возвращает `self->ob_size`. Если нет `sq_length` — пробует `__len__()`.

## 7. list_length - sq_length для PyListObject

```c
static Py_ssize_t list_length(PyListObject *self) {
    return Py_SIZE(self);              // Просто ob_size из PyVarObject
}
```

**Объяснение для людей:** `len([1,2,3])` возвращает `ob_size=3` из заголовка PyVarObject. **Мгновенно** — поле в самом
объекте.

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

**Объяснение для людей:** `LOAD_GLOBAL len` ищет имя `"len"` сначала в `globals()`, потом в `builtins`. Находит
PyCFunctionObject, увеличивает refcnt, кладёт на стек.

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

**Объяснение для людей:** `CALL 1` → `_PyObject_Vectorcall(len_obj, [lst], 1, NULL)` → `_PyCFunction_Vectorcall` →
`builtin_len(NULL, lst)`. **Без tuple создания**.

**Встроенные функции** в CPython 3.9+ — **PyCFunctionObject** из `bltinmodule.c`, **vectorcall** (METH_FASTCALL) для
скорости, **PyMethodDef таблица**, поиск через `LOAD_GLOBAL` → `globals/builtins`, диспетч `PyObject_Length` →
`sq_length`.

- [Содержание](#содержание)

---

# *Контекстные менеджеры*

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

**Объяснение для людей:** `BEFORE_WITH` вызывает `__enter__`, `STORE_FAST` сохраняет результат в `var`.
`WITH_EXCEPT_START` вызывает `__exit__(exc_type, exc_val, tb)` при выходе/исключении.

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

**Объяснение для людей:** `BEFORE_WITH` ищет `ctx.__enter__()` через `_PyObject_LookupSpecial` (быстрый поиск по MRO),
вызывает без аргументов, кладёт результат на стек вместо `ctx`.

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

**Объяснение для людей:** При исключении/выходе из `with` берём сохранённое `(exc_type, exc_val, tb)`, вызываем
`__exit__(exc_info)`, если возвращает `True` — **подавляем** исключение.

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

**Объяснение для людей:** Создаёт пустой tuple `(None,None,None)` для будущего исключения, ищет `__exit__`, кладёт оба
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

**Объяснение для людей:** Ищет `__enter__`/`__exit__` **только в типе**, не в `__dict__` экземпляра (быстрее).
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

**Объяснение для людей:** `@contextmanager` оборачивает генератор в класс с `__enter__` (next(gen)) и `__exit__` (
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

**Объяснение для людей:** `open('file.txt')` возвращает PyFileIOObject. `__enter__` просто +refcnt, `__exit__` вызывает
`f.close()`.

**Контекстные менеджеры** в CPython 3.9+ — байткоды `BEFORE_WITH` (вызов `__enter__`), `WITH_EXCEPT_START` (вызов
`__exit__(exc_info)`), структуры для сохранения `exc_info` tuple, быстрый поиск через `_PyObject_LookupSpecial`.

- [Содержание](#содержание)

---

# **Генераторы и итераторы**

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

**Объяснение для людей:** Генератор — это **замороженный фрейм** выполнения. `gi_frame` указывает, где остановились (
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

**Объяснение для людей:** `CALL_FUNCTION gen()` создаёт PyGenObject с `FRAME_CREATED`. Первый `next(g)` → `RESUME 0`
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

**Объяснение для людей:** `YIELD_VALUE 1` берёт `1` со стека, сохраняет в фрейме, возвращает **сам генератор**
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

**Объяснение для людей:** `next(g)` → `gen_send_ex(g, Py_None, 0)`: берёт фрейм из `gi_frame`, кладёт `Py_None` на
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

**Объяснение для людей:** Фрейм имеет **4 состояния**. После `yield` → `FRAME_SUSPENDED`. `RESUME oparg` восстанавливает
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

**Объяснение для людей:** `RESUME 0` — первый запуск (до первого yield). `RESUME 1` — возобновление после yield.
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

**Объяснение для людей:** `for x in lst:` → `lst.__iter__()` → PyListIterObject с `it_index=0`. Каждый `tp_iternext`
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

**Объяснение для людей:** `iter(lst)` → `PyList_Type.tp_iter(lst)` → PyListIterObject. Если нет `tp_iter` — вызывает
`obj.__iter__()`.

**Генераторы/итераторы** в CPython 3.9+ — **PyGenObject** с **замороженным _PyInterpreterFrame**, состояния
`FRAME_SUSPENDED`, байткоды `YIELD_VALUE`/`RESUME`, протокол `tp_iter`/`tp_iternext`, `PyObject_GetIter` диспетчер.

- [Содержание](#содержание)

---

# **Декораторы и замыкания**

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

**Объяснение для людей:** Функция с замыканием содержит `func_closure` — массив **ячеек** (PyCellObject), ссылающихся на
переменные внешней функции. `func_nfreevars` говорит, сколько их.

## 2. PyCellObject - "ячейка" замыкания

```c
typedef struct {
    PyObject_HEAD                 // refcnt + type=&PyCell_Type
    PyObject *ob_ref;             // Указатель на значение переменной
} PyCellObject;
```

**Объяснение для людей:** **Ячейка** — это обёртка над PyObject*. Внешняя функция пишет `x=42` →
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

**Объяснение для людей:** Компилятор помечает переменные: `co_cellvars` — в **внешней** функции (нужны для вложенных),
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

**Объяснение для людей:** `STORE_DEREF 0` пишет в ячейку #0. `LOAD_CLOSURE 0` берёт **саму ячейку**,
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

**Объяснение для людей:** `LOAD_CLOSURE 0` берёт ячейку из массива замыкания функции (`func_closure[0]`), кладёт *
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

**Объяснение для людей:** `STORE_DEREF` берёт значение со стека → `PyCell_SET(cell, value)` (меняет `cell->ob_ref`).
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

**Объяснение для людей:** `MAKE_CLOSURE 1` берёт `(cell0,)`, PyCodeObject → создаёт PyFunctionObject →
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

**Объяснение для людей:** `PyCell_Set(cell, 42)` → `cell->ob_ref = PyLong(42)`. `PyCell_Get(cell)` → возвращает
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

**Объяснение для людей:** Компилятор сканирует AST: если `inner()` читает `x` из `outer()` → `x` становится **cellvar**
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

**Объяснение для людей:** Декоратор — это **функция**, возвращающая **другую функцию** с замыканием на оригинальную
`func`. `@decorator` → `decorator(f)` во время выполнения модуля.

**Декораторы/замыкания** в CPython 3.9+ — **PyCellObject** (`ob_ref`), `co_cellvars`/`co_freevars`, байткоды
`LOAD_CLOSURE`/`STORE_DEREF`/`MAKE_CLOSURE`, `func_closure` tuple в PyFunctionObject.

- [Содержание](#содержание)

---

# **GIL (Global Interpreter Lock)**

## **Junior Level**

Представьте, что у вас есть большой офис с несколькими сотрудниками (это потоки) и всего один принтер (это интерпретатор
Python). Все сотрудники могут готовить документы одновременно, но печатать они могут только по очереди — когда принтер
свободен. GIL — это как очередь к этому единственному принтеру.

Технически GIL — это глобальная блокировка в CPython (стандартной реализации Python), которая позволяет выполнять только
одному потоку Python-кода за раз, даже если у вас многоядерный процессор. Это значит, что для задач, которые сильно
нагружают процессор (например, сложные вычисления), многопоточность в Python не даст ускорения. Все потоки будут по
очереди работать на одном ядре.

Но есть важный нюанс. Во время операций ввода-вывода — когда программа ждет ответа от сети, читает файл или общается с
базой данных — поток освобождает GIL, позволяя другим потокам работать. Поэтому для I/O-задач (например, веб-серверов)
многопоточность всё равно полезна.

## **Middle Level**

GIL (Global Interpreter Lock) — это мьютекс (mutex), блокирующий одновременное выполнение нескольких потоков в CPython.
Только один поток может владеть GIL в каждый момент времени, что делает любую многопоточную Python-программу фактически
однопоточной при выполнении байткода.

GIL был введён для упрощения управления памятью CPython и обеспечения интеграции с C-расширениями. Система подсчёта
ссылок (reference counting) в CPython не является потокобезопасной: без защиты несколько потоков могут одновременно
изменять счётчик ссылок объекта, что приводит к утечкам памяти или преждевременному удалению объектов.

Альтернативой GIL могла бы быть установка блокировок на каждый объект Python, но это привело бы к:

- Значительному overhead на множественные блокировки/разблокировки при каждой операции
- Высокому риску взаимоблокировок (deadlocks)
- Снижению производительности однопоточных программ

Вместо этого CPython использует единую глобальную блокировку интерпретатора, что исключает взаимоблокировки и
минимизирует влияние на производительность однопоточного кода.

Механизм работы GIL довольно интересен. Поток удерживает GIL не бесконечно — есть несколько условий, при которых
происходит переключение:

1. **Через определённое количество тиков байт-кода** (обычно каждые 100 инструкций)
2. **При операциях ввода-вывода** — когда поток уходит в ожидание
3. **При явном освобождении** в C-расширениях

Это создаёт некоторые проблемы. Например, если у вас есть поток, который выполняет долгие вычисления без операций
ввода-вывода, он может долго не отдавать GIL, и другие потоки будут простаивать. Это называется "голоданием" потоков.

На практике это означает, что для CPU-интенсивных задач нужно использовать другие подходы. Самый распространённый — *
*многопроцессорность** через модуль `multiprocessing`. Каждый процесс получает свой интерпретатор Python со своим GIL, и
они действительно могут работать параллельно на разных ядрах.

Есть и другие способы обойти ограничения GIL. **Асинхронное программирование** с `asyncio` позволяет эффективно работать
с I/O-задачами в одном потоке. **C-расширения** могут временно освобождать GIL во время вычислительных операций — так
работают библиотеки типа NumPy и SciPy. И существуют альтернативные реализации Python, такие как Jython или IronPython,
где GIL вообще отсутствует, но они имеют свои ограничения.

Важно понимать, что GIL — это особенность именно CPython, и у него есть свои причины для существования. Он упрощает
реализацию интерпретатора и делает более предсказуемой работу с памятью. Для многих реальных задач — веб-серверов,
скриптов автоматизации, работы с базами данных — GIL не является узким местом. Проблемы возникают в основном в научных
вычислениях и высоконагруженных вычислительных задачах, где как раз и используются специализированные библиотеки и
подходы.

Понимание GIL помогает выбрать правильную архитектуру для приложения. Если задача
CPU-интенсивная — смотрим в сторону многопроцессорности или выноса вычислений в C-расширения. Если I/O-интенсивная —
можно использовать потоки, асинхронное программирование или комбинацию подходов. GIL — это не приговор, а особенность,
которую нужно учитывать при проектировании.

## **Senior Level**

В CPython 3.9+ **GIL** (Global Interpreter Lock) — это **мьютекс** `gil->mutex` + **счётчик** `gil->recursion_count` + *
*состояние потока** `tstate->holds_gil` в `_gil_runtime_state`. **PEP 684** (3.12) добавил **per-interpreter GIL**. *
*Free-threaded** (3.13) — скомпилировано с `Py_GIL_DISABLED`. `Python/ceval_gil.c`,`Python/pythonrun.c`

## 1. _gil_runtime_state (Python/ceval_gil.c)

```c
struct _gil_runtime_state {
    _Py_atomic_int locked;             // 1=GIL занят, 0=свободен (атомарно)
    _Py_atomic_int recursion_count;    // Счётчик вложенных захватов
    PyThread_type_lock mutex;          // pthread_mutex_t или Windows CRITICAL_SECTION
    PyThread_cond_t cond;              // pthread_cond_t для ожидания
    PyThread_t owner;                  // ID потока-владельца
    int switch_interval;               // Интервал принудительного drop_gil
    uint64_t last_switch_time;         // Время последнего drop_gil
#ifdef Py_GIL_DISABLED
    int enabled;                       // 0=GIL отключен полностью
#endif
};
```

**Объяснение для людей:** GIL — это **один глобальный мьютекс** + **счётчик рекурсии** (один поток может захватить много
раз). `owner` — ID потока, который держит GIL.

## 2. PyEval_AcquireLock / take_gil() — захват GIL

```c
void _PyEval_AcquireLock(PyThreadState *tstate) {
    _Py_EnsureTstateNotNULL(tstate);   // tstate не NULL
    take_gil(tstate);                  // Захватываем GIL
}

static void take_gil(PyThreadState *tstate) {
    struct _gil_runtime_state *gil = &tstate->interp->ceval.gil;
    
    // Атомарно проверяем, свободен ли GIL
    if (_Py_atomic_load_int_relaxed(&gil->locked)) {
        // GIL занят — ждём
        MUTEX_LOCK(gil->mutex);
        while (_Py_atomic_load_int_relaxed(&gil->locked)) {
            COND_WAIT(gil->cond, gil->mutex);  // pthread_cond_wait
        }
        MUTEX_UNLOCK(gil->mutex);
    }
    
    // Атомарно захватываем GIL
    _Py_atomic_store_int_relaxed(&gil->locked, 1);
    
    // Устанавливаем владельца
    gil->owner = PyThread_get_thread_ident();
    gil->recursion_count = 1;
    
    // Отмечаем поток как держателя GIL
    tstate->holds_gil = 1;
    
    // Обновляем eval_breaker (прерывания)
    update_eval_breaker_for_thread(tstate->interp, tstate);
}
```

**Объяснение для людей:** Поток проверяет `gil->locked` атомарно. Если 1 — **ждёт** на condition variable. Захватывает →
`locked=1`, `owner=мой_ID`, `tstate->holds_gil=1`.

## 3. PyEval_ReleaseLock / drop_gil() — освобождение GIL

```c
void _PyEval_ReleaseLock(PyThreadState *tstate) {
    drop_gil(tstate->interp, tstate, 0);  // 0=не финальное освобождение
}

static void drop_gil(PyInterpreterState *interp, PyThreadState *tstate, int final) {
    struct _gil_runtime_state *gil = &interp->ceval.gil;
    
#ifdef Py_GIL_DISABLED
    if (!gil->enabled) {
        return;                        // GIL отключен — выходим
    }
#endif
    
    // Проверяем, что мы владелец
    if (!_Py_atomic_load_int_relaxed(&gil->locked) || 
        gil->owner != PyThread_get_thread_ident()) {
        Py_FatalError("drop_gil: GIL is not locked");
    }
    
    // Уменьшаем счётчик рекурсии
    if (--gil->recursion_count > 0) {
        return;                        // Ещё вложенные захваты
    }
    
    // Сбрасываем флаги потока
    tstate->holds_gil = 0;
    
    // Освобождаем GIL атомарно
    _Py_atomic_store_int_release(&gil->locked, 0);
    
    // Разбудить ждущие потоки
    MUTEX_LOCK(gil->mutex);
    PyThread_cond_broadcast(gil->cond);  // pthread_cond_broadcast
    MUTEX_UNLOCK(gil->mutex);
}
```

**Объяснение для людей:** `--recursion_count`. Если >0 — остаёмся владельцем. Иначе `holds_gil=0`, `locked=0`, **будим
ВСЕ** ждущие потоки (`broadcast`).

## 4. Автоматический drop_gil в ceval.c (каждые N инструкций)

```c
#define INSTRUCTION_COUNTER() \
    if (--tstate->cframe->instr_counter == 0) { \
        tstate->cframe->instr_counter = INSTR_COUNTER_STEP; \
        _PyEval_SignalAsyncioEventLoop(tstate); \
    }

#define PyEval_EvalFrameDefault _PyEval_EvalFrameDefault

static inline void
frame_insn_counter(PyThreadState *tstate, _PyInterpreterFrame *frame) {
    if (_Py_atomic_load_relaxed(&tstate->gilstate_counter) == 0) {
        // GIL счётчик истёк — пробуем освободить
        _PyEval_ReleaseLock(tstate);
        _PyEval_AcquireLock(tstate);
    }
}
```

**Объяснение для людей:** Каждые ~1000 инструкций байткода проверяется `gilstate_counter`. Если 0 — **drop_gil() +
take_gil()** (шанс другому потоку).

## 5. PyGILState_Ensure/Release — C API

```c
PyGILState_STATE PyGILState_Ensure(void) {
    PyThreadState *tstate = PyThreadState_Get();  // Текущий поток
    
    if (tstate == NULL) {
        tstate = _PyThreadState_GetUnattached();  // Создаём новый
        if (tstate == NULL) {
            Py_FatalError("PyGILState_Ensure: no thread state");
        }
    }
    
    // Атомарно увеличиваем счётчик
    int gilstate_counter = _Py_atomic_fetch_add_int_relaxed(
        &tstate->interp->ceval.gil.gilstate_counter, 1);
    
    if (gilstate_counter == -1) {
        // Первый захват — берём GIL
        _PyEval_AcquireLock(tstate);
        return PyGILState_LOCKED;
    }
    
    return PyGILState_UNLOCKED;
}

void PyGILState_Release(PyGILState_STATE oldstate) {
    PyThreadState *tstate = PyThreadState_Get();
    
    // Атомарно уменьшаем счётчик
    int gilstate_counter = _Py_atomic_fetch_sub_int_relaxed(
        &tstate->interp->ceval.gil.gilstate_counter, 1);
    
    if (gilstate_counter == 0) {
        // Последний — освобождаем GIL
        _PyEval_ReleaseLock(tstate);
    }
}
```

**Объяснение для людей:** C-расширения вызывают `PyGILState_Ensure()` → атомарно `++gilstate_counter`. Если был -1 →
захват GIL. `Release()` → `--counter`, если 0 → drop_gil.

## 6. Per-interpreter GIL (PEP 684, 3.12+)

```c
// Каждый PyInterpreterState имеет свой GIL
struct _ceval_state {
    struct _gil_runtime_state gil;     // GIL состояния интерпретатора
    int own_gil;                       // Этот интерпретатор владеет GIL
    // ...
};

PyInterpreterState *PyInterpreterState_New(void) {
    PyInterpreterState *interp = PyMem_Calloc(1, sizeof(*interp));
    init_own_gil(interp, &interp->ceval.gil);  // Создаём GIL для интерпретатора
    return interp;
}
```

**Объяснение для людей:** **PEP 684**: каждый subinterpreter имеет **свой GIL**. `Py_NewInterpreter()` создаёт отдельный
`interp->ceval.gil`.

## 7. Free-threaded CPython (PEP 703, 3.13+)

```c
#ifdef Py_GIL_DISABLED
static inline void take_gil(PyThreadState *tstate) {
    struct _gil_runtime_state *gil = &tstate->interp->ceval.gil;
    if (!gil->enabled) {               // GIL отключен
        tstate->holds_gil = 1;
        return;
    }
    // Обычная логика захвата
}
#endif
```

**Объяснение для людей:** `--disable-gil` компиляция: `gil->enabled=0`. Захват/освобождение — **no-op**. Потоки работают
**параллельно**.

## 8. Eval breaker integration

```c
void _PyEval_SignalAsyncioEventLoop(PyThreadState *tstate) {
    struct _ceval_state *ceval = &tstate->interp->ceval;
    
    if (ceval->gil.enabled && tstate->holds_gil) {
        // Устанавливаем бит "drop GIL request"
        _Py_atomic_store_relaxed(&ceval->eval_breaker, _PY_EVAL_BREAKER_DROP_GIL);
    }
}
```

**Объяснение для людей:** `asyncio`/`signal` сигнализируют через `eval_breaker`. Если держим GIL — бит
`_PY_EVAL_BREAKER_DROP_GIL` → следующий `drop_gil()` прерывается.

## 9. Байткод без GIL (3.13 free-threaded)

```c
case LOAD_GLOBAL: {
#ifdef Py_GIL_DISABLED
    if (!tstate->holds_gil) {
        // Без GIL — атомарный поиск
        res = _PyDict_LookupWithCache(global_dict, name, hash);
    } else {
        // С GIL — обычный поиск
        res = PyDict_GetItemWithCache(global_dict, name, hash);
    }
#else
    // GIL версия
#endif
}
```

**Объяснение для людей:** Free-threaded использует **атомарные** `PyDict_LookupWithCache`. GIL версия — обычный поиск.

**GIL** в CPython 3.9+ — **_gil_runtime_state** (`mutex + recursion_count`), **take_gil/drop_gil**, **PyGILState_Ensure
** (C API), **per-interp GIL** (3.12), **free-threaded** (`Py_GIL_DISABLED`, 3.13).

- [Содержание](#содержание)

---

# **Изменение коллекции во время итерации**

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

**Объяснение для людей:** Итератор списка копирует `list->ob_version` при создании. Каждый `next()` проверяет версии —
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

**Объяснение для людей:** Перед чтением элемента сравниваем `it_version` (копия при создании) с `list->ob_version` (
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

**Объяснение для людей:** **Любое** изменение списка (`append`, `pop`, `del lst[i]`, `lst[i]=x`) → `self->ob_version++`.
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

**Объяснение для людей:** Словарь имеет **64-битный атомарный** `ma_version_tag`. Изменение (set/del) →
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

**Объяснение для людей:** `for k in d:` создаёт PyDictIterObject с копией `d->ma_version_tag`. Каждый `next()` проверяет
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

**Объяснение для людей:** **Любое** изменение словаря (`d[k]=v`, `del d[k]`, `d.clear()`) → атомарное
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

**Объяснение для людей:** Множества работают **идентично** словарям: `version_tag`, проверка в итераторе → **"set
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

**Объяснение для людей:** **Неизменяемые** типы (tuple, str, bytes) **НЕ** проверяют версию — они **физически** не могут
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

**Объяснение для людей:** `iter(lst)` → PyListIterObject → `it_version = lst->ob_version`. **Любое** изменение списка →
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

**Объяснение для людей:** `FOR_ITER` вызывает `it->tp_iternext()` → проверка версии → RuntimeError или значение.
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

**Объяснение для людей:** Пользовательские классы **могут** проверять версии в `__next__()`.

**Изменение коллекции** в CPython 3.9+ детектируется **version tag** (`ob_version`/`ma_version_tag`) в
PyListObject/PyDictObject + проверкой в `tp_iternext()` итераторов → **RuntimeError** при несоответствии.

- [Содержание](#содержание)

---

### **Области видимости**

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

**Объяснение для людей:** Компилятор создаёт таблицу символов для каждой функции/класса/модуля. Для каждого имени `x`
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

**Объяснение для людей:** Компилятор проходит по AST, собирает все `x=1`, `for x in y`, `def f(x):`. Затем
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

**Объяснение для людей:** `LOAD_FAST 3` = `f_localsplus[3]` (массив указателей в фрейме). **Мгновенно** — без
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

**Объяснение для людей:** `LOAD_NAME "x"` → **3 поиска**: locals dict → globals dict → builtins dict. **Кеш** `co_namei`
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

**Объяснение для людей:** `LOAD_GLOBAL "print"` → **только** globals + builtins (НЕ locals). **Двойной кеш** (16 бит): 8
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

**Объяснение для людей:** `LOAD_DEREF 0` → `func_closure[0]->ob_ref` (значение ячейки). `STORE_DEREF 0` →
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

**Объяснение для людей:** Компилятор по флагам символа генерирует **правильный** опкод: LOCAL→FAST, FREE→DEREF,
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

**Объяснение для людей:** **Единый массив** `localsplus[]`: сначала параметры/локальные (`co_varnames`), потом ячейки (
`co_cellvars`), потом свободные (`co_freevars`). `LOAD_FAST 0` = `localsplus[0]`.

## 9. Кеш LOAD_NAME/LOAD_GLOBAL (co_opcache)

```c
typedef struct _PyCodeObject {
    // ...
    uint16_t *co_opcache;              // Кеш для LOAD_NAME/LOAD_GLOBAL
    // ...
} PyCodeObject;
```

**Объяснение для людей:** `co_opcache[oparg]` хранит **индекс** в `co_varnames` или хеш для globals/builtins. Промах →
полный поиск.

**Области видимости** в CPython 3.9+ — **symtable** (флаги DEF_*), компилятор → `LOAD_FAST`/`LOAD_NAME`/`LOAD_GLOBAL`/
`LOAD_DEREF`, поиск localsplus→locals→globals→builtins, кеш `co_opcache`.

- [Содержание](#содержание)

---

# **Lambda-функции**

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

**Объяснение для людей:** `lambda x: x+1` → компилятор создаёт **отдельный scope** с именем `<lambda>`, компилирует тело
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

**Объяснение для людей:** Lambda — **обычная функция** с **автогенерированным** PyCodeObject `<lambda>`.
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

**Объяснение для людей:** `MAKE_FUNCTION 0` берёт PyCodeObject → создаёт PyFunctionObject → `func_name="<lambda>"`,
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

**Объяснение для людей:** Lambda = PyFunctionObject с `func_code=<lambda bytecode>`, `func_globals=текущие globals`,
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

**Объяснение для людей:** `lambda_func(42)` → `CALL_FUNCTION 1` → `_PyObject_Vectorcall(lambda, [42], 1, NULL)` →
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

**Объяснение для людей:** Lambda выполняется в **новом фрейме** с `f_localsplus[0]=x=42`. `LOAD_FAST 0` → `42`,
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

**Объяснение для людей:** `lambda x: x + y` захватывает `y` в **ячейку**. `outer(10)` → lambda с
`func_closure=(cell_y=10)`. Вызов → `LOAD_DEREF 0` читает `cell_y->ob_ref=10`.

## 8. Различия lambda vs def (компилятор)

```c
// compiler_function() для def f():
COMPILER_SCOPE_FUNCTION  // co_flags |= CO_NEWLOCALS

// compiler_lambda() для lambda:
COMPILER_SCOPE_LAMBDA    // co_flags без CO_NEWLOCALS (использует globals)
```

**Объяснение для людей:** **def** создаёт **локальные** переменные (`f_localsplus`). **lambda** — **выражение**,
использует **globals** внешней функции (без CO_NEWLOCALS).

**Lambda** в CPython 3.9+ — **PyCodeObject** из `compiler_lambda()`, `MAKE_FUNCTION 0`, **отдельный PyFrameObject** при
вызове, **без локальных** (CO_NEWLOCALS=0), поддержка замыканий через `LOAD_DEREF`.

- [Содержание](#содержание)

---

# *Comprehensions и генераторные выражения*

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

**Объяснение для людей:** `[x**2 for x in range(10)]` → **отдельная функция** `<listcomp>` с **своим фреймом**.
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

**Объяснение для людей:** **3.12**: comprehension **встроен** в байткод. `LOAD_FAST_AND_CLEAR 1 (.0)` сохраняет
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

**Объяснение для людей:** Компилятор создаёт **scope COMPILER_SCOPE_COMPREHENSION** (без CO_NEWLOCALS), компилирует
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

**Объяснение для людей:** `LIST_APPEND 2` берёт **список из слота .0** (не со стека!), значение `x**2`, вызывает
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

**Объяснение для людей:** Генераторное выражение — **функция-генератор** с `yield x**2`. `LOAD_GENEXPR` создаёт
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

**Объяснение для людей:** Генераторное выражение компилируется как **функция с CO_GENERATOR** флагом + `yield`.
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

**Объяснение для людей:** `MAP_ADD 3` добавляет `key=value` в словарь **фиксированной позиции** (аналог LIST_APPEND).

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

**Объяснение для людей:** Comprehension использует **over-allocation** списка (allocated > ob_size). `LIST_APPEND` → *
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

**Объяснение для людей:** Async genexp → **async generator** с `async for`/`await` в байткоде.

**Comprehensions** в CPython 3.9+ — **отдельные PyCodeObject** (`COMPILER_SCOPE_COMPREHENSION`), **LIST_APPEND**/*
*MAP_ADD** с фиксированным списком/словарём в `.0`, **PEP 709 inline** (3.12+ без function call), **over-allocation**
оптимизация.

- [Содержание](#содержание)

---

# **copy() и deepcopy()**

## **Junior Level**

Когда вам нужно создать независимую копию объекта в Python, на помощь приходят функции `copy()` и `deepcopy()` из модуля
`copy`. Разница между ними напоминает разницу между созданием ярлыка на папку или её копирование.

`copy()` создаёт поверхностную (shallow) копию — новый "контейнер", который, однако, продолжает ссылаться на те же
внутренние объекты, что и оригинал. Представьте, что вы сделали копию папки с документами: сама папка новая, но лежащие
в ней файлы — те же самые. Если вы измените содержимое файла в копии папки, изменения отразятся и в оригинале, потому
что это один и тот же физический файл.

`deepcopy()` выполняет глубокое копирование — она рекурсивно создаёт полностью независимые копии всех объектов, включая
вложенные структуры. Это похоже на то, как если бы вы не только создали новую папку, но и перепечатали каждый документ
внутри неё. После такой операции изменения в копии никак не затронут оригинал, и наоборот.

Выбор между этими двумя функциями зависит от ситуации. Если вы работаете с неизменяемыми объектами (числа, строки,
кортежи) или уверены, что не будете менять вложенные элементы, достаточно `copy()`. Когда же требуется полная изоляция —
например, для тестовых данных, конфигураций или сложных изменяемых структур — без `deepcopy()` не обойтись.

## **Middle Level**

Под поверхностью этих функций скрывается довольно интересная механика. Поверхностное копирование создаёт новый объект
того же типа, но не углубляется в его содержимое — все вложенные элементы остаются общими с оригиналом. Это происходит
потому, что `copy()` копирует только ссылки, а не сами объекты. Такой подход эффективен по памяти и времени, но требует
осторожности: изменение вложенного изменяемого объекта в копии повлияет на оригинал.

Глубокое копирование устроено сложнее. Функция `deepcopy()` рекурсивно обходит всю структуру объекта, создавая новые
экземпляры для каждого встреченного изменяемого объекта. Она умеет корректно обрабатывать даже циклические ссылки (когда
объекты ссылаются друг на друга), используя внутренний словарь для отслеживания уже скопированных объектов и
предотвращения бесконечной рекурсии.

Обе функции уважают специальные методы объектов: если класс определяет `__copy__()` или `__deepcopy__()`, используются
эти реализации, что позволяет контролировать процесс копирования. Это особенно полезно для объектов со сложным
внутренним состоянием, например, открытых файлов или сетевых соединений, которые не могут быть просто продублированы.

Стоит помнить о некоторых особенностях. Некоторые объекты (модули, классы, функции) не копируются, а возвращаются как
есть, поскольку обычно в этом нет смысла. Также `deepcopy()` может быть довольно медленной для больших и сложных
структур из-за рекурсивного обхода и создания множества новых объектов.

На практике глубокое копирование незаменимо при работе с многомерными структурами данных, графами или любыми объектами,
где важна полная независимость копии. Поверхностное же копирование часто используется для создания "снимков" состояния
объекта в определённый момент, когда достаточно скопировать только верхний уровень структуры.

## **Senior Level**

`copy.copy()` и `copy.deepcopy()` — это чистый Python‑код в `Lib/copy.py`, который реализует общий протокол копирования
через поиск специальных методов, тип‑специфические таблицы и рекурсивный обход структуры с мемоизацией. Байткода как
такового там немного (обычные вызовы функций и атрибутов), вся «магия» — в логике диспетчеризации.

## copy.copy(): выбор стратегии

В CPython 3.9+ **copy.copy()** использует **tp_traverse** + `__copy__()` + **shallow copy** (`PyObject_Malloc`), *
*copy.deepcopy()** — **рекурсивный _deepcopy_dispatch** с **memo dict** (id→copy), `_reconstruct()` для классов, защита
от циклов. `Lib/copy.py`,`Objects/object.c`

## 1. copy.copy() - Lib/copy.py (Python C API)

```python
def copy(x):
    """Shallow copy operation on arbitrary Python objects."""
    cls = type(x)

    # 1. __copy__() метод класса
    copier = _copy_dispatch.get(cls)
    if copier is not None:
        return copier(x)

    try:
        # 2. Пробуем __copy__()
        cpy = x.__copy__()
        if hasattr(cpy, '__dict__'):
            cpy.__dict__.update(x.__dict__)
        return cpy
    except AttributeError:
        pass

    # 3. Встроенные типы
    if isinstance(x, dict):
        return dict(x)
    if isinstance(x, list):
        return list(x)
    if isinstance(x, set):
        return set(x)
    # ...

    # 4. Fallback: PyObject_CallMethod("__getnewargs__")
    newargs = _copy_call_getnewargs(x)
    return _reconstruct(x, len(newargs), newargs, ())
```

**Объяснение для людей:** `copy(lst)` → `list(lst)` (shallow), `copy(obj)` → `obj.__copy__()` или
`type(obj)(*obj.__getnewargs__())`. **Не рекурсивно** — копирует только верхний уровень.

## 2. copy.deepcopy() - рекурсивный dispatch (Lib/copy.py)

```python
def deepcopy(x, memo=None):
    """Deep copy operation on arbitrary Python objects."""
    if memo is None:
        memo = {}

    d = id(x)
    y = memo.get(d, None)  # Уже копировали?
    if y is not None:
        return y  # Цикл! Возвращаем копию

    cls = type(x)

    # 1. __deepcopy__()
    copier = _deepcopy_dispatch.get(cls)
    if copier is not None:
        y = copier(x, memo)

    # 2. Встроенные типы
    elif cls in _deepcopy_dispatch:
        y = _deepcopy_dispatch[cls](x, memo)

    # 3. Классы с __reduce__
    elif hasattr(x, '__reduce_ex__'):
        args = x.__reduce_ex__(2)[1]
        y = _reconstruct(x, 0, args, (), None, None, memo)

    else:
        # 4. Fallback: __getnewargs__ + __dict__ + __slots__
        reductor = getattr(x, "__reduce_ex__", None)
        if reductor is not None:
            args = reductor(4)[1]
        else:
            args = _copy_call_getnewargs(x)
        y = _reconstruct(x, len(args), args, (), None, None, memo)

    memo[d] = y  # Записываем в memo
    return y
```

**Объяснение для людей:** `deepcopy(obj)` рекурсивно копирует **все вложенные объекты**. `memo[id(obj)]=copy`
предотвращает **бесконечный цикл** при `l.append(l)`.

## 3. _deepcopy_dispatch таблица (Lib/copy.py)

```python
_deepcopy_dispatch = d = {}
d[dict] = _deepcopy_dict
d[list] = _deepcopy_list
d[set] = _deepcopy_set
d[tuple] = _deepcopy_tuple
d[frozenset] = _deepcopy_frozenset


def _deepcopy_dict(x, memo):
    y = {}
    memo[id(x)] = y
    for key, value in x.items():
        y[deepcopy(key, memo)] = deepcopy(value, memo)  # Рекурсия!
    return y


def _deepcopy_list(x, memo):
    y = []
    memo[id(x)] = y
    append = y.append  # Быстрый append
    for item in x:
        append(deepcopy(item, memo))  # Рекурсия!
    return y
```

**Объяснение для людей:** `_deepcopy_dict({a: [1]})` → `{copy(a): copy([1])}` → рекурсивно копирует **каждый**
ключ/значение. `memo` сохраняет уже сделанные копии.

## 4. PyList_Type.tp_richcompare для list.copy()

```c
static PyObject *list_copy(PyListObject *self) {
    Py_ssize_t len = Py_SIZE(self);
    PyObject **src = self->ob_item;
    PyObject **dest;
    
    // Создаём новый список той же ёмкости
    PyListObject *newlist = _PyList_New(len);  # Выделяем ob_item[len+overalloc]
    if (newlist == NULL)
        return NULL;
    
    dest = newlist->ob_item;
    
    // Копируем указатели (shallow!)
    for (Py_ssize_t i = 0; i < len; i++) {
        PyObject *v = src[i];
        Py_INCREF(v);              // +refcnt на каждый элемент
        dest[i] = v;
    }
    
    return (PyObject *)newlist;
}
```

**Объяснение для людей:** `lst.copy()` → `_PyList_New(len)` → копирует **указатели** `ob_item[]` с `Py_INCREF()`. *
*Shallow** — вложенные списки **не копируются**.

## 5. _PyDict_NewPresized() для dict.copy() (3.9+)

```c
PyObject *_PyDict_NewPresized(Py_ssize_t expected_size) {
    PyDictObject *mp;
    Py_ssize_t table_size = _PyDict_NewPresizedTableSize(expected_size);
    
    mp = PyObject_GC_New(PyDictObject, &PyDict_Type);
    if (mp == NULL)
        return NULL;
    
    mp->ma_used = 0;
    mp->ma_version_tag = DICT_NEXT_VERSION();
    mp->ma_table = PyMem_Calloc(table_size, sizeof(PyDictUnicodeEntry));
    mp->ma_keys = NULL;  // Создадим позже
    mp->ma_values = NULL;
    
    // Выделяем ma_keys той же ёмкости
    if (dictkeys_new(mp, table_size) < 0) {
        PyDictObject_Clear(mp);
        Py_DECREF(mp);
        return NULL;
    }
    
    _PyObject_GC_TRACK(mp);
    return (PyObject *)mp;
}
```

**Объяснение для людей:** `dict.copy()` → новый PyDictObject с **пустой** `ma_keys` + `ma_values`, затем
`dict_update(new, old)` копирует все пары **shallow**.

## 6. _reconstruct() для пользовательских классов

```python
def _reconstruct(x, len_args, args, state, listitems, dictitems, memo):
    # 1. Создаём новый экземпляр
    cls = type(x)
    newobj = object.__new__(cls)

    # 2. Восстанавливаем состояние
    if state is not None:
        state = deepcopy(state, memo)
        newobj.__dict__ = state
    elif hasattr(newobj, '__slots__'):
        # __slots__ копируем вручную
        for key, value in state.items():
            setattr(newobj, key, deepcopy(value, memo))

    # 3. Восстанавливаем mutable атрибуты
    if len_args > 0:
        newobj.__init__(*deepcopy(args, memo))

    return newobj
```

**Объяснение для людей:** `deepcopy(MyClass())` → `MyClass.__new__()` → `deepcopy(state)` → `__dict__` →
`__init__(*args)`. **Полная копия состояния**.

## 7. Защита от рекурсии: memo[id(obj)]

```python
l = [1, 2]
l.append(l)  # Цикл!
deep_l = deepcopy(l, memo={})

# Первый вызов: memo={} пустой
y = _deepcopy_list(l, memo)
memo[id(l)] = y  # Записали!

# Рекурсивный вызов l[3] = l:
item = deepcopy(l[3], memo)  # l[3] == l
d = id(l[3])  # == id(l)
y = memo.get(d)  # НАЙДЕН! Возвращаем копию
```

**Объяснение для людей:** `memo[id(original)]=copy` **предотвращает** копирование уже обработанных объектов.
`[1, [1, [1, ...]]]` копируется **один раз**.

## 8. C-level: PyObject_Malloc для shallow copy

```c
static PyObject *generic_copy(PyObject *self) {
    PyObject *newobj;
    
    // 1. Выделяем память той же структуры
    newobj = PyType_GenericAlloc(Py_TYPE(self), 1);
    if (newobj == NULL)
        return NULL;
    
    // 2. Копируем PyObject_HEAD
    Py_INCREF(Py_TYPE(self));
    newobj->ob_refcnt = 1;
    newobj->ob_type = Py_TYPE(self);
    
    // 3. Копируем __dict__ shallow
    if (Py_TYPE(self)->tp_dictoffset != 0) {
        PyObject **dictptr = _PyObject_GetDictPtr(self);
        if (dictptr != NULL && *dictptr != NULL) {
            PyObject *dict = PyDict_Copy(*dictptr);  # Shallow dict
            if (dict == NULL) {
                Py_DECREF(newobj);
                return NULL;
            }
            _PyObject_SetDict(newobj, dict);
        }
    }
    
    return newobj;
}
```

**Объяснение для людей:** `__copy__()` → `PyType_GenericAlloc()` → копирует `ob_type` + `__dict__.copy()` (shallow). *
*Не трогает атрибуты**.

## 9. Не копируемые типы (deepcopy игнорирует)

```python
# Модули, функции, файлы, сокеты НЕ копируются
def _deepcopy_func(x, memo):
    return x  # Возвращаем оригинал!


def _deepcopy_module(x, memo):
    return x


_deepcopy_dispatch[type(open('file.txt'))] = _deepcopy_file_like
```

**Объяснение для людей:** `deepcopy(open())` → **оригинал** (нельзя клонировать дескриптор). Функции/классы — **shallow
** (refcnt++).

**copy.copy()/deepcopy()** в CPython 3.9+ — **Lib/copy.py** с `_deepcopy_dispatch`, **memo[id→copy]** против циклов,
`PyList_New()`/`PyDict_NewPresized()` + `Py_INCREF()` (shallow), `_reconstruct()` для классов, `__copy__()`/
`__deepcopy__()` хуки.

- [Содержание](#содержание)

---

# **Асинхронность**

## **Junior Level**

Асинхронность в Python — это модель программирования, позволяющая эффективно управлять множеством операций, которые
много ждут, но мало вычисляют. Представьте, что вы — единственный водитель, который развозит нескольких пассажиров по
одному маршруту. Вместо того чтобы ждать у дверей каждого, пока он соберётся (блокирующее ожидание), вы отвозите
первого, а пока он выходит, едете за следующим. Так работает асинхронный код: он не блокируется на одной задаче, а
переключается на другие, пока первая ожидает, например, ответа от сервера или чтения файла.

Ключевыми инструментами здесь являются `async` и `await`. `async def` определяет асинхронную функцию (корутину) —
специальную функцию, которая умеет ставить себя на паузу. Ключевое слово `await` как раз и ставит корутину на паузу,
говоря: "Я буду ждать результата этой операции, а пока можешь заняться чем-то другим". Всей этой каруселью управляет
диспетчер — **цикл событий (event loop)**, который решает, какую корутину запустить или возобновить дальше.

Этот подход идеален для задач, связанных с вводом-выводом (I/O): сетевые запросы, работа с базами данных, чтение файлов.
Он позволяет обрабатывать тысячи одновременных соединений в одном потоке, экономя ресурсы. Важно помнить, что
асинхронность не ускоряет вычисления — для сложной математики лучше подходят другие инструменты, например,
многопроцессорность.

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

**Объяснение для людей:** `async def` → **PyCoroObject** (замороженный фрейм с `await`), `async def gen(): yield` → *
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

**Объяснение для людей:** `await obj` → `GET_AWAITABLE` (obj.__await__() → iterator) → `YIELD_FROM` (iterator.send(
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

**Объяснение для людей:** `GET_AWAITABLE sleep()` → `sleep.__await__()` → **iterator** на стек. `YIELD_FROM` вызывает
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

**Объяснение для людей:** `async def f():` → `PyFunctionObject` с `CO_COROUTINE` → `f()` → `_Py_MakeCoro()` → *
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

**Объяснение для людей:** `asyncio.create_task(coro())` → **TaskObj** с `task_coro=PyCoroObject`, `task_future=Future`.
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

**Объяснение для людей:** Event loop → `task_step()` → `coro.send(None)` → **awaitable** → **callback** `task_wakeup` в
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

**Объяснение для людей:** `async for` → `GET_AITER` (`agen.__aiter__()`) → `GET_ANEXT` (`aiter.__anext__()`) → `await` →
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

**Объяснение для людей:** Event loop **круг**: callbacks → I/O poll (epoll/kqueue) → I/O callbacks → timers. *
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

**Объяснение для людей:** `awaitable.send(None)` → `f_stacktop[0]=None` → выполнение до `YIELD_FROM` → **возврат
корутины** event loop'у.

**Асинхронность** в CPython 3.9+ — **PyCoroObject** (`CO_COROUTINE`), `GET_AWAITABLE`/`YIELD_FROM`, **TaskObj** в
`_asynciomodule.c`, **event loop** (epoll + callbacks), **кооперативное** переключение на `await`.

- [Содержание](#содержание)

---

# **Многопоточность**

## **Junior Level**

Многопоточность в Python позволяет нескольким потокам выполняться в рамках одного процесса, подобно сотрудникам,
работающим над разными задачами в общем офисе. Каждый поток — это независимая последовательность инструкций, которая
может выполняться параллельно с другими, используя общую память и ресурсы программы.

Главная цель многопоточности — повысить отзывчивость приложений, особенно когда они сталкиваются с операциями
ввода-вывода (I/O), такими как сетевые запросы или чтение файлов. Пока один поток ждёт ответа от сервера, другие могут
продолжать работу, что создаёт иллюзию одновременного выполнения.

Однако у многопоточности в Python есть особенность — **Global Interpreter Lock (GIL)**, который позволяет только одному
потоку в любой момент времени выполнять байт-код Python. Это делает многопоточность неэффективной для задач, интенсивно
использующих процессор (например, сложных математических вычислений), но вполне подходящей для операций, связанных с
ожиданием, так как во время ожидания I/O поток освобождает GIL, давая возможность работать другим.

Для создания и управления потоками используется модуль `threading`, который предоставляет класс `Thread` для создания
потоков и различные примитивы синхронизации, такие как блокировки (`Lock`), семафоры (`Semaphore`) и события (`Event`),
помогающие координировать доступ к общим ресурсам.

## **Middle Level**

В основе многопоточности Python лежит **Global Interpreter Lock (GIL)** — механизм, который защищает память
интерпретатора CPython, поскольку его система управления памятью не является потокобезопасной. GIL гарантирует, что
только один поток в любой момент времени выполняет байт-код Python, что упрощает работу с объектами, но ограничивает
истинный параллелизм. Тем не менее, во время операций ввода-вывода или при вызове внешнего кода (например, функций из
библиотек, написанных на C, таких как `numpy`), GIL может быть отпущен, позволяя другим потокам активироваться.

Поток в течение своего жизненного цикла проходит через состояния: создание, готовность к выполнению, запуск,
блокировка (при ожидании I/O или освобождения блокировки) и завершение. Управление этими состояниями осуществляется
планировщиком операционной системы, который распределяет время процессора между потоками.

Для координации потоков модуль `threading` предлагает набор примитивов синхронизации.

- **Блокировка (`Lock`)** обеспечивает взаимное исключение, разрешая доступ к общему ресурсу только одному потоку.
- **Реентерабельная блокировка (`RLock`)** позволяет одному потоку захватывать её несколько раз, что полезно в
  рекурсивных функциях.
- **Семафор (`Semaphore`)** ограничивает количество потоков, одновременно получающих доступ к ресурсу.
- **Событие (`Event`)** позволяет потокам ждать сигнала от других потоков
- **Условная переменная (`Condition`)** — ждать выполнения определённого условия, связанного с общим ресурсом.

Потоки могут быть демоническими (`daemon=True`), что означает их автоматическое завершение при завершении основного
потока программы. Не-демонические потоки продолжают выполнение, даже если основной поток завершился. Для хранения
данных, уникальных для каждого потока, используется `threading.local()`, что удобно для хранения состояния, специфичного
для потока.

Для удобного управления множеством задач применяется `ThreadPoolExecutor` из модуля `concurrent.futures`,
предоставляющий высокоуровневый интерфейс для пула потоков. Это упрощает параллельное выполнение задач, автоматически
управляя созданием и завершением потоков.

Понимание работы GIL и правильное использование примитивов синхронизации критически важны для написания безопасного и
эффективного многопоточного кода. Многопоточность в Python — мощный инструмент для I/O-задач, но для CPU-интенсивных
вычислений стоит рассмотреть многопроцессорность или альтернативные реализации Python, такие как PyPy или использование
библиотек, отпускающих GIL.

## **Senior Level**

В CPython 3.9+ **многопоточность** — **PyThreadState** на поток (`tstate->thread_id`), **GIL** (`take_gil/drop_gil`), **
PyThreadState_Swap()`, **PyThread_start_new_thread()` + **_PyRuntime.threads.list**. **PEP 684** (3.12) *
*per-interpreter** потоки. `Python/pylifecycle.c`,`Python/ceval_gil.c`,`Include/pystate.h`

## 1. PyThreadState структура (Include/pystate.h)

```c
typedef struct _ts {
    struct _ts *next;              // Список потоков интерпретатора
    PyInterpreterState *interp;    // Принадлежит интерпретатору
    PyThread_id thread_id;         // pthread_self() или GetCurrentThreadId()
    int gilstate_counter;          // Счётчик PyGILState_Ensure()
    int recursion_depth;           // Глубина рекурсии
    int tracing;                   // sys.settrace()
    int use_tracing;               // Активен ли tracing
    PyObject *dict;                // thread-local storage
    PyObject *async_exc;           // Текущее исключение
    PyFrameObject *frame;          // Текущий фрейм (для inspect)
    PyObject *frame_obj;           // PyFrameObject
    uint64_t coroutines;           // Счётчик корутин
    // GIL состояние
    int holds_gil;                 // Держит ли GIL
    PyObject *py_thread_state;     // Python threading._get_thread_ident()
} PyThreadState;
```

**Объяснение для людей:** Каждый поток имеет **свой** PyThreadState с `thread_id`, `holds_gil`, `frame`. **GIL**
принадлежит **только одному** tstate.

## 2. Py_NewInterpreter() / PyThreadState_New (pylifecycle.c)

```c
PyThreadState *PyThreadState_New(PyInterpreterState *interp) {
    PyThreadState *tstate;
    
    tstate = (PyThreadState*)PyObject_MALLOC(sizeof(PyThreadState));
    if (tstate == NULL)
        return NULL;
    
    tstate->interp = interp;           // Связываем с интерпретатором
    tstate->thread_id = PyThread_get_thread_ident();  // pthread_self()
    tstate->frame = NULL;
    tstate->recursion_depth = 0;
    tstate->tracing = 0;
    tstate->use_tracing = 0;
    tstate->holds_gil = 0;
    tstate->gilstate_counter = 0;
    tstate->dict = NULL;
    
    // Добавляем в список потоков интерпретатора
    tstate->next = interp->tstate_head;
    interp->tstate_head = tstate;
    
    _PyObject_INIT((PyObject*)tstate, &PyThreadState_Type);
    return tstate;
}
```

**Объяснение для людей:** Новый поток → `PyThreadState_New(main_interp)` → `tstate->thread_id=my_tid` → добавляем в
`interp->tstate_head` список.

## 3. PyThread_start_new_thread() - запуск потока

```c
long PyThread_start_new_thread(PyThread_start_func_t func, void *arg) {
    PyThreadState *tstate = PyThreadState_Get();  // Текущий поток
    
    // Создаём PyThreadState для нового потока
    PyThreadState *new_tstate = PyThreadState_New(tstate->interp);
    if (new_tstate == NULL) {
        return -1;
    }
    
    // C-thread функция
    void *thread_arg = PyMem_Malloc(sizeof(struct thread_arg));
    ((struct thread_arg*)thread_arg)->interp = tstate->interp;
    ((struct thread_arg*)thread_arg)->tstate = new_tstate;
    ((struct thread_arg*)thread_arg)->start_func = func;
    ((struct thread_arg*)thread_arg)->start_arg = arg;
    
    // Запускаем pthread_create()
    int err = pthread_create(&tid, NULL, pythread_run, thread_arg);
    
    if (err != 0) {
        PyThreadState_Clear(new_tstate);
        PyMem_Free(thread_arg);
        PyThreadState_DeleteCurrent();
        return -1;
    }
    
    return 0;
}
```

**Объяснение для людей:** `threading.Thread().start()` → `PyThread_start_new_thread()` →
`pthread_create(pythread_run, arg)` → **новый C-поток**.

## 4. pythread_run() - входная точка потока

```c
static void *pythread_run(void *arg_) {
    struct thread_arg *arg = (struct thread_arg*)arg_;
    
    PyThreadState_Swap(arg->tstate);   // Активируем PyThreadState
    _PyRuntime.threads.list_append(arg->tstate);  // В глобальный список
    
    // Захватываем GIL
    PyEval_AcquireLock(arg->tstate);
    
    // Вызываем Python функцию
    PyObject *res = arg->start_func(arg->start_arg);
    
    // Освобождаем ресурсы
    PyEval_ReleaseLock(arg->tstate);
    _PyRuntime.threads.list_remove(arg->tstate);
    
    PyThreadState_Swap(NULL);          // Деактивируем tstate
    PyThreadState_Clear(arg->tstate);
    PyThreadState_DeleteCurrent();
    
    PyMem_Free(arg);
    return (void*)res;
}
```

**Объяснение для людей:** C-поток → `PyThreadState_Swap(tstate)` → **GIL захват** → `target_func()` → **GIL освобождение
** → `PyThreadState_Clear()`.

## 5. PyThreadState_Swap() - переключение контекста

```c
PyThreadState *PyThreadState_Swap(PyThreadState *new_tstate) {
    PyThreadState *old_tstate = _PyThreadState_UncheckedGet();  // TLS
    
    if (old_tstate == new_tstate) {
        return old_tstate;             // Уже активен
    }
    
    // Сохраняем старый контекст
    if (old_tstate != NULL) {
        // Сохраняем фрейм, исключения, GIL
        old_tstate->frame = _PyThreadState_GetFrame(old_tstate);
        old_tstate->recursion_depth = PyThreadState_Get()->recursion_depth;
    }
    
    // Активируем новый
    _PyThreadState_Set(new_tstate);    // Thread Local Storage (TLS)
    
    if (new_tstate != NULL) {
        // Восстанавливаем контекст
        PyEval_RestoreThread(new_tstate);
    }
    
    return old_tstate;
}
```

**Объяснение для людей:** **TLS** (Thread Local Storage) хранит **активный** PyThreadState. `Swap(NULL)` → деактивируем
Python. `Swap(tstate)` → активируем поток.

## 6. threading.Thread C API (Lib/threading.py → Modules/_threadmodule.c)

```c
static PyObject *thread_PyThread_start_new_thread(PyObject *self, PyObject *args) {
    PyObject *func, *arg;
    
    if (!PyArg_ParseTuple(args, "OO:start_new_thread", &func, &arg))
        return NULL;
    
    // Вызываем PyThread_start_new_thread()
    long retval = PyThread_start_new_thread(
        (PyThread_start_func_t)pythread_wrapper, func);
    
    if (retval < 0) {
        PyErr_SetFromErrno(PyExc_RuntimeError);
        return NULL;
    }
    
    Py_RETURN_NONE;
}
```

**Объяснение для людей:** `Thread(target=f).start()` → `_thread.start_new_thread(f, ())` →
`PyThread_start_new_thread(pythread_wrapper, f)`.

## 7. Per-interpreter потоки (PEP 684, 3.12+)

```c
PyStatus PyInterpreterState_New(PyThreadState *tstate) {
    PyInterpreterState *interp = PyInterpreterState_New();
    
    // Каждый интерпретатор имеет свой список потоков
    interp->tstate_head = NULL;
    interp->threads = NULL;           // PyThreadState.list
    interp->threads_lock = PyThread_create_lock();
    
    // Создаём главный поток для интерпретатора
    PyThreadState *interp_tstate = PyThreadState_New(interp);
    interp->tstate_head = interp_tstate;
    
    return PyStatus_OK();
}
```

**Объяснение для людей:** **3.12+**: `subinterp = Py_NewInterpreter()` → **отдельный** список `tstate_head`. Потоки **не
делят** tstate между интерпретаторами.

## 8. _PyRuntime.threads - глобальный реестр (3.13+)

```c
struct _PyRuntimeState {
    PyThreadStateList threads;     // Все активные потоки
    PyThread_type_lock threads_lock;
};

void _PyRuntime_ThreadsList_Init(struct _PyRuntimeState *runtime) {
    runtime->threads_lock = PyThread_create_lock();
    runtime->threads.head = NULL;
}

void _PyRuntime_ThreadsList_Append(PyThreadState *tstate) {
    PyThread_acquire_lock(_PyRuntime.threads_lock, 1);
    tstate->threads_next = _PyRuntime.threads.head;
    _PyRuntime.threads.head = tstate;
    PyThread_release_lock(_PyRuntime.threads_lock);
}
```

**Объяснение для людей:** **Глобальный** список **всех** PyThreadState. `sys._current_frames()` перебирает
`threads.head`.

## 9. GIL + многопоточность взаимодействие

```c
// Каждый байткод под GIL
case LOAD_FAST: {
    if (!tstate->holds_gil) {
        Py_FatalError("bytecode without GIL");
    }
    // ...
}

// Авто drop_gil каждые 1000 инструкций
if (--tstate->gilstate_counter <= 0) {
    _PyEval_ReleaseLock(tstate);
    _PyEval_AcquireLock(tstate);   // Другой поток может захватить
}
```

**Объяснение для людей:** **Все** байткоды требуют `tstate->holds_gil=1`. Каждые ~1000 инструкций — **шанс** другому
потоку.

## 10. Free-threaded (PEP 703, 3.13+ --disable-gil)

```c
#ifdef Py_GIL_DISABLED
case LOAD_GLOBAL: {
    PyObject *name = GETITEM(names, oparg>>1);
    
    // Атомарный поиск без GIL!
    res = _PyDict_LookupWithCacheAtomic(
        frame->f_globals, name, hints);
        
    if (res == NULL) {
        res = _PyDict_LookupWithCacheAtomic(
            frame->f_builtins, name, hints);
    }
}
#endif
```

**Объяснение для людей:** **3.13 free-threaded**: **атомарные** операции словарей/списков. **Настоящий** параллелизм
байткода.

**Многопоточность** в CPython 3.9+ — **PyThreadState/thread_id**, `PyThreadState_Swap()` + **TLS**,
`PyThread_start_new_thread()` → `pthread_create()`, **per-interpreter** списки (3.12), **GIL** (`holds_gil`), *
*free-threaded** атомарные операции (3.13).

- [Содержание](#содержание)

---

# **Мультипроцессинг**

## **Junior Level**

Мультипроцессинг в Python — это подход к параллельным вычислениям, использующий несколько независимых процессов вместо
потоков в рамках одной программы. Представьте, что вместо одного офиса с сотрудниками (потоками), работающими в общем
пространстве, вы создаёте несколько полностью отдельных офисов, каждый со своим собственным помещением, оборудованием и
набором сотрудников.

Каждый процесс работает в своём собственном пространстве памяти и имеет отдельный экземпляр интерпретатора Python с
собственным Global Interpreter Lock (GIL). Это позволяет полностью обойти ограничения GIL и задействовать все доступные
ядра процессора для выполнения задач, требующих интенсивных вычислений — таких как обработка изображений, сложные
математические расчёты или анализ больших данных.

Полная изоляция процессов обеспечивает большую стабильность: если один процесс завершится с ошибкой, остальные продолжат
работу. Однако эта же изоляция усложняет обмен данными между процессами — они не могут просто обращаться к общей памяти,
как это делают потоки. Для взаимодействия приходится использовать специальные механизмы межпроцессного взаимодействия (
IPC). Для работы с процессами в Python используется модуль `multiprocessing`, который предоставляет API, во многом
похожий на `threading`, но предназначенный для процессов.

## **Middle Level**

Архитектурно каждый процесс в Python представляет собой отдельный экземпляр интерпретатора со своим адресным
пространством, кучей и стеком. Это обеспечивает настоящий параллелизм на уровне операционной системы — процессы могут
выполняться одновременно на разных ядрах процессора, что делает мультипроцессинг эффективным решением для
CPU-интенсивных задач.

Создание процессов может происходить тремя основными способами, каждый со своими особенностями. **Fork** (доступен в
Unix-системах) быстро копирует память родительского процесса, но при этом наследует все открытые файловые дескрипторы и
блокировки, что иногда приводит к неожиданным проблемам. **Spawn** (используется по умолчанию начиная с Python 3.8)
запускает новый интерпретатор Python, который импортирует модуль и выполняет целевую функцию — это безопаснее, но
требует больше времени. **Forkserver** предлагает компромисс: создаётся серверный процесс, от которого затем порождаются
все остальные процессы, что позволяет избежать многократного копирования ненужных ресурсов.

Для обмена данными между изолированными процессами модуль `multiprocessing` предоставляет несколько механизмов
межпроцессного взаимодействия (IPC).

- **Очереди (Queue)** представляют собой потокобезопасные FIFO-структуры,
  реализованные через системные каналы (pipes).
- **Каналы (Pipe)** обеспечивают двунаправленную связь между двумя
  процессами.
- **Разделяемая память (Shared Memory)** через `multiprocessing.Value` и `multiprocessing.Array` позволяет
  создавать переменные, доступные из разных процессов.
- **Менеджеры (Managers)** предлагают высокоуровневый API для
  создания разделяемых объектов (словарей, списков) через прокси-объекты, абстрагируя сложности IPC.

Синхронизация процессов осуществляется с помощью примитивов, аналогичных тем, что используются в многопоточности (Lock,
Semaphore, Event, Condition), но работающих через механизмы операционной системы или общую память. Для управления
множеством задач часто применяются **пулы процессов (Process Pool)** через `multiprocessing.Pool` или
`concurrent.futures.ProcessPoolExecutor`, которые автоматически распределяют задачи между фиксированным количеством
рабочих процессов, предоставляя удобные методы вроде `map()` и `apply_async()`.

Производительность мультипроцессинга определяется балансом между выгодами от истинного параллелизма и накладными
расходами на создание процессов и IPC. Процессы эффективны для задач, требующих значительных вычислений, но могут быть
избыточны для простых операций из-за высокой стоимости создания. Кроме того, при использовании fork необходимо
учитывать, что дочерний процесс получает копию всей памяти родителя, что может привести к неожиданному потреблению
памяти.

Мультипроцессинг в Python — это мощный инструмент для преодоления ограничений GIL и использования всех вычислительных
ресурсов системы, особенно когда задачи могут быть эффективно распараллелены и требуют минимального обмена данными между
процессами.

## **Senior Level**

В CPython 3.9+ **мультипроцессинг** — **отдельные процессы** через `fork()`/`spawn()`/`forkserver()` в
`Lib/multiprocessing/spawn.py`, **IPC** через `Pipe`/`Queue` (`_multiprocessing.so`), **новый PyInterpreterState** в
каждом процессе, **semaphore/shm** для синхронизации. `Lib/multiprocessing/`,`Modules/_multiprocessing.c`

## 1. multiprocessing.set_start_method() - выбор метода (Lib/multiprocessing/context.py)

```python
def set_start_method(method, force=False):
    if method == 'fork':
        _fork_posix()
    elif method == 'spawn':
        _spawn_posix()
    elif method == 'forkserver':
        _forkserver_posix()
    else:
        raise ValueError(f"unknown start method {method}")

    _current_context._set_start_method(method, force)
```

Когда ты пишешь `mp.set_start_method('spawn')`, Python **выбирает способ** создания
нового процесса. `'fork'` — **быстрое копирование** текущего процесса (только Linux), `'spawn'` — **с нуля** (
Windows/macOS/Linux), `'forkserver'` — **сервер копий**. Это **критично** влияет на производительность и безопасность.

## 2. Process._bootstrap() - входная точка процесса (Lib/multiprocessing/process.py)

```python
def _bootstrap():
    # 1. Восстанавливаем аргументы из командной строки
    sys.argv = _args_from_interpreter_flags()

    # 2. Импортируем main модуль заново
    code, filename, main_path = _args_from_interpreter_flags()
    assert main_path is not None
    _run_module_as_main(main_path, code)
```

Новый процесс запускается с **аргументами**
`python -c "import main; main.worker()"`. **Главное** — `if __name__ == '__main__':` **обязательно**, иначе *
*бесконечный импорт** main модуля в дочернем процессе.

## 3. spawn._main() - spawn метод (Lib/multiprocessing/spawn.py)

```python
def _main(fd):
    # 1. Подключаемся к родительскому процессу через Unix pipe
    parent_r, parent_w = os.pipe()
    code, filename = _read_signed(fd)  # Читаем код main модуля

    # 2. Создаём новый Python интерпретатор
    interp = PyInterpreterState_New()
    tstate = PyThreadState_New(interp)
    PyThreadState_Swap(tstate)

    # 3. Выполняем код worker'а
    exec(code, {'__file__': filename})
```

**spawn** = **новый процесс** → **pipe** с родителем → **чтение** `main.py`
байткода → **PyInterpreterState_New()** → **отдельный интерпретатор** → `exec(main.worker())`. **Ничего** от родителя не
наследуется!

## 4. _multiprocessing.Connection - IPC Pipe (Lib/multiprocessing/connection.py)

```python
class Connection:
    def __init__(self, handle, readable=True):
        self._handle = handle  # file descriptor (Unix pipe/socket)
        self._readable = readable

    def send(self, obj):
        # Сериализуем объект pickle → пишем в pipe
        buf = pickle.dumps(obj)
        self._send_bytes(_header(buf) + buf)

    def recv(self):
        # Читаем из pipe → десериализуем
        buf = self._recv_bytes()
        return pickle.loads(buf)
```

`Queue.put(123)` → `pickle.dumps(123)` → **запись** в Unix pipe → другой процесс
`pipe.read()` → `pickle.loads()`. **Единственный способ** передачи данных между процессами.

## 5. _multiprocessing.SemLock - семафоры (Modules/_multiprocessing/semaphore.c)

```c
typedef struct {
    PyObject_HEAD
    volatile int state;            // 0=locked, 1=unlocked
    HANDLE sem;                    // Windows: HANDLE, Unix: sem_t*
    int wait_flag;                 // Для acquire()
    int release_flag;
} SemLockObject;

static PyObject *semlock_acquire(SemLockObject *self) {
    if (self->state == 1) {
        self->state = 0;           // Быстрое захватывание
        Py_RETURN_TRUE;
    }
    
    // Блокируемся на семафоре
    int success = WaitForSingleObject(self->sem, INFINITE);
    if (success == WAIT_OBJECT_0) {
        self->state = 0;
        Py_RETURN_TRUE;
    }
    Py_RETURN_FALSE;
}
```

`Lock.acquire()` → **system semaphore** (Unix `sem_t`, Windows `HANDLE`). **Один**
процесс захватывает, **другие ждут**. **Не Python lock** — **ОС-level** синхронизация.

## 6. multiprocessing.Pool - пул процессов (Lib/multiprocessing/pool.py)

```python
class Pool:
    def __init__(self, processes=None, initializer=None):
        self._processes = processes
        self._pool = []  # Список Process

        # Создаём worker процессы
        for i in range(self._processes):
            p = self.Process(target=worker)
            p.start()
            self._pool.append(p)

    def map(self, func, iterable):
        # Разбиваем задачу → отправляем в Queue
        # Ждём результаты из result_queue
        pass
```

`Pool(4)` → **4 постоянных** процесса-worker'ов → `map(f, lst)` → **разбивает**
`lst` на куски → **отправляет** в Queue каждому worker'у → **собирает** результаты.

## 7. fork() системный вызов (только Unix, spawn контекст)

```c
// В Lib/multiprocessing/forking.py (C вызов)
pid_t pid = fork();
if (pid == 0) {
    // Дочерний процесс
    close(parent_fd);
    _bootstrap_child();        // Входная точка
} else {
    // Родительский
    close(child_fd);
    return pid;
}
```

`fork()` — **клонирует** **весь** процесс **мгновенно** (копирует **page table**
памяти). **Оба** процесса видят **одинаковую** память до **первой записи** (Copy-on-Write). **Опасно** с GIL/threads!

## 8. PyInterpreterState_New() в дочернем процессе

```c
PyStatus PyInterpreterState_New(PyThreadState *tstate) {
    PyInterpreterState *interp;
    
    interp = PyMem_RawCalloc(1, sizeof(PyInterpreterState));
    if (interp == NULL) {
        return PyStatus_NoMemory();
    }
    
    // Инициализируем **новый** интерпретатор
    interp->tstate_head = NULL;
    interp->modules = NULL;
    interp->modules_reloading = 0;
    interp->sysdict = NULL;
    
    // Создаём PyThreadState для этого процесса
    PyThreadState *new_tstate = PyThreadState_New(interp);
    PyThreadState_Swap(new_tstate);
    
    // Инициализация интерпретатора
    if (_PyInterpreterState_Init(interp) < 0) {
        PyInterpreterState_Delete(interp);
        return PyStatus_Err();
    }
    
    return PyStatus_OK();
}
```

Каждый процесс имеет **свой** `PyInterpreterState` + `PyThreadState`. **Никаких**
общих globals/modules/builtins. **Полная изоляция**.

## 9. Shared Memory (multiprocessing.shared_memory)

```c
// Modules/_multiprocessing/shm_posix.c
int shm_open(const char *name, int oflag, mode_t mode) {
    // Создаёт /dev/shm/name сегмент памяти
    return syscall(SYS_shm_open, name, oflag, mode);
}

void *mmap(void *addr, size_t length, int prot, int flags, int fd, off_t offset) {
    // Отображает shm в адресное пространство процесса
}
```

`SharedMemory('name', size=1e6)` → **/dev/shm/name** файл → `mmap()` → **общая
физическая память**. **Байтовое** копирование между процессами.

## 10. multiprocessing.Manager() - proxy объекты

```python
class Server:
    def serve_forever(self):
        # Запускает сокет сервер
        while True:
            conn = self.listener.accept()
            t = threading.Thread(target=Dispatcher(conn))
            t.start()
```

`Manager().dict()` → **отдельный процесс-сервер** → **Unix socket** → **pickle
запросы** → **Python RPC**. **Автоматическая** сериализация.

**Мультипроцессинг** в CPython 3.9+ — **spawn/fork/forkserver**, **новый PyInterpreterState** в каждом процессе, **Pipe
** (`pickle` + Unix sockets), **sem_t/HANDLE** семафоры, **Pool** worker'ы, **Copy-on-Write** (`fork`), **SharedMemory
** (`mmap`).

- [Содержание](#содержание)

---

# **Dataclass**

## **Junior Level**

Dataclass — это декоратор из модуля `dataclasses`, который автоматически добавляет в класс стандартные методы, избавляя
разработчика от написания шаблонного кода. Если обычный класс, предназначенный в основном для хранения данных, требует
ручного определения методов `__init__`, `__repr__` и `__eq__`, то dataclass делает это за вас одной строкой.

Представьте, что вам нужно создать простой класс для представления точки с координатами. Без dataclass пришлось бы
писать несколько методов, которые часто выглядят одинаково для разных классов. С dataclass достаточно объявить поля с
аннотациями типов, и всё необходимое появится автоматически. Это делает код чище, уменьшает вероятность ошибок и
упрощает поддержку.

Например, класс `Point` можно определить всего в две строки, и он сразу будет иметь конструктор, читаемое строковое
представление и корректное сравнение экземпляров по значению полей. Dataclass также предлагает гибкие настройки: можно
создавать неизменяемые классы, управлять участием полей в сравнении, задавать значения по умолчанию и даже определять
поля, которые вычисляются динамически.

## **Middle Level**

Под капотом dataclass анализирует аннотации полей класса и на их основе генерирует необходимые методы. Помимо очевидных
`__init__`, `__repr__` и `__eq__`, он может создавать и другие специальные методы в зависимости от параметров
декоратора. Например, при `order=True` генерируются методы сравнения (`__lt__`, `__le__`, `__gt__`, `__ge__`), что
позволяет упорядочивать экземпляры. При `frozen=True` экземпляры становятся неизменяемыми, и любая попытка изменить поле
вызывает исключение `FrozenInstanceError`. Это полезно для создания объектов, которые должны оставаться постоянными,
например, для использования в качестве ключей словаря.

Функция `field()` предоставляет тонкий контроль над каждым полем. Через неё можно указать, должно ли поле участвовать в
конструкторе, строковом представлении, сравнении или вычислении хеша. Особенно важны параметры `default` для значений по
умолчанию и `default_factory` для изменяемых объектов — например, чтобы каждый экземпляр получал свой собственный
список, а не ссылался на один и тот же. Параметр `metadata` позволяет прикреплять к полям произвольные метаданные,
которые не влияют на поведение dataclass, но могут быть использованы сторонними инструментами для валидации или
генерации документации.

Dataclass поддерживает наследование, собирая поля от всех родительских классов в порядке MRO (Method Resolution Order).
Однако при наследовании требуется аккуратность с значениями по умолчанию, чтобы избежать проблем с порядком аргументов в
конструкторе. Для дополнительной инициализации или валидации можно определить метод `__post_init__()`, который
вызывается автоматически после основного конструктора.

Начиная с Python 3.10, dataclass поддерживает паттерн-матчинг через параметр `match_args=True`, что делает экземпляры
класса удобными для использования в конструкциях `match/case`. Также при использовании `slots=True` dataclass может
генерировать классы с оптимизацией памяти, аналогичные классам с `__slots__`, что особенно полезно при создании большого
количества экземпляров.

В целом, dataclass — это не просто удобное сокращение кода, а полноценный инструмент для создания надёжных и эффективных
классов данных, который сочетает простоту использования с широкими возможностями настройки под конкретные задачи.

1. **Автоматически генерируемые методы**:
    - `__init__()`: Инициализатор с параметрами для всех полей
    - `__repr__()`: Читаемое строковое представление
    - `__eq__()`: Сравнение по значениям всех полей
    - `__ne__()`: Автоматически на основе `__eq__`
    - Опционально (через параметры декоратора):
        - `__lt__()`, `__le__()`, `__gt__()`, `__ge__()`: при `order=True`
        - `__hash__()`: при `frozen=True` или явном указании

2. **Параметры декоратора `@dataclass`**:
    - `init=True`: Генерация `__init__`
    - `repr=True`: Генерация `__repr__`
    - `eq=True`: Генерация `__eq__`
    - `order=False`: Генерация методов сравнения
    - `frozen=False`: Запрет изменения полей после создания
    - `unsafe_hash=False`: Принудительная генерация `__hash__`
    - `match_args=True`: Поддержка паттерн-матчинга (Python 3.10+)

3. **Поле `field()`**: Функция для тонкой настройки полей:
    - `default`: Значение по умолчанию
    - `default_factory`: Фабрика для изменяемых значений по умолчанию
    - `init`: Включать ли поле в `__init__`
    - `repr`: Включать ли поле в `__repr__`
    - `compare`: Участвует ли поле в сравнении
    - `hash`: Участвует ли поле в вычислении хеша
    - `metadata`: Произвольные метаданные

## **Senior Level**

В CPython 3.9+ **dataclass** — **Python декоратор** `Lib/dataclasses.py` с `_process_class()`, генерирует **специальные
методы** (`__init__`, `__repr__`, `__eq__`) через `_create_fn()` + `exec()`, использует **metaclass** `DataclassType`. *
*PEP 557**. `Lib/dataclasses.py`

## 1. @dataclass декоратор (Lib/dataclasses.py)

```python
def dataclass(cls=None, *, init=True, repr=True, eq=True, order=False,
              unsafe_hash=False, frozen=False, match_args=True,
              kw_only=False, slots=False, weakref_slot=False):
    def wrap(cls):
        # 1. Собираем поля класса
        fields = fields(cls)

        # 2. Генерируем специальные методы
        ns = dict(cls.__dict__)

        # __init__
        if init:
            __init__ = _init_fn(fields, locals())
            ns['__init__'] = __init__

        # __repr__
        if repr:
            __repr__ = _repr_fn(fields, globals())
            ns['__repr__'] = __repr__

        # __eq__
        if eq:
            __eq__ = _eq_fn(fields, globals())
            ns['__eq__'] = __eq__

        # __hash__ / __setattr__
        if frozen:
            ns['__setattr__'] = _frozen_setattr()
            ns['__delattr__'] = _frozen_delattr()

        # 3. Создаём новый класс
        return type(cls)(cls.__name__, cls.__bases__, ns)

    return wrap if cls is None else wrap(cls)
```

**Объяснение для тупого человека (очень подробно):** Представь, что у тебя класс `Person` с полями `name: str` и
`age: int`. Когда ты пишешь `@dataclass`, Python **не меняет** твой исходный класс. Вместо этого он **создаёт новый
класс** с **такими же** полями, но **добавляет** готовые методы `__init__`, `__repr__`, `__eq__`. Это как **фабрика**,
которая берёт твой чертеж дома и **достраивает** коммуникации, двери, окна. Ты написал только стены — Python добавил
остальное. `_process_class()` — это **главная фабрика**, которая анализирует аннотации `: str`, `: int` и понимает, что
это **поля dataclass'а**.

## 2. fields() - сбор полей класса

```python
def fields(cls):
    flds = []
    bases = list(cls.__bases__)

    # Ищем __dataclass_fields__
    ann = getattr(cls, '__annotations__', {})
    for name, type_ in ann.items():
        if isinstance(type_, Field):
            flds.append(type_)
        else:
            # Обычная аннотация -> Field(name, type_)
            flds.append(Field(name, type_, default=MISSING))

    # Наследование
    for b in bases[::-1]:  # MRO
        flds = fields(b) + flds

    return tuple(flds)
```

Python **сканирует** `__annotations__` = `{'name': str, 'age': int}`. Каждое имя *
*становится Field**. Если написал `name: str = "Bob"` — это **default значение**. Если класс **наследуется** от другого
dataclass — **берёт** поля родителя **первым**. Представь, что собираешь конструктор LEGO: сначала **база** от
родителей, потом **твои** кубики сверху.

## 3. _init_fn() - генерация __init__

```python
def _init_fn(fields, locals):
    # Сигнатура: def __init__(self, name: str, age: int = 0):
    sig_lines = ['def __init__(self, /, ']

    # Параметры из полей
    for idx, field in enumerate(fields):
        if field.init_arg is False:
            continue
        arg = field.name
        if field.default is not MISSING:
            arg += '=' + repr(field.default)
        sig_lines.append(arg)

    sig = ', '.join(sig_lines) + '):'

    # Тело: self.name = name
    body_lines = []
    for field in fields:
        if field.init_arg:
            line = f'self.{field.name} = {field.name}'
            body_lines.append(line)

    # exec() создаёт функцию
    fn = _create_fn('__init__', ('self',),
                    [sig] + body_lines,
                    locals=locals)
    return fn
```

Для `@dataclass class Person: name: str; age: int = 0` создаётся **строка кода**:

```
def __init__(self, /, name: str, age: int = 0):
    self.name = name
    self.age = 0
```

Затем `exec()` **компилирует** эту строку в **настоящую** Python функцию и **вставляет** в класс. Это **магия**: твой
класс получает **готовый** `__init__`, который принимает **именно те параметры**, что ты объявил.

## 4. _create_fn() - компиляция кода в функцию

```python
def _create_fn(name, args, body_lines, globals=None, locals=None,
               return_type=None):
    # Формируем полный код функции
    code = ['def ' + name + '(' + ', '.join(args) + '):']
    code.extend('    ' + line for line in body_lines)

    # Компилируем в PyCodeObject
    src = '\n'.join(code)
    co = compile(src, '<dataclasses>', 'exec')

    # Создаём функцию
    fn = types.FunctionType(co, globals, name, (), closure=None)
    return fn
```

`_create_fn()` — **мини-компилятор**. Берёт строки
`def __init__(self, name): self.name=name`, **склеивает** в одну большую строку, вызывает `compile()` → **PyCodeObject
**, затем `types.FunctionType()` → **готовую функцию**. Это **точно так же**, как если бы ты написал эту функцию *
*руками** — никакого отличия в скорости!

## 5. __repr__ генерация

```python
def _repr_fn(fields, globals):
    # 'Person(name=\'Bob\', age=30)'
    body_lines = []
    body_lines.append('r = self.__class__.__name__ + \'(\'')

    for field in fields:
        if field.repr:
            line = f"r += '{field.name}=' + repr({field.name}) + ', '"
            body_lines.append(line)

    # Убираем последнюю запятую
    body_lines.append('return r.rstrip(\", \") + \')\'')

    return _create_fn('__repr__', ('self',), body_lines, globals)
```

`print(Person)` → `'Person(name=\'Bob\', age=30)'`. Python **проходит** по всем
полям, для **каждого** делает `name='Bob'`, **склеивает** через запятую. **Точно** как `str(dict)`, но **для твоего
класса**. Если поле `repr=False` — **пропускает** его.

## 6. DataclassType метакласс

```python
class DataclassType(type):
    def __new__(cls, name, bases, ns):
        # Автоматическая обработка @dataclass
        if hasattr(ns, '__dataclass_transform__'):
            return super().__new__(cls, name, bases, ns)

        # Обрабатываем поля
        fields = []
        for base in bases[::-1]:
            fields.extend(getattr(base, '__dataclass_fields__', {}))

        # Добавляем __dataclass_fields__
        ns['__dataclass_fields__'] = dict(fields)

        return super().__new__(cls, name, bases, ns)
```

**Метакласс** — это **фабрика классов**. Когда Python видит `class Person:`, он *
*спрашивает** метакласс: "как создать этот класс?". `DataclassType` **добавляет** `__dataclass_fields__` =
`{'name': Field(...), 'age': Field(...)}`. Это **словарь полей** для наследования.

## 7. Field() - дескриптор полей

```python
class Field:
    __slots__ = ('name', 'type', 'default', 'default_factory',
                 'init', 'repr', 'eq', 'order', 'hash',
                 'compare', 'metadata', 'init_arg')

    def __init__(self, default, *, default_factory, init, repr, eq,
                 order, unsafe_hash, frozen, compare, metadata):
        self.name = None
        self.default = default
        self.default_factory = default_factory
        self.init = init
        self.repr = repr
        self.eq = eq
        self.order = order
        self.hash = unsafe_hash
        self.compare = compare
        self.metadata = metadata
```

`name: str = field(default_factory=list)` → **Field объект** с флагами `repr=True`,
`init=True`, `default_factory=<class 'list'>`. Когда `__init__` видит `default_factory` — вызывает `list()` вместо
копирования значения.

## 8. __post_init__() поддержка

```python
# Автоматический вызов после __init__
def _init_fn(fields, locals):
    # ... обычный __init__ ...

    # Добавляем __post_init__ если есть
    if '__post_init__' in locals:
        body_lines.append('self.__post_init__()')
```

Dataclass **всегда** вызывает `self.__post_init__()` **после** заполнения полей.
Полезно для **валидации**: `def __post_init__(self): assert self.age >= 0`.

## 9. frozen=True - неизменяемый dataclass

```python
def _frozen_setattr():
    return _create_fn('__setattr__', ('self', 'name', 'value'),
                      ['if _is_dataclass_field(self, name):',
                       '    raise FrozenInstanceError(f"cannot assign to {name}")',
                       'object.__setattr__(self, name, value)'])
```

`@dataclass(frozen=True)` → `__setattr__` **блокирует** изменение полей.
`p.name = "new"` → **FrozenInstanceError**. **Только** `object.__setattr__` для служебных полей.

## 10. Наследование dataclass

```python
@dataclass
class Person:
    name: str


@dataclass
class Employee(Person):
    salary: int
```

**Результат:** `__init__(self, name: str, salary: int)`. **Поля родителей** идут **первыми**.

**Dataclass** в CPython 3.9+ — **Lib/dataclasses.py** декоратор, **_process_class()** анализ `__annotations__`, **
_create_fn() + exec()** генерация `__init__`/`__repr__`, **Field дескрипторы**, **DataclassType метакласс**, `frozen`/
`__post_init__` поддержка.

- [Содержание](#содержание)

---

# **Enum**

## **Junior Level**

Enum (перечисление) в Python — это специальный тип данных, позволяющий создавать набор именованных констант, что делает
код более читаемым и защищённым от опечаток. Члены Enum являются иммутабельными синглтонами и поддерживают итерацию,
сравнение и доступ по имени или значению. Python предоставляет несколько вариантов: базовый `Enum`, `IntEnum` (
наследующий `int`), `Flag` для битовых операций и `StrEnum` (с Python 3.11) для строковых констант.

Например, вместо использования чисел 0, 1, 2 для статусов задачи можно создать Enum:

```python
from enum import Enum


class TaskStatus(Enum):
    PENDING = 0
    RUNNING = 1
    COMPLETED = 2
    FAILED = 3
```

Теперь в коде можно использовать `TaskStatus.RUNNING` вместо просто `1`. Enum предоставляет итерацию, сравнение и доступ
к членам по имени или значению. Особенно полезны они для ограничения допустимых значений параметров функции и замены
строковых констант.

## **Middle Level**

1. **Типы Enum**:
    - `Enum`: Базовый класс для создания перечислений
    - `IntEnum`: Члены являются подклассами `int`, могут использоваться везде, где ожидается целое число
    - `Flag`, `IntFlag`: Для битовых флагов, поддерживают побитовые операции
    - `StrEnum` (Python 3.11+): Члены являются подклассами `str`

2. **Свойства членов Enum**:
    - `name`: Имя члена (строка)
    - `value`: Значение члена (может быть любого типа)
    - `__members__`: Словарь всех членов {имя: член}

3. **Автоматические значения**:
    - `auto()`: Автоматически присваивает уникальные значения (начиная с 1)
    - `@enum.unique`: Декоратор, гарантирующий уникальность значений

4. **Методы и операции**:
    - Итерация: `for status in TaskStatus:`
    - Доступ по значению: `TaskStatus(1)`
    - Доступ по имени: `TaskStatus['RUNNING']`
    - Проверка принадлежности: `isinstance(value, TaskStatus)`

5. **Расширенные возможности**:
    - Методы класса: можно добавлять методы в класс Enum
    - Свойства (property): вычисляемые атрибуты
    - Наследование: Enum может наследоваться от других классов (кроме другого Enum)

6. **Иммутабельность**: Члены Enum — синглтоны. Нельзя изменить их значение после создания.

## **Senior Level**

В CPython 3.9+ **Enum** — **EnumType метакласс** (`Lib/enum.py`), **EnumDict** (отслеживает порядок `_member_names_`), *
*`_member_map_`** (name→member), **`__new__`/`__init__`** для создания **EnumMember** объектов, *
*`_generate_next_value_`** + `auto()`. **Singleton-подобные** immutable объекты. `Lib/enum.py`

## 1. EnumType.__new__() - метакласс создания (Lib/enum.py)

```python
class EnumType(type):
    def __new__(metacls, cls, bases, classdict):
        # 1. EnumDict уже обработал поля
        enum_dict = classdict

        # 2. Определяем тип значений (_member_type_)
        member_type, first_enum = metacls._get_mixins_(cls, bases)

        # 3. Создаём enum класс
        enum_class = super().__new__(metacls, cls, bases, classdict)

        # 4. Инициализируем структуры
        enum_class._member_names_ = []  # ['RED', 'GREEN']
        enum_class._member_map_ = {}  # {'RED': <RED>, 'GREEN': <GREEN>}
        enum_class._member_type_ = member_type
        enum_class._value_ = None  # Для StrEnum/IntEnum

        # 5. Создаём члены enum
        for member_name in enum_dict.member_names:
            value = enum_dict[member_name]
            member = metacls._create_(cls, member_name, value)
            enum_class._member_names_.append(member_name)
            enum_class._member_map_[member_name] = member

        return enum_class
```

**Объяснение для тупого человека (очень подробно):** Представь, что пишешь `class Color(Enum): RED = 1; GREEN = 2`.
Python **НЕ** создаёт обычный класс. Вместо этого **метакласс** `EnumType` берёт твой словарь `{'RED': 1, 'GREEN': 2}` и
**превращает** его в **специальные структуры**: `_member_names_ = ['RED', 'GREEN']` (порядок объявления) и
`_member_map_ = {'RED': <Color.RED>, 'GREEN': <Color.GREEN>}` (словарь для быстрого поиска). Каждый член — **отдельный
объект** `Color.RED`, а **НЕ** число `1`. Это как **фабрика**, которая берёт твои константы и делает из них **умные
объекты** с методами.

## 2. EnumDict - специальный словарь (Lib/enum.py)

```python
class EnumDict(dict):
    def __init__(self):
        super().__init__()
        self._member_names = {}  # Имена членов в порядке объявления
        self._last_values = []  # Для auto()

    def __setitem__(self, key, value):
        """Запрещаем дубликаты имён"""
        if key in self._member_names:
            raise TypeError(f'Attempted to reuse member {key!r}')
        if _is_dunder(key):
            super().__setitem__(key, value)
        else:
            self._member_names[key] = None  # Запоминаем порядок
            super().__setitem__(key, value)

    @property
    def member_names(self):
        return list(self._member_names)
```

Обычный `dict` **позволит** `class Color: RED=1; RED=2` (перезапишет). `EnumDict` *
*запрещает** дубликаты: `RED` второй раз → **TypeError**. `_member_names` хранит **порядок** объявления (
`['RED', 'GREEN']`), **НЕ** порядок хеш-таблицы. Это **очень важно** для `list(Color)` → `Color.RED, Color.GREEN`.

## 3. _create_() - создание EnumMember (EnumType)

```python
def _create_(cls, member_name, value):
    # 1. Создаём EnumMember объект
    enum_member = object.__new__(cls)

    # 2. Устанавливаем свойства
    enum_member._name_ = member_name
    enum_member._value_ = value
    enum_member.__objclass__ = cls
    enum_member.__init__(value)

    # 3. Алиасы (дубликаты значений)
    for name, canonical_member in cls._member_map_.items():
        if canonical_member._value_ == value and canonical_member is not enum_member:
            enum_member._missing_(name)  # Алиас

    return enum_member
```

`Color.RED = 1` → **новый объект** `Color.RED` с `_name_='RED'`, `_value_=1`. Если
позже `ALIEN = 1` → `Color.ALIEN` **указывает** на тот же **самый** объект `Color.RED` (алиас).
`Color.RED is Color.ALIEN` = `True`. Это **экономит память** и гарантирует **единственность**.

## 4. Enum.__new__() / __init__() членов

```python
class Enum:
    def __new__(cls, value):
        # Ищем существующий член с таким value
        for member in cls._member_map_.values():
            if member._value_ == value:
                return member

        # Новый член (только через класс!)
        member = object.__new__(cls)
        member._value_ = value
        return member

    def __init__(self, value):
        self._value_ = value

    def __repr__(self):
        return f"<{self.__class__.__name__}.{self._name_}>"
```

`Color(1)` → ищет в `_member_map_` член с `value=1` → **возвращает** `Color.RED` (
существующий объект). `Color.RED` → `'Color.RED'`. **НЕ** создаёт новые экземпляры — возвращает **сигнатоны** (единичные
объекты).

## 5. auto() + _generate_next_value_ (Lib/enum.py)

```python
def auto():
    """Генерирует значение автоматически"""
    value = _next_value_
    _next_value_ += 1
    return value


class Color(Enum):
    def _generate_next_value_(name, start, count, last_values):
        # Автоинкремент
        return count

    RED = auto()  # 1
    GREEN = auto()  # 2
    BLUE = auto()  # 3
```

`RED = auto()` → вызывает `_generate_next_value_('RED', 1, 0, [])` → возвращает `0`.
`GREEN = auto()` → `_generate_next_value_('GREEN', 1, 1, [0])` → `1`. **Глобальный счётчик** `count` (номер члена).
Можно **переопределить**: `return 3**count` → `RED=1, GREEN=3, BLUE=9`.

## 6. Enum.__iter__() / __len__()

```python
def __iter__(cls):
    return (cls._member_map_[name] for name in cls._member_names_)


def __len__(cls):
    return len(cls._member_names_)


def __getitem__(cls, name):
    return cls._member_map_[name]
```

`list(Color)` → `[Color.RED, Color.GREEN]` **в порядке объявления**. `len(Color)` →
`3`. `Color['RED']` → `Color.RED`. **`_member_names_`** гарантирует **порядок** и **быстрый поиск**.

## 7. IntEnum / StrEnum наследование

```python
class IntEnum(int, Enum):
    pass


class StrEnum(str, Enum):
    pass


class Color(IntEnum):
    RED = 1
    GREEN = 2
```

`Color.RED + 1` → `2` (потому что `IntEnum` наследует `int`). `Color.GREEN == '2'` →
`False` (потому что `_value_=2`, а **НЕ** строка). `Color.RED.value` → `1` (сырое значение).

## 8. _simple_enum() - оптимизация (3.11+)

```python
def _simple_enum(etype, seq, value=1):
    """Быстрое создание простого enum"""
    members = dict()
    for i, name in enumerate(seq):
        val = etype._generate_next_value_(name, 1, i, [])
        member = object.__new__(etype)
        member._value_ = val
        member._name_ = name
        member.__objclass__ = etype
        members[name] = member

    # Создаём класс
    cls = type(seq.__class__.__name__, (etype,), members)
    cls._member_names_ = list(members)
    cls._member_map_ = members
    return cls
```

`Enum('RED GREEN BLUE')` → **оптимизированный** путь без метакласса. **Быстрее** для
простых случаев.

## 9. Flag / IntFlag - битовые флаги

```python
class Perm(Flag):
    R = 4
    W = 2
    X = 1

    RWX = R | W | X
```

`Perm.RWX` → **новый** объект с `_value_=7`. `Perm.RWX & Perm.R` → `Perm.R`.
`Perm.RWX | Perm.W` → `Perm.RWX`. **Битовые операции** работают с `_value_`.

## 10. pickle поддержка

```python
def __reduce_ex__(self, proto):
    return (self.__class__, (self._value_,))
```

`pickle.dumps(Color.RED)` → `(Color, (1,))` → при восстановлении `Color(1)` → **тот
же** `Color.RED` объект. **НЕ** новый экземпляр!

**Enum** в CPython 3.9+ — **EnumType метакласс**, **EnumDict** (порядок + уникальность), *
*`_member_map_/ _member_names_`**, **сигнатоны** через `__new__`, **`auto()`** + `_generate_next_value_`, *
*IntEnum/StrEnum** наследование, **Flag** битовые операции.

- [Содержание](#содержание)

---

# **Garbage Collector (Сборщик мусора)**

## **Junior Level**

Сборщик мусора в Python — это автоматический механизм управления памятью, который освобождает память от объектов,
которые больше не используются программой. Он использует комбинированный подход: **подсчет ссылок (reference counting)**
для немедленного освобождения памяти и **циклический сборщик мусора (cycle collector)** для обнаружения и удаления
циклических ссылок, которые не может обработать первый механизм.

Представьте, что каждый объект имеет счётчик (`ob_refcnt`), который увеличивается при создании новой ссылки на объект и
уменьшается при её удалении. Когда счётчик достигает нуля, память объекта немедленно освобождается. Однако, если два или
более объекта ссылаются друг на друга (образуя цикл), их счётчики никогда не станут нулевыми, даже если они уже
недоступны из программы. Для таких случаев существует циклический сборщик.

Для оптимизации процесса объекты делятся на три поколения (0, 1, 2), основываясь на наблюдении, что большинство объектов
живут недолго. Сборка мусора чаще всего происходит в самом молодом поколении (0). Модуль `gc` позволяет управлять этим
процессом, получать статистику и настраивать поведение сборщика.

## **Middle Level**

1. **Два механизма управления памятью**:
    * **Подсчет ссылок**: Быстрый и детерминированный механизм. Каждая операция присваивания или удаления ссылки
      изменяет счётчик. При достижении нуля сразу вызывается деструктор (`__del__`, если он есть) и освобождается
      память.
    * **Циклический сборщик**: Специальный алгоритм (использующий трёхцветную маркировку) для обнаружения и удаления
      недостижимых циклических ссылок между объектами. Работает периодически.

2. **Поколения объектов (Generational GC)**:
    * Все объекты делятся на три поколения. Новые объекты попадают в поколение 0.
    * Частота сборки зависит от поколения: поколение 0 сканируется чаще всего, поколение 1 — реже, поколение 2 — ещё
      реже. Это повышает эффективность, так как сосредотачивает усилия на «молодых» объектах, где скапливается больше
      всего мусора.

3. **Пороги сборки**:
    * Параметры `gc.get_threshold()` возвращают кортеж `(threshold0, threshold1, threshold2)`.
    * Сборка в поколении 0 запускается, когда количество созданных объектов Python с момента последней сборки в этом
      поколении минус количество удалённых превышает `threshold0`.
    * После `threshold0` сборок в поколении 0 происходит одна сборка в поколении 1. После `threshold1` сборок в
      поколении 1 — одна сборка в поколении 2.

4. **Модуль `gc`**:
    * `gc.enable()` / `gc.disable()`: Включение/выключение циклического сборщика.
    * `gc.collect(generation=None)`: Принудительный запуск сборки для указанного поколения (по умолчанию для всех).
    * `gc.get_referents(obj)`: Возвращает объекты, на которые ссылается `obj`.
    * `gc.get_referrers(obj)`: Возвращает объекты, которые ссылаются на `obj`.
    * `gc.set_debug(flags)`: Установка флагов отладки для логирования процесса сборки.

5. **Проблемные случаи**:
    * **Циклические ссылки с `__del__`**: Если объекты в цикле имеют определенный метод `__del__`, сборщик может не
      определить порядок их удаления и оставить такие объекты в памяти (в списке `gc.garbage`), чтобы избежать
      непредсказуемого поведения.
    * **Слабые ссылки (`weakref`)**: Позволяют ссылаться на объект, не увеличивая его счётчик ссылок. Они полезны для
      создания кэшей или наблюдателей, которые не должны мешать удалению основного объекта.

6. **Производительность**:
    * Подсчёт ссылок — это операция с низкими накладными расходами, выполняемая при каждой манипуляции со ссылками.
    * Запуск циклического сборщика (особенно для старших поколений) может вызывать заметные паузы (stop-the-world), так
      как требует обхода всех проверяемых объектов.

## **Senior Level**

В CPython 3.9+ **GC** — **двухфазный**: **refcount** (`Py_INCREF`/`Py_DECREF`) + **generational cycle detection** (
`Modules/gcmodule.c`). **PyGC_Head** (16 байт) в начале каждого GC-объекта, **3 поколения** (threshold 700/10/10), **DFS
** для циклов. `Modules/gcmodule.c`,`Include/objimpl.h`

## 1. PyGC_Head структура (Include/objimpl.h)

```c
typedef union _gc_head {
    struct {
        union _gc_head *gc_next;   // Следующий в списке поколения
        union _gc_head *gc_prev;   // Предыдущий в списке поколения
        PyGC_Head *gc_gc_head;     // Указатель на себя (для GC)
        Py_ssize_t gc_refs;        // Количество ссылок (отдельно от ob_refcnt)
    } gc;
    double dummy;                          // Выравнивание 16 байт
} PyGC_Head;
```

**Объяснение для тупого человека (очень подробно):** Представь, что каждый объект в Python — это **коробка** с
игрушками. Обычный `refcnt` (`ob_refcnt`) считает **сколько коробок ссылается** на эту игрушку. Но если две игрушки *
*указывают друг на друга** (`a.x = b; b.y = a`), refcnt **НЕ упадёт до 0** — **цикл**! GC добавляет **специальный
заголовок** `PyGC_Head` **перед** каждым объектом (list, dict, set, custom с `tp_traverse`). Это **16 байт** со *
*списком** (`gc_next/gc_prev`) для быстрого прохода по всем объектам поколения. `gc_refs` считает **только GC-ссылки** (
отдельно от обычных).

## 2. PyObject + PyGC_Head layout (реальная структура)

```c
// PyListObject в памяти:
// [PyGC_Head 16 байт] + [PyObject_HEAD 16 байт] + [PyListObject поля]
typedef struct {
    PyGC_Head gc;                      // ПЕРВЫЕ 16 байт!
    PyObject_HEAD                       // ob_refcnt + ob_type
    Py_ssize_t ob_size;                // Длина списка
    PyObject **ob_item;                // Массив указателей
} PyListObject;
```

**ВСЯ** память объекта:
`[gc.gc_next(8)][gc.gc_prev(8)][gc.gc_refs(8)][PyObject ob_refcnt(8)][ob_type(8)][ob_size(8)][ob_item(8)]`.
`Py_REFCNT(obj)` → `obj->gc.gc_refs` для GC-объектов. **Overhead** 16 байт на **каждый** list/dict!

## 3. PyObject_GC_New / PyObject_GC_Track (gcmodule.c)

```c
PyObject *_PyObject_GC_New(PyTypeObject *tp) {
    PyObject *op = _PyObject_New(tp);  // Обычный malloc
    if (op != NULL) {
        _PyObject_GC_Link(op);         // Добавляем в GC список
    }
    return op;
}

void _PyObject_GC_Link(PyObject *op) {
    GCState *gcstate = get_gc_state();
    PyGC_Head *gchead = GC_HEAD(op);   // &op->gc
    
    // Добавляем в конец поколения 0 (молодые объекты)
    PyGC_Head *gen0 = &gcstate->generations[0].head;
    gchead->gc.gc_next = gen0->gc_next;
    gchead->gc.gc_prev = gen0;
    gen0->gc_next->gc.gc_prev = gchead;
    gen0->gc_next = gchead;
    
    // Увеличиваем счётчики поколений
    gcstate->generations[0].count++;   // +1 в gen0
    for (int i = 1; i <= NUM_GENERATIONS; i++) {
        gcstate->generations[i].count++;  // +1 во всех старших
    }
    
    gchead->gc.gc_refs = Py_REFCNT(op);  // Копируем refcnt
}
```

`lst = []` → `_PyObject_GC_New(&PyList_Type)` → **malloc** → `_PyObject_GC_Link()` →
**вставляем** `lst.gc` в **двусвязный список** поколения 0. **Все поколения** получают `count++`. **Поколение 0** — *
*самое частое** (каждые 700 объектов).

## 4. Py_DECREF + GC check (Objects/object.c)

```c
Py_ssize_t _Py_DecRef(PyObject *op) {
    Py_ssize_t refcnt = Py_REFCNT(op) - 1;
    
    if (refcnt == 0) {
        // Refcnt=0 → tp_dealloc
        op->ob_type->tp_dealloc(op);
        return 0;
    }
    
    Py_REFCNT(op) = refcnt;
    
    // GC объект? Проверяем gc_refs
    if (PyObject_IS_GC(op) && refcnt < _PyGC_Threshold && 
        Py_REFCNT(op) == 0) {
        _PyObject_GC_Unlink(op);       // Удаляем из списка
        _Py_Dealloc(op);               // Освобождаем
    }
    
    return refcnt;
}
```

`del lst` → `Py_DECREF(lst)` → `ob_refcnt--`. Если `refcnt=0` → **освобождаем**. Для
GC-объектов: если `gc.gc_refs < threshold` (700) **и** `refcnt=0` → `_PyObject_GC_Unlink()` (удаляем из списка) → *
*malloc free**.

## 5. _PyGC_CollectNoFail() - триггер GC (gcmodule.c)

```c
Py_ssize_t _PyGC_CollectNoFail(void) {
    GCState *gcstate = get_gc_state();
    
    // Проверяем все поколения
    for (int gen = NUM_GENERATIONS - 1; gen >= 0; gen--) {
        Py_ssize_t count = gcstate->generations[gen].count;
        Py_ssize_t threshold = gcstate->threshold[gen];
        
        if (count > threshold) {
            // ТРИГГЕР! Собираем это поколение
            return gc_collect(gcstate, gen);
        }
    }
    
    return 0;
}
```

**Каждое** выделение памяти → `_PyGC_CollectNoFail()`. Проверяет **сначала старшее
поколение** (gen2), потом gen1, gen0. Если `gen0.count > 700` → **собираем gen0**. `gen1.count > 10` → собираем *
*gen0+gen1**. **Автоматически**!

## 6. gc_collect() - основной цикл сборки (gcmodule.c)

```c
static Py_ssize_t gc_collect(GCState *gcstate, int gen) {
    Py_ssize_t n = 0;
    
    // Собираем поколения младше gen
    for (int i = 0; i <= gen; i++) {
        n += collect_generation(gcstate, i);
    }
    
    return n;
}

static Py_ssize_t collect_generation(GCState *gcstate, int gen) {
    PyGC_Head *head = &gcstate->generations[gen].head;
    PyGC_Head *next, *cur;
    
    // Проходим по всему поколению
    for (cur = head->gc.gc_next; cur != head; cur = next) {
        next = cur->gc.gc_next;
        
        // Проверяем refcnt
        if (cur->gc.gc_refs == 0) {
            // Мусор! Удаляем
            _PyObject_GC_Unlink((PyObject*)cur);
            PyObject_GC_Del((PyObject*)cur);
            gcstate->collected++;
        }
    }
    
    return gcstate->collected;
}
```

GC **проходит** по **двусвязному списку** поколения:
`head → obj1 → obj2 → ... → head`. Для **каждого** проверяет `gc.gc_refs`. `0` → **мусор** → `_PyObject_GC_Unlink()` +
`free()`. **Живые** остаются.

## 7. Цикловая детекция: _PyGC_traverse() (gcmodule.c)

```c
static int _PyGC_traverse(PyObject *op) {
    Py_ssize_t refs;
    PyTypeObject *tp = Py_TYPE(op);
    
    // tp_traverse для контейнеров
    if (tp->tp_traverse != NULL) {
        refs = tp->tp_traverse(op, (visitproc)_PyGC_traverse);
        if (refs < 0) {
            return refs;
        }
    }
    
    // Считаем живые ссылки
    gc_refs = gc_refs + refs;
    
    return gc_refs;
}
```

**Фаза 2** (циклы): для **каждого** живого объекта вызываем
`tp_traverse(list_traverse)` → **рекурсивно** считаем ссылки. `list_traverse(lst)` проходит `lst->ob_item[]`, вызывает
`_PyGC_traverse(item)`. **Нулируем** `gc.gc_refs` для unreachable.

## 8. PyList_Type.tp_traverse для списков

```c
int list_traverse(PyListObject *op, visitproc visit, void *arg) {
    Py_ssize_t i, len = Py_SIZE(op);
    PyObject **p;
    
    // Проходим все элементы списка
    for (i = 0, p = op->ob_item; i < len; i++, p++) {
        Py_VISIT(*p);              // Вызываем visit(*p)
    }
    
    return 0;
}
```

`list_traverse([a,b,c])` → `Py_VISIT(a)` → `Py_VISIT(b)` → `Py_VISIT(c)`.
`Py_VISIT(obj)` → если `obj` живой → `obj->gc.gc_refs++`. **Цепочка** ссылок!

## 9. Поколения и продвижение (gcmodule.c)

```c
static void move_objects_to_new_gen(GCState *gcstate, int gen0, int gen1) {
    PyGC_Head *gen0_head = &gcstate->generations[gen0].head;
    PyGC_Head *gen1_head = &gcstate->generations[gen1].head;
    
    // Перемещаем выжившие из gen0 → gen1
    for (PyGC_Head *cur = gen0_head->gc.gc_next; cur != gen0_head; ) {
        PyGC_Head *next = cur->gc.gc_next;
        if (cur->gc.gc_refs > 0) {     // Живой!
            _PyObject_GC_Unlink((PyObject*)cur);
            _PyObject_GC_LinkTo((PyObject*)cur, gen1);  // В старшее поколение
        }
        cur = next;
    }
}
```

**Выжившие** из gen0 (700 объектов) **переезжают** в gen1 (собирается реже). Gen1
выжившие → gen2 (самое старшее). **Старые** проверяются **реже** — **оптимизация**!

## 10. gc.collect() C API (gcmodule.c)

```c
static PyObject *gc_collect(PyObject *self, PyObject *args) {
    int n;
    GCState *gcstate = get_gc_state();
    
    if (!PyArg_ParseTuple(args, "|i:collect", &n)) {
        return NULL;
    }
    
    if (n == 0) {
        n = NUM_GENERATIONS - 1;       // Полная сборка gen2
    }
    
    Py_ssize_t collected = _PyGC_CollectNoFail();
    
    return PyLong_FromSsize_t(collected);
}
```

`gc.collect()` → `_PyGC_CollectNoFail()` → собирает **самое старшее** поколение (
gen2 + все младшие). Возвращает **количество** освобождённых объектов.

**GC** в CPython 3.9+ — **PyGC_Head** (16 байт), **3 поколения** (700/10/10), **refcount** + **tp_traverse DFS**, *
*двусвязные списки** поколений, **продвижение** выживших, **триггер** при `count > threshold`.

- [Содержание](#содержание)

---

# **Сложность кода (Asymptotic, Cyclomatic, Coupling, Maintainability)**

## **Junior Level**

Сложность кода — это набор характеристик, определяющих, насколько программа понятна, эффективна и удобна для работы.
Выделяют четыре ключевых типа.

1. **Асимптотическая сложность (Asymptotic Complexity)** — показывает, как время выполнения или использование памяти
   программой зависит от объема входных данных (например, количества элементов в списке). Выражается в "О-нотации": O(
    1) — время постоянно, O(n) — растет линейно, O(n²) — растет квадратично (намного быстрее).

2. **Цикломатическая сложность (Cyclomatic Complexity)** — это мера количества независимых путей выполнения в коде (
   например, в функции). Чем больше операторов ветвления (`if/else`, `switch`) и циклов (`for`, `while`), тем она выше.
   Высокая цикломатическая сложность делает код трудным для понимания и тестирования.

3. **Сложность связей (Coupling Complexity)** — определяет, насколько сильно разные модули, классы или функции зависят
   друг от друга. Сильно связанный код сложно изменять, тестировать изолированно и повторно использовать.

4. **Сложность поддержки (Maintainability Complexity)** — общая оценка того, насколько легко анализировать, изменять и
   расширять код. Зависит от его читаемости, документированности, структуры и соответствия стандартам.

**Практическое значение для разработчика**:

- **Высокая асимптотическая сложность** → проверка производительности на больших данных.
- **Высокая цикломатическая сложность** → необходимость в большем количестве тестов для покрытия всех путей выполнения.
- **Высокая связанность** → сложности с изоляцией модулей для модульного тестирования.
- **Низкая поддерживаемость** → больше времени тратится на понимание кода перед внесением изменений или написанием
  тестов.

## **Middle Level**

### **1. Асимптотическая сложность (Asymptotic Complexity)**

Это математическая оценка роста ресурсоемкости алгоритма.

- **Верхняя граница (O-нотация)**: `O(f(n))` означает, что время выполнения не превысит `c * f(n)` для всех достаточно
  больших `n`. Это наиболее часто используемая оценка.
- **Нижняя граница (Ω-нотация)**: `Ω(f(n))` означает, что время выполнения не меньше `c * f(n)`.
- **Точная граница (Θ-нотация)**: `Θ(f(n))` означает, что алгоритм является одновременно и `O(f(n))`, и `Ω(f(n))`.
- **Амортизированный анализ**: оценивает среднюю производительность операции в худшем случае за всю последовательность
  вызовов, что полезно для анализа структур данных.
- **Космическая сложность**: оценивает рост потребления памяти аналогично временной сложности.

**Практика для разработки и тестирования**:

- **Тестирование на разных объемах данных**: малые (пограничные случаи), средние, большие (нагрузочное тестирование).
- **Профилирование**: использование профилировщиков (cProfile, memory_profiler) для поиска узких мест.
- **Анализ в CI/CD**: интеграция проверки сложности в конвейер сборки с помощью инструментов статического анализа.

### **2. Цикломатическая сложность (Cyclomatic Complexity)**

Количественная метрика, основанная на графе потока управления программы.

- **Расчет**: `M = E - N + 2P`, где `E` — число рёбер в графе, `N` — число узлов, `P` — число компонентов связности (
  обычно 1). На практике рассчитывается автоматически.
- **Практические рекомендации** по значениям для функции/метода:
    - **1-10**: простая, низкий риск.
    - **11-20**: умеренная сложность.
    - **21-50**: высокая сложность, требует рассмотрения рефакторинга.
    - **51+**: очень высокая, тестирование затруднено, высок риск ошибок.
- **Инструменты**: radon, mccabe, pylint, а также комплексные платформы (SonarQube, Codacy), которые включают этот
  анализ.

**Связь с тестированием**:

- Минимальное количество тестов для покрытия всех путей должно быть не меньше цикломатической сложности.
- Помогает выявить функции с излишней логикой ветвления, которые являются кандидатами на упрощение.

### **3. Сложность связей (Coupling Complexity)**

Оценивает степень взаимозависимости между модулями. Цель — достичь слабой связанности.

**Типы связности (от худшего к лучшему)**:

- **Content coupling**: один модуль напрямую изменяет внутренние данные другого.
- **Common coupling**: модули используют общие глобальные данные.
- **Control coupling**: один модуль передает другому флаг или данные, управляющие его логикой.
- **Stamp coupling**: модулям передается большая структура данных, хотя нужна лишь ее часть.
- **Data coupling**: модули обмениваются только необходимыми данными через параметры (идеал).

**Метрики**:

- **CBO (Coupling Between Objects)**: количество классов, с которыми непосредственно связан данный класс.
- **Ca (Afferent Coupling)**: количество классов, которые зависят от данного (входящие связи).
- **Ce (Efferent Coupling)**: количество классов, от которых зависит данный (исходящие связи).

**Влияние на разработку**:

- Высокая связанность затрудняет модульное тестирование, требуя создания множества mock-объектов.
- Повышает риск каскадных изменений: правка в одном модуле ведет к необходимости правок во многих других.
- Современные инструменты анализа кода помогают визуализировать зависимости и выявлять проблемные узлы.

### **4. Сложность поддержки (Maintainability Complexity)**

Составная характеристика, прогнозирующая усилия по сопровождению кода.

**Основные компоненты**:

- **Анализируемость**: легкость понимания кода, отладки и локализации дефектов.
- **Изменяемость**: легкость внесения изменений и минимальный риск побочных эффектов.
- **Стабильность**: устойчивость к распространению ошибок при изменениях.
- **Тестируемость**: степень, в которой код облегчает создание и проведение тестов.

**Ключевые метрики**:

- **Индекс поддерживаемости (MI)**: комплексный показатель, рассчитываемый на основе цикломатической сложности,
  количества строк кода (LOC) и объема комментариев.
- **Соотношение комментарии/код**: показатель документированности.
- **Нарушения стандартов кода**: количество отступлений от соглашений (PEP8, Google Style и др.).
- **Дублирование кода**: процент строк, повторяющихся в кодовой базе.

**Инструменты**: SonarQube, CodeClimate, Codacy. Эти платформы агрегируют метрики, устанавливают пороговые значения ("
Quality Gate") и предоставляют сводные отчеты о здоровье кода, помогая командам проактивно работать над улучшением
сопровождаемости.

Надеюсь, это объяснение было полезным и полным. Если у вас есть вопросы по какому-то из видов сложности или инструментам
для их анализа — обращайтесь.

## **Senior Level**

### **1. Асимптотическая сложность (Asymptotic Complexity) (Big O only)**

## **O(1) - Константная сложность**

```python
def get_first_element(arr):
    return arr[0]  # Всегда одна операция


def is_even(n):
    return n % 2 == 0  # Одна операция
```

## **O(log n) - Логарифмическая сложность**

```python
def binary_search(arr, target):
    left, right = 0, len(arr) - 1
    while left <= right:
        mid = (left + right) // 2
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    return -1
```

## **O(n) - Линейная сложность**

```python
def linear_search(arr, target):
    for i in range(len(arr)):  # Проход по всем элементам
        if arr[i] == target:
            return i
    return -1


def sum_array(arr):
    total = 0
    for num in arr:  # Один цикл по всем элементам
        total += num
    return total
```

## **O(n log n) - Линейно-логарифмическая сложность**

```python
def merge_sort(arr):
    if len(arr) <= 1:
        return arr

    mid = len(arr) // 2
    left = merge_sort(arr[:mid])
    right = merge_sort(arr[mid:])

    return merge(left, right)


def merge(left, right):
    result = []
    i = j = 0
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            result.append(left[i])
            i += 1
        else:
            result.append(right[j])
            j += 1
    result.extend(left[i:])
    result.extend(right[j:])
    return result
```

## **O(n²) - Квадратичная сложность**

```python
def bubble_sort(arr):
    n = len(arr)
    for i in range(n):  # Внешний цикл
        for j in range(0, n - i - 1):  # Внутренний цикл
            if arr[j] > arr[j + 1]:
                arr[j], arr[j + 1] = arr[j + 1], arr[j]
    return arr


def find_duplicates(arr):
    duplicates = []
    for i in range(len(arr)):  # Двойной цикл
        for j in range(i + 1, len(arr)):
            if arr[i] == arr[j]:
                duplicates.append(arr[i])
    return duplicates
```

## **O(n³) - Кубическая сложность**

```python
def find_triplets(arr):
    n = len(arr)
    triplets = []
    for i in range(n):  # Три вложенных цикла
        for j in range(i + 1, n):
            for k in range(j + 1, n):
                if arr[i] + arr[j] + arr[k] == 0:
                    triplets.append((arr[i], arr[j], arr[k]))
    return triplets
```

## **O(2ⁿ) - Экспоненциальная сложность**

```python
def fibonacci_recursive(n):
    if n <= 1:
        return n
    return fibonacci_recursive(n - 1) + fibonacci_recursive(n - 2)


def generate_subsets(arr):
    def backtrack(start, current):
        subsets.append(current.copy())
        for i in range(start, len(arr)):
            current.append(arr[i])
            backtrack(i + 1, current)
            current.pop()

    subsets = []
    backtrack(0, [])
    return subsets
```

## **O(n!) - Факториальная сложность**

```python
def generate_permutations(arr):
    def backtrack(path):
        if len(path) == len(arr):
            permutations.append(path.copy())
            return
        for num in arr:
            if num not in path:
                path.append(num)
                backtrack(path)
                path.pop()

    permutations = []
    backtrack([])
    return permutations

# Пример: для 3 элементов будет 3! = 6 перестановок
```

## **O(√n) - Сложность квадратного корня**

```python
def is_prime(n):
    if n < 2:
        return False
    for i in range(2, int(n ** 0.5) + 1):  # До квадратного корня
        if n % i == 0:
            return False
    return True
```

**Ключевые выводы:**

1. **O(1)** и **O(log n)** - самые эффективные
2. **O(n)** и **O(n log n)** - приемлемы для больших данных
3. **O(n²)**, **O(2ⁿ)**, **O(n!)** - становятся непрактичными при росте n
4. При выборе алгоритма важно оценивать ожидаемый размер входных данных

### **2. Цикломатическая сложность (Cyclomatic Complexity)**

### **V(G) = 1** - Простая функция

```python
def simple_greeting(name):
    """Цикломатическая сложность = 1"""
    return f"Hello, {name}!"

# Граф: один путь выполнения
```

### **V(G) = 2** - Одно условие

```python
def check_age(age):
    """Цикломатическая сложность = 2 (1 условие + 1)"""
    if age >= 18:
        return "Adult"
    else:
        return "Minor"

# Условия: 1
# Пути: 1) age >= 18, 2) age < 18
```

### **V(G) = 3** - Два условия (или if-elif)

```python
def grade_score(score):
    """Цикломатическая сложность = 3"""
    if score >= 90:
        return "A"
    elif score >= 70:
        return "B"
    else:
        return "C"

# Условия: 2 (score >= 90 и score >= 70)
# Пути: 3 возможных пути
```

### **V(G) = 4** - Несколько условий и цикл

```python
def categorize_temperature(temp):
    """Цикломатическая сложность = 4"""
    if temp < 0:
        category = "Freezing"
    elif temp < 15:
        category = "Cold"
    elif temp < 25:
        category = "Warm"
    else:
        category = "Hot"

    # Добавляем дополнительный путь
    if "Cold" in category or "Freezing" in category:
        return f"{category} - Wear jacket"
    return f"{category} - Light clothes"

# Условия: 4
# Пути: множество комбинаций
```

### **V(G) = 6** - Вложенные условия

```python
def check_access(user_role, subscription_type, age):
    """Цикломатическая сложность = 6"""
    if user_role == "admin":
        if subscription_type == "premium":
            return "Full access"
        else:
            return "Admin basic access"
    elif user_role == "user":
        if age >= 18:
            if subscription_type == "premium":
                return "Premium user access"
            else:
                return "Basic user access"
        else:
            return "Restricted access (minor)"
    else:
        return "No access"

# Граф:
# - 3 внешних условия (admin/user/else)
# - 2 вложенных условия в admin
# - 3 вложенных условия в user
```

### **V(G) = 8** - Сложная логика с циклами

```python
def process_items(items, threshold, max_attempts):
    """Цикломатическая сложность = 8"""
    result = []

    for item in items:  # Цикл добавляет 1 к сложности
        if item is None:
            continue  # Дополнительный путь

        attempts = 0
        success = False

        while attempts < max_attempts and not success:  # Еще +1
            if item["value"] > threshold:
                if item.get("valid", False):
                    success = True
                    result.append(item["value"] * 2)
                else:
                    result.append(item["value"])
            elif item["value"] > 0:
                result.append(item["value"] // 2)
            else:
                break  # Еще один путь

            attempts += 1

        if not success:
            result.append(-1)

    return result

# Составляющие сложности:
# - for loop: +1
# - while loop: +1  
# - if item is None: +1
# - if item["value"] > threshold: +1
# - if item.get("valid", False): +1
# - elif item["value"] > 0: +1
# - else в цикле while: +1
# - if not success: +1
# Итого: 8
```

### **V(G) = 15+** - "Плохой" код с высокой сложностью

```python
def monster_function(a, b, c, d, e, f):
    """Цикломатическая сложность > 15 - ТРЕБУЕТ РЕФАКТОРИНГА!"""
    result = 0

    if a > 0:
        if b < 10:
            result += 1
        elif b < 20:
            result += 2
        else:
            if c == "type1":
                result += 3
            elif c == "type2":
                result += 4
            else:
                result += 5
    elif a == 0:
        if d:
            for i in range(e):  # +1
                if i % 2 == 0:  # +1
                    result += i
                else:
                    if f:  # +1
                        result -= i
                    else:
                        result += i * 2
        else:
            result = -1
    else:
        if e > 100:
            result = e * 2
        elif e > 50:
            result = e
        else:
            if f:
                result = e // 2
            else:
                result = e // 4

    # Дополнительные вложенные условия
    if result > 1000:
        if a > 50:
            result *= 1.1
        elif a > 20:
            result *= 1.05

    return result

# Эта функция имеет очень высокую цикломатическую сложность
# из-за глубокой вложенности и множества условий
```

### **3. Сложность связей (Coupling Complexity)**

#### **Content Coupling (Худший вид)**

Один модуль напрямую изменяет внутренние данные или логику другого.

```python
# ПЛОХО: Модуль A манипулирует внутренним состоянием модуля B
class UserDatabase:
    def __init__(self):
        self._users = []  # Приватное поле


class AdminPanel:
    def clear_database(self, db):
        db._users.clear()  # Нарушение инкапсуляции!


# Использование:
db = UserDatabase()
admin = AdminPanel()
admin.clear_database(db)  # Прямой доступ к приватному полю
```

#### **Common/Global Coupling**

Модули используют общие глобальные данные.

```python
# ПЛОХО: Множество компонентов зависят от глобальной переменной
CONFIG = {"theme": "dark", "timeout": 30}  # Глобальное состояние


class UserService:
    def get_timeout(self):
        return CONFIG["timeout"]  # Прямая зависимость


class ApiClient:
    def request(self):
        timeout = CONFIG["timeout"]  # Та же зависимость
        # ... использование timeout
```

#### **Control Coupling**

Один модуль передает флаги или данные, управляющие логикой другого.

```python
# ПЛОХО: Флаг управляет внутренней логикой метода
def process_data(data, format_type):
    """format_type: 'json', 'xml', 'csv'"""
    if format_type == "json":
        return json.dumps(data)
    elif format_type == "xml":
        return to_xml(data)  # Предположим, есть такая функция
    elif format_type == "csv":
        return to_csv(data)
    else:
        raise ValueError("Unknown format")


# Вызывающая сторона управляет внутренним поведением
result = process_data(my_data, "json")
```

#### **Stamp Coupling**

Передача больших структур данных, когда нужна только их часть.

```python
# ПЛОХО: Передается весь объект пользователя, хотя нужен только email
def send_email(user):
    """Требуется только user.email, но получает целый объект"""
    email = user.email
    name = user.name  # Не используется в отправке
    # ... логика отправки


# Более четкий интерфейс:
def send_email_to_address(email_address, message):
    # Использует только необходимые данные
    pass
```

#### **Data Coupling (Идеал)**

Модули обмениваются только необходимыми данными через параметры.

```python
# ХОРОШО: Слабая связанность через четкие интерфейсы и зависимости
class EmailSender:
    def send(self, to_address: str, subject: str, body: str) -> bool:
        # Логика отправки
        return True


class UserNotifier:
    def __init__(self, email_sender: EmailSender):  # Внедрение зависимости
        self._email_sender = email_sender

    def notify_password_change(self, user_email: str):
        # Использует только необходимые данные и явную зависимость
        self._email_sender.send(
            to_address=user_email,
            subject="Password Changed",
            body="Your password was successfully changed."
        )


# Использование с Dependency Injection
sender = EmailSender()
notifier = UserNotifier(sender)
notifier.notify_password_change("user@example.com")
```

### **4. Сложность поддержки (Maintainability Complexity)**

**Низкая поддерживаемость (Technical Debt):**

```python
# ПЛОХО: Множество проблем в одной функции
def proc(d, t, s, f, m, c):  # Непонятные параметры
    r = 0
    # Магические числа
    if d > 100 and t == "A" or t == "B" and s:  # Сложное условие
        for i in range(len(f)):  # C-стиль итерации
            if f[i] and m.get(c, False):  # Смесь логики
                try:
                    r += complex_operation(d, f[i])  # Неочевидная функция
                except:
                    r = -1  # Подавление исключений
    elif d < 0:  # Не покрывает edge-case d == 0
        r = None
    # Отсутствует обработка других случаев
    return r  # Может вернуть число, None или -1
```

**Высокая поддерживаемость:**

```python
# ХОРОШО: Чистый, самодокументирующийся код
from typing import Optional, List
from enum import Enum


class DeviceType(Enum):
    TYPE_A = "A"
    TYPE_B = "B"
    UNKNOWN = "UNKNOWN"


class ProcessingResult:
    def __init__(self, value: float, status: str):
        self.value = value
        self.status = status


def process_device_data(
        device_id: int,
        device_type: DeviceType,
        is_active: bool,
        sensor_readings: List[float],
        calibration_map: dict,
        calibration_key: str
) -> Optional[ProcessingResult]:
    """
    Обрабатывает данные устройства и возвращает результат калибровки.
    
    Args:
        device_id: Уникальный идентификатор устройства
        device_type: Тип устройства из перечисления DeviceType
        is_active: Флаг активности устройства
        sensor_readings: Показания датчиков устройства
        calibration_map: Карта калибровочных коэффициентов
        calibration_key: Ключ для поиска коэффициента калибровки
    
    Returns:
        ProcessingResult с значением и статусом или None при ошибке
    
    Raises:
        ValueError: При некорректных входных данных
    """
    VALID_DEVICE_THRESHOLD = 100

    # Валидация входных данных
    if not sensor_readings:
        raise ValueError("Sensor readings cannot be empty")

    # Проверка условий обработки
    should_process = (
            device_id > VALID_DEVICE_THRESHOLD and
            device_type in (DeviceType.TYPE_A, DeviceType.TYPE_B) and
            is_active
    )

    if not should_process:
        return None

    # Обработка показаний датчиков
    calibration_factor = calibration_map.get(calibration_key)
    if not calibration_factor:
        return ProcessingResult(
            value=0.0,
            status="CALIBRATION_MISSING"
        )

    try:
        total = sum(
            reading * calibration_factor
            for reading in sensor_readings
            if reading is not None
        )
        average = total / len(sensor_readings)

        return ProcessingResult(
            value=average,
            status="SUCCESS"
        )

    except (ZeroDivisionError, TypeError) as e:
        logger.error(f"Processing failed for device {device_id}: {e}")
        return ProcessingResult(
            value=0.0,
            status="PROCESSING_ERROR"
        )
```

- [Содержание](#содержание)

---

# **ООП**

## **Junior Level**

Объектно-ориентированное программирование (ООП) — это подход к разработке программ, где основными строительными блоками
являются объекты, которые объединяют данные и методы для работы с ними.

Основные концепции ООП:

1. **Класс** — чертёж или шаблон для создания объектов (например, класс `Dog`).
2. **Объект** — конкретный экземпляр класса (например, `my_dog = Dog()`).
3. **Инкапсуляция** — объединение данных и методов в одном объекте и сокрытие внутренней реализации от внешнего мира.
4. **Наследование** — возможность создавать новые классы на основе существующих, перенимая их свойства и методы (
   например, класс `Poodle` наследует от `Dog`).
5. **Полиморфизм** — возможность использовать объекты разных классов через одинаковый интерфейс (например, методы
   `speak()` для `Dog` и `Cat` могут работать по-разному).
6. **Абстракция** - это концепция, которая позволяет выделить существенные характеристики объекта, игнорируя
   несущественные детали реализации. Она помогает работать со сложными системами, представляя их в виде упрощённых
   моделей.

ООП помогает организовать код, делает его более понятным, удобным для повторного использования и изменения.

## **Middle Level**

В Python ООП реализовано динамически и поддерживает все классические принципы, а также некоторые уникальные особенности:

**1. Динамическая природа классов и объектов**:

- Классы являются объектами первого класса (можно присваивать переменным, передавать в функции).
- Атрибуты и методы могут добавляться, изменяться или удаляться во время выполнения.
- Поддержка динамического создания классов через `type()`.

**2. Множественное наследование и MRO**:

- Python поддерживает множественное наследование.
- Алгоритм C3 Linearization определяет порядок разрешения методов (Method Resolution Order, MRO).
- `super()` используется для корректного вызова методов родительских классов.

**3. Дескрипторы и свойства**:

- **Дескрипторы** — объекты, реализующие протокол `__get__`, `__set__`, `__delete__`. Лежат в основе свойств, методов,
  статических методов и методов класса.
- **Свойства (property)** — позволяют использовать getter, setter и deleter для атрибутов, сохраняя синтаксис доступа к
  атрибуту.

**4. Абстрактные классы**:

- Модуль `abc` позволяет создавать абстрактные классы и методы.
- Абстрактный класс не может быть инстанциирован, требует переопределения абстрактных методов в дочерних классах.

**5. Магические методы (dunder methods)**:

- Методы вида `__init__`, `__str__`, `__add__` и т.д.
- Позволяют переопределять поведение операторов и встроенных функций для объектов пользовательских классов.

**6. Метаклассы**:

- Классы, экземпляры которых являются другими классами.
- Позволяют вмешиваться в процесс создания класса, добавлять валидацию, регистрацию и т.д.

**7. Принципы SOLID в Python**:

- **Single Responsibility**: Каждый класс должен иметь одну причину для изменения.
- **Open/Closed**: Классы должны быть открыты для расширения, но закрыты для изменения.
- **Liskov Substitution**: Объекты базового класса должны быть заменяемы объектами производных классов без изменения
  корректности программы.
- **Interface Segregation**: Много специализированных интерфейсов лучше одного универсального.
- **Dependency Inversion**: Зависимости должны строиться на абстракциях, а не на деталях.

**8. Протоколы и утиная типизация**:

- Python использует утиную типизацию: интерфейс объекта определяется его поведением (наличием методов), а не явным
  объявлением.
- **Протоколы** — неформальные интерфейсы, например, протокол итератора требует методы `__iter__` и `__next__`.

## **Senior Level**

В CPython 3.9+ **ООП** — **PyTypeObject** (классы как объекты), **MRO** (`tp_mro` tuple), **descriptor протокол** (
`__get__`/`__set__`), **`PyType_Ready()`** инициализация, **`super()`** → `_PySuperObject`, **tp_call** для
`Class()`. `Objects/typeobject.c`,`Objects/classobject.c`,`Python/compile.c`

## 1. PyTypeObject структура (Include/cpython/object.h)

```c
typedef struct _typeobject {
    PyObject_VAR_HEAD         // ob_base = PyVarObject_HEAD_INIT + tp_name
    Py_ssize_t tp_basicsize;  // Размер экземпляра (без переменных полей)
    Py_ssize_t tp_itemsize;   // Размер для массивов (list/tuple=sizeof(PyObject*))
    
    // Методы типа
    destructor tp_dealloc;          // __del__
    printfunc tp_print;             // print(obj)
    getattrfunc tp_getattr;         // __getattr__
    setattrfunc tp_setattr;         // __setattr__
    PyAsyncMethods *tp_as_async;    // async методы
    reprfunc tp_repr;               // repr()
    
    // Кэш атрибутов
    Py_ssize_t tp_hash;             // Кэш хеша типа
    
    // Итерация
    unaryfunc tp_call;              // obj() вызов
    reprfunc tp_str;                // str()
    getattrofunc tp_getattro;       // obj.name поиск
    setattrofunc tp_setattro;       // obj.name = value
    
    // Дескрипторный протокол
    PyBufferProcs *tp_as_buffer;    // buffer протокол
    
    unsigned long tp_flags;         // Py_TPFLAGS_BASETYPE и т.д.
    
    const char *tp_doc;             // __doc__
    traversal tp_traverse;          // GC traverse
    inquiry tp_clear;               // GC clear
    richcmpfunc tp_richcompare;     // ==
    
    // MRO и наследование
    Py_ssize_t tp_weaklistoffset;   // Слабые ссылки
    Py_ssize_t tp_dictoffset;       // __dict__ offset
    Py_ssize_t tp_init;             // __init__
    Py_ssize_t tp_alloc;            // __alloc__
    vectornewfunc tp_new;           // __new__
    freefunc tp_free;               // tp_free
    
    // Классовые атрибуты
    PyObject *tp_bases;             // tuple базовых классов
    tuple mro;                      // Method Resolution Order tuple
    PyObject *tp_cache;             // Кэш атрибутов
    PyObject *tp_subclasses;        // Список подклассов
    PyObject *tp_weaklist;          // Слабые ссылки
    destructor tp_del;              // __del__
    
    // 3.8+ Finalization
    destructor tp_version_tag;      // Версия кэша
    destructor tp_finalize;         // __del__
} PyTypeObject;
```

`class MyClass:` → **PyTypeObject** размером ~300 байт в памяти.
`MyClass.__bases__` = `(object,)`, `MyClass.mro` = `(MyClass, object)`. `MyClass()` → `MyClass.tp_new(MyClass)` →
`MyClass.tp_alloc()` → `MyClass.tp_init()`. **Всё** в одном объекте!

## 2. PyType_Ready() - инициализация класса (Objects/typeobject.c)

```c
int PyType_Ready(PyTypeObject *type) {
    // 1. Инициализируем базовые поля
    if (type->tp_flags & Py_TPFLAGS_READY) {
        return 0;                      // Уже готов
    }
    
    // 2. Вычисляем MRO (Method Resolution Order)
    if (mro_internal(type) < 0) {
        return -1;
    }
    
    // 3. Инициализируем dictoffset (для __dict__)
    if (type->tp_dictoffset && 
        type->tp_dictoffset != _PyObject_DictOffset()) {
        PyErr_Format(PyExc_TypeError,
            "instance dictoffset must be %zd, not %zd",
            _PyObject_DictOffset(), type->tp_dictoffset);
        return -1;
    }
    
    // 4. Инициализируем слабые ссылки
    if (type->tp_weaklistoffset && 
        !PyType_SUPPORTS_WEAKREFS(type)) {
        return -1;
    }
    
    // 5. Устанавливаем флаги READY
    type->tp_flags |= Py_TPFLAGS_READY;
    
    // 6. Обновляем кэш подклассов
    update_all_slots(type);
    
    return 0;
}
```

После компиляции `class MyClass:` Python вызывает `PyType_Ready(&MyClass_Type)`. *
*Главное** — `mro_internal()` вычисляет `(MyClass, object)` → `MyClass.tp_mro`. Проверяет `__dict__` offset (всегда 184
байта в PyDictObject*). Устанавливает `Py_TPFLAGS_READY` флаг. **Только после этого** класс готов к использованию!

## 3. mro_internal() - вычисление MRO (typeobject.c)

```c
static int mro_internal(PyTypeObject *type) {
    PyObject *mro;
    PyObject *bases;
    
    // Базовый случай
    if (PyTuple_GET_SIZE(type->tp_bases) == 0) {
        mro = PyTuple_New(1, (PyObject *)type);
        type->tp_mro = mro;
        return 0;
    }
    
    // C3 линейнаязация (псевдокод)
    mro = PyTuple_New(0);
    todo = list(type->tp_bases);
    
    while (todo) {
        candidate = find_candidate(todo);  // Первый общий класс
        if (!candidate) {
            // Diamond problem!
            PyErr_SetObject(PyExc_TypeError, "MRO conflict");
            return -1;
        }
        append(candidate, mro);
        remove_from_todos(candidate, todo);
    }
    
    type->tp_mro = mro;
    return 0;
}
```

`class D(B, C):` → MRO должен быть `D, B, C, object`. **C3 алгоритм** решает *
*алмазную проблему** наследования. Ищет **первый** класс, **присутствующий** во **всех** путях. **Гарантирует**:
дочерний **перед** родителями, **линейный** порядок. `super()` следует **этому** порядку!

## 4. obj.name атрибутный поиск (Objects/object.c)

```c
PyObject *PyObject_GenericGetAttr(PyObject *obj, PyObject *name) {
    PyTypeObject *tp = Py_TYPE(obj);
    PyObject *descr;
    
    // 1. Ищем дескриптор в типе
    descr = _PyType_Lookup(tp, name);
    if (descr != NULL) {
        descr_getfunc f = Py_TYPE(descr)->tp_descr_get;
        if (f != NULL) {
            // Дескриптор! Вызываем __get__
            PyObject *res = f(descr, obj, (PyObject *)tp);
            if (res != NULL) {
                return res;
            }
        }
    }
    
    // 2. Ищем в __dict__ экземпляра
    if (tp->tp_dictoffset != 0) {
        PyObject **dictptr = _PyObject_GetDictPtr(obj);
        if (dictptr && *dictptr != NULL) {
            PyObject *res = PyDict_GetItem(*dictptr, name);
            if (res != NULL) {
                Py_INCREF(res);
                return res;
            }
        }
    }
    
    // 3. Ищем в __dict__ типа
    if (PyType_HasFeature(tp, Py_TPFLAGS_HAVE_CLASS)) {
        PyObject *res = _PyType_Lookup(tp, name);
        if (res != NULL) {
            Py_INCREF(res);
            return res;
        }
    }
    
    // AttributeError
    PyErr_Format(PyExc_AttributeError,
        "'%.50s' object has no attribute '%.400s'",
        tp->tp_name, PyObject_GETITEM(name));
    return NULL;
}
```

`obj.name` → **3 места поиска**: 1) **класс** (property/method), 2) **__dict__
экземпляра**, 3) **__dict__ класса**. **Дескрипторы** (`property`) имеют **приоритет** — вызывается
`property.__get__(obj)`. `__dict__` — **PyDictObject** по offset в `tp_dictoffset`.

## 5. super() байткод + _PySuperObject (Objects/classobject.c)

```python
class B:
    def f(self): pass


class C:
    def f(self): pass


class D(B, C):
    def f(self):
        super().f()  # B.f(self)
```

```python
# Байткод D.f():
0
LOAD_SUPER_ATTR
0  # super().__getattr__('f')
2
LOAD_FAST
0(self)
4
CALL
1  # B.f(self)
```

**LOAD_SUPER_ATTR (ceval.c):**

```c
case LOAD_SUPER_ATTR: {
    PyObject *super = GETITEM(names, oparg>>8);     // super объект
    PyObject *attr = GETITEM(names, oparg&0xFF);   // 'f'
    
    // super = _PySuperObject(type=D, obj=self)
    _PySuperObject *s = (_PySuperObject*)super;
    
    // Ищем в MRO после D
    PyTypeObject *start = s->type;
    PyObject *mro = start->tp_mro;
    
    for (Py_ssize_t i = PySequence_Index(mro, (PyObject*)start) + 1; 
         i < PyTuple_GET_SIZE(mro); i++) {
        PyTypeObject *cls = PyTuple_GET_ITEM(mro, i);
        PyObject *res = _PyType_Lookup(cls, attr);
        if (res != NULL) {
            // НАЙДЕН! B.f
            Py_INCREF(res);
            PUSH(res);
            DISPATCH();
        }
    }
    
    PyErr_SetObject(PyExc_AttributeError, attr);
    goto error;
}
```

`super().f()` → `LOAD_SUPER_ATTR` → `_PySuperObject{D, self}` → **MRO**
`D, B, C, object` → **пропускаем D** → ищем в `B, C, object` → **B.f найден** → возвращаем **bound method** `B.f(self)`.
**Автоматически** вызывает **правильный** родитель!

## 6. Класс создание: compiler_class() (Python/compile.c)

```c
static int compiler_class(struct compiler *c, location loc, stmt_ty s) {
    identifier name = s->v.ClassDef.name;     // 'MyClass'
    asdl_expr_seq *bases = s->v.ClassDef.bases;  // (object,)
    asdl_keyword_seq *keywords = s->v.ClassDef.keywords;
    
    // 1. Создаём namespace для класса
    if (!compiler_enter_scope(c, name, COMPILER_SCOPE_CLASSBLOCK, s, loc)) {
        return -1;
    }
    
    // 2. Базовые классы
    VISIT_SEQ(c, expr, bases);                 // LOAD_GLOBAL object
    
    // 3. Ключевые аргументы (metaclass)
    if (keywords) {
        VISIT_KEYWORDS(c, keywords);
    }
    
    // 4. Тело класса
    VISIT_SEQ(c, stmt, s->v.ClassDef.body);
    
    // 5. Генерируем: MyClass = type(name, (object,), {})
    ADDINSTR(c, loc, BUILD_TUPLE, 1);          // bases tuple
    ADDINSTR(c, loc, LOAD_CONST, add_empty_dict);  // {}
    ADDINSTR(c, loc, CALL_FUNCTION_EX, 1);     // type(name, bases, {})
    ADDINSTR(c, loc, STORE_NAME, add(name));   // MyClass =
    
    compiler_exit_scope(c);
    return 0;
}
```

`class MyClass(object): pass` → байткод: `bases=(object,)`,
`dict={}, CALL_FUNCTION_EX(1)` → **вызов** `type('MyClass', (object,), {})` → **PyTypeObject** создания!

## 7. type.__call__() = Class() создание (Objects/typeobject.c)

```c
static PyObject *type_call(PyTypeObject *type, PyObject *args, PyObject *kwds) {
    PyObject *obj;
    
    // 1. Выделяем экземпляр
    if (type->tp_new == NULL) {
        PyErr_Format(PyExc_TypeError,
            "cannot create '%.100s' instances",
            type->tp_name);
        return NULL;
    }
    
    obj = type->tp_new(type, args, kwds);  // MyClass.__new__()
    if (obj == NULL) {
        return NULL;
    }
    
    // 2. Инициализируем
    if (PyType_HasFeature(type, Py_TPFLAGS_HAVE_INIT) &&
        type->tp_init != NULL) {
        int res = type->tp_init(obj, args, kwds);  // MyClass.__init__()
        if (res < 0) {
            Py_DECREF(obj);
            return NULL;
        }
    }
    
    return obj;
}
```

`MyClass()` → `type_call(&MyClass_Type)` → `MyClass.tp_new(MyClass)` → **malloc** →
`MyClass.tp_init(obj)` → **готовый экземпляр**. По умолчанию `object.__new__()` + `object.__init__()`.

## 8. Property дескриптор (Objects/descrobject.c)

```c
static PyObject *property_getter(PyObject *self, PyObject *obj, PyObject *type) {
    // self = property(fget=name)
    PyObject *func = ((PyPropertyObject *)self)->prop_get;
    
    if (func == NULL) {
        PyErr_SetString(PyExc_AttributeError, "unreadable property");
        return NULL;
    }
    
    // Вызываем getter функцию
    PyObject *args = PyTuple_New(1, obj);  // (self,)
    PyObject *result = PyObject_Call(func, args, NULL);
    Py_DECREF(args);
    return result;
}
```

`class C: @property def x(self): return 42` → **C.x** = **PyPropertyObject** с
`prop_get=fget`. `c = C(); c.x` → `property_getter(property, c, C)` → `fget(c)` → `42`. **Автоматически**!

**ООП** в CPython 3.9+ — **PyTypeObject** (класс=тип), **MRO** C3 алгоритм, **descriptor протокол** (`__get__`),
`PyType_Ready()`, **`super()`** → MRO поиск, `type.__call__()` → `__new__`+`__init__`, **property** дескрипторы.

- [Содержание](#содержание)

---

# **Абстракция**

## **Junior Level**

Абстракция в объектно-ориентированном программировании — это процесс выделения существенных характеристик объекта и
игнорирования несущественных деталей. Представьте, что вы заказываете пиццу: вас интересует её состав и цена, но не
детали того, как её готовят на кухне или доставляют курьером. Вы абстрагируетесь от сложных процессов, фокусируясь
только на том, что важно для вас как заказчика.

В программировании абстракция позволяет создавать модели реальных объектов, которые содержат только те свойства и
методы, которые нужны для решения конкретной задачи. Например, в программе для библиотеки класс "Книга" может иметь
свойства "автор", "название", "год издания" и методы "взять", "вернуть". Но он не будет включать такие детали, как "вес
книги", "цвет обложки" или "материал страниц", если они не важны для работы библиотечной системы.

Абстракция делает код проще, понятнее и легче для изменения.

## **Middle Level**

В Python абстракция реализуется через несколько механизмов, каждый из которых предоставляет свой уровень сокрытия
деталей:

1. **Абстрактные классы**:
    - Определяются с помощью модуля `abc` (abstract base classes).
    - Не могут быть инстанциированы напрямую.
    - Содержат абстрактные методы (помеченные `@abstractmethod`), которые должны быть реализованы в дочерних классах.
    - `@abstractproperty` (устаревшее в Python 3.3+) и комбинация `@property` с `@abstractmethod`.

2. **Интерфейсы**:
    - В Python нет явных интерфейсов как в Java/C#, но их роль выполняют абстрактные классы и протоколы.
    - Протоколы — неформальные интерфейсы, основанные на утиной типизации.

3. **Инкапсуляция как средство абстракции**:
    - `_single_underscore`: защищённый атрибут (соглашение).
    - `__double_underscore`: приватный атрибут с name mangling.
    - Свойства (`@property`) для контроля доступа к атрибутам.

4. **Уровни абстракции**:
    - **Высокоуровневые абстракции**: интерфейсы, абстрактные классы.
    - **Среднеуровневые**: конкретные классы с детализированным поведением.
    - **Низкоуровневые**: вспомогательные классы, детали реализации.

5. **Шаблоны проектирования, основанные на абстракции**:
    - **Фабричный метод**: абстрагирует процесс создания объектов.
    - **Мост (Bridge)**: разделяет абстракцию и реализацию.
    - **Стратегия (Strategy)**: абстрагирует семейство алгоритмов.

6. **Абстракция данных**:
    - Создание типов данных, которые скрывают свою внутреннюю структуру.
    - Предоставление только операций для работы с данными.

## **Senior Level**

В CPython 3.9+ **абстракция** — **ABC** (`Lib/abc.py` + `Lib/_collections_abc.py`) с **ABCMeta** метаклассом, *
*`abstractmethod`** (`__isabstractmethod__=True`), **`register()`** виртуальное наследование, *
*`__subclasscheck__`/`__instancecheck__`** переопределение `issubclass`/`isinstance`, **structural typing** через
`typing.Protocol`. `Lib/abc.py`,`Objects/abstract.c`

## 1. ABCMeta.__new__() - метакласс ABC (Lib/abc.py)

```python
class ABCMeta(type):
    def __new__(mcls, name, bases, namespace, **kwargs):
        # 1. Создаём обычный класс
        cls = super().__new__(mcls, name, bases, namespace, **kwargs)

        # 2. Отмечаем абстрактные методы
        abstracts = []
        for key, value in namespace.items():
            if getattr(value, '__isabstractmethod__', False):
                abstracts.append(key)

        # 3. Если есть абстрактные методы - класс абстрактный
        cls.__abstractmethods__ = set(abstracts)

        # 4. Регистрируем в _abc_registry
        _abc_registry[cls] = None

        return cls
```

`class MyABC(ABC): @abstractmethod def method(self): pass` → **ABCMeta** видит `__isabstractmethod__=True` на `method` →
`MyABC.__abstractmethods__ = {'method'}`. **ABCMeta** — **обычный метакласс**, но **добавляет** `__abstractmethods__` и
регистрирует в глобальном `_abc_registry` словаре.

## 2. @abstractmethod декоратор (Lib/abc.py)

```python
def abstractmethod(funcobj):
    """Устанавливает __isabstractmethod__ = True"""
    funcobj.__isabstractmethod__ = True
    return funcobj
```

`@abstractmethod def f(): pass` → `f.__isabstractmethod__ = True`. **Метакласс** сканирует `__dict__` класса, находит
все такие методы → **помечает класс абстрактным**. **Очень просто** — один атрибут!

## 3. ABCMeta.__subclasscheck__() - переопределение issubclass()

```python
def __subclasscheck__(cls, subclass):
    # 1. Обычная проверка наследования
    if cls in getattr(subclass, '__mro__', ()):
        return True

    # 2. Проверяем register()
    for s in subclass.__mro__:
        if cls in _abc_impl_cache.get(s, ()):
            return True

    # 3. Проверяем реализацию абстрактных методов
    ok = all(cls.__abstractmethod_subset(obj) for obj in subclass.__mro__)
    if ok:
        _abc_impl_cache[subclass].add(cls)
        return True

    return False
```

`issubclass(MyClass, Sequence)` → **НЕ** смотрит на наследование! Проверяет: 1) **наследуется** ли от
зарегистрированного класса, 2) **реализует** ли все методы `Sequence` (`__len__`, `__getitem__`). `MyList(Sequence)` → *
*True** даже **без** `class MyList(Sequence):`!

## 4. ABC.register() - виртуальное наследование (Lib/abc.py)

```python
def register(cls, subclass):
    """Регистрирует subclass как виртуальный"""
    if not issubclass(subclass, cls):
        for obj in subclass.__mro__:
            if cls in _abc_cache.get(obj, ()):
                break
        else:
            raise RuntimeError('Refusing to register %r as subclass of %r' %
                               (subclass, cls))

    # Добавляем в кэш
    _abc_impl_cache.setdefault(subclass, set()).add(cls)
    _abc_impl_cache.setdefault(cls, set()).add(subclass)

    return subclass
```

`Sequence.register(MyList)` → `issubclass(MyList, Sequence) = True` **навсегда**! `MyList` **НЕ** наследует код
`Sequence`, но **поддерживает** протокол (`__len__`, `__getitem__`). **Duck typing** + **явная регистрация**.

## 5. collections.abc.Sequence реализация (Lib/_collections_abc.py)

```python
class Sequence(Collection):
    """Последовательности: list, tuple, str"""
    __slots__ = 'register', '__abstractmethods__'

    @abstractmethod
    def __getitem__(self, index):
        """Получить элемент по индексу"""
        return NotImplemented

    @abstractmethod
    def __len__(self):
        """Длина последовательности"""
        return NotImplemented

    def __iter__(self):
        """Итератор"""
        return SequenceIter(self)

    def __contains__(self, value):
        """Проверка вхождения"""
        for item in self:
            if item == value:
                return True
        return False
```

`Sequence` **НЕ** содержит кода! Только **список абстрактных методов** (`__len__`, `__getitem__`).
`isinstance(my_list, Sequence)` → проверяет **наличие** этих методов. **Реализация** в `list.c` (`list_length`,
`list_subscript`).

## 6. PySequence_Check() C API (Objects/abstract.c)

```c
int PySequence_Check(PyObject *s) {
    PyTypeObject *tp = Py_TYPE(s);
    
    // 1. Быстрая проверка по tp_as_sequence
    if (tp->tp_as_sequence && tp->tp_as_sequence->sq_item != NULL) {
        return 1;                          // Реальная последовательность
    }
    
    // 2. ABC проверка
    if (PyType_FastSubclass(tp, Py_TPFLAGS_SEQUENCE)) {
        return 1;
    }
    
    // 3. Проверяем __getitem__
    PyObject *item = PyObject_GetItem(s, PyLong_FromLong(0));
    Py_XDECREF(item);
    if (!PyErr_Occurred()) {
        return 1;                          // Поддерживает __getitem__
    }
    PyErr_Clear();
    
    return 0;
}
```

`len(my_list)` → `PySequence_Check(my_list)` → `PyList_Type.tp_as_sequence->sq_length != NULL` → **True**. **Быстрый
путь** через C слоты, **медленный** через `__getitem__`.

## 7. list_subscript() - реализация __getitem__ (Objects/listobject.c)

```c
static PyObject *list_subscript(PyListObject *self, PyObject *item) {
    if (PyLong_Check(item)) {
        Py_ssize_t i = PyLong_AsSsize_t(item);
        if (i < 0)
            i += Py_SIZE(self);
        if (i >= 0 && i < Py_SIZE(self)) {
            return Py_NewRef(self->ob_item[i]);
        }
    }
    PyErr_SetString(PyExc_IndexError, "list index out of range");
    return NULL;
}
```

`my_list[0]` → `PyObject_GetItem(my_list, 0)` → `list_subscript(my_list, 0)` → `my_list->ob_item[0]`. **Прямой доступ**
к массиву указателей!

## 8. typing.Protocol - structural typing (typing.py)

```python
class Protocol(Generic, metaclass=ProtocolMeta):
    """Структурная типизация - duck typing"""
    __slots__ = ()

    def __instancecheck__(cls, instance):
        # Проверяем наличие методов по сигнатуре
        return _is_protocol_compatible(instance, cls.__dict__)

    def __subclasscheck__(cls, subclass):
        # Проверяем наличие абстрактных методов
        return _is_protocol_compatible(subclass, cls.__dict__)


def _is_protocol_compatible(obj, protocol_attrs):
    for attr_name, attr_value in protocol_attrs.items():
        if attr_name.startswith('_'):
            continue
        obj_attr = getattr(obj, attr_name, None)
        if obj_attr != attr_value:
            return False
    return True
```

`class HasLen(Protocol): def __len__(self) -> int: ...` → `isinstance(my_list, HasLen)` → **НЕ** наследование! Проверяет
**наличие** `__len__` с **правильной** сигнатурой. **Структурная типизация** — работает **любая** реализация!

## 9. PyList_Type.tp_as_sequence - C протокол (Include/listobject.h)

```c
static PySequenceMethods list_as_sequence = {
    .sq_length = list_length,          // len(lst)
    .sq_concat = list_concat,          // lst + lst2
    .sq_repeat = list_repeat,          // lst * 3
    .sq_item = list_subscript,         // lst[i]
    .sq_slice = list_slice,            // lst[1:3]
    .sq_ass_item = list_ass_item,      // lst[i] = x
    .sq_ass_slice = list_ass_slice,    // lst[1:3] = [...]
    .sq_contains = list_contains,      // x in lst
    .sq_inplace_concat = list_inplace_concat,
    .sq_inplace_repeat = list_inplace_repeat,
};
```

`PyList_Type.tp_as_sequence = &list_as_sequence` → `len(lst)` → `PySequence_Length(lst)` →
`lst->tp_as_sequence->sq_length(lst)` → `list_length()`. **Слоты** — **таблица указателей** на C функции!

## 10. Байткод: isinstance(obj, Sequence) (ceval.c)

```c
case ISINSTANCE: {
    PyObject *inst = TOP();            // obj
    PyObject *cls = SECOND();          // Sequence
    
    int retval = PyObject_IsInstance(inst, cls);  // Вызывает __instancecheck__
    
    Py_DECREF(inst);
    Py_DECREF(cls);
    
    if (retval < 0) {
        goto error;
    }
    
    Py_SETREF(SECOND(), PyBool_FromLong(retval));
    DISPATCH();
}
```

`isinstance(lst, Sequence)` → `PyObject_IsInstance(lst, Sequence)` → `Sequence.__instancecheck__(lst)` → **True**. *
*Делегирует** ABC метаклассу!

**Абстракция** в CPython 3.9+ — **ABCMeta** (`__subclasscheck__`), **`abstractmethod`** (`__isabstractmethod__`), *
*`register()`** виртуальное наследование, **`tp_as_sequence`** C протоколы, **`typing.Protocol`** structural typing, *
*`PySequence_Check()`** duck typing.

- [Содержание](#содержание)

---

# **Инкапсуляция**

## **Junior Level**

Инкапсуляция — это принцип объектно-ориентированного программирования, который объединяет данные и методы, работающие с
этими данными, внутри одного объекта и скрывает внутренние детали реализации от внешнего мира.

Представьте, что у вас есть банковский счет. Вы можете пополнять его, снимать деньги и проверять баланс, но вам не нужно
знать, как именно банк хранит ваши данные, как они обрабатывают транзакции или как обновляют баланс. Вам предоставляют
простой интерфейс (например, банкомат или мобильное приложение), а все сложности скрыты внутри.

В Python инкапсуляция реализуется через модификаторы доступа: публичные (public), защищенные (protected) и приватные (
private) атрибуты и методы. Это помогает защитить данные от неправильного использования и обеспечивает контроль над тем,
как объект взаимодействует с внешним миром.

## **Middle Level**

В Python инкапсуляция имеет свои особенности из-за динамической природы языка:

1. **Уровни доступа**:
    - **Публичные (public)**: Обычные атрибуты и методы, доступные отовсюду.
    - **Защищенные (protected)**: Имена с одним подчёркиванием (`_attr`). Это соглашение, а не строгая защита. Говорит
      разработчикам: "это для внутреннего использования".
    - **Приватные (private)**: Имена с двумя подчёркиваниями (`__attr`). Python применяет **name mangling** (искажение
      имён), превращая `__attr` в `_ClassName__attr`. Это затрудняет случайный доступ, но не делает атрибут полностью
      недоступным.

2. **Свойства (property)**:
    - Декораторы `@property`, `@attr.setter`, `@attr.deleter`.
    - Позволяют контролировать доступ к атрибутам, добавляя логику при чтении, записи или удалении.
    - Сохраняют синтаксис доступа как к обычному атрибуту.

3. **Дескрипторы**:
    - Классы, реализующие протокол `__get__`, `__set__`, `__delete__`.
    - Лежат в основе свойств, методов класса, статических методов.
    - Позволяют создавать переиспользуемые механизмы управления доступом.

4. **Методы доступа (getters/setters)**:
    - В Python не принято создавать простые getters/setters для всех атрибутов.
    - Используются только при необходимости добавить логику (валидацию, вычисления).
    - Иначе нарушается принцип "We're all consenting adults here".

5. **`__slots__`**:
    - Ограничивает набор атрибутов экземпляра.
    - Экономит память, предотвращая создание `__dict__`.
    - Ограничивает динамическое добавление атрибутов.

6. **Интерфейсы и абстракции**:
    - Абстрактные классы (`abc.ABC`) определяют контракты без раскрытия реализации.
    - Протоколы (`typing.Protocol`) для структурной типизации.

## **Senior Level**

В CPython 3.9+ **инкапсуляция** — **name mangling** (`_Class__private` в compile.c), **`__slots__`** (
`tp_dictoffset=0`), **descriptor протокол** (`property`/`@property`), **`__getattribute__`** защита, **`__dict__`** по
`tp_dictoffset`. **НЕ** есть `private/protected`. `Python/compile.c`,`Objects/object.c`,`Objects/descrobject.c`

## 1. Name mangling в компиляторе (Python/compile.c)

```c
static const char * _Py_Mangle(const char *privateobj, const char *privatename) {
    /* Name mangling: __private -> _Class__private */
    const char *p;
    size_t len_privateobj = strlen(privateobj);
    size_t len_privatename = strlen(privatename);
    size_t len = len_privateobj + len_privatename + 1;
    char *mangled = PyMem_Malloc(len + 1);
    
    if (mangled == NULL)
        return NULL;
    
    /* Skip leading underscores (_Class -> Class) */
    p = privateobj;
    while (*p == '_')
        p++;
    
    /* privateobj + privatename */
    strcpy(mangled, "_");
    strcat(mangled, p);           // _Class
    strcat(mangled, privatename); // _Class__private
    
    return mangled;
}

static identifier compiler_nameop(struct compiler *c, location loc, identifier name, expr_context_ty ctx) {
    if (ctx == Load && c->u->u_scope_type == COMPILER_SCOPE_CLASS &&
        PyUnicode_GET_LENGTH(name) >= 2 &&
        PyUnicode_READ_CHAR(name, 0) == '_' &&
        PyUnicode_READ_CHAR(name, 1) == '_') {
        
        /* __private -> _Class__private */
        identifier class_name = c->u->u_private;  // 'MyClass'
        mangled = _Py_Mangle(PyUnicode_AsUTF8(class_name), PyUnicode_AsUTF8(name));
        if (mangled == NULL)
            return NULL;
        
        name = PyUnicode_FromString(mangled);
        PyMem_Free(mangled);
    }
    return name;
}
```

`class MyClass: def __private(self): pass` → компилятор **заменяет** `__private` на `_MyClass__private` **в байткоде**.
`LOAD_NAME '__private'` → `LOAD_NAME '_MyClass__private'`. **Подкласс** `class Sub(MyClass):` ищет `__private` →
`_Sub__private` (НЕ находит родительский). **Не private** — просто **избегает конфликтов** имён!

## 2. Байткод с mangling

```python
class MyClass:
    def __private(self):
        pass


class Sub(MyClass):
    def f(self):
        self.__private()  # _Sub__private()
```

```
# Байткод Sub.f():
  0 LOAD_FAST           0 (self)
  2 LOAD_ATTR           0 (_Sub__private)  # <- mangled!
  4 CALL_FUNCTION       1
  6 POP_TOP
  8 LOAD_CONST          0 (None)
 10 RETURN_VALUE
```

В байткоде **НЕ** `self.__private()`, а `self._Sub__private()`. Компилятор **заменил** имя **статически**.
`MyClass().__private()` → **AttributeError** (ищет `_MyClass__private`).

## 3. __slots__ - без __dict__ (Objects/typeobject.c)

```c
int PyType_Ready(PyTypeObject *type) {
    // ...
    
    /* __slots__ = () -> tp_dictoffset=0 */
    if (type->tp_dictoffset == 0) {
        /* Нет __dict__ - экономим память */
        type->tp_flags |= Py_TPFLAGS_HAVE_SLOTS;
    }
    
    /* Вычисляем реальный размер экземпляра */
    if (type->tp_itemsize != 0) {
        /* Переменная длина (str, tuple) */
        type->tp_basicsize = -type->tp_basicsize;
    }
    
    type->tp_flags |= Py_TPFLAGS_READY;
    return 0;
}

PyObject *type_new(PyTypeObject *type, PyObject *args, PyObject *kwargs) {
    Py_ssize_t nslots = type->tp_dictoffset / sizeof(PyObject *);
    
    /* Выделяем память */
    PyObject *inst = PyObject_MALLOC(type->tp_basicsize + 
                                   nslots * sizeof(PyObject *));
    
    if (inst == NULL)
        return PyErr_NoMemory();
    
    /* НЕТ __dict__ если slots */
    if (type->tp_dictoffset == 0) {
        /* slots = ['name', 'age'] -> фиксированные поля */
        memset(inst + type->tp_basicsize, 0, nslots * sizeof(PyObject *));
    }
    
    return inst;
}
```

`class Point: __slots__ = ['x', 'y']` → `Point.tp_dictoffset = 0` → **НЕТ** `PyDictObject __dict__` (экономия ~64
байт!). `p.x` → **фиксированное поле** по offset (не поиск в хеш-таблице). `p.z = 1` → **AttributeError**!

## 4. __getattribute__ защита (Objects/object.c)

```c
static PyObject *instance_getattro(PyObject *self, PyObject *name) {
    PyInstanceObject *inst = (PyInstanceObject*)self;
    PyTypeObject *type = Py_TYPE(self);
    
    /* 1. __getattribute__ перехватывает */
    if (type->tp_getattro != PyObject_GenericGetAttr) {
        return type->tp_getattro(self, name);
    }
    
    /* 2. Дескрипторы класса */
    PyObject *descr = _PyType_Lookup(type, name);
    if (descr != NULL) {
        descrgetfunc f = Py_TYPE(descr)->tp_descr_get;
        if (f != NULL) {
            PyObject *res = f(descr, self, (PyObject *)type);
            return res;
        }
    }
    
    /* 3. __dict__[name] */
    PyObject **dictptr = _PyObject_GetDictPtr(self);
    if (dictptr != NULL && *dictptr != NULL) {
        PyObject *res = PyDict_GetItem(*dictptr, name);
        if (res != NULL) {
            Py_INCREF(res);
            return res;
        }
    }
    
    /* 4. __getattr__ */
    if (type->tp_getattro == PyObject_GenericGetAttr) {
        PyObject *res = PyObject_GenericGetAttr(self, name);
        if (res != NULL)
            return res;
    }
    
    PyErr_Format(PyExc_AttributeError,
        "'%.50s' object has no attribute '%.400s'",
        type->tp_name, PyUnicode_AsUTF8(name));
    return NULL;
}
```

`class C: def __getattribute__(self, name): if name=='_private': raise AttributeError` → **все** `c._private` →
`__getattribute__` → **блок**. **НЕ** доходит до `__dict__`!

## 5. Property дескриптор (Objects/descrobject.c)

```c
typedef struct {
    PyObject_HEAD
    PyGetterDescrObject *prop_get;     // fget функция
    PySetterDescrObject *prop_set;     // fset функция
    PyDeleterDescrObject *prop_del;    // fdel функция
    PyObject *prop_doc;                // __doc__
} PyPropertyObject;

static PyObject *property_get(PyPropertyObject *self, PyObject *obj, PyObject *type) {
    PyObject *func = (PyObject*)self->prop_get;
    
    if (func == NULL) {
        PyErr_SetString(PyExc_AttributeError,
            "can't get attribute");
        return NULL;
    }
    
    /* Вызываем getter(self) */
    PyObject *args = PyTuple_New(1);
    PyTuple_SET_ITEM(args, 0, Py_NewRef(obj));
    PyObject *result = PyObject_Call(func, args, NULL);
    Py_DECREF(args);
    return result;
}
```

`@property def x(self): return self._x` → **PyPropertyObject** с `prop_get=fget`. `c.x` →
`property_get(property, c, C)` → `fget(c)` → `_x`. **НЕ** хранит значение — **вычисляет**!

## 6. __slots__ layout в памяти

```c
class Point:
    __slots__ = ['x', 'y']  # 2 * PyObject* = 16 байт

# Point в памяти:
PyObject_HEAD (16 байт) + x (8 байт) + y (8 байт) = 32 байт
# Без slots: PyObject_HEAD + __dict__ (64+ байт) = 80+ байт!
```

**Обычный класс**: `[refcnt][type][__dict__ ptr][данные]`. **`__slots__`**: `[refcnt][type][x ptr][y ptr]`. **НЕТ**
хеш-таблицы `__dict__` — **фиксированные поля** по offset. **10x быстрее** доступ + **экономия памяти**.

## 7. Байткод доступа к атрибуту

```python
class C:
    def __init__(self):
        self.x = 42


c = C()
print(c.x)
```

```
# Байткод:
  0 LOAD_GLOBAL         0 (C)
  2 CALL                0
  4 STORE_FAST          0 (c)
  6 LOAD_FAST           0 (c)
  8 LOAD_ATTR           0 (x)        # c.x -> PyObject_GetAttr
 10 CALL                1 (print)
```

`LOAD_ATTR 0 (x)` → `PyObject_GetAttr(c, 'x')` → `instance_getattro(c, 'x')` → **поиск**: класс → `__dict__` →
`__getattr__`.

## 8. Защита через __setattr__ (user-level)

```python
class SafeDict(dict):
    def __setattr__(self, name, value):
        if name.startswith('_'):
            raise AttributeError(f"cannot set {name}")
        object.__setattr__(self, name, value)

    def __setitem__(self, key, value):
        if key.startswith('_'):
            raise KeyError(f"cannot set {key}")
        super().__setitem__(key, value)
```

`sd = SafeDict(); sd._secret = 1` → `__setattr__` → **блок**. `sd['_secret'] = 1` → `__setitem__` → **блок**. **Двойная
защита**!

**Инкапсуляция** в CPython 3.9+ — **name mangling** (`_Class__private` compile-time), **`__slots__`** (
`tp_dictoffset=0`), **`__getattribute__`/`__setattr__`** перехват, **property дескрипторы** (`prop_get`), *
*`PyObject_GenericGetAttr()`** порядок поиска.

- [Содержание](#содержание)

---

# **Наследование**

## **Junior Level**

Наследование в ООП — это механизм, позволяющий создавать новый класс на основе существующего, перенимая его свойства и
методы. Новый класс называется **дочерним** (подклассом), а существующий — **родительским** (суперклассом).

Представьте, что у вас есть класс `Animal` с методами `eat()` и `sleep()`. Вы можете создать класс `Dog`, который
наследует от `Animal`, и автоматически получит эти методы. Затем вы можете добавить специфичное поведение для собаки:
метод `bark()`. Это позволяет избежать дублирования кода и создавать логические иерархии объектов.

Наследование представляет отношение **"является"** (is-a): собака является животным. Это один из основных способов
достижения полиморфизма — возможности использовать объекты разных классов через общий интерфейс.

## **Middle Level**

В Python наследование имеет несколько ключевых особенностей:

1. **Синтаксис наследования**:
   ```python
   class Parent:
       pass
   
   class Child(Parent):
       pass
   ```

2. **Множественное наследование**:
   Python поддерживает наследование от нескольких классов:
   ```python
   class Child(Parent1, Parent2, Parent3):
       pass
   ```

3. **Переопределение методов**:
   Дочерний класс может переопределять методы родительского класса, предоставляя свою реализацию.

4. **Расширение методов**:
   Использование `super()` для вызова родительской реализации:
   ```python
   def method(self):
       super().method()  # Вызов родительского метода
       # Дополнительная логика
   ```

5. **Method Resolution Order (MRO)**:
   Порядок поиска методов при множественном наследовании. Определяется алгоритмом C3 и доступен через
   `ClassName.__mro__`.

6. **Абстрактные классы и наследование**:
   Использование `abc.ABC` для создания классов, которые нельзя инстанциировать, но от которых можно наследоваться.

7. **Миксины (Mixins)**:
   Классы, предназначенные для добавления функциональности через множественное наследование. Не предназначены для
   самостоятельного использования.

8. **Доступ к родительским атрибутам**:
    - Через `super()`
    - Через явное указание имени родительского класса: `ParentClass.method(self)`

9. **Наследование встроенных типов**:
   Можно наследоваться от `list`, `dict`, `str` и других встроенных типов, но это требует осторожности.

10. **Композиция vs наследование**:
    Наследование создаёт жёсткую связь "является", композиция — гибкую связь "имеет". Композиция часто предпочтительнее.

## **Senior Level**

В CPython 3.9+ **наследование** реализовано через **PyTypeObject.tp_bases** (кортеж баз), **tp_base** (один общий
базовый), **tp_mro** (готовый MRO C3), и работу `PyType_Ready()` + `type.__call__` + `LOAD_SUPER_ATTR` для
`super()`.[1][2]

***

## PyTypeObject и хранение баз

```c
// Упрощённая часть PyTypeObject (Objects/typeobject.c)
typedef struct _typeobject {
    PyObject_VAR_HEAD              // refcnt, type="type", имя класса
    const char *tp_name;           // "MyClass"
    Py_ssize_t tp_basicsize;       // размер экземпляра
    Py_ssize_t tp_itemsize;        // для переменной длины

    // ... много слотов опущено ...

    PyObject *tp_bases;            // tuple базовых классов (B, C)
    PyTypeObject *tp_base;         // один "основной" базовый тип
    PyObject *tp_mro;              // tuple MRO: (D, B, C, object)
    PyObject *tp_dict;             // __dict__ самого класса
    PyObject *tp_subclasses;       // список слабых ссылок на подклассы

    unsigned long tp_flags;        // флаги (Py_TPFLAGS_HAVE_GC, BASETYPE и т.п.)
} PyTypeObject;
```

- `tp_bases` — кортеж всех указанных баз (`(B, C)` у `class D(B, C): ...`).
- `tp_base` — “основный” один базовый тип (обычно первый в `tp_bases`, либо `object`).
- `tp_mro` — кортеж **линейного порядка разрешения методов** (`(D, B, C, object)` для ромбовидного наследования).
- `tp_subclasses` — связка “родитель знает, какие у него дети” (для `__subclasses__()`).

каждый класс в CPython — это **C-структура** `PyTypeObject`, где есть: имя класса, размеры объектов, список родителей,
MRO, словарь методов, список детей. Наследование пишется в питоне, но *хранится* как поля `tp_bases/tp_mro` в этой
структуре.

***

## Создание класса и заполнение tp_bases (compiler_class)

```c
// Python/compile.c — генерация кода для class
static int
compiler_class(struct compiler *c, location loc, stmt_ty s) {
    // s->v.ClassDef.name = имя класса (identifier)
    // s->v.ClassDef.bases = список выражений базовых классов

    // 1) скомпилировать выражения баз, они окажутся на стеке
    VISIT_SEQ(c, expr, s->v.ClassDef.bases);  // LOAD_NAME B, LOAD_NAME C, ...

    // 2) собрать их в tuple
    ADDOP_I(c, loc, BUILD_TUPLE, PySequence_SIZE(s->v.ClassDef.bases));
    // теперь на стеке tuple(bases)

    // 3) создать namespace (обычно dict)
    ADDOP(c, loc, LOAD_BUILD_CLASS);       // функция builtins.__build_class__
    // ... компиляция тела класса в отдельную функцию-прослойку ...

    // 4) вызвать __build_class__(body_func, name, bases_tuple, **kw)
    // возвращённый объект – готовый класс (= PyTypeObject на C стороне)
    ADDOP(c, loc, CALL_FUNCTION_EX, 1);    // CALL_FUNCTION_EX оparg=1: есть *args

    // 5) сохранить класс в locals
    ADDOP_NAME(c, loc, STORE_NAME, s->v.ClassDef.name, names);

    return 1;
}
```

`class D(B, C): ...` компилируется в вызов `__build_class__`, которому передаётся: функция-тело класса, строка `"D"`,
кортеж `(B, C)` и kwargs (метакласс, если был). Дальше всё делает C-реализация `type`/метакласса.

***

## type.__new__ и PyType_Ready: инициализация наследования

```c
// Objects/typeobject.c — создание нового типа (класса)
static PyObject *
type_new(PyTypeObject *metatype, PyObject *args, PyObject *kwds)
{
    // args: (name, bases, dict)
    PyObject *name, *bases, *dict;
    // ... разбор args ...
    // создаём "сырую" структуру PyHeapTypeObject
    PyHeapTypeObject *et = (PyHeapTypeObject *)PyType_GenericAlloc(metatype, 0);
    PyTypeObject *type = &et->ht_type;

    // заполняем tp_name, tp_bases, tp_dict
    type->tp_name = /* C-строка имени */;
    Py_INCREF(bases);
    type->tp_bases = bases;
    Py_INCREF(dict);
    type->tp_dict = dict;

    // вычисляем tp_base = подходящий один базовый
    type->tp_base = best_base(bases);    // обычно первый в bases

    // готовим тип (mro, слоты, flags и т.п.)
    if (PyType_Ready(type) < 0) {
        Py_DECREF(et);
        return NULL;
    }
    return (PyObject *)type;
}
```

`type.__new__` выделяет память под новый класс, кладёт туда имя, кортеж баз, словарь методов, выбирает один “главный”
базовый (`best_base`) и потом зовёт `PyType_Ready`, чтобы посчитать MRO и прочее. До `PyType_Ready` класс “сырой”,
после — “готовый”.

***

## MRO: mro_internal и C3-алгоритм

```c
// Objects/typeobject.c
static int
mro_internal(PyTypeObject *type)
{
    PyObject *mro;

    // Если метакласс переопределил mro() — вызвать его
    PyObject *meth = lookup_mro(type, &_Py_ID(mro)); // type.mro?
    if (meth != NULL) {
        mro = PyObject_CallNoArgs(meth);  // собственный mro()
        // проверка что это tuple типов...
    }
    else {
        // стандартный C3 linearization
        mro = mro_implementation(type);   // см ниже
    }

    if (mro == NULL)
        return -1;

    Py_XSETREF(type->tp_mro, mro);  // tp_mro = кортеж типов
    return 0;
}

// C3 linearization: D(B,C), B(A), C(A), A(object)
static PyObject *
mro_implementation(PyTypeObject *type)
{
    // строит списки: [bases], [mro(base1)], [mro(base2)], ...
    // и затем делает C3 merge, выбирая головные элементы,
    // которые не встречаются в хвостах других списков.
}
```

когда класс создаётся, нужно решить **в каком порядке** искать методы при множественном наследовании. MRO C3
гарантирует:

- дочерний класс **всегда** раньше родителей,
- порядок родителей **сохраняется**,
- каждый класс встречается **один раз**.

Это всё заранее кладётся в `tp_mro = (D, B, C, A, object)`.

***

## Поиск атрибутов по MRO: _PyType_Lookup

```c
// Objects/typeobject.c
PyObject *
_PyType_Lookup(PyTypeObject *type, PyObject *name)
{
    PyObject *mro = type->tp_mro;      // кортеж типов
    Py_ssize_t i, n = PyTuple_GET_SIZE(mro);

    for (i = 0; i < n; i++) {
        PyTypeObject *base = (PyTypeObject *)PyTuple_GET_ITEM(mro, i);
        PyObject *dict = base->tp_dict;            // __dict__ класса
        PyObject *res;

        // обычный поиск в dict
        res = PyDict_GetItemWithError(dict, name);
        if (res != NULL) {
            return res;  // нашли атрибут на этом уровне MRO
        }
        if (PyErr_Occurred()) {
            return NULL;
        }
    }
    return NULL;  // атрибут не найден
}
```

`obj.method` → `PyObject_GetAttr` → `_PyType_Lookup(type(obj), 'method')` → идём по `tp_mro`: сначала класс самого
объекта, потом его родителя, и т.д., пока не `object`. **Первое совпадение** выигрывает.

***

## Наследование реализаций слотов (tp_* из базового типа)

```c
// Objects/typeobject.c — после вычисления MRO
static int
type_ready_set_bases_and_slots(PyTypeObject *type)
{
    // берём "base" как tp_base (один главный базовый)
    PyTypeObject *base = type->tp_base;

    // наследуем слоты, если они не определены в потомке
    if (type->tp_as_number == NULL)
        type->tp_as_number = base->tp_as_number;
    if (type->tp_as_sequence == NULL)
        type->tp_as_sequence = base->tp_as_sequence;
    if (type->tp_as_mapping == NULL)
        type->tp_as_mapping = base->tp_as_mapping;
    // и т.д. для tp_iter, tp_call, tp_hash, tp_str, ...
}
```

если ты не переопределил, например, `__len__` в своём классе, а родитель — `list`, то у твоего класса `tp_as_sequence`
будет указывать на **те же** C-функции, что и у `PyList_Type`. То есть потомок **по-настоящему наследует** реализацию
методов на C-уровне, а не только имена.

***

## type.__call__ и вызов конструктора с наследованием

```c
// Objects/typeobject.c
static PyObject *
type_call(PyTypeObject *type, PyObject *args, PyObject *kwds)
{
    // 1) вызываем tp_new (может быть у потомка или у базового)
    PyObject *obj = type->tp_new(type, args, kwds);
    if (obj == NULL)
        return NULL;

    // 2) вызываем tp_init (конструктор), если есть
    if (type->tp_init != NULL) {
        int res = type->tp_init(obj, args, kwds);
        if (res < 0) {
            Py_DECREF(obj);
            return NULL;
        }
    }
    return obj;
}
```

`D()` при наследовании ведёт себя так:

- сначала вызывается `D.__new__` (если его нет — унаследованный от родителя),
- затем `D.__init__` (если не переопределён — срабатывает родительский).  
  С точки зрения C — это два указателя `tp_new` и `tp_init`, которые либо свои, либо скопированы из базового класса.

***

## super() и наследование: LOAD_SUPER_ATTR

```python
class B:
    def f(self): ...


class C(B):
    def f(self):
        super().f()
```

```
# Байткод C.f (3.11+ условно):
  0 LOAD_FAST           0 (self)
  2 LOAD_GLOBAL         0 (super)
  4 LOAD_FAST           0 (self)
  6 LOAD_CONST          1 (C)
  8 CALL                2        # super(C, self)
 10 LOAD_ATTR           1 (f)    # ищем f начиная после C в MRO
 12 CALL                0
 14 RETURN_VALUE
```

Центр механизма — суперкласс `_PySuper_Type`:

```c
// Objects/typeobject.c/Objects/classobject.c
typedef struct {
    PyObject_HEAD
    PyTypeObject *type;   // текущий класс (C)
    PyObject *obj;        // экземпляр (self)
    PyTypeObject *obj_type; // type(self)
} superobject;

// В __getattribute__ super'а:
static PyObject *
super_getattro(superobject *su, PyObject *name)
{
    // MRO объекта
    PyObject *mro = su->obj_type->tp_mro;

    // находим индекс su->type (C) в mro и начинаем поиск с i+1
    Py_ssize_t i = index_of(mro, (PyObject *)su->type);
    for (i = i+1; i < PyTuple_GET_SIZE(mro); i++) {
        PyTypeObject *base = (PyTypeObject *)PyTuple_GET_ITEM(mro, i);
        PyObject *descr = _PyType_Lookup(base, name);
        if (descr != NULL) {
            // возвращаем bound method/descriptor
            return PyObject_Get(descr, su->obj, (PyObject *)su->obj_type);
        }
    }
    // AttributeError, если не нашли
}
```

`super(C, self).f()` ищет `f` **не в C**, а в **следующих** по MRO классах (`B`, `object`, ...). Это делается, чтобы при
множественном наследовании все родительские реализации вызывались **один раз** и в **правильном порядке**.

***

## __bases__, __mro__ и __subclasses__ на C-уровне

```c
// Возврат __bases__ (type.__getattribute__)
static PyObject *
type_get_bases(PyTypeObject *type, void *context)
{
    if (type->tp_bases == NULL)
        Py_RETURN_NONE;
    Py_INCREF(type->tp_bases);
    return type->tp_bases;
}

static PyObject *
type_get_mro(PyTypeObject *type, void *context)
{
    if (type->tp_mro == NULL)
        Py_RETURN_NONE;
    Py_INCREF(type->tp_mro);
    return type->tp_mro;
}

static PyObject *
type_get_subclasses(PyTypeObject *type, PyObject *args)
{
    // tp_subclasses — список (list) weakref'ов на дочерние классы
    return _PyWeakref_GetRefList(type->tp_subclasses);
}
```

`D.__bases__` — это просто `tp_bases` (кортеж из `PyTypeObject*`), `D.__mro__` — `tp_mro`, `D.__subclasses__()` →
развёрнутый список weakref’ов из `tp_subclasses`. Все эти связи поддерживаются автоматически при создании/удалении
типов.

***

## Наследование встроенных типов (пример list)

```c
// Include/cpython/listobject.h
PyTypeObject PyList_Type = {
    PyVarObject_HEAD_INIT(&PyType_Type, 0)
    "list",                         // tp_name
    sizeof(PyListObject),           // tp_basicsize
    0,                              // tp_itemsize
    (destructor)list_dealloc,       // tp_dealloc
    // ...
    &list_as_sequence,              // tp_as_sequence
    &list_as_mapping,               // tp_as_mapping
    // ...
    &PyBaseObject_Type,             // tp_base = object
    // ...
};
```

Если написать:

```python
class MyList(list):
    pass
```

то создаётся `PyHeapTypeObject`, у которого:

- `tp_base` = `&PyList_Type`,
- слоты `tp_as_sequence`/`tp_as_mapping` по умолчанию **унаследованы** от листа,
- у тебя дополнительно будет свой `tp_dict` с методами класса Python’а.

экземпляр `MyList` в памяти — точно такой же массив элементов, как и `list`, плюс всё, что ты добавишь. Вся арифметика,
конкатенация, индексирование работают через `list_*` функции, потому что твой класс **наследует** их слоты.

***

Итого: наследование в CPython “под капотом” — это:

- создание нового `PyTypeObject` на базе `tp_bases`,
- вычисление `tp_mro` по C3,
- наследование функциональных слотов от `tp_base`,
- поиск атрибутов по `_PyType_Lookup` с проходом по `tp_mro`,
- `super()` реализован как объект, который стартует поиск не с текущего класса, а далее по MRO.


- [Содержание](#содержание)

---

# **Полиморфизм**

## **Junior Level**

Полиморфизм — это способность объектов с разной внутренней структурой иметь одинаковый интерфейс и реагировать на одни и
те же команды по-разному.

Представьте, что у вас есть пульт дистанционного управления, который работает с разными
устройствами: нажатие кнопки "включить" включает и телевизор, и кондиционер, и музыку, но делает это по-разному. Каждое
устройство понимает команду "включить" по-своему.

В программировании полиморфизм позволяет использовать один и тот же метод или функцию для работы с объектами разных
классов. Например, если у классов `Dog` и `Cat` есть метод `speak()`, то при вызове `dog.speak()` мы услышим "Гав!", а
при `cat.speak()` — "Мяу!". Это позволяет писать более гибкий и универсальный код.

В Python полиморфизм тесно связан с концепцией "утиной типизации" (duck typing): если объект ходит как утка и крякает
как утка, то он утка. То есть Python не проверяет тип объекта заранее, а смотрит, есть ли у него нужный метод в момент
вызова.

## **Middle Level**

В Python полиморфизм проявляется в нескольких формах:

1. **Утиная типизация (Duck Typing)**:
    - Основной механизм полиморфизма в Python
    - Объекты используются на основе наличия методов/атрибутов, а не их типа
    - Пример: любой объект с методом `__len__()` можно передать в функцию `len()`

2. **Переопределение методов в наследовании**:
    - Дочерние классы могут переопределять методы родительских классов
    - Вызов метода у объекта дочернего класса использует переопределённую версию

3. **Перегрузка операторов**:
    - Магические методы (`__add__`, `__str__`, `__getitem__`) позволяют определить поведение операторов для
      пользовательских классов
    - Один и тот же оператор (`+`, `==`, `[]`) работает по-разному для разных типов

4. **Абстрактные базовые классы (ABC)**:
    - Определяют интерфейсы, которые должны быть реализованы
    - `collections.abc` содержит абстрактные классы для коллекций (`Iterable`, `Sequence`, `Mapping`)

5. **Протоколы**:
    - Неформальные интерфейсы, основанные на наличии определённых методов
    - Пример: протокол итератора требует методы `__iter__()` и `__next__()`

6. **Функции высшего порядка**:
    - Функции, принимающие другие функции как аргументы или возвращающие функции
    - `map()`, `filter()`, `sorted()` с параметром `key`

7. **Множественная диспетчеризация**:
    - Используется в библиотеках типа `multipledispatch`
    - Выбор реализации функции на основе типов нескольких аргументов

8. **`singledispatch` из `functools`**:
    - Позволяет создавать перегруженные функции на основе типа первого аргумента

## **Senior Level**

В CPython 3.9+ **полиморфизм** “под капотом” – это:

1) **таблицы слотов в PyTypeObject** (`tp_as_number`, `tp_as_sequence`, `tp_call` и т.п.);
2) **общие “абстрактные” функции** (`PyNumber_Add`, `PySequence_GetItem`), которые по `Py_TYPE(obj)` берут нужную
   реализацию;
3) **дескрипторный протокол** и `tp_getattro` для полиморфных атрибутов и методов.

***

## 1. Слоты в PyTypeObject – “виртуальные таблицы”

```c
/* Упрощённый фрагмент PyTypeObject (Include/cpython/object.h) */
typedef struct _typeobject {
    PyObject_VAR_HEAD                /* стандартный заголовок */
    const char *tp_name;             /* имя типа, например "int" */
    Py_ssize_t tp_basicsize;         /* размер объекта */
    Py_ssize_t tp_itemsize;          /* для переменной длины */

    /* Числовой протокол: +, -, *, /, ... */
    struct PyNumberMethods *tp_as_number;   /* указатель на таблицу числовых операций */

    /* Последовательность: len, concat, повтор, индексирование */
    struct PySequenceMethods *tp_as_sequence; /* таблица seq-операций */

    /* Отображение: словарный интерфейс */
    struct PyMappingMethods *tp_as_mapping; /* таблица mapping-операций */

    unaryfunc tp_call;               /* вызов объекта: obj() */
    reprfunc tp_str;                 /* str(obj) */
    getattrofunc tp_getattro;        /* получение атрибута obj.attr */
    setattrofunc tp_setattro;        /* установка атрибута */

    /* ... много других слотов опущено ... */
} PyTypeObject;
```

Комментарии по строкам:

- `tp_as_number` – указатель на структуру `PyNumberMethods`, где лежат C-функции для операций вроде `+`, `-`, `*`, `==`
  и т.д.
- `tp_as_sequence` – структура `PySequenceMethods` с реализациями `len`, `obj[i]`, конкатенации и т.п.
- `tp_as_mapping` – структура `PyMappingMethods` с реализацией словарного интерфейса (`obj[key]`, `len(obj)` как
  mapping).
- `tp_call` – функция “вызвать объект”, используется для функций, классов, объектов с `__call__`.
- `tp_getattro` / `tp_setattro` – общий вход для доступа к атрибутам; конкретный тип может переопределять поведение.

“По‑человечески для тупых”: у каждого типа в CPython есть **набор указателей на функции**, которые реализуют операции.
Это аналог **vtable** в C++: есть один интерфейс “сложить два объекта”, но реализация берётся из `tp_as_number`
конкретного типа (`int`, `float`, `list` и т.п.). То есть “полиморфизм” – это просто **выбор строки из таблицы** по
`Py_TYPE(obj)`.

***

## 2. Числовой полиморфизм: PyNumber_Add и nb_add

```c
/* Include/cpython/object.h */
typedef struct {
    binaryfunc nb_add;         /* a + b */
    binaryfunc nb_subtract;    /* a - b */
    binaryfunc nb_multiply;    /* a * b */
    /* ... ещё много числовых операций ... */
} PyNumberMethods;

/* Objects/abstract.c – абстрактная операция сложения */
PyObject *PyNumber_Add(PyObject *v, PyObject *w)
{
    /* Берём типы операндов */
    PyTypeObject *vt = Py_TYPE(v);    /* тип левого */
    PyTypeObject *wt = Py_TYPE(w);    /* тип правого */

    /* Пытаемся вызвать nb_add у левого операнда */
    if (vt->tp_as_number && vt->tp_as_number->nb_add) {
        PyObject *res = vt->tp_as_number->nb_add(v, w);
        if (res != Py_NotImplemented) {
            return res;
        }
        Py_XDECREF(res);
    }

    /* Пытаемся вызвать nb_add у правого (обратное сложение) */
    if (wt->tp_as_number && wt->tp_as_number->nb_add &&
        wt != vt) {
        PyObject *res = wt->tp_as_number->nb_add(v, w);
        if (res != Py_NotImplemented) {
            return res;
        }
        Py_XDECREF(res);
    }

    /* Ни один тип не смог – выбрасываем TypeError */
    PyErr_Format(PyExc_TypeError,
                 "unsupported operand type(s) for +: '%.100s' and '%.100s'",
                 vt->tp_name, wt->tp_name);
    return NULL;
}
```

- `PyNumber_Add(a, b)` не знает, какие именно типы у `a` и `b`.
- Он смотрит на `tp_as_number->nb_add` у типа левого аргумента. Если есть – вызывает её.
- Если левый не умеет или вернул `Py_NotImplemented` – пробует правый аргумент (обратная операция, как `__radd__`).
- Если оба “не умеют” – выбрасывается `TypeError`.

когда ты пишешь `a + b`, интерпретатор **сначала смотрит**, какой у `a` тип, и есть ли у этого типа
реализация `+`. Если есть – вызывает её. Если этот тип не знает, как сложиться с типом `b`, он говорит “я не умею” (
`Py_NotImplemented`), и тогда пробуют уже реализацию `+` у типа `b`. Это и есть **динамический полиморфизм**.

***

## 3. Пример: nb_add для int и list

```c
/* Objects/longobject.c – для int */
static PyNumberMethods long_as_number = {
    .nb_add = long_add,     /* сложение больших целых */
    .nb_subtract = long_sub,
    .nb_multiply = long_mul,
    /* ... */
};

/* Objects/listobject.c – для list */
static PySequenceMethods list_as_sequence = {
    .sq_length = list_length,
    .sq_concat = list_concat,   /* lst1 + lst2 */
    /* ... */
};

/* В PyTypeObject типов */
PyTypeObject PyLong_Type = {
    /* ... */
    .tp_as_number = &long_as_number,
    /* ... */
};

PyTypeObject PyList_Type = {
    /* ... */
    .tp_as_sequence = &list_as_sequence,
    /* list не заполняет tp_as_number->nb_add ! */
};
```

- `int` реализует арифметические операции через `long_as_number.nb_add = long_add`.
- `list` **не** заполняет `tp_as_number->nb_add`, а операцию `+` реализует как последовательность (`sq_concat`), которую
  использует байткод `BINARY_ADD`, если оба операнда – последовательности.
- Разные типы реализуют “сложение” **через разные таблицы слотов**.

оператор `+` может значить “арифметику” (для чисел), а может значить “конкатенацию” (для списков,
строк). На C-уровне это реализовано тем, что у разных типов разные таблицы функций: у int – одна реализация, у list –
другая. Но Python‑код везде одинаковый: `a + b`.

***

## 4. Полиморфизм последовательностей: PySequence_GetItem

```c
/* Include/cpython/object.h */
typedef struct {
    lenfunc sq_length;            /* len(o) */
    binaryfunc sq_concat;         /* o1 + o2 */
    ssizeargfunc sq_repeat;       /* o * n */
    ssizeargfunc sq_item;         /* o[i] */
    /* ... прочие методы ... */
} PySequenceMethods;

/* Objects/abstract.c – абстрактный доступ по индексу */
PyObject *PySequence_GetItem(PyObject *s, Py_ssize_t i)
{
    PySequenceMethods *m;

    if (s == NULL)
        return null_error();

    /* Берём таблицу seq-операций у типа объекта */
    m = Py_TYPE(s)->tp_as_sequence;
    if (m && m->sq_item) {
        return m->sq_item(s, i);
    }

    /* Fallback: пробуем obj.__getitem__(i) */
    return PyObject_GetItem(s, PyLong_FromSsize_t(i));
}
```

- Для “настоящих” последовательностей (`list`, `tuple`, `str` и т.п.) заполнен `tp_as_sequence->sq_item`, и
  индексирование идёт **напрямую** в C.
- Для “псевдо‑последовательностей”, которые просто реализуют `__getitem__`, но не заполнили слоты, идёт медленный путь
  через `PyObject_GetItem`.

операция `obj[i]` **сначала** пытается использовать быстрый C‑слот (`sq_item`). Если тип
“неофициальный” и слоты не заполнены, используется “медленный” путь через обычный метод `__getitem__`. Но интерфейс (
`obj[i]`) один и тот же – полиморфизм.

***

## 5. Атрибутный полиморфизм: PyObject_GenericGetAttr + дескрипторы

```c
/* Objects/object.c */
PyObject *PyObject_GenericGetAttr(PyObject *obj, PyObject *name)
{
    PyTypeObject *tp = Py_TYPE(obj);

    /* 1. Ищем атрибут в MRO класса (включая родителей) */
    PyObject *descr = _PyType_Lookup(tp, name);
    descrgetfunc f = NULL;
    if (descr != NULL) {
        f = Py_TYPE(descr)->tp_descr_get;  /* есть ли __get__? */
        if (f != NULL && PyDescr_IsData(descr)) {
            /* data-дескриптор имеет приоритет над __dict__ */
            return f(descr, obj, (PyObject *)tp);
        }
    }

    /* 2. Ищем в __dict__ экземпляра */
    PyObject **dictptr = _PyObject_GetDictPtr(obj);
    if (dictptr && *dictptr) {
        PyObject *res = PyDict_GetItemWithError(*dictptr, name);
        if (res != NULL) {
            Py_INCREF(res);
            return res;
        }
        if (PyErr_Occurred())
            return NULL;
    }

    /* 3. Если был не-data дескриптор – вызываем его __get__ */
    if (descr != NULL && f != NULL) {
        return f(descr, obj, (PyObject *)tp);
    }

    /* 4. Если это просто атрибут класса – вернуть его как есть */
    if (descr != NULL) {
        Py_INCREF(descr);
        return descr;
    }

    /* 5. Ничего не нашли – AttributeError */
    PyErr_Format(PyExc_AttributeError,
                 "'%.50s' object has no attribute '%.400s'",
                 tp->tp_name, ...);
    return NULL;
}
```

- `_PyType_Lookup` идёт по `tp_mro` типа (наследование!) и ищет атрибут.
- Если атрибут – дескриптор (имеет `tp_descr_get`), он может вернуть *совсем другой объект* (например, bound‑метод).
- Порядок: сначала data‑дескрипторы (property с setter), потом `__dict__` экземпляра, потом non‑data дескрипторы (
  обычные методы), потом просто атрибуты класса.

`obj.attr` – это **полиморфный вызов**:

- если `attr` – property → вызывается его `fget(self)`;
- если `attr` – функция в классе → создаётся bound‑метод;
- если `attr` – просто поле в экземпляре → читается из `__dict__`.

Логика одна, а поведение зависит от конкретного типа и того, что лежит в его `__dict__`.

***

## 6. Полиморфный вызов объектов: tp_call

```c
/* Objects/abstract.c */
PyObject *PyObject_Call(PyObject *callable,
                        PyObject *args, PyObject *kwargs)
{
    PyTypeObject *tp = Py_TYPE(callable);
    ternaryfunc call = tp->tp_call;  /* указатель на функцию вызова */

    if (call == NULL) {
        PyErr_Format(PyExc_TypeError,
            "'%.200s' object is not callable",
            tp->tp_name);
        return NULL;
    }
    return call(callable, args, kwargs);
}
```

- `tp_call` реализует `()` для объектов: функции, классы, экземпляры с `__call__`.
- Для `PyFunctionObject` (`def f(...)`) в `tp_call` стоит функция, запускающая байткод.
- Для типа (`type`) – `tp_call` создаёт новый экземпляр (см. `type_call` выше).
- Для пользовательского объекта с `__call__` – `tp_call` у типа `type(object)` вызывает его метод `__call__`.

`something()` работает **одинаково** для функций, классов, объектов с `__call__`. Но на C‑уровне это
просто вызов `tp_call` конкретного типа – опять таблица из указателей.

***

## 7. Байткод полиморфного вызова: CALL / PRECALL

```python
def f(x): return x


f(10)
```

```
# Байткод (3.11+):
  0 LOAD_GLOBAL              0 (f)      # берём объект-вызываемый
  2 LOAD_CONST               1 (10)
  4 PRECALL                  1         # подготовка стека/fastcall
  6 CALL                     1         # общий вызов
  8 POP_TOP
 10 LOAD_CONST               0 (None)
 12 RETURN_VALUE
```

В интерпретаторе (`Python/ceval.c`):

```c
case CALL: {
    PyObject *callable = PEEK(oparg);      /* вызываемый объект */
    PyObject **args = stack_pointer - oparg; /* указатель на аргументы */

    /* Полиморфный fast-path: vectorcall, tp_call и т.п. */
    PyObject *res = _PyObject_Vectorcall(callable, args, oparg, NULL);
    if (res == NULL)
        goto error;

    /* снимаем callable и аргументы, кладём результат */
    STACKADJ(-(oparg+1));
    PUSH(res);
    DISPATCH();
}
```

байткод `CALL` вообще **не знает**, кого он вызывает: обычную функцию, метод, класс, объект‑функтор.
Он берёт объект и аргументы и передаёт их в `_PyObject_Vectorcall` / `PyObject_Call`, которые, глядя на тип, выбирают
правильный `tp_call` / fastcall‑функцию.

***

Итого, полиморфизм в CPython 3.9+ реализован через:

- **слоты в PyTypeObject** (`tp_as_number`, `tp_as_sequence`, `tp_call` и т.д.),
- **абстрактные функции** (`PyNumber_*`, `PySequence_*`, `PyMapping_*`), которые выбирают C‑реализацию по
  `Py_TYPE(obj)`,
- **общий механизм атрибутов** (`PyObject_GenericGetAttr` + дескрипторы),
- **полиморфный вызов** через `tp_call` и байткод `CALL`.

Всё это даёт привычный на уровне языка эффект: “одна и та же операция выполняется по‑разному в зависимости от типа
объекта”.

- [Содержание](#содержание)

---

# **Diamond Problem**

## **Junior Level**

Проблема ромбовидного наследования (Diamond Problem) — это классическая проблема в объектно-ориентированном
программировании, возникающая при множественном наследовании.
Представьте себе ромб (алмаз), где вершина — это базовый класс A, от него наследуются
два класса B и C, а от обоих наследуется класс D. Если в классе A есть метод `some_method()`, и классы B и C
переопределяют его по-разному, то возникает вопрос: какую реализацию унаследует класс D — от B или от C?

Эта проблема создает неоднозначность в поведении программы. В Python эта проблема решается с помощью четко определенного
**порядка разрешения методов (MRO - Method Resolution Order)**, который определяет, в каком порядке интерпретатор будет
искать метод в иерархии наследования.

## **Middle Level**

Технически в Python проблема решается алгоритмом **C3 linearization**, который гарантирует детерминированный и
предсказуемый порядок поиска методов.

1. **MRO (Method Resolution Order):**
    - MRO — это порядок, в котором Python ищет методы в иерархии классов. Его можно посмотреть через атрибут класса
      `__mro__` или метод `mro()`.
    - Алгоритм C3 гарантирует, что:
        - Каждый класс встречается в MRO ровно один раз.
        - Подклассы идут перед своими суперклассами.
        - Порядок наследования, указанный в определении класса, сохраняется.

2. **Пример:**
   Для иерархии A -> B, A -> C, B -> D, C -> D, MRO для D будет `[D, B, C, A, object]`. При вызове метода из D Python
   будет искать его в этом порядке.

3. **Механизм `super()`:**
    - `super()` — это не вызов родительского класса в традиционном понимании, а вызов следующего класса в MRO.
    - Это позволяет реализовать **кооперативное множественное наследование**, где каждый класс в цепочке может добавить
      свою функциональность, вызывая `super()`.
    - При правильном использовании `super()` во всех классах иерархии (A, B, C, D) метод будет вызван у каждого из них,
      что позволяет избежать "потери" вызова.

## **Senior Level**

В CPython 3.9+ **Diamond Problem** решается **C3-линеаризацией MRO** в `PyType_Ready()` (`Objects/typeobject.c`),
которая строит **единую последовательность** `(D, B, C, A, object)` без дублирования `A`. Поиск атрибутов идёт строго по
`tp_mro`, **первый** найденный метод выигрывает. `Objects/typeobject.c`,`Python/ceval.c`

## 1. Diamond иерархия и MRO (Python/compile.c → type_new)

```python
class A:  # tp_mro = (A, object)
    def m(self): print("A")


class B(A):  # tp_mro = (B, A, object)
    pass


class C(A):  # tp_mro = (C, A, object)
    pass


class D(B, C):  # tp_mro = (D, B, C, A, object) ← C3!
    pass
```

```c
// Objects/typeobject.c — type_new() при создании D
static PyObject *type_new(PyTypeObject *metatype, PyObject *args, PyObject *kwds) {
    PyObject *name, *bases, *dict;  // ("D", (B, C), {...})
    
    // ... парсинг args ...
    
    PyHeapTypeObject *et = (PyHeapTypeObject*)PyType_GenericAlloc(metatype, 0);
    PyTypeObject *type = &et->ht_type;  // новый тип D
    
    type->tp_name = PyUnicode_AsUTF8(name);    // "D"
    Py_INCREF(bases);                          // tp_bases = (B, C)
    type->tp_bases = bases;
    Py_INCREF(dict);
    type->tp_dict = dict;                      // методы D
    
    type->tp_base = best_base(bases);          // B (первый в tp_bases)
    
    if (PyType_Ready(type) < 0) {              // ← СЮДА C3-магия!
        Py_DECREF(et);
        return NULL;
    }
    return (PyObject*)type;
}
```

При `class D(B, C):` создаётся новый `PyTypeObject *D`, куда кладут `tp_bases=(B,C)`. Затем
`PyType_Ready(D)` **вычисляет** `tp_mro=(D,B,C,A,object)` по алгоритму C3 и сохраняет его. **НЕТ** дублирования `A`!

## 2. C3-линеаризация: mro_internal() и merge

```c
// Objects/typeobject.c — вычисление MRO для D
static int mro_internal(PyTypeObject *type) {
    PyObject *mro;  // результат: tuple(D, B, C, A, object)
    
    // если метакласс переопределил mro() — вызываем его
    PyObject *mro_meth = _PyType_Lookup(type, &_Py_ID(mro));
    if (mro_meth != NULL) {
        mro = PyObject_CallNoArgs(mro_meth);
        // проверяем, что это корректный tuple типов
        if (!PyTuple_Check(mro) || !mro_is_linearized(mro)) {
            PyErr_SetString(PyExc_TypeError, "bad mro");
            return -1;
        }
    } else {
        // стандартный C3
        mro = mro_implementation(type);
    }
    
    if (mro == NULL)
        return -1;
    
    Py_XSETREF(type->tp_mro, mro);     // D->tp_mro = (D,B,C,A,object)
    return 0;
}
```

`mro_internal(D)` строит **линейный список** классов для поиска методов. **C3 гарантирует**:
`D` первый, `B` перед `C` (порядок в `bases`), `A` **один раз** (не дублируется), `object` последний.

## 3. Алгоритм C3 merge (упрощённо в mro_implementation)

```c
// Псевдокод C3 для D(B,C) где B(A), C(A):
static PyObject *mro_implementation(PyTypeObject *type) {
    PyObject *result = PyList_New(0);  // [D]
    PyList_Append(result, (PyObject*)type);  // result = [D]
    
    // L1 = mro(B) = [B, A]
    // L2 = mro(C) = [C, A]  
    // L3 = bases(D) = [B, C]
    PyObject *L[3] = {mro_of_bases[0], mro_of_bases[1], bases_tuple};
    
    while (true) {
        PyObject *candidates = find_heads(L);  // головы списков: B, C
        
        if (PyList_GET_SIZE(candidates) == 0) {
            // цикл! TypeError
            PyErr_SetString(PyExc_TypeError, "Cannot create a consistent MRO");
            return NULL;
        }
        
        PyObject *candidate = choose_head(candidates);  // B (левый первый!)
        PyList_Append(result, candidate);               // result = [D, B]
        
        // удаляем B из всех L[] где он встречается
        remove_from_lists(L, candidate);
        
        if (all_lists_empty(L))
            break;
    }
    
    return PyList_AsTuple(result);  // (D, B, C, A, object)
}
```

C3 — это **алгоритм слияния списков**. Берём **головы** всех MRO родителей (`B` из `[B,A]`, `C`
из `[C,A]`). Выбираем **левую первую** (`B`), добавляем в результат, **удаляем** `B` из всех списков. Повторяем: головы
`C`, добавляем `C`, потом `A`.

## 4. Поиск метода по MRO: _PyType_Lookup

```c
// Objects/typeobject.c — поиск d.m() идёт СТРОГО по tp_mro!
PyObject *_PyType_Lookup(PyTypeObject *type, PyObject *name) {
    Py_ssize_t i, n;
    PyObject *mro = type->tp_mro;  // (D, B, C, A, object)
    n = PyTuple_GET_SIZE(mro);
    
    for (i = 0; i < n; i++) {                    // 0:D, 1:B, 2:C, 3:A
        PyTypeObject *base = PyTuple_GET_ITEM(mro, i);
        PyObject *dict = base->tp_dict;          // __dict__ класса
        
        PyObject *res = PyDict_GetItemWithError(dict, name);
        if (res != NULL) {
            Py_INCREF(res);
            return res;                          // ← ПЕРВЫЙ НАЙДЕННЫЙ!
        }
        if (PyErr_Occurred())
            return NULL;
    }
    return NULL;  // AttributeError
}
```

`d.m()` → `PyObject_GetAttr(d, 'm')` → `_PyType_Lookup(D, 'm')` → идём по `D->tp_mro`:
`D.tp_dict['m']`? нет → `B.tp_dict['m']`? нет → `C.tp_dict['m']`? нет → `A.tp_dict['m']`? **ДА** → возвращаем метод
`A.m`. **НЕ** доходим до второго `A` — его нет в MRO!

## 5. Байткод вызова: LOAD_ATTR по MRO

```python
d = D()
d.m()  # ищет m по MRO: D→B→C→A→object
```

```
# Байткод:
  0 LOAD_GLOBAL         0 (D)
  2 CALL                0
  4 STORE_FAST          0 (d)
  6 LOAD_FAST           0 (d)
  8 LOAD_ATTR           0 (m)     # ← _PyType_Lookup(D, 'm') → A.m!
 10 CALL                0
```

В `Python/ceval.c` (LOAD_ATTR):

```c
case LOAD_ATTR: {
    PyObject *name = GETITEM(names, oparg);    // PyUnicode "m"
    PyObject *owner = POP();                   // d (экземпляр D)
    
    PyObject *res = PyObject_GetAttr(owner, name);  // GenericGetAttr
    PUSH(res);
    DISPATCH();
}
```

`LOAD_ATTR 'm'` вызывает `PyObject_GetAttr(d, 'm')` → `instance_getattro(d, 'm')` →
`_PyType_Lookup(D, 'm')` по MRO → **находит в A** → bound method → `CALL`.

## 6. super() в Diamond: пропуск текущего класса

```python
class B(A):
    def m(self):
        super().m()  # ищет начиная ПОСЛЕ B в MRO!


class C(A):
    def m(self):
        super().m()
```

```c
// Objects/typeobject.c — super(B, self).m()
static PyObject *super_getattro(superobject *su, PyObject *name) {
    PyObject *mro = su->obj_type->tp_mro;      // (D,B,C,A,object)
    Py_ssize_t i = find_index(mro, su->type);  // индекс B = 1
    
    // начинаем поиск С i+1 = 2 (C!)
    for (i++; i < PyTuple_GET_SIZE(mro); i++) {
        PyTypeObject *cls = PyTuple_GET_ITEM(mro, i);
        PyObject *method = _PyType_Lookup(cls, name);
        if (method != NULL) {
            // descriptor_get(method, self, D)
            return method;
        }
    }
    PyErr_Format(PyExc_AttributeError, "no attr '%U'", name);
    return NULL;
}
```

`super(B,self).m()` в MRO `(D,B,C,A)` **стартует с C** (пропускает B и раньше). Поэтому
`B.m() → C.m() → A.m()`, **НЕ** `B.m() → A.m()` дважды!

## 7. Ошибка C3: когда MRO невозможен

```python
class X: pass


class Y: pass


class A(X): pass


class B(Y): pass


class C(A, B): pass  # OK: (C,A,X,B,Y,object)


class D(B, A): pass  # TypeError! C3 fail
```

```c
// В mro_implementation при цикле:
if (no_candidates()) {
    PyErr_SetString(PyExc_TypeError,
        "Cannot create a consistent method resolution order (MRO) "
        "for bases ...");
    return NULL;
}
```

Если порядок баз создаёт **цикл** (нельзя выбрать голову без нарушения монотонности),
`PyType_Ready(D)` выбрасывает `TypeError`. C3 **отказывается** строить некорректный MRO.

## 8. Проверка MRO корректности (type_mro_is_valid)

```c
// Objects/typeobject.c — после C3
static int mro_is_linearized(PyObject *mro) {
    Py_ssize_t i, n = PyTuple_GET_SIZE(mro);
    
    // 1. все элементы — типы
    // 2. type всегда ДО своих базовых
    for (i = 0; i < n; i++) {
        PyTypeObject *type = PyTuple_GET_ITEM(mro, i);
        Py_ssize_t j;
        for (j = 0; j < PyTuple_GET_SIZE(type->tp_bases); j++) {
            PyObject *base = PyTuple_GET_ITEM(type->tp_bases, j);
            if (index_of(mro, base) < i) {  // base раньше type? FAIL!
                return 0;
            }
        }
    }
    return 1;
}
```

После построения `(D,B,C,A)` проверяем: `B` после `D`? да. `C` после `D`? да. `A` после `B` и
`C`? да. **MRO валиден**.

## Итог Diamond в CPython

**Diamond Problem** **НЕ существует** в Python 3.9+ благодаря:

1. **C3-линеаризации** в `PyType_Ready()` → `tp_mro=(D,B,C,A,object)` **без дублирования**.
2. **Линейный поиск** `_PyType_Lookup()` по `tp_mro` → **первый** найденный атрибут.
3. **`super()`** пропускает текущий класс → вызывает **следующий** по MRO.
4. **TypeError** при невозможности C3 → отказ от неоднозначного наследования.

Всё вычисляется **один раз** при создании класса, хранится в `tp_mro`, используется **быстро** при каждом `LOAD_ATTR`.

- [Содержание](#содержание)

---

# **Магические методы**

## **Junior Level**

Магические методы в Python — это специальные методы, которые начинаются и заканчиваются двойным подчеркиванием (dunder),
например `__init__`, `__str__`, `__add__`. Они позволяют настраивать поведение объектов при выполнении стандартных
операций: создание, вывод на печать, сложение, сравнение и т.д.

Когда вы пишете `obj + other`, Python внутри вызывает `obj.__add__(other)`. Если в вашем классе определен этот метод, вы
можете задать, как именно будет работать сложение для ваших объектов. Это делает код интуитивно понятным и позволяет
вашим объектам вести себя как встроенные типы данных.

Самые распространенные магические методы:

- `__init__` — инициализатор объекта (конструктор)
- `__str__` — строковое представление для человека (`str(obj)`, `print(obj)`)
- `__repr__` — строковое представление для разработчика (показывается в консоли)
- `__len__` — поддержка функции `len(obj)`
- `__getitem__`, `__setitem__` — доступ по индексу или ключу: `obj[key]`

Использование магических методов делает ваш код более "питоничным" и понятным.

## **Middle Level**

Магические методы — это основа протоколов в Python, реализующих полиморфизм через утиную типизацию. Они делятся на
несколько категорий:

1. **Методы жизненного цикла**:
    - `__new__` — создание объекта (вызывается до `__init__`)
    - `__init__` — инициализация объекта
    - `__del__` — финализатор (вызывается перед удалением объекта)

2. **Строковые представления**:
    - `__str__` — для неформального представления (читабельного для человека)
    - `__repr__` — для формального представления (должно позволять воссоздать объект)
    - `__format__` — для поддержки `format(obj, spec)`
    - `__bytes__` — для `bytes(obj)`

3. **Протоколы сравнения**:
    - `__eq__`, `__ne__` — равенство/неравенство (`==`, `!=`)
    - `__lt__`, `__le__`, `__gt__`, `__ge__` — сравнения (`<`, `<=`, `>`, `>=`)
    - `__hash__` — вычисление хеша (обязателен для использования в множествах и как ключ словаря)

4. **Протоколы числовых операций**:
    - Арифметические: `__add__`, `__sub__`, `__mul__`, `__truediv__`
    - Унарные: `__neg__`, `__pos__`, `__abs__`
    - С преобразованием типов: `__int__`, `__float__`, `__bool__`

5. **Протоколы коллекций**:
    - `__len__` — длина
    - `__getitem__`, `__setitem__`, `__delitem__` — доступ по индексу/ключу
    - `__contains__` — поддержка оператора `in`
    - `__iter__`, `__next__` — итерация

6. **Протоколы вызова и контекста**:
    - `__call__` — вызов объекта как функции
    - `__enter__`, `__exit__` — контекстные менеджеры (оператор `with`)

7. **Дескрипторы и атрибуты**:
    - `__getattr__`, `__setattr__`, `__delattr__` — доступ к атрибутам
    - `__getattribute__` — перехват всех обращений к атрибутам
    - `__dir__` — список атрибутов для `dir(obj)`

8. **Сериализация**:
    - `__getstate__`, `__setstate__` — для pickle
    - `__reduce__`, `__reduce_ex__` — кастомная сериализация

## **Senior Level**

В CPython 3.9+ **магические методы** (`__init__`, `__str__`, `__add__` и т.д.) — это **обычные атрибуты** в `tp_dict`
класса, которые **находятся** по MRO через `_PyType_Lookup()` и **вызываются** через **дескрипторный протокол** (
`tp_descr_get`) или **слоты** `tp_*` (`tp_call`,
`tp_as_number->nb_add`). [Objects/typeobject.c][Objects/object.c][Objects/descrobject.c]

## 1. Хранение магических методов в tp_dict

```c
// Objects/typeobject.c — после компиляции класса
static int
PyType_Ready(PyTypeObject *type)
{
    // ... MRO, bases ...
    
    // tp_dict содержит ВСЕ атрибуты класса, включая __init__, __str__
    PyObject *dict = type->tp_dict;  // {"__init__": func, "__str__": func, ...}
    
    // Магические методы ищутся как _PyType_Lookup(type, "__str__")
    // → PyDict_GetItem(dict, "__str__") → PyCFunctionObject или PyFunctionObject
}
```

`__init__`, `__str__` — **не специальные** поля в `PyTypeObject`. Это **обычные ключи** в `type.__dict__` (`tp_dict`).
`PyObject_GetAttr(MyClass, '__str__')` → `_PyType_Lookup(MyClass, '__str__')` → `tp_dict['__str__']`.

## 2. Поиск магического метода: _PyType_Lookup + дескриптор

```c
// Objects/typeobject.c
PyObject *
_PyType_Lookup(PyTypeObject *type, PyObject *name)  // name = "__str__"
{
    PyObject *mro = type->tp_mro;  // (MyClass, object)
    Py_ssize_t i, n = PyTuple_GET_SIZE(mro);
    
    for (i = 0; i < n; i++) {
        PyTypeObject *cls = PyTuple_GET_ITEM(mro, i);
        PyObject *res = PyDict_GetItem(cls->tp_dict, name);  // cls.__dict__["__str__"]
        if (res != NULL) {
            Py_INCREF(res);
            return res;  // ← НАШЛИ __str__!
        }
    }
    return NULL;
}
```

```c
// Objects/object.c — obj.__str__()
PyObject *PyObject_Str(PyObject *v)
{
    PyObject *res = NULL;
    
    // 1. Ищем __str__ по MRO
    PyObject *str_meth = _PyType_Lookup(Py_TYPE(v), &_Py_ID(__str__));
    if (str_meth != NULL) {
        res = call_method(v, str_meth);  // obj.__str__()
        if (res != NULL)
            return res;
        PyErr_Clear();  // __str__ вернул ошибку → fallback
    }
    
    // 2. Fallback: __repr__
    PyObject *repr_meth = _PyType_Lookup(Py_TYPE(v), &_Py_ID(__repr__));
    if (repr_meth != NULL) {
        return call_method(v, repr_meth);
    }
    
    // 3. Fallback: tp_str слот типа
    reprfunc f = Py_TYPE(v)->tp_repr;
    if (f != NULL)
        return f(v);
    
    PyErr_Format(PyExc_TypeError, "'%.200s' object is not repr-able", ...);
    return NULL;
}
```

`str(obj)` → `PyObject_Str(obj)` → `_PyType_Lookup(type(obj), '__str__')` → находит `MyClass.__str__` в `tp_dict` →
вызывает как **обычный метод** `obj.__str__()`.

## 3. Дескрипторный протокол для магических методов

```c
// Objects/descrobject.c — метод как дескриптор
typedef struct {
    PyObject_HEAD
    PyFunctionObject *func;        // def __str__(self): ...
} PyMethodDescrObject;

static PyObject *
method_descr_get(PyMethodDescrObject *descr, PyObject *obj, PyObject *type)
{
    if (obj == NULL) {
        Py_INCREF(descr);
        return (PyObject *)descr;  // unbound method
    }
    
    // Создаём bound method: meth.__func__ = descr->func, meth.__self__ = obj
    return PyMethod_New(descr->func, obj, type);
}
```

`MyClass.__str__` — это **PyMethodDescrObject**. При `obj.__str__` срабатывает `tp_descr_get` → создаётся **bound method
** (с `self=obj`). При `MyClass.__str__` (без obj) — **unbound method**.

## 4. Байткод вызова магического метода

```python
class C:
    def __str__(self):
        return "C!"


c = C()
print(str(c))  # → c.__str__()
```

```
# Байткод str(c):
  0 LOAD_GLOBAL         0 (str)      # builtin str()
  2 LOAD_FAST           0 (c)
  4 CALL                1
  6 CALL                1 (print)

# Внутри str(c) → PyObject_Str(c) → LOAD_ATTR '__str__' → CALL
```

В `Python/ceval.c` (LOAD_ATTR для `__str__`):

```c
case LOAD_ATTR: {
    PyObject *name = GETITEM(names, oparg);  // "__str__"
    PyObject *owner = TOP();                 // c
    
    PyObject *res = PyObject_GetAttr(owner, name);  // c.__str__
    Py_DECREF(TOP());  // снимаем c
    SET_TOP(res);      // кладём bound method
    DISPATCH();
}
```

`str(c)` → `PyObject_Str(c)` → `LOAD_ATTR '__str__'` → `PyObject_GetAttr(c, '__str__')` → **bound method** →
`CALL_FUNCTION`.

## 5. Слоты tp_* для магических методов (ускорение)

```c
// Include/cpython/object.h
typedef struct _typeobject {
    reprfunc tp_repr;              // __repr__
    reprfunc tp_str;               // __str__
    objobjproc tp_richcompare;     // __eq__, __lt__
    
    /* Number: __add__, __mul__ */
    struct {
        binaryfunc nb_add;         // obj + other
        /* ... */
    } *tp_as_number;
    
    /* Sequence: __len__, __getitem__ */
    struct {
        lenfunc sq_length;         // len(obj)
        ssizeargfunc sq_item;      // obj[i]
    } *tp_as_sequence;
} PyTypeObject;
```

```c
// Objects/longobject.c — int.__add__
static PyObject *
long_add(PyLongObject *a, PyObject *bb)
{
    PyLongObject *b;
    // ... реализация сложения больших чисел ...
}

// PyLong_Type.tp_as_number->nb_add = long_add;
```

Для **встроенных типов** (`int`, `list`) магические методы **НЕ** в `tp_dict`, а в **C-слотах** (`tp_str`,
`tp_as_number->nb_add`). `1 + 2` → `PyNumber_Add(1, 2)` → `PyLong_Type.tp_as_number->nb_add` → **C-код**, **без поиска
по MRO**!

## 6. PyObject_Str: полный алгоритм с fallback'ами

```c
// Objects/abstract.c
PyObject *PyObject_Str(PyObject *v)
{
    PyObject *res = NULL;
    PyTypeObject *tp = Py_TYPE(v);
    
    // 1. Быстрый путь: tp_str слот (только для встроенных типов)
    if (tp->tp_str != NULL) {
        res = tp->tp_str(v);
        if (res != NULL)
            return res;
        PyErr_Clear();
    }
    
    // 2. Медленный путь: obj.__str__()
    PyObject *meth = _PyObject_LookupAttr(v, &_Py_ID(__str__));
    if (meth != NULL) {
        res = PyObject_CallOneArg(meth, v);
        Py_DECREF(meth);
        if (res != NULL)
            return res;
        PyErr_Clear();
    }
    
    // 3. Fallback: obj.__repr__()
    meth = _PyObject_LookupAttr(v, &_Py_ID(__repr__));
    if (meth != NULL) {
        res = PyObject_CallOneArg(meth, v);
        Py_DECREF(meth);
        return res ? res : NULL;
    }
    
    // 4. Финальный fallback: "<obj object at 0x...>"
    return PyUnicode_FromFormat("<%s object at %p>", tp->tp_name, v);
}
```

`str(obj)` пробует **4 пути по скорости**:

1. `type.tp_str` (C-функция, **самый быстрый**).
2. `obj.__str__()` (Python-метод).
3. `obj.__repr__()` (fallback).
4. Сырой `"<obj at 0x...>"`.

## 7. Арифметические магические методы через слоты

```c
// Objects/abstract.c — универсальное сложение
PyObject *PyNumber_Add(PyObject *v, PyObject *w)
{
    PyTypeObject *vt = Py_TYPE(v), *wt = Py_TYPE(w);
    
    // 1. v.__add__(w)
    if (vt->tp_as_number && vt->tp_as_number->nb_add) {
        PyObject *res = vt->tp_as_number->nb_add(v, w);
        if (res != Py_NotImplemented)
            return res;
    }
    
    // 2. w.__radd__(v)
    if (wt->tp_as_number && wt->tp_as_number->nb_add) {
        PyObject *res = wt->tp_as_number->nb_add(v, w);
        if (res != Py_NotImplemented)
            return res;
    }
    
    PyErr_Format(PyExc_TypeError, "unsupported operand type(s) for +: '%s' and '%s'",
                 vt->tp_name, wt->tp_name);
    return NULL;
}
```

`a + b` → `PyNumber_Add(a, b)` → **СНАЧАЛА** `type(a).tp_as_number.nb_add(a, b)` (т.е. `a.__add__(b)`), **ПОТОМ**
`type(b).tp_as_number.nb_add(a, b)` (`b.__radd__(a)`). **НЕ** ищет `__add__` в `tp_dict`!

## 8. Создание bound method из магического метода

```c
// Objects/classobject.c
PyObject *
PyMethod_New(PyFunctionObject *func, PyObject *self, PyObject *cls)
{
    PyMethodObject *meth = PyObject_GC_New(PyMethodObject, &PyMethod_Type);
    if (meth == NULL)
        return NULL;
    
    Py_INCREF(func);
    meth->im_func = func;      // __str__ функция
    
    Py_XINCREF(self);
    meth->im_self = self;      // экземпляр obj
    
    Py_XINCREF(cls);
    meth->im_class = cls;      // MyClass
    
    PyObject_GC_Track(meth);
    return (PyObject *)meth;
}
```

`c.__str__` → `PyMethod_New(__str__, c, C)` → **bound method** с тремя полями: `im_func` (код `__str__`), `im_self` (=
`c`), `im_class` (=`C`). При вызове: `im_func(im_self)`.

## 9. __init__ и tp_init слот

```c
// Objects/typeobject.c — type.__call__()
static PyObject *
type_call(PyTypeObject *type, PyObject *args, PyObject *kwds)
{
    // 1. tp_new: создаём "пустой" объект
    PyObject *obj = type->tp_new(type, args, kwds);
    if (obj == NULL)
        return NULL;
    
    // 2. tp_init: вызываем __init__(obj, *args, **kwds)
    if (type->tp_init != NULL) {
        int res = type->tp_init(obj, args, kwds);
        if (res < 0) {
            Py_DECREF(obj);
            return NULL;
        }
    }
    
    return obj;
}
```

`C()` → `type_call(C, (), {})` → `C.tp_new(C)` (создаёт объект) → `C.tp_init(obj)` → ищет `__init__` в `tp_dict` →
вызывает `obj.__init__()`.

## Итог работы магических методов в CPython

**Магические методы** работают через **3 механизма**:

1. **Слоты `tp_*`** (`tp_str`, `tp_as_number->nb_add`) — **быстрые C-функции** для встроенных типов.
2. **Поиск в `tp_dict`** по MRO (`_PyType_Lookup`) — Python-методы `__str__`, `__add__`.
3. **Дескрипторный протокол** — превращение в **bound method** с `self`.

Байткод `str(c)` → `LOAD_ATTR '__str__'` → **bound method** → `CALL`. **ВСЁ одинаково** для всех магических методов!

- [Содержание](#содержание)

---

# **Инвариантность и ковариантность**

## **Junior Level**

Инвариантность и ковариантность — это понятия из теории типов, которые описывают, как отношения между типами сохраняются
при использовании в обобщённых (generic) конструкциях.

Представьте, что у вас есть коробки для фруктов. У вас есть:

- Коробка для любых фруктов (`Box[Fruit]`)
- Коробка для яблок (`Box[Apple]`), где `Apple` — подтип `Fruit`

**Ковариантность** означает, что отношение "является подтипом" сохраняется и для коробок: если `Apple` — подтип `Fruit`,
то `Box[Apple]` — подтип `Box[Fruit]`. То есть коробку яблок можно использовать там, где ожидается коробка фруктов.

**Инвариантность** означает, что эти типы не связаны: `Box[Apple]` и `Box[Fruit]` — совершенно разные, независимые типы.
Коробку яблок нельзя использовать вместо коробки фруктов, и наоборот.

**Контравариантность** (обратное ковариантности) — когда отношение обращается: если `Apple` — подтип `Fruit`, то
`Box[Fruit]` — подтип `Box[Apple]`.

В Python эти концепции важны при работе с типизацией (type hints), особенно при использовании обобщённых типов вроде
`List[T]`, `Callable[[T], R]`.

## **Middle Level**

В контексте Python и статической типизации:

1. **Ковариантность (covariant)**:
    - Обозначается `+T` в определении типа
    - Если `A` — подтип `B`, то `Generic[A]` — подтип `Generic[B]`
    - Пример: `typing.Sequence` ковариантен по типу элемента

2. **Контравариантность (contravariant)**:
    - Обозначается `-T`
    - Если `A` — подтип `B`, то `Generic[B]` — подтип `Generic[A]` (обратное отношение)
    - Пример: `typing.Callable` контравариантен по типам аргументов

3. **Инвариантность (invariant)**:
    - Наиболее распространённый случай по умолчанию
    - Никаких отношений между `Generic[A]` и `Generic[B]`, даже если `A` и `B` связаны
    - Пример: `List[T]` инвариантен (по соображениям безопасности)

4. **Проблема изменяемости**:
    - Ковариантность безопасна для "read-only" типов (`Sequence`, `Iterable`)
    - Изменяемые типы (`List`, `Dict`) должны быть инвариантны для безопасности типов
    - Иначе можно было бы добавить апельсин в список яблок

5. **Практическое применение в Python**:
   ```python
   from typing import TypeVar, Generic, List, Sequence
   
   class Fruit: pass
   class Apple(Fruit): pass
   
   # Ковариантный тип
   T_co = TypeVar('T_co', covariant=True)
   class ReadOnlyBox(Generic[T_co]): pass
   
   # Контравариантный тип  
   T_contra = TypeVar('T_contra', contravariant=True)
   class Consumer(Generic[T_contra]): pass
   ```

6. **Проверка типов (mypy, pyright)**:
    - Статические анализаторы используют информацию о вариативности для проверки безопасности типов
    - Ошибки вариативности — частые причины ошибок типизации

## **Senior Level**

В CPython 3.9+ **инвариантность/ковариантность** реализованы в **generic alias** (`list[T]`) через
`PyGenericAliasObject` (`Objects/genericaliasobject.c`), где **variance** хранится в `ga_variance` поля TypeVar (
`covariant=True`). **Runtime** — только `__class_getitem__`, **статическая проверка** — в type checker'ах (
mypy). `Objects/genericaliasobject.c`,`Include/cpython/typeobject.h`

## 1. Generic alias: list[T] как PyGenericAliasObject

```c
// Objects/genericaliasobject.c — list[int] = PyGenericAliasObject
typedef struct {
    PyObject_HEAD
    PyObject *ga_origin;           // list (оригинальный тип)
    PyObject *ga_params;           // tuple(int,) параметры
    int ga_flags;                  // флаги
    Py_hash_t ga_hash;             // кэш хэша
    unsigned char ga_variance;     // ← VARIANCE! 0=invariant, 1=covariant, 2=contravariant
} PyGenericAliasObject;
```

`list[int]` — **НЕ** класс, а **специальный объект** `PyGenericAliasObject` с полями: `ga_origin=list`,
`ga_params=(int,)`, `ga_variance=1` (covariant). **Runtime** не проверяет типы — только хранит!

## 2. Создание generic alias: __class_getitem__

```c
// Objects/typeobject.c — List.__class_getitem__(int)
static PyObject *
type_subscript(PyTypeObject *type, PyObject *item)
{
    // type=list, item=int → list[int]
    if (PyTuple_Check(item)) {
        // list[int, str]
        return PyGenericAlias_New(type, item, NULL);
    }
    
    // list[int]
    PyObject *params = PyTuple_New(1);
    PyTuple_SET_ITEM(params, 0, Py_NewRef(item));
    return PyGenericAlias_New(type, params, NULL);
}

// Objects/genericaliasobject.c
PyObject *
PyGenericAlias_New(PyObject *origin, PyObject *params, PyObject *context)
{
    PyGenericAliasObject *ga = PyObject_GC_New(PyGenericAliasObject, &PyGenericAlias_Type);
    if (ga == NULL)
        return NULL;
    
    Py_INCREF(origin);
    ga->ga_origin = origin;        // list
    
    Py_INCREF(params);
    ga->ga_params = params;        // (int,)
    
    ga->ga_flags = 0;
    ga->ga_variance = 0;           // по умолчанию invariant
    
    PyObject_GC_Track(ga);
    return (PyObject *)ga;
}
```

`list[int]` → `list.__class_getitem__(int)` → `PyGenericAlias_New(list, (int,), NULL)` → **новый объект** с
`ga_origin=list`, `ga_params=(int,)`. **НЕ** создаёт новый класс!

## 3. TypeVar с variance (typing.TypeVar)

```c
// Lib/typing.py (упрощённо) → компилируется в C-объекты
class TypeVar:
    def __init__(self, name, *constraints, covariant=False, contravariant=False):
        self.__name__ = name
        self.__covariant__ = covariant      # True для covariant
        self.__contravariant__ = contravariant  # True для contravariant
        
# T_co = TypeVar('T_co', covariant=True)
# → T_co.__covariant__ = True
```

```python
from typing import TypeVar

T_co = TypeVar('T_co', covariant=True)  # variance=1
T_inv = TypeVar('T_inv')  # variance=0 (по умолчанию)
```

В CPython это **PyTypeVarObject** (внутреннее представление):

```c
typedef struct {
    PyObject_HEAD
    PyObject *tv_name;             // 'T_co'
    int tv_variance;               // 1=covariant, 0=invariant, -1=contravariant
    PyObject *tv_bound;            // ограничение типа
} PyTypeVarObject;
```

`TypeVar('T', covariant=True)` создаёт объект с флагом `tv_variance=1`. При подстановке в `list[T_co]` этот флаг
копируется в `ga_variance`.

## 4. Подстановка параметров: list[T_co][Animal] → list[Cat]

```c
// Objects/genericaliasobject.c — list[T_co][Animal]
PyObject *
generic_alias_subscript(PyGenericAliasObject *ga, PyObject *item)
{
    // ga = list[T_co], item = Animal
    PyObject *params = ga->ga_params;  // (T_co,)
    PyObject *tv = PyTuple_GET_ITEM(params, 0);  // T_co
    
    // подставляем T_co = Animal
    PyObject *sub = PyGenericAlias_Substitute(ga->ga_origin, params, item);
    return sub;  // list[Animal]
}
```

`list[T_co][Animal]` **НЕ** `list[Animal]` напрямую. Сначала `T_co` замещается на `Animal`, **с учётом variance**.
Runtime **не проверяет** `Cat <: Animal`.

## 5. Variance в isinstance() и issubclass() (ограниченная проверка)

```c
// Objects/abstract.c — isinstance(obj, list[int])
int
PyObject_IsInstance(PyObject *inst, PyObject *cls)
{
    // если cls — generic alias (list[int])
    if (PyGenericAlias_CheckExact(cls)) {
        PyGenericAliasObject *ga = (PyGenericAliasObject*)cls;
        PyObject *origin = ga->ga_origin;  // list
        
        // рекурсивно: isinstance(obj, list) И params совместимы
        int res = PyObject_IsInstance(inst, origin);
        if (res == 0)
            return 0;
            
        // ПРОВЕРКА VARIANCE ТОЛЬКО ДЛЯ BUILTIN!
        return generic_alias_isinstance(inst, ga);
    }
}
```

`isinstance(x, list[int])` работает **только для builtin** (`list`, `tuple`). Для пользовательских классов **игнорирует
** параметры. **НЕ** проверяет `Cat <: Animal`!

## 6. Проверка variance в generic_alias_isinstance()

```c
// Objects/genericaliasobject.c (упрощённо)
static int
generic_alias_isinstance(PyObject *inst, PyGenericAliasObject *ga)
{
    PyObject *origin = ga->ga_origin;      // list
    PyObject *inst_type = Py_TYPE(inst);   // type(x)
    
    // 1. x должен быть list
    if (!PyObject_IsInstance(inst, origin))
        return 0;
    
    // 2. для list[T_co] где T_co covariant:
    //    Cat <: Animal → list[Cat] OK для list[Animal]
    PyObject *params = ga->ga_params;      // (Animal,)
    unsigned char variance = ga->ga_variance;  // 1=covariant
    
    if (variance == 1) {  // covariant
        // проверим подтипность параметров
        return check_covariant_params(inst_type, params);
    }
    
    // invariant: точное совпадение
    return check_exact_params(inst_type, params);
}
```

**Runtime проверка** работает **только для covariant builtin** типа `list[int]`. `list[Cat](cat)` **проходит**
`isinstance(cat, list[Animal])` **если** `Cat <: Animal`. **НЕ работает** для пользовательских классов!

## 7. __class_getitem__ для пользовательских generic (PEP 585, 3.9+)

```c
// Objects/typeobject.c — MyGeneric[int]
static PyObject *
type_subscript_user_generic(PyTypeObject *type, PyObject *item)
{
    // пользовательский класс с Generic[T]
    if (has_generic_base(type)) {
        // возвращаем GenericAlias с origin=type
        return PyGenericAlias_New((PyObject*)type, PyTuple_New(1, item), NULL);
    }
}
```

```python
from typing import Generic, TypeVar

T = TypeVar('T', covariant=True)


class Box(Generic[T]):  # Box[int]
    pass
```

Пользовательский `Box[int]` тоже `PyGenericAliasObject` с `ga_origin=Box`. **Но** `isinstance(x, Box[int])` **НЕ
РАБОТАЕТ** — только статическая проверка в mypy!

## 8. Байткод работы с generic alias

```python
from typing import List

x: List[int] = [1, 2, 3]
```

```
# Байткод (аннотации):
  0 LOAD_GLOBAL          0 (List)
  2 LOAD_GLOBAL          1 (int)
  4 CALL_FUNCTION        1          # List[int] ← __class_getitem__
  6 LOAD_CONST           0 ([])     # дефолт []
  8 STORE_FAST           0 (x)
```

`List[int]` в байткоде — **обычный вызов** `List.__class_getitem__(int)` → `PyGenericAliasObject`. **Runtime** не
проверяет типы в списке!

## 9. Variance в type checker (mypy, НЕ CPython!)

```python
T_co = TypeVar('T_co', covariant=True)


class Box(Generic[T_co]): pass


def accept_box(box: Box[Animal]): ...


b: Box[Cat] = Box()  # Cat <: Animal
accept_box(b)  # OK! covariant
```

**Mypy логика (псевдокод):**

```
if Box.ga_variance == COVARIANT and issubclass(Cat, Animal):
    OK: Box[Cat] <: Box[Animal]
else:
    Error!
```

**CPython runtime** хранит `ga_variance`, **НО** проверку **НЕ делает**. `isinstance()` работает только для **builtin
covariant** коллекций. Полная проверка **только в type checker** (mypy, pyright).

## Итог инвариантности/ковариантности в CPython 3.9+

1. **Хранение**: `PyGenericAliasObject.ga_variance` из `TypeVar(covariant=True)`.
2. **Создание**: `list[int]` → `__class_getitem__` → `PyGenericAlias_New()`.
3. **Runtime**: `isinstance()` **только для builtin** (`list`, `tuple`), **игнорирует** пользовательские классы.
4. **Type checking**: **mypy** анализирует `ga_variance` + подтипность параметров **статически**.

**НЕ путай**: variance — это **информация для статических анализаторов**, **НЕ runtime проверка типов**!

- [Содержание](#содержание)

---

# **ABC**

## **Junior Level*

Представьте, что вы архитектор, который разрабатывает проект дома (интерфейс, контракт). Вы создаете подробный чертеж,
где указано: "здесь должна быть дверь", "здесь должно быть не менее двух окон", "обязательно должна быть крыша". Однако
сам чертеж — это не дом, по нему нельзя жить. Вы просто задаете стандарт, которому должен следовать любой построенный по
этому проекту дом.

В Python ABC (Abstract Base Classes) — это и есть такие "чертежи" для классов. Это специальный механизм, который
позволяет нам декларативно сказать: "Любой класс, который будет моим наследником, ОБЯЗАН реализовать конкретные методы (
например, `save()`, `load()`, `validate()`)". Если наследник не реализует все обязательные методы, Python не даст
создать его экземпляр и сразу укажет на ошибку. Это делает код предсказуемым, помогает избежать ошибок в рантайме и
четко документирует ожидания от объекта. Для QA инженера это мощный инструмент для стандартизации тестовых утилит,
плагинов и проверки контрактов в системе.

## **Middle Level**

На техническом уровне ABC — это классы, наследующиеся от встроенного класса `abc.ABC` и использующие декоратор
`@abstractmethod` (а также `@abstractclassmethod`, `@abstractstaticmethod`, `@abstractproperty`) для пометки абстрактных
методов. Ключевые аспекты:

1. **Инстанцирование:** Попытка создать экземпляр класса, у которого есть хотя бы один не переопределенный
   `@abstractmethod`, немедленно вызовет исключение `TypeError`. Это проверка происходит в момент вызова `__new__`
   класса, еще до вызова `__init__`.

2. **Регистрация (Registration):** Помимо явного наследования, класс может быть "зарегистрирован" как виртуальный
   подкласс ABC с помощью метода `register()`. После этого `isinstance()` и `issubclass()` будут возвращать для него
   `True`, но при этом **не проводится проверка на наличие абстрактных методов!** Это "мягкий" способ заявить о
   соответствии интерфейсу, полезный для интеграции сторонних или унаследованных классов.

3. **`__subclasshook__`:** Это специальный метод класса (`@classmethod`), который позволяет кастомизировать логику
   проверки `issubclass()`. Метод может проверять наличие у класса требуемых атрибутов или методов (через `hasattr`), а
   не только факт прямого наследования или регистрации. Это самый гибкий и "питонический" способ определения виртуальных
   подклассов.

4. **Встроенные ABCs в `collections.abc`:** Модуль предоставляет богатейший набор готовых абстрактных классов для
   стандартных протоколов: `Iterable`, `Iterator`, `Container`, `Sequence`, `Mapping`, `Callable` и т.д. Использование
   `isinstance(obj, collections.abc.Sequence)` вместо `isinstance(obj, list)` делает код полиморфным и независимым от
   конкретной реализации.

## **Senior Level**

В CPython 3.9+ **ABC** (Abstract Base Classes) реализованы через **C-модуль `_abc`** (`Modules/_abc.c`), **регистрацию**
в `_abc_registry` (weakref dict), **кэш** `_abc_cache`/`_abc_negative_cache`, перехват `isinstance()`/`issubclass()`
через `__instancecheck__`/`__subclasscheck__` на `ABCMeta`. Абстрактные методы — флаг `__abstractmethods__` в
`tp_dict`. `Modules/_abc.c`,`Objects/abstract.c`

## 1. Структура _abc_data в C-модуле

```c
// Modules/_abc.c — глобальное состояние ABC на модуль
typedef struct {
    PyObject *_abc_registry;       // dict: ABC → set(слабые ссылки на подклассы)
    PyObject *_abc_cache;          // dict: (subclass, ABC) → True/False
    PyObject *_abc_negative_cache; // dict: (subclass, ABC) → False (не подкласс)
    uint64_t _abc_negative_cache_version; // версия кэша (меняется при register)
} _abc_data;
```

Каждый ABC-модуль имеет **3 словаря**:

- `_abc_registry` — кто кого зарегистрировал (`MyABC → {tuple, str}`).
- `_abc_cache` — положительный кэш проверок.
- `_abc_negative_cache` — отрицательный кэш.  
  **Кэш ускоряет** `isinstance()` в 1000x раз!

## 2. Регистрация подкласса: _abc_register_impl()

```c
// Modules/_abc.c — MyABC.register(tuple)
static PyObject *
_abc__abc_register_impl(PyObject *module, PyObject *self, PyObject *subclass)
{
    _abc_data *impl = _get_impl(module, self);  // получаем данные ABC
    if (impl == NULL)
        return NULL;
    
    PyObject *registry;  // set слабых ссылок
    Py_BEGIN_CRITICAL_SECTION(impl);  // атомарно
    registry = impl->_abc_registry;
    Py_END_CRITICAL_SECTION();
    
    if (registry == NULL) {
        registry = PySet_New(NULL);  // новый set
        Py_BEGIN_CRITICAL_SECTION(impl);
        Py_XSETREF(impl->_abc_registry, registry);
        Py_END_CRITICAL_SECTION();
    }
    
    // добавляем weakref(subclass) в registry
    PyObject *ref = PyWeakref_NewRef(subclass, NULL);
    PySet_Add(registry, ref);
    Py_DECREF(ref);
    
    // инвалидируем кэш
    impl->_abc_negative_cache_version++;
    
    Py_RETURN_NONE;
}
```

`MyABC.register(tuple)` → `weakref(tuple)` **добавляется** в `MyABC._abc_registry`. **НЕ** меняет `tuple.__bases__`!
`issubclass(tuple, MyABC)` смотрит **только** в registry. **Кэш сбрасывается** (`_abc_negative_cache_version++`).

## 3. Проверка issubclass: _abc_abc_subclasscheck()

```c
// Modules/_abc.c — issubclass(MyClass, MyABC)
static int
_abc_abc_subclasscheck_impl(PyObject *self, PyObject *subclass)
{
    _abc_data *impl = _get_impl(NULL, self);
    if (impl == NULL)
        return -1;
    
    // 1. Проверяем кэш
    int cached = check_cache(impl, subclass, self);
    if (cached != -1)
        return cached;
    
    // 2. Обычное наследование (MRO)
    if (PyObject_IsSubclass(subclass, (PyObject *)Py_TYPE(self)))
        return 1;
    
    // 3. Регистрация в _abc_registry
    PyObject *registry = impl->_abc_registry;
    if (registry != NULL && PySet_Contains(registry, subclass))
        return 1;
    
    // 4. __subclasshook__
    PyObject *hook = lookup_method(self, &_Py_ID(__subclasshook__));
    if (hook != NULL) {
        PyObject *res = PyObject_CallFunctionObjArgs(hook, subclass, NULL);
        if (res != NULL) {
            int result = PyObject_IsTrue(res);
            Py_DECREF(res);
            return result;
        }
    }
    
    // 5. Кэшируем False
    cache_negative_result(impl, subclass, self);
    return 0;
}
```

`issubclass(C, ABC)` проверяет **4 пути**:

1. **Кэш** (быстро).
2. **Обычное наследование** `C.__mro__` содержит `ABC`.
3. **Регистрация** `ABC._abc_registry` содержит `weakref(C)`.
4. **`ABC.__subclasshook__(C)`** (custom логика).  
   **НЕ реализует** — возвращает `False`.

## 4. Перехват isinstance/issubclass в PyObject_IsInstance()

```c
// Objects/abstract.c — isinstance(obj, ABC)
int PyObject_IsInstance(PyObject *inst, PyObject *cls)
{
    PyTypeObject *tp = Py_TYPE(inst);
    
    // 1. Обычная проверка по MRO
    if (PyTuple_Contains(tp->tp_mro, cls))
        return 1;
    
    // 2. ABC перехват через __instancecheck__
    PyObject *inst_cls = (PyObject *)Py_TYPE(cls);
    if (Py_TYPE(inst_cls) == &ABCMeta_Type) {  // это ABC!
        PyObject *meth = _PyType_Lookup(Py_TYPE(inst_cls), &_Py_ID(__instancecheck__));
        if (meth != NULL) {
            PyObject *res = PyObject_CallFunctionObjArgs(meth, cls, inst, NULL);
            int result = PyObject_IsTrue(res);
            Py_XDECREF(res);
            return result;
        }
    }
    
    return 0;
}
```

`isinstance(obj, ABC)` **СНАЧАЛА** проверяет `type(obj).__mro__`, **ПОТОМ** вызывает `ABC.__instancecheck__(obj)`. *
*ABCMeta** перехватывает и делегирует в `_abc_abc_instancecheck()`.

## 5. __abstractmethods__ и проверка абстрактности

```c
// Lib/abc.py (компиляция → tp_dict)
class MyABC(ABC):
    @abstractmethod
    def method(self):
        pass
```

В `PyType_Ready()` (Objects/typeobject.c):

```c
static int
PyType_Ready(PyTypeObject *type)
{
    // ... MRO ...
    
    // проверяем __abstractmethods__ из tp_dict
    PyObject *abstracts = PyDict_GetItem(type->tp_dict, &_Py_ID(__abstractmethods__));
    if (abstracts != NULL && PySet_GET_SIZE(abstracts) > 0) {
        // есть нереализованные абстрактные методы!
        type->tp_flags |= Py_TPFLAGS_IS_ABSTRACT;
    }
}
```

`@abstractmethod` добавляет имя метода в `MyABC.__abstractmethods__ = {'method'}`. При создании `MyABC` если set **НЕ
пустой** — класс помечается `Py_TPFLAGS_IS_ABSTRACT`. **НЕ** блокирует создание экземпляра!

## 6. Блокировка инстанцирования абстрактного класса

```c
// Objects/typeobject.c — type_call() для абстрактного класса
static PyObject *
type_call(PyTypeObject *type, PyObject *args, PyObject *kwds)
{
    // ПРОВЕРКА АБСТРАКТНОСТИ!
    if (type->tp_flags & Py_TPFLAGS_IS_ABSTRACT) {
        PyObject *abstracts = PyDict_GetItem(type->tp_dict, &_Py_ID(__abstractmethods__));
        if (abstracts != NULL && PySet_GET_SIZE(abstracts) > 0) {
            // выводим НЕ реализованные методы
            PyErr_Format(PyExc_TypeError,
                "Can't instantiate abstract class %s with abstract methods %R",
                type->tp_name, abstracts);
            return NULL;
        }
    }
    
    // обычное создание: tp_new → tp_init
    return type->tp_new(type, args, kwds);
}
```

`MyABC()` → `type_call(MyABC)` → видит `Py_TPFLAGS_IS_ABSTRACT` + `__abstractmethods__ != set()` → **TypeError** со
списком нереализованных методов!

## 7. __subclasshook__ пример (collections.abc.Container)

```python
# Lib/_collections_abc.py
class Container(ABC):
    @classmethod
    def __subclasshook__(cls, C):
        # любой класс с __contains__ — Container!
        if cls is Container:
            return hasattr(C, '__contains__')
        return NotImplemented
```

В C (`_abc_abc_subclasshook__`):

```c
PyObject *hook_res = PyObject_CallFunctionObjArgs(hook, subclass, NULL);
int result = PyObject_IsTrue(hook_res);  // True/False/NotImplemented
```

`Container.__subclasshook__(MyClass)` → `hasattr(MyClass, '__contains__')` → `True`. `MyClass` **НЕ наследует**
`Container`, **НО** `issubclass(MyClass, Container) == True`!

## 8. Байткод регистрации ABC

```python
from abc import ABC


class MyABC(ABC): pass


MyABC.register(tuple)
print(issubclass(tuple, MyABC))  # True
```

```
# Байткод MyABC.register(tuple):
  0 LOAD_GLOBAL         0 (MyABC)
  2 LOAD_GLOBAL         1 (tuple)
  4 CALL_METHOD         1          # register(tuple)
  6 POP_TOP
  8 LOAD_GLOBAL         0 (issubclass)
 10 LOAD_GLOBAL         1 (tuple)
 12 LOAD_GLOBAL         0 (MyABC)
 14 CALL_FUNCTION       2
```

`MyABC.register(tuple)` → `CALL_METHOD` → C `_abc__abc_register_impl()` → `weakref(tuple)` в `_abc_registry`.

## Итог реализации ABC в CPython 3.9+

**ABC** — **виртуальное наследование** через:

1. **C-модуль `_abc`** с `_abc_data` (registry + 2 кэша).
2. **`register()`** → weakref в `_abc_registry`.
3. **`issubclass(C, ABC)`** → кэш → MRO → registry → `__subclasshook__`.
4. **`isinstance(obj, ABC)`** → `ABC.__instancecheck__(obj)`.
5. **`@abstractmethod`** → `__abstractmethods__` set → `Py_TPFLAGS_IS_ABSTRACT`.
6. **Инстанцирование** → проверка флага + set → `TypeError`.

**НЕ меняет MRO**, **НЕ добавляет методы**, **только** перехватывает `isinstance`/`issubclass`!

- [Содержание](#содержание)

---

# **Протокол (Protocol)**

## **Junior Level*

Представьте, что вы описываете не конкретный предмет, а его роль. Например, "то, что можно включить". Под это описание
подходит и лампа, и компьютер, и телевизор — у всех есть кнопка "включить". Вам неважно, что это за устройство внутри;
важно, что оно поддерживает операцию "включения".

В Python **Протокол (Protocol)** — это именно такое формальное описание роли или поведения. Он говорит: "Если у объекта
есть вот такие методы и атрибуты, то он автоматически считается подходящим для определенной цели, даже если он не был
изначально для этого предназначен". Это продвинутая, "официальная" версия принципа утиной типизации ("если ходит как
утка и крякает как утка, то это утка"). В отличие от Abstract Base Class (ABC), где класс должен явно заявить "я
наследуюсь от этого чертежа", протокол работает на уровне "если ты имеешь эти черты, то ты соответствуешь".

## **Middle Level**

Технически `Protocol` — это конструкция системы типизации, определенная в PEP 544 и доступная в модуле `typing`. Его
ключевые аспекты:

1. **Структурная, а не номинативная типизация:** Класс соответствует протоколу, если его **структура** (набор методов и
   атрибутов с правильными сигнатурами) совпадает с протоколом. Ему не нужно явно наследоваться от протокола. Это
   фундаментальное отличие от ABC, где требуется явное номинативное объявление (наследование или регистрация).

2. **Синтаксис и `@runtime_checkable`:** Протокол определяется как класс, наследующийся от `typing.Protocol`. Его методы
   часто помечаются как абстрактные, используя `...` (ellipsis) в теле. По умолчанию протоколы используются **только
   статическими анализаторами типов** (mypy, pyright). Чтобы позволить проверку соответствия протоколу во время
   выполнения (`isinstance(obj, MyProtocol)`), протокол необходимо декорировать `@typing.runtime_checkable`. Однако
   такая проверка ограничена: она проверяет только **наличие** указанных атрибутов, но не их сигнатуры или типы.

3. **Generic и вариативность:** Протоколы могут быть параметризованы (Generic), что позволяет описывать типы, зависящие
   от других типов (например, `Iterable[T]`). Они также поддерживают ковариантность и контравариантность через параметры
   `covariant=True`/`contravariant=True`, что критично для точного описания отношений между сложными типами.

4. **Встроенные протоколы:** Модуль `typing` и `collections.abc` предоставляют множество встроенных протоколов:
   `SupportsInt`, `SupportsBytes`, `ContextManager`, `Iterable`, `Sized` и др. Их использование в аннотациях делает код
   гораздо более выразительным и безопасным.

## **Senior Level**

`Protocol` в CPython — это чистый Python‑код в `Lib/typing.py`: специальный базовый класс c метаклассом
`typing._ProtocolMeta`, который помечает класс как протокольный (структурный) и запоминает множество его членов для
рантайм‑проверок `isinstance`/`issubclass` при включённом `@runtime_checkable`. Никакой спецподдержки в байткоде или
интерпретаторе нет, всё на уровне обычных классов/метаклассов.

## Базовая реализация Protocol в typing.py

В `Lib/typing.py` `Protocol` определён примерно так (упрощённо):

- Класс `Protocol` наследует от `Generic` и использует `_ProtocolMeta` как метакласс:

  ```python
  class Protocol(metaclass=_ProtocolMeta):
      pass
  ```

- `_ProtocolMeta` сам наследует от `abc.ABCMeta` (или `type` в старых версиях), переопределяя `__instancecheck__`/
  `__subclasscheck__` и `__new__`.
- При создании класса‑протокола `class P(Protocol): ...` управление идёт в `_ProtocolMeta.__new__`.

`_ProtocolMeta.__new__(mcls, name, bases, namespace, **kwargs)` делает:

1. Создаёт класс через базовый метакласс (`abc.ABCMeta.__new__`/`type.__new__`).
2. Помечает класс как протокол, устанавливая флаги:

    - `cls._is_protocol = True`;
    - `cls.__protocol_attrs__ = frozenset(<имена членов>)`.

3. Список протокольных членов вычисляется сбором всех атрибутов, объявленных в теле класса и его базовых протоколах,
   исключая служебные (`__mro__`, `__dict__` и т.п.).

Это всё — обычный Python‑метакласс: `class` компилируется как вызов `ProtocolMeta.__new__`/`__init__`, как и для любого
метакласса.

## Сбор членов протокола

Внутри `_ProtocolMeta.__new__` используется helper вроде `_get_protocol_attrs(cls)`:

- Берётся `cls.__dict__` и `__mro__` (кроме `Protocol` и `Generic`).
- Отбираются имена, у которых:
    - нет префикса `'__'` или явно разрешены (`__call__` и т.п.);
    - значение не является `typing.ClassVar`/`Final` и не помечено как приватное.
- Результат — `frozenset({'method1', 'attr2', ...})` — это интерфейс протокола.

Это множество не используется интерпретатором, а нужно только для `@runtime_checkable` и утилит
`typing_extensions.get_protocol_members`.

## runtime_checkable и __instancecheck__/__subclasscheck__

По умолчанию `Protocol` **не** должен использоваться с `isinstance`/`issubclass` — это только для статического анализа.
Декоратор `@typing.runtime_checkable` меняет поведение:

- `runtime_checkable(proto_cls)` оборачивает класс, проверяет, что это протокол (`_is_protocol=True`), и ставит ему флаг
  `_is_runtime_protocol = True`.

`_ProtocolMeta.__instancecheck__(cls, obj)` реализует:

1. Если `not getattr(cls, "_is_runtime_protocol", False)` — вызывает обычный `abc.ABCMeta.__instancecheck__` (по
   наследованию) → `False`/`TypeError`.
2. Иначе проверяет объект структурно:

    - Для каждого имени в `cls.__protocol_attrs__`:
        - через `getattr(obj, name, _marker)` проверяет наличие;
        - для методов/функций — только наличие, без точной проверки сигнатур на рантайме.

3. Если все имена есть → `True`, иначе `False`.

`_ProtocolMeta.__subclasscheck__(cls, sub)` для runtime‑протоколов делает аналогичный анализ по `dir(sub)`/MRO, но опять
же полностью в `typing.py`, без участия VM.

Сам `isinstance(x, P)` в VM вызывает `PyObject_IsInstance` → `cls.__instancecheck__`, т.е. всё поведение под контролем
`_ProtocolMeta`.

## Generic Protocol и параметры типов

`Protocol` часто комбинируется с `Generic`:

```python
class P(Protocol[T]):
    def f(self, x: T) -> T: ...
```

На уровне реализации:

- `Protocol` наследует от `Generic`, а `_ProtocolMeta` наследует от `typing._GenericAlias`‑совместимого метакласса,
  поэтому `P[int]` создаёт `typing._GenericAlias(P, (int,))`.
- При `P[int]` вызывается `Protocol.__class_getitem__`, унаследованный от `Generic`, который создаёт alias‑объект и не
  меняет поведение рантайм‑проверок; параметры типов хранятся в `__args__`/`__origin__` alias’а.

Для рантайма это всего лишь ещё один generic alias, без логики в байткоде.

## Ограничения на конструктор и наследование

`_ProtocolMeta.__call__` *не* запрещает инстанцирование протоколов (в отличие от `ABCMeta`) — протоколы используются как
обычные классы, но для статического анализа важен только их интерфейс.

Однако `typing` накладывает ограничения при наследовании/параметризации:

- В `Protocol.__mro__` не допускаются конфликтующие базы (миксы с обычными классами, не основанными на Protocol) —
  `typing` проверяет это в `__init_subclass__`/`_check_protocol`.
- При субскрипции (`Protocol[...]`) `_ProtocolMeta.__getitem__` проверяет, что все параметры — `TypeVar` (для объявления
  generic‑протокола), а не конкретные типы.

Эти проверки — обычные Python‑вычисления в `typing.py`, без участия VM.

## typing_extensions.Protocol

В старых версиях CPython большинство логики протоколов жило в `typing_extensions.Protocol`, а позже мигрировало в stdlib
`typing.Protocol`.

- `typing_extensions.Protocol` имеет свой metaclass `_ProtocolMeta` с почти идентичным кодом (иногда даже бэкпортит
  патчи из будущих версий CPython, например поведение `__init__` в 3.11).
- Реальный рантайм‑поведение зависит от того, какой именно класс стоит в MRO (stdlib vs extensions); при смешивании
  обоих в MRO возможны несостыковки, но это всё решается в чистом Python‑коде, без изменений интерпретатора.

***

Итого, `Protocol` в CPython — это обычные Python‑классы с метаклассом `_ProtocolMeta`, который при создании класса
собирает множество имён членов и, при `@runtime_checkable`, реализует структурные `isinstance`/`issubclass` через
переопределение `__instancecheck__`/`__subclasscheck__`. Никакого спецбайткода, дополнительных флагов в `PyTypeObject`
или логики в `ceval.c` под протоколы нет.

- [Содержание](#содержание)

---

# **Паттерны проектирования**

## **Junior Level*

Паттерны проектирования — это проверенные временем решения типовых проблем в разработке программного обеспечения. Можно
сравнить их с архитектурными чертежами для строительства: вместо того чтобы каждый раз изобретать велосипед, опытные
архитекторы используют готовые схемы организации кода, которые уже доказали свою эффективность для конкретных ситуаций.

Это не готовые куски кода, а скорее концепции или шаблоны мышления о том, как структурировать взаимодействие между
компонентами системы. Например, "Фабрика" — это паттерн для создания объектов, не привязываясь к их конкретным классам,
а "Наблюдатель" описывает, как один объект может уведомлять множество других о произошедших изменениях.

Для AQA инженера понимание паттернов критически важно по нескольким причинам: во-первых, чтобы понимать архитектуру
тестируемого приложения и находить в ней слабые места; во-вторых, чтобы проектировать собственные тестовые фреймворки,
которые будут гибкими, расширяемыми и поддерживаемыми; в-третьих, чтобы говорить с разработчиками на одном языке при
обсуждении дизайна и потенциальных проблем.

## **Middle Level**

С технической точки зрения, паттерны проектирования (особенно из классического каталога GoF — "Банды четырех") в Python
часто реализуются с учетом уникальных особенностей языка.

1. **Идиоматичность Python:** Многие классические паттерны в Python могут быть реализованы проще и элегантнее, чем в
   статически типизированных языках. Например:
    * **Стратегия (Strategy):** Вместо создания иерархии классов можно просто передавать callable-объекты (функции,
      лямбды, экземпляры с `__call__`).
    * **Декоратор (Decorator):** Прямо соответствует синтаксическому декоратору Python, который является "обёрткой"
      вокруг функции или метода, модифицирующей его поведение.
    * **Адаптер (Adapter):** Часто реализуется не через наследование, а через композицию и магический метод
      `__getattr__` для перенаправления вызовов.

2. **Категории паттернов:**
    * **Порождающие (Creational):** Решают задачу гибкого и контролируемого создания объектов (Синглтон, Фабрика,
      Строитель). В Python Синглтон часто реализуют через метакласс или модуль (сам модуль по своей природе — синглтон).
    * **Структурные (Structural):** Организуют композицию классов и объектов для образования более крупных структур (
      Адаптер, Мост, Компоновщик, Декоратор, Фасад).
    * **Поведенческие (Behavioral):** Определяют эффективные способы взаимодействия и распределения ответственности
      между объектами (Наблюдатель, Стратегия, Команда, Цепочка обязанностей, Состояние).

3. **Для AQA:**
    * **Page Object** — это специализированный структурный паттерн для UI-тестирования, по сути являющийся Фасадом,
      скрывающим детали HTML-структуры за удобным API.
    * **Фабрика** и **Строитель (Builder)** незаменимы для создания сложных тестовых данных и фикстур.
    * **Наблюдатель (Observer)** или **Издатель-Подписчик (Pub/Sub)** лежит в основе систем событийного логирования или
      сбора результатов тестов в реальном времени.
    * **Прокси (Proxy)** и **Заместитель (Mock/Stub)** — основа библиотек для мокирования (unittest.mock), позволяющих
      подменять реальные зависимости в тестах.

4. **Dependency Injection (Внедрение зависимостей):** Хотя формально не входит в каталог GoF, это ключевой архитектурный
   паттерн, который в Python часто реализуется просто через передачу зависимостей в конструктор (Constructor Injection),
   без использования тяжеловесных фреймворков. Это краеугольный камень тестируемого дизайна.


- [Содержание](#содержание)

---

# **Связность и связанность**

## **Junior Level*

Связность и связанность — это две фундаментальные концепции качества кода, которые описывают, насколько хорошо
организованы компоненты внутри модуля и насколько сильно они зависят друг от друга.

**Связность (Cohesion)** отвечает на вопрос: "Насколько хорошо элементы внутри одного модуля (класса, функции) связаны
между собой и работают для достижения одной четкой цели?". Высокая связность — это хорошо. Это означает, что модуль
делает что-то одно, целостное и понятное. Например, модуль `MathUtils`, который содержит только функции для
математических вычислений, обладает высокой связностью. А модуль `Utils`, который смешивает функции для работы со
строками, отправки email и логирования, имеет низкую связность — это "мусорная корзина", которую трудно поддерживать.

**Связанность (Coupling)** отвечает на вопрос: "Насколько сильно один модуль зависит от внутреннего устройства другого
модуля?". Низкая связанность — это хорошо. Это означает, что модули взаимодействуют через четкие, простые интерфейсы и
могут изменяться независимо друг от друга. Если же модуль "залезает" в приватные детали другого модуля, то любое
изменение в одном сломает другой — это высокая связанность (сильная связь).

Простой принцип: нужно стремиться к **высокой связности и низкой связанности**. Для QA инженера это напрямую влияет на
тестируемость: модули с высокой связностью легко тестировать изолированно, а при низкой связанности можно легко заменять
зависимости на моки.

## **Middle Level**

С технической точки зрения эти концепции реализуются через конкретные практики проектирования и имеют четкие индикаторы.

1. **Типы связности (от худшего к лучшему):**
    * **Случайная (Coincidental):** Элементы собраны вместе случайно (например, модуль `MiscHelpers`). Тестировать такое
      невозможно — нет единой ответственности.
    * **Логическая (Logical):** Элементы сгруппированы по категории (например, класс `DataProcessor`, который
      обрабатывает и CSV, и JSON, и XML). Тесты становятся размазанными и хрупкими.
    * **Временная (Temporal):** Элементы выполняются в одно время (например, функция `initialize_all()`, которая
      настраивает логгер, БД и кеш). Приводит к сложным фикстурам и непредсказуемым побочным эффектам в тестах.
    * **Процедурная (Procedural):** Элементы объединены последовательностью шагов (например, функция, которая читает
      файл, парсит данные и сохраняет в БД). В тестах приходится эмулировать всю последовательность.
    * **Коммуникационная (Communicational):** Элементы работают с одними и теми же данными (например, класс
      `CustomerReport`, который и вычисляет статистику, и форматирует отчет). Уже лучше, но еще есть смесь
      ответственностей.
    * **Последовательная (Sequential):** Выход одного элемента является входом для другого (конвейер). Хорошо для
      тестирования каждого шага.
    * **Функциональная (Functional):** Все элементы вносят вклад в выполнение одной четкой задачи — **идеал**. Класс
      `InvoiceCalculator` только считает, `InvoiceFormatter` только форматирует. Юнит-тесты пишутся легко и
      изолированно.

2. **Типы связанности (от худшего к лучшему):**
    * **Содержание (Content):** Модуль напрямую обращается к приватным данным или коду другого модуля (нарушение
      инкапсуляции). Тесты становятся хрупкими к любым изменениям.
    * **Общая (Common):** Модули используют глобальные данные. Для тестов это катастрофа — состояние тестов влияет друг
      на друга, параллельный запуск невозможен.
    * **Внешняя (External):** Модули зависят от внешнего формата данных или протокола. Требует сложных интеграционных
      тестов и моков.
    * **Управление (Control):** Один модуль управляет логикой другого (например, передача флагов). Усложняет
      тестирование, так как нужно проверять множество ветвлений.
    * **Структурная (Stamp):** Модуль принимает сложную структуру данных, но использует только часть полей. Создает
      скрытые зависимости и усложняет создание тестовых данных.
    * **Данных (Data):** Модули взаимодействуют через минимальный интерфейс (например, передача примитивных значений). *
      *Идеал** для тестирования — зависимости легко заглушить.

3. **Инструменты для достижения:**
    * **Принцип единственной ответственности (SRP)** ведет к высокой связности.
    * **Инверсия зависимостей (DIP)** через абстракции (ABC, Protocol) ведет к низкой связанности.
    * **Закон Деметры ("не разговаривай с незнакомцами")** снижает связанность.

- [Содержание](#содержание)

---

# **SOLID**

## **Junior Level*

SOLID — это набор из пяти ключевых принципов проектирования в объектно-ориентированном программировании, которые
помогают создавать гибкий, поддерживаемый и расширяемый код. Каждая буква акронима представляет отдельный принцип:

**S - Single Responsibility Principle (Принцип единственной ответственности):** Каждый класс или модуль должен иметь
только одну причину для изменения. Он должен отвечать за одну конкретную задачу или ответственность.

**O - Open/Closed Principle (Принцип открытости/закрытости):** Классы должны быть открыты для расширения, но закрыты для
модификации. Это означает, что мы можем добавлять новое поведение, не меняя существующий код.

**L - Liskov Substitution Principle (Принцип подстановки Барбары Лисков):** Объекты в программе должны быть заменяемы на
экземпляры их подтипов без изменения корректности программы. Проще говоря: если что-то работает с родительским классом,
это должно работать и с любым его наследником.

**I - Interface Segregation Principle (Принцип разделения интерфейса):** Лучше иметь много специализированных
интерфейсов, чем один универсальный. Клиенты не должны зависеть от методов, которые они не используют.

**D - Dependency Inversion Principle (Принцип инверсии зависимостей):** Модули верхнего уровня не должны зависеть от
модулей нижнего уровня. Оба должны зависеть от абстракций. Абстракции не должны зависеть от деталей — детали должны
зависеть от абстракций.

Для QA инженера понимание SOLID критично, потому что код, написанный по этим принципам, гораздо легче тестировать: он
лучше изолирован, зависимости явные и заменяемые, а поведение предсказуемо.

## **Middle Level**

С технической точки зрения, каждый принцип SOLID реализуется через конкретные паттерны и механизмы Python.

1. **SRP (Single Responsibility):** На уровне модуля (файла .py) это означает, что модуль должен экспортировать
   логически связанный набор функций/классов. На уровне класса — класс должен иметь минимальное количество публичных
   методов, связанных одной целью. Нарушение SRP приводит к God-объектам, которые невозможно адекватно покрыть
   юнит-тестами. Метрика: если вы не можете назвать ответственность класса одним коротким предложением без союза "и" —
   принцип нарушен.

2. **OCP (Open/Closed):** В Python реализуется через:
    * **Наследование и полиморфизм:** Создание подклассов для добавления поведения.
    * **Композицию и паттерн "Стратегия":** Передача поведения через callable-объекты.
    * **Декораторы:** Обертывание функций без изменения их исходного кода.
    * **Абстрактные базовые классы (ABC):** Определение интерфейсов для расширения.
      Код, соответствующий OCP, позволяет добавлять новые тестовые сценарии и проверки без модификации ядра тестового
      фреймворка.

3. **LSP (Liskov Substitution):** Это контрактный принцип. В Python он реализуется через:
    * **Соблюдение сигнатур методов:** Подкласс не должен ужесточать предусловия (требовать больше) или ослаблять
      постусловия (обещать меньше).
    * **Сохранение инвариантов:** Состояние объекта после операций должно оставаться валидным с точки зрения базового
      класса.
    * **Исключения:** Подкласс не должен выбрасывать новые типы исключений, не являющиеся подтипами исключений базового
      класса.
      Нарушение LSP — классическая причина падения тестов при замене реализации. Для QA это означает, что моки и стабы
      должны точно следовать контракту реальных объектов.

4. **ISP (Interface Segregation):** В Python, где нет формальных интерфейсов, принцип реализуется через:
    * **Абстрактные базовые классы (ABC)** с минимальным набором абстрактных методов.
    * **Протоколы (Protocol),** которые позволяют описывать узкие, специфичные наборы методов.
    * **Миксины (Mixins)** — классы, предоставляющие конкретную функциональность.
      Следствие для тестирования: нам не нужно мокировать гигантские интерфейсы, достаточно реализовать только
      используемую часть.

5. **DIP (Dependency Inversion):** Практическая реализация:
    * **Зависимость от абстракций:** Вместо `ConcreteService` в коде указывается `AbstractService` (ABC или Protocol).
    * **Внедрение зависимостей (DI):** Зависимости передаются извне (через конструктор, сеттеры, контекст), а не
      создаются внутри класса.
    * **IoC-контейнеры** (в продвинутых случаях) для автоматического управления зависимостями.
      Это краеугольный камень тестируемости: он позволяет легко подменять реальные сервисы моками в тестах.

## **Senior Level**

### **Пример 1: Single Responsibility Principle (SRP)**

#### ❌ **Неправильно: God Object в тестовом фреймворке**

```python
class TestFramework:
    """Нарушение SRP: класс делает слишком много"""

    def __init__(self):
        self.driver = None
        self.config = {}
        self.test_data = {}
        self.report_data = []
        self.logger = None

    def setup_driver(self):
        # Создание драйвера
        pass

    def load_config(self, path):
        # Загрузка конфигурации
        pass

    def generate_test_data(self):
        # Генерация тестовых данных
        pass

    def run_test(self, test_case):
        # Запуск теста
        pass

    def generate_report(self):
        # Генерация отчета
        pass

    def send_email(self):
        # Отправка email
        pass

    def cleanup(self):
        # Очистка
        pass

    def backup_results(self):
        # Бэкап результатов
        pass
```

#### ✅ **Правильно: Разделение ответственностей**

```python
from abc import ABC, abstractmethod
from typing import Protocol


class WebDriverProvider(Protocol):
    def get_driver(self): ...

    def quit_driver(self): ...


class ConfigLoader:
    def __init__(self, path: str):
        self.path = path

    def load(self) -> dict:
        """Только загрузка конфигурации"""
        with open(self.path, 'r') as f:
            return json.load(f)


class TestDataGenerator:
    def __init__(self, strategy: DataGenerationStrategy):
        self.strategy = strategy

    def generate(self) -> dict:
        """Только генерация тестовых данных"""
        return self.strategy.execute()


class TestRunner:
    def __init__(self, driver_provider: WebDriverProvider):
        self.driver_provider = driver_provider

    def run(self, test_case: TestCase) -> TestResult:
        """Только запуск теста"""
        driver = self.driver_provider.get_driver()
        # логика теста
        return result


class ReportGenerator:
    def __init__(self, formatter: ReportFormatter):
        self.formatter = formatter

    def generate(self, results: list[TestResult]) -> Report:
        """Только генерация отчета"""
        return self.formatter.format(results)


# Композиция компонентов
class TestSuite:
    def __init__(
            self,
            config_loader: ConfigLoader,
            data_generator: TestDataGenerator,
            test_runner: TestRunner,
            report_generator: ReportGenerator
    ):
        self.config_loader = config_loader
        self.data_generator = data_generator
        self.test_runner = test_runner
        self.report_generator = report_generator

    def execute(self):
        config = self.config_loader.load()
        test_data = self.data_generator.generate()
        # ... выполнение
```

### **Пример 2: Open/Closed Principle (OCP)**

#### ❌ **Неправильно: Модификация при добавлении новой проверки**

```python
class AssertionChecker:
    """Нарушение OCP: нужно модифицировать при добавлении новых проверок"""

    def check(self, actual, expected, check_type):
        if check_type == "equals":
            return actual == expected
        elif check_type == "contains":
            return expected in actual
        elif check_type == "greater":
            return actual > expected
        # Добавление новой проверки требует изменения кода
        # elif check_type == "regex":
        #     return bool(re.match(expected, actual))
        else:
            raise ValueError(f"Unknown check type: {check_type}")
```

#### ✅ **Правильно: Расширение через новые классы**

```python
from abc import ABC, abstractmethod
from typing import Any


class CheckStrategy(ABC):
    @abstractmethod
    def execute(self, actual: Any, expected: Any) -> bool:
        pass


class EqualsCheck(CheckStrategy):
    def execute(self, actual, expected):
        return actual == expected


class ContainsCheck(CheckStrategy):
    def execute(self, actual, expected):
        return expected in actual


class RegexCheck(CheckStrategy):
    def execute(self, actual, expected):
        import re
        return bool(re.match(expected, actual))


class CustomCheck(CheckStrategy):
    def __init__(self, custom_func):
        self.custom_func = custom_func

    def execute(self, actual, expected):
        return self.custom_func(actual, expected)


class AssertionChecker:
    """Соблюдение OCP: расширяется без модификации"""

    def __init__(self):
        self._strategies = {}
        self._register_default_strategies()

    def _register_default_strategies(self):
        self.register("equals", EqualsCheck())
        self.register("contains", ContainsCheck())

    def register(self, name: str, strategy: CheckStrategy):
        """Добавление новой стратегии без изменения кода"""
        self._strategies[name] = strategy

    def check(self, name: str, actual: Any, expected: Any) -> bool:
        if name not in self._strategies:
            raise ValueError(f"Unknown check: {name}")
        return self._strategies[name].execute(actual, expected)


# Использование
checker = AssertionChecker()
checker.register("regex", RegexCheck())  # Расширение без модификации

# Новую кастомную проверку можно добавить динамически
checker.register("custom", CustomCheck(lambda a, e: len(a) > e))
```

### **Пример 3: Liskov Substitution Principle (LSP)**

#### ❌ **Неправильно: Нарушение контракта базового класса**

```python
class Database:
    def connect(self, timeout: int = 10) -> bool:
        """Возвращает True при успешном подключении"""
        # Логика подключения
        return True

    def query(self, sql: str) -> list:
        """Возвращает список результатов"""
        return []


class MockDatabase(Database):
    def connect(self, timeout: int = 10) -> bool:
        """Нарушение LSP: изменяет контракт - требует больше"""
        if timeout < 20:  # Ужесточение предусловия
            raise ValueError("Mock requires timeout >= 20")
        return True

    def query(self, sql: str) -> list:
        """Нарушение LSP: изменяет контракт - возвращает другой тип"""
        return {}  # Должен возвращать list, а возвращает dict


# В тестах это приведет к падению
def test_database_operations(db: Database):
    db.connect(15)  # Упадет с MockDatabase
    results = db.query("SELECT * FROM users")
    # Упадет при попытке использовать results как list
```

#### ✅ **Правильно: Соблюдение контракта**

```python
from typing import Protocol, runtime_checkable


@runtime_checkable
class DatabaseProtocol(Protocol):
    def connect(self, timeout: int) -> bool: ...

    def query(self, sql: str) -> list: ...


class RealDatabase:
    def connect(self, timeout: int = 10) -> bool:
        print(f"Connecting with timeout {timeout}")
        return True

    def query(self, sql: str) -> list:
        print(f"Executing: {sql}")
        return ["result1", "result2"]


class MockDatabase:
    def connect(self, timeout: int = 10) -> bool:
        """Соблюдает контракт: те же предусловия"""
        print(f"Mock connecting with timeout {timeout}")
        return True  # Всегда успешно

    def query(self, sql: str) -> list:
        """Соблюдает контракт: возвращает list"""
        print(f"Mock executing: {sql}")
        return ["mock_result1", "mock_result2"]  # Корректный тип


class InMemoryDatabase:
    def __init__(self):
        self.data = {}

    def connect(self, timeout: int = 10) -> bool:
        """Соблюдает контракт"""
        return True  # Всегда успешно

    def query(self, sql: str) -> list:
        """Соблюдает контракт"""
        # Парсинг SQL и работа с памятью
        return list(self.data.values())


# Все реализации могут быть использованы взаимозаменяемо
def run_database_test(db: DatabaseProtocol):
    assert isinstance(db, DatabaseProtocol)  # Проверка протокола

    success = db.connect(15)
    assert success is True

    results = db.query("SELECT * FROM users")
    assert isinstance(results, list)  # Гарантировано

    return results


# Все реализации работают корректно
for db in [RealDatabase(), MockDatabase(), InMemoryDatabase()]:
    run_database_test(db)
```

### **Пример 4: Interface Segregation Principle (ISP)**

#### ❌ **Неправильно: "Толстый" интерфейс**

```python
class TestFrameworkInterface:
    """Нарушение ISP: заставляет реализовывать ненужные методы"""

    def setup_driver(self):
        pass

    def teardown_driver(self):
        pass

    def load_config(self):
        pass

    def parse_results(self):
        pass

    def generate_report(self):
        pass

    def send_notification(self):
        pass

    def backup_results(self):
        pass

    def cleanup_temp_files(self):
        pass


class SimpleTestRunner(TestFrameworkInterface):
    """Вынужден реализовывать все методы, хотя нужны только некоторые"""

    def setup_driver(self):
        # Нужно
        pass

    def teardown_driver(self):
        # Нужно
        pass

    def load_config(self):
        # Нужно
        pass

    def parse_results(self):
        # Не нужно, но вынужден реализовать
        raise NotImplementedError("Not needed for simple runner")

    def generate_report(self):
        # Не нужно, но вынужден реализовать
        raise NotImplementedError("Not needed for simple runner")

    # ... остальные ненужные методы
```

#### ✅ **Правильно: Разделенные интерфейсы**

```python
from typing import Protocol


class DriverManager(Protocol):
    def setup_driver(self): ...

    def teardown_driver(self): ...


class ConfigLoader(Protocol):
    def load_config(self) -> dict: ...


class TestExecutor(Protocol):
    def execute_test(self, test_case) -> TestResult: ...


class ResultParser(Protocol):
    def parse_results(self, results) -> ParsedResults: ...


class ReportGenerator(Protocol):
    def generate_report(self, data) -> Report: ...


class NotificationSender(Protocol):
    def send_notification(self, message): ...


# Классы реализуют только нужные интерфейсы
class SimpleTestRunner:
    def __init__(
            self,
            driver_manager: DriverManager,
            config_loader: ConfigLoader,
            test_executor: TestExecutor
    ):
        self.driver_manager = driver_manager
        self.config_loader = config_loader
        self.test_executor = test_executor

    def run(self):
        # Использует только нужные зависимости
        config = self.config_loader.load_config()
        self.driver_manager.setup_driver()
        # ... выполнение тестов
        self.driver_manager.teardown_driver()


class FullFeaturedRunner(SimpleTestRunner):
    def __init__(
            self,
            driver_manager: DriverManager,
            config_loader: ConfigLoader,
            test_executor: TestExecutor,
            result_parser: ResultParser,
            report_generator: ReportGenerator,
            notifier: NotificationSender
    ):
        super().__init__(driver_manager, config_loader, test_executor)
        self.result_parser = result_parser
        self.report_generator = report_generator
        self.notifier = notifier

    def run_with_reporting(self):
        results = self.run()
        parsed = self.result_parser.parse_results(results)
        report = self.report_generator.generate_report(parsed)
        self.notifier.send_notification(f"Report: {report}")
```

### **Пример 5: Dependency Inversion Principle (DIP)**

#### ❌ **Неправильно: Зависимость от конкретных реализаций**

```python
import requests
import smtplib
import json


class TestReporter:
    """Нарушение DIP: зависит от конкретных библиотек"""

    def __init__(self):
        # Прямая зависимость от конкретных реализаций
        self.http_client = requests.Session()
        self.email_client = smtplib.SMTP()
        self.json_parser = json

    def send_to_webhook(self, url: str, data: dict):
        """Жесткая привязка к requests"""
        response = self.http_client.post(url, json=data)
        return response.status_code == 200

    def send_email(self, to: str, subject: str, body: str):
        """Жесткая привязка к smtplib"""
        self.email_client.connect()
        # ... логика отправки
        self.email_client.quit()

    def parse_json(self, text: str):
        """Жесткая привязка к json модулю"""
        return self.json_parser.loads(text)
```

#### ✅ **Правильно: Зависимость от абстракций**

```python
from typing import Protocol, Any
from abc import ABC, abstractmethod


class HttpClient(Protocol):
    def post(self, url: str, data: dict) -> Any: ...

    def get(self, url: str) -> Any: ...


class EmailClient(Protocol):
    def connect(self): ...

    def send(self, to: str, subject: str, body: str) -> bool: ...

    def disconnect(self): ...


class JsonParser(Protocol):
    def loads(self, text: str) -> dict: ...

    def dumps(self, obj: dict) -> str: ...


class RequestsHttpClient:
    """Конкретная реализация для production"""

    def __init__(self):
        import requests
        self.session = requests.Session()

    def post(self, url: str, data: dict):
        return self.session.post(url, json=data)

    def get(self, url: str):
        return self.session.get(url)


class MockHttpClient:
    """Реализация для тестирования"""

    def __init__(self, mock_responses=None):
        self.mock_responses = mock_responses or {}
        self.calls = []

    def post(self, url: str, data: dict):
        self.calls.append(("POST", url, data))
        return self.mock_responses.get(url, MockResponse(200))

    def get(self, url: str):
        self.calls.append(("GET", url))
        return self.mock_responses.get(url, MockResponse(200))


class TestReporter:
    """Соблюдение DIP: зависит от абстракций"""

    def __init__(
            self,
            http_client: HttpClient,
            email_client: EmailClient,
            json_parser: JsonParser
    ):
        self.http_client = http_client
        self.email_client = email_client
        self.json_parser = json_parser

    def send_to_webhook(self, url: str, data: dict) -> bool:
        """Работает с любой реализацией HttpClient"""
        response = self.http_client.post(url, data)
        return getattr(response, 'status_code', 200) == 200

    def send_email(self, to: str, subject: str, body: str) -> bool:
        """Работает с любой реализацией EmailClient"""
        self.email_client.connect()
        success = self.email_client.send(to, subject, body)
        self.email_client.disconnect()
        return success


# В production
production_reporter = TestReporter(
    http_client=RequestsHttpClient(),
    email_client=RealEmailClient(),
    json_parser=StandardJsonParser()
)

# В тестах
mock_reporter = TestReporter(
    http_client=MockHttpClient(),
    email_client=MockEmailClient(),
    json_parser=MockJsonParser()
)
```

### **Пример 6: Комплексный пример - Фреймворк для API тестирования**

#### ❌ **Неправильно: Монолитная архитектура с нарушениями SOLID**

```python
class APITestFramework:
    """Монолит с множеством нарушений SOLID"""

    def __init__(self):
        self.requests = __import__('requests')
        self.json = __import__('json')
        self.config = self._load_config()
        self.auth = self._setup_auth()
        self.results = []
        self.reports = []

    def run_test(self, endpoint, method="GET", data=None):
        # SRP: слишком много ответственностей
        # DIP: прямая зависимость от requests
        # OCP: сложно добавить новые типы тестов

        url = self.config['base_url'] + endpoint

        # Жестко закодированная логика
        if method == "GET":
            response = self.requests.get(url, auth=self.auth)
        elif method == "POST":
            response = self.requests.post(url, json=data, auth=self.auth)
        # ... другие методы

        # Разная логика обработки в одном методе
        if response.status_code == 200:
            self._log_success(response)
        else:
            self._log_failure(response)

        # Генерация отчета в том же методе
        self._generate_report(response)

        return response
```

#### ✅ **Правильно: SOLID-архитектура**

```python
from abc import ABC, abstractmethod
from typing import Protocol, Optional, Any
from dataclasses import dataclass
import json as json_module


# ===== АБСТРАКЦИИ (DIP) =====
class HttpClient(Protocol):
    def request(self, method: str, url: str, **kwargs) -> Any: ...


class Authenticator(Protocol):
    def authenticate(self, request: dict) -> dict: ...


class ResponseValidator(Protocol):
    def validate(self, response) -> bool: ...


class ReportGenerator(Protocol):
    def generate(self, test_result) -> str: ...


# ===== SRP: Каждый класс с одной ответственностью =====
@dataclass
class TestRequest:
    endpoint: str
    method: str = "GET"
    data: Optional[dict] = None
    headers: Optional[dict] = None


@dataclass
class TestResult:
    request: TestRequest
    response: Any
    success: bool
    duration: float


class RequestBuilder:
    """Только сборка запросов"""

    def __init__(self, base_url: str):
        self.base_url = base_url

    def build(self, test_request: TestRequest) -> dict:
        return {
            'method': test_request.method,
            'url': f"{self.base_url}{test_request.endpoint}",
            'json': test_request.data,
            'headers': test_request.headers or {}
        }


class TestExecutor:
    """Только выполнение тестов"""

    def __init__(
            self,
            http_client: HttpClient,
            authenticator: Authenticator,
            request_builder: RequestBuilder
    ):
        self.http_client = http_client
        self.authenticator = authenticator
        self.request_builder = request_builder

    def execute(self, test_request: TestRequest) -> TestResult:
        import time
        start = time.time()

        # Подготовка запроса
        request_data = self.request_builder.build(test_request)
        authenticated_request = self.authenticator.authenticate(request_data)

        # Выполнение
        response = self.http_client.request(**authenticated_request)

        duration = time.time() - start

        return TestResult(
            request=test_request,
            response=response,
            success=200 <= getattr(response, 'status_code', 0) < 300,
            duration=duration
        )


# ===== OCP: Легко расширяемые валидаторы =====
class StatusCodeValidator:
    def validate(self, response) -> bool:
        return 200 <= response.status_code < 300


class JsonSchemaValidator:
    def __init__(self, schema: dict):
        self.schema = schema

    def validate(self, response) -> bool:
        import jsonschema
        try:
            jsonschema.validate(response.json(), self.schema)
            return True
        except:
            return False


class CompositeValidator:
    """Композиция валидаторов"""

    def __init__(self, validators: list[ResponseValidator]):
        self.validators = validators

    def validate(self, response) -> bool:
        return all(v.validate(response) for v in self.validators)


# ===== LSP: Взаимозаменяемые аутентификаторы =====
class BasicAuthenticator:
    def authenticate(self, request: dict) -> dict:
        request['auth'] = ('user', 'pass')
        return request


class TokenAuthenticator:
    def __init__(self, token: str):
        self.token = token

    def authenticate(self, request: dict) -> dict:
        headers = request.get('headers', {})
        headers['Authorization'] = f'Bearer {self.token}'
        request['headers'] = headers
        return request


class MockAuthenticator:
    def authenticate(self, request: dict) -> dict:
        return request  # Без аутентификации для тестов


# ===== ISP: Специализированные генераторы отчетов =====
class JsonReportGenerator:
    def generate(self, test_result: TestResult) -> str:
        return json_module.dumps({
            'success': test_result.success,
            'duration': test_result.duration,
            'status': getattr(test_result.response, 'status_code', None)
        })


class HtmlReportGenerator:
    def generate(self, test_result: TestResult) -> str:
        return f"""
        <html>
            <body>
                <h1>Test Result</h1>
                <p>Success: {test_result.success}</p>
                <p>Duration: {test_result.duration:.2f}s</p>
            </body>
        </html>
        """


# ===== ФИНАЛЬНАЯ КОМПОЗИЦИЯ =====
class APITestFramework:
    """SOLID-совместимый фреймворк"""

    def __init__(
            self,
            base_url: str,
            http_client: HttpClient,
            authenticator: Authenticator,
            validator: ResponseValidator,
            report_generator: ReportGenerator
    ):
        self.request_builder = RequestBuilder(base_url)
        self.executor = TestExecutor(http_client, authenticator, self.request_builder)
        self.validator = validator
        self.report_generator = report_generator
        self.results = []

    def run_test(self, test_request: TestRequest) -> dict:
        # Выполнение
        result = self.executor.execute(test_request)

        # Валидация
        result.success = self.validator.validate(result.response)

        # Отчет
        report = self.report_generator.generate(result)

        self.results.append(result)
        return {
            'result': result,
            'report': report,
            'valid': result.success
        }

    # Легко добавить новые методы без изменения существующего кода (OCP)
    def run_test_suite(self, requests: list[TestRequest]):
        return [self.run_test(req) for req in requests]


# ===== ИСПОЛЬЗОВАНИЕ С РАЗНЫМИ КОНФИГУРАЦИЯМИ =====

# Production конфигурация
production_framework = APITestFramework(
    base_url="https://api.example.com",
    http_client=RequestsHttpClient(),
    authenticator=TokenAuthenticator("real-token"),
    validator=CompositeValidator([
        StatusCodeValidator(),
        JsonSchemaValidator({"type": "object"})
    ]),
    report_generator=JsonReportGenerator()
)

# Test конфигурация
test_framework = APITestFramework(
    base_url="https://test-api.example.com",
    http_client=MockHttpClient(),
    authenticator=MockAuthenticator(),
    validator=StatusCodeValidator(),
    report_generator=HtmlReportGenerator()
)

# Конфигурация для нагрузочного тестирования
load_framework = APITestFramework(
    base_url="https://api.example.com",
    http_client=AsyncHttpClient(),  # Новая реализация без изменения фреймворка
    authenticator=BasicAuthenticator(),
    validator=StatusCodeValidator(),
    report_generator=CsvReportGenerator()  # Новый генератор без изменения фреймворка
)
```

- [Содержание](#содержание)

---

# **Наследование vs композиция**

## **Junior Level**

Наследование и композиция — это два фундаментальных подхода к организации кода в объектно-ориентированном
программировании, и выбор между ними определяет, насколько гибкой, понятной и удобной для поддержки будет ваша система.

Основное различие лежит в типе отношений, которые они моделируют:

* **Наследование** описывает отношение **«является»** (is-a). Например, `Dog` (Собака) наследует от `Animal` (Животное),
  потому что собака *является* конкретным видом животного. Это позволяет `Dog` автоматически получить общие для всех
  животных свойства и методы (например, `eat()` или `sleep()`), а также добавить свои уникальные (`bark()`).
* **Композиция** описывает отношение **«имеет»** (has-a). Класс `Car` (Автомобиль) не является двигателем, но он *имеет*
  его как составную часть. Двигатель — это независимый компонент, с которым автомобиль взаимодействует через четко
  определенный интерфейс.

**Ключевое правило современной разработки: предпочитайте композицию наследованию.** Наследование создает жесткую,
статическую связь, подобную родственной. Изменения в «родительском» классе могут неожиданно «сломать» всех его
«потомков». Композиция же строит более гибкие, договорные отношения: вы можете заменить «двигатель» на другой, не
переделывая всю «машину». Для тестирования это преимущество критично — компоненты, переданные через композицию, легко
подменить на заглушки (mocks), что позволяет тестировать классы изолированно.

## **Middle Level**

### **Детали и тонкости наследования в Python**

1. **Механизм и порядок разрешения методов (MRO)**: Python поддерживает множественное наследование. Чтобы управлять
   потенциальным хаосом при поиске методов, используется строгий **MRO (Method Resolution Order)**, вычисляемый по
   алгоритму C3 (`ClassName.__mro__`). Этот порядок гарантирует, что каждый класс в иерархии будет проверен только один
   раз и предсказуемо.

2. **Инструмент `super()` и кооперативное наследование**: `super()` — это не просто вызов метода родителя. В условиях
   MRO он делегирует выполнение **следующему классу в цепочке**. Это позволяет нескольким классам-предкам (например,
   `Mixin`-ам) кооперативно участвовать в выполнении одного метода (например, `__init__`), не мешая друг другу.

3. **Наследование как потенциальное нарушение инкапсуляции**: Это ключевая критика наследования. Дочерний класс получает
   доступ к защищённым (`_protected`) членам родителя, что создает хрупкую, скрытую зависимость от его внутренней
   реализации. Тестировать такой класс сложно, так как для понимания его поведения необходимо глубоко знать детали
   работы родителя.

4. **Множественное наследование и Mixins**: Python разрешает множественное наследование, что часто используется для *
   *Mixins** — небольших классов, добавляющих конкретную функциональность (например, `JSONSerializableMixin`). Mixin не
   предназначен для использования отдельно. При их применении важно проектировать имена методов так, чтобы избежать
   конфликтов, разрешаемых через MRO.

5. **Абстрактные классы (ABC)**: Модуль `abc` позволяет создавать формальные «чертежи» — классы, объявляющие
   обязательные методы (через `@abstractmethod`). Они заставляют наследников соблюдать контракт, явно формализуя
   отношение «является».

### **Детали и тонкости композиции в Python**

1. **Композиция vs Агрегация: управление жизненным циклом**:
    * **Композиция (сильная связь)**: Компонент (например, `Heart` для `Human`) не существует отдельно. Он создаётся и
      уничтожается вместе с объектом-владельцем (обычно в `__init__`).
    * **Агрегация (слабая связь)**: Компонент (например, `Driver` для `Car`) существует независимо и передаётся объекту
      извне (как аргумент). Их жизненные циклы разделены.

2. **Композиция и структурная типизация (Протоколы)**: Вместо жесткого наследования от абстрактного класса для
   достижения полиморфизма в Python всё чаще используют **композицию с протоколами** (`typing.Protocol`). Объекту не
   нужно объявлять «я наследник `Reader`» — достаточно просто реализовать метод `read()`. Это резко снижает связанность.
   В тестах можно подставить любой объект с нужным методом, не строя сложных иерархий.

3. **Делегирование как явная форма композиции**: Паттерн **Делегирование** — это когда внешний объект (делегатор) явно
   передает выполнение задачи внутреннему объекту (делегату). В Python его можно элегантно реализовать через
   `__getattr__`, автоматически перенаправляя вызовы. Это основа для объеков-обёрток (адаптеров, прокси), которые дают
   полный контроль над взаимодействием и являются идеальной точкой для внедрения моков в тестах.

4. **Динамическое поведение**: Композиция позволяет менять поведение объекта во время выполнения программы, заменяя его
   компоненты. Этот принцип лежит в основе многих паттернов, таких как **Стратегия** (Strategy), где алгоритм можно
   «подменить на лету».

### **Сравнительный анализ для проектирования и тестирования**

* **Гибкость и связность**: Наследование фиксирует отношения на этапе компиляции, приводя к высокой связности.
  Композиция/агрегация определяют поведение во время выполнения, обеспечивая слабую связность и большую гибкость.

* **Тестируемость**: Класс, построенный на композиции, легко тестировать изолированно. Его зависимости — это просто
  аргументы, которые можно подменить. Тестирование глубокой иерархии наследования требует создания сложных фикстур и
  мокирования родительских методов, что увеличивает сложность тестов.

* **Проблема хрупкого базового класса**: Это главный риск наследования. Даже безопасное на вид изменение во внутренней
  логике родителя может сломать работу непредусмотревшего этого наследника. В больших проектах отследить такие побочные
  эффекты крайне трудно.

* **Борьба со сложностью архитектуры**: Глубокие иерархии наследования имеют тенденцию разрастаться и усложняться.
  Композиция предлагает альтернативную парадигму: строить сложную систему не через вертикальное ветвление, а через
  горизонтальную сборку из небольших, независимых и легко заменяемых компонентов. Это прямой путь к более
  поддерживаемому и надежному коду.

## **Senior Level**

В CPython реализация наследования и композиции строится на разных механиках внутренних структур и способах разрешения
атрибутов, без использования спецбайткода: всё работает через стандартные механизмы поиска атрибутов и MRO.

### Наследование

- При создании класса `class Child(Parent):` CPython формирует иерархию типов, вычисляя MRO (Method Resolution Order) по
  алгоритму C3.
- MRO сохраняется в `tp_mro` объекта типа `PyTypeObject`, определяя порядок поиска атрибутов и методов по базовым
  классам.
- При обращении к атрибуту `obj.attr` (опкод `LOAD_ATTR`), интерпретатор:
    - Проверяет наличие атрибута в `__dict__` экземпляра.
    - Если не найден, ищет в `__dict__` класса.
    - Далее обходит базовые классы по MRO, проверяя каждый `__dict__` до первого совпадения.
- Изменения в базовом классе могут повлиять на поведение всех наследников, так как поиск атрибутов идёт через MRO.

### Композиция

- При композиции создаётся объект другого класса как атрибут: `self.component = OtherClass()`.
- Атрибуты экземпляра хранятся в `__dict__` или слотах, без влияния на MRO.
- При обращении к методу компонента `self.component.method()` интерпретатор:
    - Находит `component` в `__dict__` экземпляра.
    - Находит `method` в `__dict__` объекта `component`.
    - Вызывает метод напрямую.
- Композиция не использует MRO, компоненты являются независимыми объектами и доступ к ним происходит через явные ссылки.

### Подробности реализации

- **Наследование**:
    - Поиск атрибутов идёт через MRO, порядок определяется алгоритмом C3.
    - `LOAD_ATTR` вызывает `PyObject_GetAttr`, который ищет в `__dict__` экземпляра, затем в `__dict__` класса и по
      `tp_mro` базовых классов.
- **Композиция**:
    - Поиск атрибутов компонента происходит только в `__dict__` компонента, без обхода MRO.
    - Доступ к компоненту осуществляется через явные ссылки.

### Сравнение

- **Наследование**:
    - Жёсткая связь, зависит от структуры MRO.
    - Изменения в базовом классе могут сломать наследников.
    - Поиск атрибутов через MRO, используется при всех операциях с классами.
- **Композиция**:
    - Гибкая связь, компоненты независимы.
    - Компоненты можно подменить или изменить без переписывания всего кода.
    - Поиск атрибутов компонента только в его собственном `__dict__`, без MRO.

Механизмы наследования и композиции в CPython реализованы через стандартные методы поиска атрибутов и MRO, что позволяет
эффективно использовать оба подхода в зависимости от задачи.

- [Содержание](#содержание)

---

# **Композиция и агрегация**

## **Junior Level*

Композиция и агрегация — это два способа создания отношений между объектами в объектно-ориентированном программировании.
Обе описывают ситуацию, когда один объект содержит в себе другой, но с критически важным различием в силе связи и
управлении жизненным циклом.

Представьте, что вы строите дом. **Композиция** — это как комната в доме. Комната не существует отдельно от дома. Когда
дом сносят, комната исчезает вместе с ним. Объект-владелец (дом) полностью контролирует жизнь объекта-части (комнаты). *
*Агрегация** — это как мебель в доме. Стол, стул, диван существуют независимо от дома. Их занесли в дом, а потом могут
вынести в другой дом или на склад. Объект-владелец (дом) использует объект-часть (мебель), но не управляет его рождением
и смертью.

В разработке композиция означает, что при уничтожении основного объекта уничтожаются и все его составные части.
Агрегация означает, что объекты собраны вместе, но могут жить самостоятельно. Для QA инженера понимание этого различия
помогает проектировать тестовые фикстуры и моки, правильно управлять их жизненным циклом и понимать, какие зависимости
нужно создавать заново, а какие можно переиспользовать между тестами.

## **Middle Level**

С технической точки зрения, композиция и агрегация реализуются через **атрибуты класса**, но с разной семантикой
создания и владения.

1. **Реализация:**
    * **Композиция (Composition):** Объект-часть создается **внутри** конструктора (или иного метода) объекта-владельца.
      Владелец полностью инкапсулирует создание и, как правило, не предоставляет публичных методов для замены этой
      части. Часть реализуется как внутренний, приватный атрибут.
    * **Агрегация (Aggregation):** Объект-часть создается **вне** объекта-владельца и передается ему в качестве
      аргумента (чаще всего в конструктор). Владелец сохраняет ссылку на эту часть, но не управляет ее созданием.
      Объект-часть может быть общим (разделяемым) ресурсом.

2. **Жизненный цикл:**
    * При **композиции** жизненный цикл части жестко привязан к жизненному циклу целого. Когда объект-владелец
      удаляется (например, сборщиком мусора), удаляется и объект-часть, если на него больше нет ссылок.
    * При **агрегации** жизненные циклы независимы. Удаление владельца не влечет удаление части, так как на нее могут
      оставаться ссылки из других объектов.

3. **Для AQA:**
    * **Фикстуры в Pytest:** Композиция часто используется для создания сложных, вложенных фикстур, которые существуют
      только в рамках одной тестовой сессии или модуля и автоматически очищаются. Агрегация похожа на фикстуры с
      областью видимости `session` или `package`, которые создаются один раз и переиспользуются многими тестами.
    * **Тестовые данные:** Понимание, когда создавать новый экземпляр тестовых данных для каждого кейса (композиция), а
      когда использовать общий, предсозданный набор данных (агрегация), критично для скорости и изоляции тестов.
    * **Page Object:** Внутри Page Object может существовать композиция из элементов (например, `Button`, `InputField`),
      которые не имеют смысла вне контекста этой страницы. И агрегация — например, общий `Header` или `Footer`, которые
      могут быть переданы в несколько Page Object.

4. **Отличия от наследования:** И композиция, и агрегация — это альтернативы наследованию, предпочитаемые в современном
   дизайне ("предпочитай композицию наследованию"). Они обеспечивают большую гибкость и слабую связанность.

## **Senior Level**

### **Пример 1: Композиция (Сильная связь)**

```python
class DatabaseConnection:
    def __init__(self, connection_string: str):
        self.connection_string = connection_string
        self._connect()

    def _connect(self):
        print(f"Соединение с БД: {self.connection_string}")

    def close(self):
        print("Закрытие соединения с БД")


class UserRepository:
    """UserRepository СОЗДАЕТ и УПРАВЛЯЕТ жизненным циклом DatabaseConnection"""

    def __init__(self, connection_string: str):
        # Композиция: создание объекта внутри конструктора
        self._db = DatabaseConnection(connection_string)  # Часть не существует без целого

    def get_user(self, user_id: int):
        return f"User {user_id} from {self._db.connection_string}"

    def __del__(self):
        self._db.close()  # При удалении репозитория закрывается соединение


# Использование
repo = UserRepository("postgresql://localhost:5432/mydb")
print(repo.get_user(1))
# При удалении repo автоматически закроется соединение с БД
```

### **Пример 2: Агрегация (Слабая связь)**

```python
class Logger:
    """Независимый компонент, может использоваться в разных контекстах"""

    def __init__(self, name: str):
        self.name = name

    def log(self, message: str):
        print(f"[{self.name}] {message}")


class PaymentService:
    """PaymentService ИСПОЛЬЗУЕТ Logger, но не управляет его жизненным циклом"""

    def __init__(self, logger: Logger):  # Агрегация: передача извне
        self._logger = logger  # Часть существует независимо от целого

    def process_payment(self, amount: float):
        self._logger.log(f"Processing payment: ${amount}")
        return True


class NotificationService:
    """Тот же логгер может использоваться в разных сервисах"""

    def __init__(self, logger: Logger):
        self._logger = logger

    def send_notification(self, message: str):
        self._logger.log(f"Sending: {message}")


# Использование
shared_logger = Logger("AppLogger")  # Общий ресурс

payment_service = PaymentService(shared_logger)
notification_service = NotificationService(shared_logger)  # Один логгер в двух сервисах

payment_service.process_payment(100.0)
notification_service.send_notification("Payment successful")
```

### **Пример 3: Практический пример для AQA (Page Object с композицией и агрегацией)**

```python
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC


class BaseElement:
    """Базовый элемент - может существовать самостоятельно"""

    def __init__(self, driver, locator):
        self.driver = driver
        self.locator = locator
        self.element = WebDriverWait(driver, 10).until(
            EC.presence_of_element_located(locator)
        )

    def click(self):
        self.element.click()

    def get_text(self):
        return self.element.text


class Button(BaseElement):
    """Специализированный элемент"""

    def is_enabled(self):
        return self.element.is_enabled()


class LoginForm:
    """Композиция: форма создает свои элементы внутри"""

    def __init__(self, driver):
        self.driver = driver
        # Композиция: элементы создаются внутри формы
        self.username_input = BaseElement(
            driver, (By.ID, "username")
        )  # Не существует без формы
        self.password_input = BaseElement(
            driver, (By.ID, "password")
        )  # Не существует без формы
        self.submit_button = Button(
            driver, (By.CSS_SELECTOR, "button[type='submit']")
        )  # Не существует без формы

    def login(self, username: str, password: str):
        self.username_input.element.send_keys(username)
        self.password_input.element.send_keys(password)
        self.submit_button.click()


class Header:
    """Агрегация: хедер может быть переиспользован на разных страницах"""

    def __init__(self, driver):
        self.driver = driver
        self.logo = BaseElement(driver, (By.CLASS_NAME, "logo"))
        self.cart_button = Button(driver, (By.ID, "cart"))


class HomePage:
    """Агрегация: страница использует готовый хедер"""

    def __init__(self, driver, header: Header = None):  # Агрегация через DI
        self.driver = driver
        # Агрегация: хедер может быть передан извне
        self.header = header if header else Header(driver)
        # Композиция: форма логина создается внутри страницы
        self.login_form = LoginForm(driver)

    def go_to_cart(self):
        self.header.cart_button.click()


# Тестовый пример
def test_login_with_shared_header():
    driver = webdriver.Chrome()

    # Создаем хедер один раз (агрегация)
    shared_header = Header(driver)

    # Переиспользуем хедер на разных страницах
    home_page = HomePage(driver, shared_header)
    product_page = ProductPage(driver, shared_header)  # Предположим, такой класс существует

    # Тестируем
    home_page.login_form.login("user", "pass")
    assert home_page.header.logo.get_text() == "MyStore"

    # Хедер продолжает существовать при переходе между страницами
    product_page.header.cart_button.click()
```

### **Пример 4: Фикстуры Pytest с учетом композиции/агрегации**

```python
import pytest


class Database:
    """Дорогой в создании ресурс"""

    def __init__(self):
        print("Creating expensive database connection...")

    def query(self, sql):
        return f"Result of {sql}"

    def close(self):
        print("Closing database connection...")


class UserService:
    """Сервис агрегирует базу данных"""

    def __init__(self, db: Database):
        self.db = db  # Агрегация

    def get_user(self, user_id):
        return self.db.query(f"SELECT * FROM users WHERE id = {user_id}")


# Фикстура с композицией (создает новый экземпляр для каждого теста)
@pytest.fixture
def user_service_with_composition():
    """Каждый тест получает свой изолированный сервис с собственной БД"""
    db = Database()  # Композиция внутри фикстуры
    service = UserService(db)
    yield service
    db.close()


# Фикстура с агрегацией (разделяемый ресурс)
@pytest.fixture(scope="session")
def shared_database():
    """Одно соединение на всю сессию тестов"""
    db = Database()
    yield db
    db.close()


@pytest.fixture
def user_service_with_aggregation(shared_database):
    """Каждый тест получает сервис, но все используют одну БД"""
    return UserService(shared_database)  # Агрегация


# Тесты
def test_user_1(user_service_with_composition):
    result = user_service_with_composition.get_user(1)
    assert "SELECT" in result
    # После теста БД будет закрыта


def test_user_2(user_service_with_aggregation):
    result = user_service_with_aggregation.get_user(2)
    assert "SELECT" in result
    # БД остается открытой для других тестов
```

### **Пример 5: Dependency Injection как реализация агрегации**

```python
from abc import ABC, abstractmethod
from typing import Protocol


# Определяем протокол (интерфейс)
class LoggerProtocol(Protocol):
    def log(self, message: str) -> None: ...


class ConsoleLogger:
    def log(self, message: str) -> None:
        print(f"LOG: {message}")


class FileLogger:
    def __init__(self, filename: str):
        self.filename = filename

    def log(self, message: str) -> None:
        with open(self.filename, 'a') as f:
            f.write(f"{message}\n")


class OrderService:
    """Сервис агрегирует логгер через внедрение зависимости"""

    def __init__(self, logger: LoggerProtocol):  # Агрегация + Dependency Injection
        self.logger = logger

    def place_order(self, order_id: str):
        self.logger.log(f"Placing order {order_id}")
        # Логика заказа
        return True


# Конфигурация зависимостей (в продакшене может быть в DI-контейнере)
def create_order_service(logger_type: str = "console") -> OrderService:
    if logger_type == "console":
        logger = ConsoleLogger()
    else:
        logger = FileLogger("app.log")

    return OrderService(logger)  # Агрегация


# Тестирование с моками
from unittest.mock import Mock


def test_order_service_with_mock():
    # Создаем mock логгера
    mock_logger = Mock(spec=LoggerProtocol)
    mock_logger.log = Mock()

    # Внедряем mock в сервис
    service = OrderService(mock_logger)  # Агрегация для тестирования

    # Выполняем тест
    result = service.place_order("ORDER123")

    # Проверяем взаимодействие
    assert result is True
    mock_logger.log.assert_called_once_with("Placing order ORDER123")
```

- [Содержание](#содержание)

---

# **Метапрограммирование**

## **Junior Level**

Метапрограммирование — это искусство написания программ, которые анализируют, генерируют или изменяют другие программы (
включая самих себя) во время выполнения. Если обычный код оперирует данными (числами, строками, объектами), то
метапрограммирование оперирует **кодом как данными**.

Представьте себе фабрику, которая не производит товары, а проектирует и собирает другие фабрики. Метапрограммирование в
Python — это и есть такая «фабрика фабрик», дающая вам инструменты для управления структурой и поведением кода на более
высоком, абстрактном уровне.

Простейшие формы метапрограммирования:

* **Декораторы** (`@decorator`): Это «обёртки», которые меняют поведение функции или класса, не трогая их исходный код.
  Например, `@staticmethod` или `@property` изменяют способ вызова метода.
* **Динамическое создание классов**: С помощью встроенной функции `type()` можно создать новый класс прямо в процессе
  выполнения программы, сгенерировав его имя, атрибуты и методы «на лету».
* **Изменение объектов**: В Python можно добавить новый метод или атрибут к уже существующему объекту после его
  создания. Эта гибкость — фундаментальное свойство языка, открывающее двери для метапрограммирования.

Метапрограммирование позволяет создавать элегантные абстракции, автоматизировать рутинный код (
например, генерацию геттеров/сеттеров), внедрять сквозную логику (логирование, проверку прав) и строить мощные
фреймворки (такие как ORM Django или система валидации Pydantic), где структура кода динамически определяется на основе
моделей или конфигураций.

## **Middle Level**

Метапрограммирование в Python — это не единая техника, а целый арсенал инструментов разного уровня воздействия: от
точечной модификации функций до управления самой природой создания классов. Их можно представить как матрешку: каждый
следующий инструмент работает на более глубоком уровне абстракции.

### **1. Декораторы: Модификация поведения «снаружи»**

Декоратор — это функция высшего порядка, принимающая в качестве аргумента другую функцию (или класс) и возвращающая
новую, модифицированную версию. Синтаксис `@` — это лишь удобная запись.

* **Суть**: Декоратор действует как «обёртка» или «плагин», добавляющий дополнительную логику *до* или *после* вызова
  исходной функции, не изменяя её ядра.
* **Эволюция**: Декораторы могут быть вложенными, принимать собственные аргументы (`@retry(attempts=3)`) и применяться к
  методам класса. Они стали стандартным способом реализации аспектно-ориентированного программирования в Python для
  задач кеширования, логирования, аутентификации.

### **2. Дескрипторы: Управление доступом к атрибутам**

Дескрипторы — это объекты, реализующие протокол с методами `__get__`, `__set__` и `__delete__`. Когда вы обращаетесь к
атрибуту объекта, Python проверяет, не является ли этот атрибут дескриптором, и если да, то вызывает соответствующий
метод.

* **Суть**: Дескрипторы перехватывают операции чтения, записи или удаления атрибута, позволяя вам внедрить свою логику (
  валидацию, ленивые вычисления, логирование доступа).
* **Где встречается**: Вся система `@property`, `@classmethod`, `@staticmethod`, а также механизм `__slots__` построены
  на дескрипторах. Это мощный инструмент для создания умных атрибутов, поведение которых вы можете полностью
  контролировать.

### **3. Метаклассы: Контроль над созданием классов**

Если класс — это «чертёж» для создания объектов, то метакласс — это «чертёж» для создания классов. По умолчанию все
классы создаются метаклассом `type`.

* **Суть**: Создав собственный метакласс (наследовав от `type`), вы можете перехватить момент создания класса (до
  появления первого его экземпляра). В этот момент можно проанализировать, модифицировать или валидировать атрибуты и
  методы будущего класса, зарегистрировать его в реестре или автоматически добавить новые методы.
* **Применение**: ORM-фреймворки (например, Django) используют метаклассы для трансформации объявлений моделей
  `class User(models.Model)` в сложные объекты с доступом к базе данных. Это инструмент для глубокой, архитектурной
  магии, когда требуется, чтобы классы вели себя особым, нестандартным образом с самого рождения.

### **Сопутствующие мощные (и опасные) техники**

* **Интроспекция**: Возможность программы исследовать свою собственную структуру во время выполнения. Функции `dir()`,
  `getattr()`, `hasattr()`, а также модуль `inspect` позволяют «заглянуть внутрь» объектов, узнать их методы, аргументы,
  исходный код. Это основа для многих продвинутых инструментов: систем автоматического обнаружения тестов, фреймворков
  dependency injection, интерактивных дебаггеров и автодокументирования.
* **Monkey Patching (Обучение обезьян)**: Динамическое изменение модулей или классов *после* их загрузки в память, во
  время выполнения программы. Позволяет «исправлять» поведение стороннего кода, добавлять фичи или, что наиболее часто,
  подменять реальные объекты на **моки (mock-объекты)** в целях тестирования (как это делает библиотека
  `unittest.mock`). Это крайне мощный, но и рискованный приём, так как может привести к трудноотлавливаемым побочным
  эффектам, если делать это без чёткого понимания области видимости изменений.
* **Динамическое выполнение кода (`eval`/`exec`)**: Функции `eval()` (вычисляет выражение) и `exec()` (выполняет блок
  кода) интерпретируют строки как инструкции Python. Это самый радикальный вид метапрограммирования, позволяющий
  исполнять код, сгенерированный самой программой. **Крайне опасен** при использовании с непроверенными
  пользовательскими данными (уязвимость инъекции кода). Применяется во внутренних DSL (предметно-ориентированных
  языках), сложных системах конфигурации или шаблонизаторах, где безопасность входных данных гарантирована.

### **Иерархия и синергия**

Эти инструменты образуют иерархию по глубине воздействия:

1. **Декораторы** меняют уже готовые функции/классы.
2. **Дескрипторы** управляют доступом к отдельным атрибутам внутри класса.
3. **Метаклассы** определяют саму природу и процесс создания классов.

При этом они часто работают вместе: метакласс может использовать дескрипторы для создания «умных» атрибутов в новом
классе, а декоратор — применять к методам этого класса. Понимание этой экосистемы открывает путь к созданию
выразительных, эффективных и элегантных абстракций, которые делают Python таким мощным языком для построения сложных
фреймворков.

## **Senior Level**

Метапрограммирование в CPython реализовано через механизмы создания классов на уровне объектов и типов, без специальных
опкодов и только с использованием структур C и стандартных методов классов.

### Создание классов и метаклассы

- При создании класса оператором `class` в байткоде происходит вызов метакласса (обычно `type`), который отвечает за
  создание объекта типа.
- Метаклассы — это объекты, которые создают новые классы, используя методы `__prepare__`, `__new__`, и `__init__`.
- При создании класса метакласс:
    - Вызывает `__prepare__` для подготовки namespace (обычно dict, но может быть и другой mapping для отслеживания
      порядка объявлений).
    - Выполняет тело класса в подготовленном namespace.
    - Вызывает `__new__` для создания объекта класса.
    - Вызывает `__init__` для инициализации атрибутов и методов класса.

### Подробности реализации

- Метакласс может переопределять `__prepare__` для контроля namespace, например, использовать `OrderedDict` для
  сохранения порядка объявления.
- Метод `__new__` принимает имя класса, базы и namespace, создает объект типа (`PyTypeObject`) и может изменять его
  структуру, атрибуты, MRO и слоты.
- Метод `__init__` используется для инициализации атрибутов и методов созданного класса.
- В процессе создания класса вызывается `PyType_Ready` для инициализации слотов и методов.

### __init_subclass__

- `__init_subclass__` — специальный метод класса, который вызывается при создании подкласса.
- При создании подкласса метакласс:
    - Создает объект подкласса.
    - Вызывает `__init_subclass__` родительского класса, передавая ему объект подкласса и дополнительные
      аргументы.
- Это позволяет настраивать подклассы, добавлять атрибуты или регистрировать классы без необходимости в сложных
  метаклассах.

### Пример реализации

```python
class CustomMeta(type):
    @classmethod
    def __prepare__(metacls, name, bases):
        return OrderedDict()

    def __new__(cls, name, bases, namespace):
        namespace['custom_attr'] = 'custom_value'
        return super().__new__(cls, name, bases, namespace)

    def __init__(cls, name, bases, namespace):
        cls.custom_init = 'initialized'


class MyClass(metaclass=CustomMeta):
    pass


class SubClass(MyClass):
    pass
```

### Под капотом: CPython

- Все классы и метаклассы реализованы через структуры `PyObject`, `PyTypeObject` и механизмы MRO.
- При создании класса метакласс использует `__prepare__`, `__new__`, и `__init__` для контроля создания и инициализации
  класса.
- `__init_subclass__` позволяет настраивать подклассы без сложных метаклассов, предоставляя простой способ контроля
  создания подклассов.

Метапрограммирование в CPython предоставляет мощные инструменты для управления созданием и поведением классов, что
делает Python очень гибким для разработки сложных систем.

- [Содержание](#содержание)

---

# **Миксины**

## **Junior Level**

Миксины (mixins) — это специальные классы в Python, предназначенные для добавления конкретной функциональности к другим
классам через множественное наследование. Если представить основной класс как основное блюдо, то миксины — это специи
или добавки, которые придают ему дополнительные свойства и возможности.

Миксины не предназначены для самостоятельного использования — они не являются полноценными объектами, а служат для "
подмешивания" методов и атрибутов к другим классам. Например, можно создать миксин `LoggingMixin`, который добавляет
методы логирования, и использовать его в разных классах, где нужна эта функциональность.

Для QA инженера миксины полезны при создании тестовых фреймворков и утилит: можно вынести общую функциональность (
например, работу с базой данных, генерацию тестовых данных, создание отчетов) в миксины и переиспользовать их в разных
тестовых классах.

## **Middle Level**

Технически миксины — это обычные классы Python, которые следуют определенным соглашениям при использовании в
множественном наследовании.

1. **Синтаксис и использование:**
    - Миксины включаются в цепочку наследования перед основным классом.
    - Обычно имеют суффикс `Mixin` в названии для ясности.
    - Не вызывают `super().__init__()` в своем `__init__`, если не предназначены для участия в MRO (Method Resolution
      Order) инициализации.

2. **Метод разрешения порядка (MRO):**
    - При множественном наследовании Python использует алгоритм C3 для определения порядка поиска методов.
    - Миксины должны быть спроектированы так, чтобы не конфликтовать с методами основных классов и других миксинов.
    - Важно правильно располагать миксины в списке наследования: обычно миксины идут слева перед основными классами.

3. **Особенности проектирования:**
    - **Одна ответственность:** Каждый миксин должен добавлять одну четкую функциональность.
    - **Независимость:** Миксины должны быть максимально независимы от деталей реализации классов, к которым они
      подмешиваются.
    - **Гибкость:** Миксины могут определять абстрактные методы (`@abstractmethod`), которые должны быть реализованы в
      основном классе.

4. **Для AQA:**
    - **Тестовые утилиты:** Создание миксинов для общих операций: `DatabaseMixin` (для работы с БД), `APIClientMixin` (
      для HTTP-запросов), `ScreenshotMixin` (для создания скриншотов при падении теста).
    - **Расширение Page Object:** Добавление функциональности к Page Object через миксины (например, `ModalDialogMixin`
      для работы с модальными окнами).
    - **Кастомизация тестовых классов:** В `unittest.TestCase` можно использовать миксины для добавления методов
      подготовки данных, ассертов.

5. **Примеры в экосистеме Python:**
    - Django использует миксины для CBV (Class-Based Views).
    - DRF (Django REST Framework) активно использует миксины для ViewSets.
    - В тестировании: `unittest.TestCase` можно расширять миксинами.

## **Senior Level**

1. **Миксины и линейная MRO:**
    - Алгоритм C3 гарантирует сохранение порядка и отсутствие циклов в наследовании.
    - При использовании нескольких миксинов их порядок в списке наследования критичен. Методы миксина, стоящего левее,
      имеют приоритет.
    - **Проблема diamond inheritance:** Если два миксина наследуются от одного базового класса, может возникнуть
      неочевидное поведение. C3 решает эту проблему, но важно понимать итоговый MRO.

2. **Миксины как протоколы дескрипторов:**
    - Миксины могут использовать дескрипторы для управления доступом к атрибутам. Например, миксин может добавлять
      property или управляемые атрибуты.
    - Дескрипторы в миксинах могут быть использованы для автоматической валидации данных, ленивой загрузки или
      кэширования.

3. **Динамическое подмешивание (монкей-патчинг vs наследование):**
    - В отличие от статического наследования, миксины можно "подмешивать" динамически с помощью `type()` с тремя
      аргументами:
      ```python
      DynamicClass = type('DynamicClass', (ExistingClass, SomeMixin), {})
      ```
    - Это позволяет создавать классы на лету с нужной комбинацией функциональности, что полезно в тестовых фреймворках
      для создания специализированных тестовых классов.
    - Однако такой подход усложняет отладку и анализ кода.

4. **Миксины и метаклассы:**
    - Миксины могут иметь свои метаклассы, что может привести к конфликту метаклассов при множественном наследовании.
    - Python разрешает конфликты, требуя чтобы один метакласс был подклассом другого. Если миксин использует метакласс,
      нужно убедиться в его совместимости.
    - Метаклассы миксинов могут использоваться для автоматической регистрации классов, добавления декораторов или
      изменения структуры класса.

5. **Для AQA:**
    - **Динамическая композиция тестовых классов:** В сложных тестовых фреймворках можно использовать фабрику классов,
      которая на основе конфигурации (например, тестируемой среды, типа браузера, нужных фич) динамически создает
      тестовый класс, подмешивая соответствующие миксины:
        - `SeleniumMixin` для UI-тестов.
        - `APITestMixin` для API-тестов.
        - `MobileTestingMixin` для мобильного тестирования.
        - `PerformanceMetricsMixin` для сбора метрик.
          Это позволяет избежать взрыва комбинаторного наследования.

    - **Миксины с зависимостями и инициализацией:** Создание миксинов, которые требуют определенной последовательности
      инициализации. Использование `__init_subclass__` для настройки класса при создании. Например, миксин может
      автоматически добавлять декораторы к методам класса, начинающимся с `test_`.

    - **Тестирование самих миксинов:** Написание тестов для миксинов — отдельная задача, так как они не предназначены
      для использования в изоляции. Стратегии:
        - Создание тестового класса-заглушки, который использует миксин.
        - Использование метаклассов для автоматического тестирования всех комбинаций миксинов.
        - Проверка, что миксин не ломает MRO основного класса.

    - **Миксины для cross-cutting concerns в тестах:**
        - `RetryMixin`: автоматически повторяет падающий тест (опасно, может маскировать проблемы).
        - `TimeoutMixin`: добавляет таймауты к тестам.
        - `IsolationMixin`: обеспечивает изоляцию тестов (создание уникальных данных, очистка состояния).
          Эти миксины могут переопределять методы выполнения тестов (`setUp`, `tearDown`, `runTest`), что требует
          глубокого понимания механизмов выполнения тестов во фреймворке.

    - **Конфликты и разрешение методов:** При использовании нескольких миксинов, определяющих методы с одинаковыми
      именами, можно реализовать механизм разрешения конфликтов:
        - Использование `super()` с явным указанием класса (но это ломает гибкость).
        - Создание миксина-медиатора, который координирует вызовы.
        - Использование декораторов для "сшивания" методов (например, вызывать оба метода).

    - **Миксины для интеграции с внешними системами:**
        - `AllureReportingMixin`: автоматически добавляет шаги и аттачменты в Allure-отчет.
        - `JiraIntegrationMixin`: создает баг-репорты в JIRA при падении теста.
        - `SlackNotificationMixin`: отправляет уведомления в Slack о начале/конце тестового прогона.
          Такие миксины часто требуют настройки (учетные данные, URL), что можно реализовать через атрибуты класса или
          конфигурационные объекты.

6. **Альтернативы миксинам в современном Python:**
    - **Композиция через классы-делегаты:** Вместо множественного наследования можно использовать композицию и
      делегирование. Это делает зависимости более явными, но требует больше кода.
    - **Протоколы и ABC:** Для определения интерфейсов лучше использовать абстрактные базовые классы или протоколы, а
      реализацию — через композицию.
    - **Декораторы классов:** Многие задачи, решаемые миксинами, можно реализовать через декораторы классов, которые
      модифицируют поведение.
    - **Хуки и плагины:** В pytest функциональность добавляется через фикстуры и плагины, что является более
      декларативным и гибким подходом, чем миксины.

7. **Опасности и антипаттерны:**
    - **God Mixin:** Миксин, который делает слишком много и становится монолитом.
    - **Неявные зависимости:** Миксин, который зависит от определенных атрибутов или методов в классе, но не декларирует
      это явно.
    - **Хрупкость MRO:** Изменение порядка миксинов может сломать работу класса.
    - **Тестируемость:** Классы с большим количеством миксинов сложно тестировать, так как приходится учитывать все
      подмешанные поведения.

- [Содержание](#содержание)

---

# **typing**

## **Junior Level*

`typing` модуль в Python предоставляет инструменты для добавления подсказок типов (type hints) в код. Это не меняет
поведение программы во время выполнения, но помогает разработчикам, IDE и статическим анализаторам понимать, какие типы
данных ожидаются.

**Optional[X]** означает, что значение может быть либо типа `X`, либо `None`. Это удобный способ сказать "этот аргумент
может быть передан, а может и нет".

**Union[X, Y, ...]** означает, что значение может быть одного из перечисленных типов. Например, `Union[int, str]` — это
либо целое число, либо строка.

**TypeVar** используется для создания обобщенных (generic) типов. Например, если у вас есть функция, которая работает со
списками любого типа, вы можете использовать `TypeVar` чтобы показать, что тип элементов входного списка и выходного
значения одинаков.

**Generic** — это способ создавать классы или функции, которые могут работать с разными типами, но сохранять информацию
о конкретном типе. Например, `List[int]` — это список целых чисел, а `List[str]` — список строк. `Generic` позволяет вам
создавать свои собственные классы, которые могут быть параметризованы типами.

## **Middle Level**

Технически, эти конструкции являются частью системы типизации, которая реализована в модуле `typing` и поддерживается
статическими анализаторами (mypy, pyright, PyCharm).

1. **Optional и Union:**
    - `Optional[X]` это просто сокращение для `Union[X, None]`.
    - `Union` может использоваться для любого количества типов. В Python 3.10 появился синтаксис `X | Y` как
      альтернатива `Union[X, Y]`.
    - Важно: `Optional` и `Union` не выполняют проверку типов во время выполнения. Они только для статического анализа.
    - Для проверки в runtime можно использовать `isinstance()` с кортежем типов, но это не связано напрямую с
      аннотациями.

2. **TypeVar:**
    - Создается вызовом `TypeVar(name, *constraints, bound=None, covariant=False, contravariant=False)`.
    - **Ограничения (constraints):** TypeVar может быть ограничен конкретными типами. Тогда он может представлять только
      один из них.
    - **Связывание (bound):** TypeVar может быть связан с базовым классом или протоколом. Тогда он представляет любой
      подтип этого базового класса.
    - **Ковариантность/контравариантность:** Позволяют выразить отношения между производными типами. Например, если `C`
      ковариантен по `T`, то `C[Dog]` является подтипом `C[Animal]` (при условии, что `Dog` подтип `Animal`). Это важно
      для корректного определения подтипов в обобщенных классах.

3. **Generic:**
    - Класс, наследующийся от `Generic[T]`, где `T` — это TypeVar, становится обобщенным.
    - Можно использовать несколько TypeVar: `Generic[T, U]`.
    - Внутри класса можно использовать `T` как обычный тип в аннотациях методов и атрибутов.
    - При наследовании от обобщенного класса можно либо конкретизировать тип (`ChildClass[int]`), либо передать TypeVar
      дальше.

4. **Для AQA:**
    - Type hints помогают документировать интерфейсы тестовых утилит и фикстур.
    - `Optional` часто используется для необязательных аргументов в функциях-фикстурах.
    - `Union` полезен, когда функция может возвращать разные типы данных в зависимости от условий (например,
      `Union[WebElement, None]` при поиске элемента).
    - `Generic` и `TypeVar` позволяют создавать гибкие, переиспользуемые компоненты тестовых фреймворков, например,
      абстрактный репозиторий для тестовых данных `Repository[T]`, где `T` — тип модели.

## **Senior Level**

1. **Optional и Union как алгебраические типы:**
    - В теории типов `Union` является суммой типов (sum type), а кортеж — произведением типов (product type). Это основа
      для построения сложных типов данных.
    - `Optional` — это частный случай суммы типа с единичным типом (`None`).
    - В Python 3.10 появился `TypeGuard` и `isinstance` с `Union` теперь работает лучше, но все еще есть ограничения.

2. **TypeVar и вариативность:**
    - **Инвариантность по умолчанию:** `List[T]` инвариантен: `List[Dog]` не является ни подтипом, ни надтипом
      `List[Animal]`, потому что список можно изменять. Это предотвращает ошибки типа "добавление кота в список собак".
    - **Ковариантность:** `Sequence[T]` (как и `Iterable[T]`) может быть ковариантен, потому что он только читает
      данные. Если `Dog` подтип `Animal`, то `Sequence[Dog]` подтип `Sequence[Animal]`.
    - **Контравариантность:** `Callable[[Animal], ...]` может быть контравариантен по аргументам. Функция, которая
      принимает `Animal`, может принимать и `Dog`, поэтому `Callable[[Animal], ...]` является подтипом
      `Callable[[Dog], ...]` (обратное отношение).
    - Правильное указание вариативности в пользовательских дженериках критично для корректной работы статических
      анализаторов.

3. **Generic и `__class_getitem__`:**
    - Когда вы пишете `List[int]`, это не создает новый класс. Вместо этого вызывается метод `__class_getitem__` класса
      `List`, который возвращает объект-алиас с сохраненной информацией о параметре типа.
    - Пользовательские классы могут реализовать `__class_getitem__` для поддержки такой нотации, не наследуясь от
      `Generic`. Но наследование от `Generic` автоматически предоставляет эту реализацию и другие возможности.
    - `Generic` использует метаклассы и магию, чтобы сохранить параметры типов в `__parameters__` и `__args__`.

4. **TypeVar и `NewType`:**
    - `NewType` создает непрозрачный псевдоним типа, который статически считается подтипом исходного типа, но в runtime
      это отдельный тип. Полезно для предотвращения логических ошибок (например, `UserId = NewType('UserId', int)`).
    - `TypeVar` же используется для параметризации, а не создания новых типов.

5. **Для AQA:**
    - **Создание типобезопасных DSL для тестов:** Используя `Generic` и `TypeVar`, можно создать DSL, который статически
      проверяет цепочки вызовов. Например, `browser.element(by.id("login")).click()` — если `element` возвращает
      `WebElement[T]`, где `T` — это тип элемента (кнопка, поле ввода), то можно гарантировать, что методы `click()` или
      `send_keys()` доступны только для соответствующих типов.
    - **Генерация тестов на основе типов:** Метакласс может анализировать аннотации типов тестовых методов и
      автоматически генерировать тесты для разных типов данных. Например, для параметризованного теста
      `test_sort[T: (int, str)]` можно сгенерировать два теста.
    - **Валидация конфигураций тестов:** Используя `TypedDict` и `Literal`, можно статически проверять конфигурационные
      файлы тестов (YAML/JSON) на соответствие схеме.
    - **Мокирование с проверкой типов:** Создание моков, которые наследуются от `Generic[T]` и проверяют, что
      возвращаемые значения соответствуют аннотациям. Можно использовать `__annotations__` реального класса для
      автоматической настройки мока.
    - **Протоколы и дженерики:** Комбинирование `Protocol` и `Generic` для описания обобщенных интерфейсов. Например,
      `class Repository[T, U]: ...` — где `T` — тип модели, `U` — тип идентификатора. Это позволяет создавать
      типобезопасные абстракции для тестовых данных.
    - **Зависимые типы (эмуляция):** Хотя Python не поддерживает зависимые типы полноценно, можно использовать `Literal`
      и `TypeVar` с bound для эмуляции простых случаев. Например, функция, которая возвращает длину списка, может быть
      аннотирована так, чтобы статический анализатор понимал, что индекс не выйдет за границы.
    - **Проблемы с рекурсивными типами:** Определение рекурсивных структур данных (например, дерева) требует
      использования строковых аннотаций (`'Tree'`) или `from __future__ import annotations`. В тестовых фреймворках это
      может возникать при описании сложных JSON-схем.
    - **Производительность:** Использование `typing` модуля может замедлить запуск программы, потому что он содержит
      много сложной логики. В продакшн-коде иногда используют `from typing import TYPE_CHECKING` для условного импорта.
      В тестах это менее критично, но стоит помнить.

6. **Ограничения и будущее:**
    - **PEP 563 (Postponed Evaluation of Annotations):** Строковые аннотации по умолчанию с Python 3.10 (в 3.7—3.9 через
      `from __future__ import annotations`). Это решает проблему forward references, но делает аннотации недоступными
      для интроспекции в runtime (нужно использовать `get_type_hints`).
    - **PEP 646 (Variadic Generics):** В Python 3.11 добавлены `TypeVarTuple` и `Unpack` для работы с произвольным
      количеством типов (например, для многомерных массивов).
    - **PEP 675 (LiteralString):** Для более точной типизации строковых литералов, полезно в SQL-запросах или командах
      shell, чтобы предотвранить injection.
    - **PEP 655 (Required/NotRequired для TypedDict):** Для указания обязательных и необязательных ключей.

- [Содержание](#содержание)

---

# **Literal, TypedDict, Protocol**

## **Junior Level**

**Literal** позволяет указать конкретное значение, которое может принимать переменная. Например,
`Literal["GET", "POST"]` означает, что параметр может быть только строкой "GET" или "POST". Это полезно для ограничения
допустимых значений.

**TypedDict** позволяет описывать структуру словаря с конкретными ключами и типами их значений. Это как схема для
словаря. Например, можно описать тип пользователя: `{"name": str, "age": int}`. Статические анализаторы будут проверять,
что словарь имеет именно такую структуру.

**Protocol** (о котором уже говорили) позволяет описывать интерфейсы через набор методов и атрибутов, которые должны
быть у объекта. Объект считается соответствующим протоколу, если у него есть все указанные методы и атрибуты, даже если
он не наследуется от протокола явно.

Для QA инженера эти инструменты помогают четко описывать форматы тестовых данных, конфигураций и ожидаемых структур
ответов, а также создавать типобезопасные моки и заглушки.

## **Middle Level**

Рассмотрим технические особенности каждой конструкции:

1. **Literal:**
    - Введен в PEP 586 (Python 3.8).
    - Может содержать конкретные значения: строки, числа, булевы значения, `None`.
    - Часто используется с `Union` для ограничения допустимых значений параметра.
    - Пример: `def request(method: Literal["GET", "POST", "PUT"]) -> Response: ...`
    - Статические анализаторы проверяют, что передаваемые значения входят в указанный набор.

2. **TypedDict:**
    - Введен в PEP 589 (Python 3.8).
    - Определяется через наследование от `TypedDict` или с использованием синтаксиса `TypedDict()`.
    - Поддерживает обязательные и необязательные ключи (через `total=False` и `NotRequired` в Python 3.11+).
    - Пример:
      ```python
      class User(TypedDict):
          name: str
          age: int
          email: NotRequired[str]  # Python 3.11+
      ```
    - В рантайме `TypedDict` ведет себя как обычный `dict`, но статические анализаторы используют его для проверки
      структуры.

3. **Protocol:**
    - Введен в PEP 544 (Python 3.8).
    - Определяет интерфейс через аннотации методов и атрибутов.
    - Может быть параметризован с помощью `Generic`.
    - Декоратор `@runtime_checkable` позволяет использовать `isinstance()` для проверки соответствия протоколу (но
      проверяет только наличие атрибутов, не их типы).
    - Пример:
      ```python
      class SupportsClose(Protocol):
          def close(self) -> None: ...
      ```

4. **Для AQA:**
    - **Literal:** Типизация параметров тестов (например, имена браузеров, окружения).
    - **TypedDict:** Описание форматов JSON-ответов API, конфигураций тестового окружения, фикстур.
    - **Protocol:** Определение интерфейсов для зависимостей, которые нужно мокировать. Создание абстракций для тестовых
      утилит.

## **Senior Level**

Глубокий анализ внутренней механики и продвинутое применение:

1. **Literal и зависимые типы:**
    - `Literal` может использоваться для уточнения типов в зависимости от других значений (эмуляция зависимых типов).
    - В сочетании с `@overload` и `TypeGuard` позволяет создавать точные аннотации для функций с ветвлением.
    - **Проблема:** `Literal` значения не считаются подтипами базового типа (`Literal["GET"]` не подтип `str`) для
      статического анализатора, что может привести к неожиданностям.
    - **Для AQA:** Использование `Literal` для параметризации тестов гарантирует, что тестовые значения принадлежат
      допустимому набору. Можно создавать перечисления тестовых сценариев.

2. **TypedDict и динамическая валидация:**
    - `TypedDict` использует метаклассы для создания классов-схем.
    - Атрибуты `__required_keys__` и `__optional_keys__` доступны для интроспекции.
    - **Проблема:** Наследование `TypedDict` работает, но требует осторожности при переопределении ключей.
    - **Интеграция с runtime-валидацией:** Библиотеки типа `pydantic` или `marshmallow` могут использовать `TypedDict`
      для генерации схем валидации.
    - **Для AQA:** Создание самодокументирующихся и валидируемых конфигураций тестового фреймворка. Автогенерация
      тестовых данных на основе схем.

3. **Protocol и метаклассы:**
    - `Protocol` наследуется от `ABCMeta`, что делает его метаклассом.
    - Декоратор `@runtime_checkable` добавляет метод `__instancecheck__`, который проверяет наличие атрибутов через
      `hasattr()`.
    - **Generic протоколы:** Позволяют создавать параметризованные интерфейсы. Например,
      `class Repository[T](Protocol): ...`.
    - **Проблема:** `@runtime_checkable` не проверяет сигнатуры методов и типы атрибутов. Это ограничение.
    - **Для AQA:** Создание библиотеки типобезопасных моков. Мок, созданный на основе протокола, будет иметь все
      необходимые методы, и IDE сможет предоставлять автодополнение.

4. **Комбинирование конструкций:**
    - Создание сложных типов: `TypedDict`, где некоторые значения имеют тип `Literal`.
    - Протоколы, требующие наличия атрибутов определенного `TypedDict` типа.
    - Пример: `class APIResponse(TypedDict): status: Literal["success", "error"]`.

5. **Ограничения и обходные пути:**
    - **Производительность:** Большое количество `TypedDict` и `Protocol` может замедлить статический анализ.
    - **Динамические ключи:** `TypedDict` не подходит для словарей с динамическими ключами. Для этого есть
      `typing.TypedDict` с `total=False`.
    - **Рекурсивные типы:** Для рекурсивных `TypedDict` или `Protocol` требуются строковые аннотации.

6. **Для AQA:**
    - **Создание DSL для тестов:** Используя `Protocol` и `Literal`, можно построить DSL, который статически проверяет
      корректность цепочек вызовов. Например, `page.form().fill().submit()` — если `form()` возвращает объект,
      соответствующий протоколу `Form`, то доступны только методы `fill()` и `submit()`.
    - **Генерация тестов из OpenAPI/Swagger:** На основе схемы API (которая сама по себе `TypedDict`-подобна) можно
      генерировать типизированные клиенты и тесты.
    - **Валидация тестовых сценариев:** Статическая проверка, что тестовые сценарии (описанные как `TypedDict`)
      соответствуют ожидаемой структуре. Например, сценарий теста UI:
      `{"action": Literal["click", "fill"], "element": str, ...}`.
    - **Мокирование внешних сервисов:** Создание моков, которые точно соответствуют протоколу внешнего сервиса.
      Использование `TypedDict` для описания форматов запросов и ответов.
    - **Статический анализ тестовых данных:** Написание скриптов, которые с помощью `TypedDict` проверяют, что тестовые
      данные (JSON, YAML) соответствуют ожидаемой структуре. Интеграция этой проверки в CI/CD.
    - **Протоколы для плагинов:** Описание интерфейсов плагинов для тестового фреймворка через `Protocol`. Это позволяет
      сторонним плагинам быть типобезопасными.
    - **Селекторы и локаторы:** Использование `Literal` для типизации селекторов: `by: Literal["id", "xpath", "css"]`.

7. **Будущее:**
    - **PEP 655 (NotRequired/Required):** Улучшает работу с необязательными ключами в `TypedDict`.
    - **PEP 681 (Data Class Transforms):** Упрощает создание классов, похожих на `TypedDict`.
    - **PEP 692 (TypedDict для **kwargs):** Позволит типизировать `**kwargs` с помощью `TypedDict`.

- [Содержание](#содержание)

---

# **Ковариантность, контравариантность**

## **Junior Level*

Ковариантность и контравариантность — это понятия из теории типов, которые описывают, как отношения между типами (
например, "является подтипом") переносятся на более сложные типы, содержащие эти типы (например, на списки или функции).

Представьте, что у вас есть иерархия классов: `Animal` -> `Dog` (собака — это животное). Теперь рассмотрим контейнеры:

- **Ковариантность** означает, что если `Dog` является подтипом `Animal`, то `Container[Dog]` является подтипом
  `Container[Animal]`. То есть контейнер сохраняет направление отношения. Это безопасно, если контейнер только читает
  данные (например, итератор).
- **Контравариантность** означает обратное: если `Dog` является подтипом `Animal`, то `Container[Animal]` является
  подтипом `Container[Dog]`. То есть контейнер инвертирует отношение. Это имеет смысл для контейнеров, которые только
  записывают данные (например, функция, которая принимает аргументы).

Простой пример: если у вас есть функция, которая может принять любое `Animal`, то она может принять и `Dog` (
контравариантность по аргументу). Если функция возвращает `Dog`, то эта возвращаемая собака также является `Animal` (
ковариантность по возвращаемому значению).

## **Middle Level**

Технически, ковариантность и контравариантность определяются в параметризованных (generic) типах. В Python они задаются
с помощью `TypeVar` с соответствующими флагами.

1. **Инвариантность (по умолчанию):**
    - Если `TypeVar('T')` объявлен без указания `covariant` или `contravariant`, то `Generic[T]` инвариантен.
    - Это значит, что `Container[Dog]` не является ни подтипом, ни надтипом `Container[Animal]`, даже если `Dog` —
      подтип `Animal`.
    - Пример: `List[T]` инвариантен, потому что список можно и читать, и изменять. Если бы он был ковариантен, то можно
      было бы присвоить `List[Dog]` переменной типа `List[Animal]` и добавить `Cat`, что привело бы к ошибке типа.

2. **Ковариантность:**
    - Объявляется как `TypeVar('T', covariant=True)`.
    - Пример: `Iterable[T]` ковариантен, потому что он только производит (yield) значения типа `T`. Если `Dog` подтип
      `Animal`, то `Iterable[Dog]` можно использовать везде, где требуется `Iterable[Animal]`.

3. **Контравариантность:**
    - Объявляется как `TypeVar('T', contravariant=True)`.
    - Пример: `Callable[[T], ...]` контравариантен по параметру `T`. Функция, которая может обработать любое `Animal`,
      может обработать и `Dog`. Поэтому `Callable[[Animal], ...]` является подтипом `Callable[[Dog], ...]`.

4. **Правила вариативности:**
    - Ковариантные типовые параметры могут использоваться только в "выходных" позициях (например, возвращаемый тип
      метода).
    - Контравариантные типовые параметры могут использоваться только во "входных" позициях (например, аргументы метода).
    - Инвариантные параметры могут использоваться и там, и там.

5. **Для AQA:**
    - При создании собственных generic-классов для тестовых фреймворков (например, `Repository[T]`) важно правильно
      указать вариативность, чтобы обеспечить типобезопасность.
    - Понимание вариативности помогает правильно аннотировать моки и стабы, особенно когда они используются в
      полиморфных контекстах.

## **Senior Level**

1. **Теоретические основы и Liskov Substitution Principle (LSP):**
    - Вариативность — это формальное выражение принципа подстановки Барбары Лисков для параметризованных типов.
    - Ковариантность гарантирует, что если `A` — подтип `B`, то `F[A]` — подтип `F[B]` (при условии, что `F`
      ковариантен).
    - Контравариантность гарантирует, что если `A` — подтип `B`, то `F[B]` — подтип `F[A]`.
    - Эти правила обеспечивают безопасность типов при использовании полиморфизма.

2. **Система типов Python и вариативность:**
    - Python использует структурную типизацию для протоколов, что делает вариативность еще более важной.
    - Например, протокол `SupportsLessThan[T]` может быть ковариантен по `T`, потому что если `Dog` можно сравнивать, то
      и `Animal` (если `Dog` — подтип `Animal`) можно сравнивать? На самом деле, это не всегда так, и поэтому
      вариативность должна быть определена аккуратно.
    - Статические анализаторы (mypy, pyright) проверяют вариативность и выдают ошибки, если ковариантный параметр
      появляется в контравариантной позиции (и наоборот).

3. **Вариативность и исключения:**
    - В Python исключения ковариантны. Это означает, что `Exception[Dog]` является подтипом `Exception[Animal]`. Это
      безопасно, потому что исключения обычно только читаются (их ловят и обрабатывают).
    - Однако, если бы исключения могли изменяться, это могло бы привести к проблемам. Но поскольку исключения в Python
      неизменяемы (immutable) после создания, ковариантность безопасна.

4. **Практические проблемы и обходные пути:**
    - **Проблема с изменяемыми данными:** Изменяемые контейнеры не могут быть ковариантными или контравариантными,
      потому что это привело бы к нарушению типобезопасности. Поэтому `list` инвариантен.
    - **Обходной путь:** Использование неизменяемых типов, таких как `Sequence` (ковариантен) или `Mapping` (ковариантен
      по значениям, но инвариантен по ключам).
    - **Self-тип:** Паттерм `Self` (введенный в Python 3.11) позволяет методам возвращать экземпляр текущего класса, что
      полезно для fluent-интерфейсов. `Self` является ковариантным.

5. **Для AQA:**
    - **Создание типобезопасных API для тестов:** При проектировании фреймворка для тестирования API можно использовать
      ковариантность для представления иерархий ответов. Например, `APIResponse[Success]` и `APIResponse[Error]` могут
      быть подтипами `APIResponse[Any]`. Это позволяет писать общие обработчики ответов.
    - **Мокирование с вариативностью:** При создании моков для generic-интерфейсов важно учитывать вариативность.
      Например, если интерфейс `Repository[T]` инвариантен, то мок для `Repository[Dog]` не может быть использован там,
      где ожидается `Repository[Animal]`, даже если `Dog` — подтип `Animal`. Это может потребовать создания отдельных
      моков для каждого типа.
    - **Параметризация тестов:** Используя вариативность, можно создавать параметризованные тесты, которые безопасно
      работают с иерархиями типов. Например, тест для `Animal` должен работать и для `Dog`, если только тест не изменяет
      объект (в этом случае требуется инвариантность).
    - **Статическая проверка тестовых сценариев:** С помощью правильной вариативности в аннотациях можно заставить
      статический анализатор проверять, что тестовые данные соответствуют ожидаемым типам, даже при использовании
      полиморфизма.
    - **Проблема с двойной вариативностью:** Некоторые структуры данных, такие как `Comparable`, могут требовать, чтобы
      тип был одновременно и ковариантным, и контравариантным. Это невозможно в большинстве систем типов. В Python это
      решается через протоколы с `Self` или через перегрузку (overloading).
    - **Использование `typing.covariant_check` и `contravariant_check`:** Эти декораторы (гипотетические) могли бы
      использоваться для проверки вариативности в runtime, но в стандартной библиотеке их нет. Однако можно создать свои
      проверки, используя `isinstance` и `issubclass`.

6. **Ограничения и будущее:**
    - **Производительность:** Проверка вариативности во время статического анализа может быть сложной и замедлять
      анализ.
    - **Динамическая природа Python:** В runtime информация о типах стирается (type erasure), поэтому вариативность
      важна только для статической проверки.
    - **PEP 484 и последующие:** Постепенное улучшение поддержки вариативности в системе типов Python.

- [Содержание](#содержание)

---

# **Pytest**

## **Junior Level*

Pytest — это современный фреймворк для тестирования в Python, который делает написание и выполнение тестов простым,
интуитивным и эффективным. В отличие от стандартного модуля unittest, pytest требует меньше шаблонного кода и предлагает
более мощные возможности.

Ключевые преимущества:

1. **Простой синтаксис:** Тесты пишутся как обычные функции с префиксом `test_`, а проверки — с помощью оператора
   `assert`. Не нужно запоминать множество методов вроде `assertEqual`, `assertTrue`.
2. **Фикстуры (Fixtures):** Механизм для подготовки и очистки тестового окружения. Фикстуры объявляются декоратором
   `@pytest.fixture` и могут использоваться в тестах путем указания их имен в параметрах функции.
3. **Параметризация:** Легко запустить один тест с разными наборами данных с помощью декоратора
   `@pytest.mark.parametrize`.
4. **Плагины:** Богатая экосистема плагинов расширяет возможности pytest (например, `pytest-cov` для измерения покрытия,
   `pytest-xdist` для параллельного запуска).
5. **Подробные отчеты:** При падении теста pytest предоставляет детальную информацию, что упрощает отладку.

## **Middle Level**

Технически pytest — это сложная система, построенная вокруг нескольких ключевых концепций:

1. **Архитектура запуска:**
    - Основная точка входа — функция `pytest.main()`, которая вызывает внутренний механизм, собирающий тесты,
      выполняющий их и формирующий отчет.
    - Процесс включает **обнаружение тестов** (сканирование файлов и каталогов по соглашениям), **сбор тестов** (
      создание объектов `Item` для каждого теста) и **выполнение** (запуск тестов с учетом фикстур и хуков).

2. **Фикстуры (Fixtures):**
    - Это функции, помеченные декоратором `@pytest.fixture`. Они могут возвращать данные или объекты, которые затем
      передаются в тестовые функции.
    - **Области видимости (scope):** Фикстуры могут быть уровня функции, класса, модуля или сессии, что определяет, как
      часто они создаются и уничтожаются.
    - **Зависимости фикстур:** Фикстуры могут зависеть от других фикстур, образуя граф зависимостей, который pytest
      разрешает автоматически.
    - **Finalizer и yield:** Фикстуры могут использовать `yield` для разделения кода установки и очистки (альтернатива
      `addfinalizer`).

3. **Хуковая система (Hook System):**
    - Pytest построен на плагинах, которые могут перехватывать и изменять его поведение через **хуки**. Хуки определены
      в `pytest.hookspec` и реализуются плагинами.
    - Примеры хуков: `pytest_collection_modifyitems` (для изменения списка тестов), `pytest_runtest_setup` (выполняется
      перед каждым тестом).
    - Эта система позволяет глубоко кастомизировать процесс тестирования.

4. **Маркеры (Markers):**
    - Декоратор `@pytest.mark` позволяет помечать тесты для выборочного запуска (например, `@pytest.mark.slow`).
    - Можно создавать собственные маркеры и использовать их для фильтрации или передачи метаданных.

5. **Параметризация:**
    - `@pytest.mark.parametrize` создает несколько тестовых случаев из одной функции. Каждый набор параметров становится
      отдельным тестовым случаем в отчете.

6. **Для AQA:**
    - **Интеграция с Allure:** Генерация детальных отчетов через `pytest-allure`.
    - **Параллельный запуск:** Использование `pytest-xdist` для ускорения выполнения тестов.
    - **Мокирование:** Хотя pytest сам не предоставляет моки, он хорошо интегрируется с `unittest.mock`.

## **Senior Level**

Глубокое погружение во внутреннюю механику pytest, расширяемость и интеграция с экосистемой Python.

1. **Архитектура на основе плагинов:**
    - Сам pytest реализован как набор **встроенных плагинов**. Даже базовые функции, такие как сбор тестов и выполнение,
      являются плагинами.
    - **Загрузка плагинов:** Плагины могут быть зарегистрированы через `setup.cfg`, `pyproject.toml`, аргументы
      командной строки или автоматически обнаружены через setuptools entry points.
    - **Конфликты плагинов:** При наличии нескольких плагинов, переопределяющих одни и те же хуки, порядок их выполнения
      определяется порядком загрузки и может быть критичным.

2. **Механизм фикстур: как работает внедрение зависимостей:**
    - **Разрешение зависимостей:** При обнаружении тестовой функции pytest анализирует ее сигнатуру и ищет имена
      параметров среди зарегистрированных фикстур. Затем он строит **граф зависимостей** и определяет порядок создания
      фикстур, учитывая их области видимости.
    - **Кэширование фикстур:** Фикстуры с областью видимости выше, чем `function`, кэшируются. Например, фикстура
      `session` создается один раз и переиспользуется. Это реализовано через объект `FixtureCache`.
    - **Время жизни фикстур и очистка:** При использовании `yield` в фикстуре, код после `yield` выполняется в момент
      окончания области видимости. Это не просто генератор, а специальная обработка внутри pytest, которая гарантирует
      выполнение финализатора даже при падении теста.

3. **Сбор тестов и интроспекция:**
    - **Объекты Node:** Pytest представляет тестовую сессию как дерево объектов: `Session` -> `Collection` -> `Item` (
      тест). Каждый узел соответствует каталогу, файлу, классу или функции.
    - **Плагин `pytest_pycollect`:** Отвечает за сбор тестов из Python-модулей. Он использует интроспекцию для поиска
      функций и классов, соответствующих шаблонам.
    - **Кастомизация сбора:** Можно написать плагин, который изменяет процесс сбора, например, добавляет тесты из
      не-Python файлов или динамически генерирует тесты на основе внешних данных.

4. **Хуковая система и взаимодействие с event loop:**
    - **Асинхронная поддержка:** Pytest поддерживает асинхронные тесты через плагин `pytest-asyncio`. Он предоставляет
      фикстуру `event_loop` и маркер `@pytest.mark.asyncio`.
    - **Внутренности:** Асинхронные фикстуры и тесты требуют особой обработки, так как хуки pytest сами не являются
      асинхронными. Плагин `pytest-asyncio` использует свой собственный цикл событий и оборачивает асинхронные функции.

5. **Для AQA:**
    - **Динамическое создание фикстур:** В продвинутых сценариях можно создавать фикстуры программно во время
      выполнения, используя `pytest.fixture` как обычную функцию и регистрируя ее через `request.addfixturedef`. Это
      позволяет создавать фикстуры на основе конфигурации или внешних данных.
    - **Кастомизация отчетов через хуки:** Хуки `pytest_runtest_makereport` и `pytest_terminal_summary` позволяют
      перехватывать результаты тестов и формировать кастомные отчеты, интегрируясь с внешними системами (например,
      TestRail, JIRA).
    - **Распределенное тестирование:** Используя `pytest-xdist`, можно распределять тесты по нескольким worker'ам. При
      этом каждая worker-нода имеет свою копию фикстур сессии, что может привести к избыточному созданию ресурсов. Для
      оптимизации можно использовать фикстуры с областью `"worker"` (только в xdist).
    - **Интеграция с контейнеризацией:** Запуск тестов в Docker-контейнерах с помощью кастомного плагина, который через
      хук `pytest_configure` поднимает инфраструктуру, а через `pytest_unconfigure` — останавливает.
    - **Мокирование на уровне импорта:** Плагины могут использовать `importlib` и `sys.modules` для подмены модулей до
      начала сбора тестов, что полезно для изоляции тестов от внешних зависимостей.
    - **Профилирование тестов:** Написание плагина, который с помощью `cProfile` или `py-spy` собирает метрики
      производительности для каждого теста и выводит топ медленных тестов.
    - **Валидация тестовых данных:** Использование хука `pytest_collection_modifyitems` для проверки, что все тесты
      имеют определенные маркеры или что параметризованные тесты используют валидные данные.
    - **Селективный запуск на основе изменений:** Плагин, который интегрируется с Git и анализирует diff, чтобы
      запускать только тесты, связанные с измененными файлами.
    - **Работа с базами данных:** Создание фикстур, которые управляют транзакциями БД, откатывают изменения после
      каждого теста (используя `rollback`) или применяют миграции для тестовой базы.

6. **Ограничения и граничные случаи:**
    - **Циклические зависимости фикстур:** Pytest обнаружит цикл и выдаст ошибку. Нужно перепроектировать фикстуры,
      введя третью, или использовать ленивую инициализацию.
    - **Параллелизм и состояние:** При использовании `pytest-xdist` нужно избегать разделяемого изменяемого состояния
      между worker'ами. Фикстуры `session` создаются в каждом worker отдельно.
    - **Моники-патчинг и фикстуры:** Если фикстура делает monkeypatch, то важно убедиться, что патч применяется и
      откатывается в правильной области видимости. Использование встроенной фикстуры `monkeypatch` решает эту проблему.
    - **Наследование фикстур в классах:** Фикстуры, определенные в родительском классе, не автоматически доступны в
      дочерних. Нужно явно объявить их или использовать миксины.

- [Содержание](#содержание)

---

# **Pytest hooks**

## **Junior Level*

Pytest hooks (хуки) — это специальные функции, которые позволяют расширять и кастомизировать поведение pytest на разных
этапах выполнения тестов. Если представить pytest как кинотеатр, то хуки — это моменты, когда можно вставить свою
рекламу или изменить сценарий: перед началом сеанса, во время показа или после его завершения.

Хуки позволяют плагинам (включая ваши собственные) вмешиваться в процесс тестирования: изменять список тестов, добавлять
дополнительную обработку перед или после каждого теста, модифицировать отчеты, интегрироваться с внешними системами. Для
QA инженера понимание хуков открывает возможность создания кастомных плагинов для специфичных нужд проекта: интеграция с
системой отчетности, подготовка тестового окружения, сбор дополнительных метрик.

## **Middle Level**

Технически, хуки — это часть архитектуры pytest, построенной на библиотеке `pluggy`. Это система точек расширения, где
каждая точка соответствует определенному этапу жизненного цикла тестов.

1. **Система плагинов и pluggy:**
    - Pytest сам является набором встроенных плагинов, которые регистрируют и используют хуки.
    - `pluggy` — это отдельная библиотека, реализующая механизм «хук-спецификаций» и «хук-имплементаций». Она управляет
      обнаружением, регистрацией и вызовом хуков.

2. **Типы хуков:**
    - **Хуки настройки/завершения:** `pytest_configure`, `pytest_unconfigure`. Вызываются при инициализации и завершении
      сессии.
    - **Хуки сбора тестов:** `pytest_collection_modifyitems`, `pytest_collection_finish`. Позволяют фильтровать,
      переупорядочивать или модифицировать собранные тесты.
    - **Хуки выполнения тестов:** `pytest_runtest_setup`, `pytest_runtest_call`, `pytest_runtest_teardown`. Вызываются
      соответственно перед тестом, во время выполнения теста и после.
    - **Хуки отчетов:** `pytest_runtest_makereport`, `pytest_terminal_summary`. Позволяют создавать кастомные отчеты и
      выводить информацию в терминал.
    - **Хуки вызова:** `pytest_internalerror`, `pytest_keyboard_interrupt`. Обработка внутренних ошибок и прерываний.

3. **Реализация хуков:**
    - Хуки реализуются в плагинах (отдельных модулях или классах) как функции с именами, соответствующими спецификациям.
    - Плагин регистрирует свои хуки автоматически при загрузке (через entry points) или вручную через `pytest.addhooks`.
    - Хуки могут иметь параметры, которые pytest передает в них (например, `session`, `item`, `report`).

4. **Примеры использования для AQA:**
    - **Автоматическая маркировка тестов:** Хук `pytest_collection_modifyitems` может анализировать имена тестов и
      автоматически помечать их как `@pytest.mark.slow` или `@pytest.mark.integration`.
    - **Динамическое добавление тестов:** Хук `pytest_generate_tests` позволяет генерировать параметризованные тесты на
      основе внешних данных.
    - **Кастомная отчетность:** Хук `pytest_runtest_makereport` позволяет добавлять в отчет дополнительную информацию (
      скриншоты, логи, метрики производительности).
    - **Интеграция с внешними системами:** Хуки `pytest_sessionstart` и `pytest_sessionfinish` могут отправлять
      уведомления в Slack, JIRA или обновлять дашборды.

## **Senior Level**

Глубокий анализ системы хуков, их взаимодействия с внутренней архитектурой pytest и продвинутые сценарии использования.

1. **Архитектура pluggy и порядок вызова:**
    - **HookspecMarker и HookimplMarker:** `pluggy` использует декораторы `@hookspec` для объявления спецификации хука (
      что он делает, какие параметры принимает) и `@hookimpl` для реализации.
    - **Приоритеты и ordering:** Реализации хуков могут иметь модификаторы `tryfirst=True` или `trylast=True`, чтобы
      управлять порядком вызова среди нескольких плагинов.
    - **Hook wrappers:** Особый тип реализации, оборачивающий вызов других реализаций. Хук-обертка получает генератор,
      который `yield`'ит результат остальных реализаций. Это позволяет выполнить код до и после основного вызова хука (
      аналогично `around` advice в AOP).
    - **Пример:** Хук `pytest_runtest_makereport` является hook wrapper, позволяя плагинам модифицировать отчет до и
      после его создания.

2. **Внутренний объект `pytest` и `config`:**
    - Объект `pytest` (или `config` в хуках) — это центральный реестр, содержащий все зарегистрированные плагины, хуки и
      состояние сессии.
    - Плагины могут добавлять свои атрибуты в `config`, чтобы делиться данными между хуками (например,
      `config.myplugin_data = {}`).
    - `config.hook` — это объект `pluggy.PluginManager`, через который происходит вызов всех хуков.

3. **Хуки и жизненный цикл тестового элемента (Item):**
    - Каждый тест (функция, метод) представляется объектом `Item`. Хуки `pytest_runtest_*` получают этот объект.
    - Внутри `Item` есть `_request` — объект, содержащий контекст выполнения, включая фикстуры.
    - Хуки могут модифицировать `Item` (например, добавлять пользовательские атрибуты через `item.user_properties`),
      которые затем могут быть использованы в отчетах.

4. **Динамическая регистрация хуков:**
    - Плагины могут регистрировать новые хуки во время выполнения через `pluggy.PluginManager.add_hookspecs`. Это
      позволяет создавать плагины, которые расширяют не только поведение, но и интерфейс pytest.
    - Однако, такой подход требует глубокого понимания архитектуры и может привести к конфликтам.

5. **Для AQA:**
    - **Создание DSL через хуки:** Плагин может через хук `pytest_pycollect_makeitem` перехватывать сбор тестов и
      преобразовывать функции с определенными декораторами в кастомные тестовые объекты. Например, можно создать
      синтаксис `@scenario("ID-123")`, который будет превращать функцию в тест-кейс с привязанным ID из TestRail.
    - **Распределенное выполнение тестов:** Плагин, использующий `pytest_collection_modifyitems` для разделения тестов
      на группы (sharding) и `pytest_sessionstart` для координации между несколькими инстансами runner'ов через
      распределенную очередь (Redis, RabbitMQ). Хук `pytest_runtest_protocol` может быть полностью переопределен для
      отправки теста на выполнение в удаленный worker.
    - **Адаптивное тестирование:** Плагин, который в `pytest_runtest_makereport` анализирует результаты и на лету
      изменяет план тестирования (например, при падении smoke-теста, пропускает dependent тесты). Это требует сохранения
      состояния между вызовами хуков через `config` или внешнее хранилище.
    - **Глубокая интеграция с мониторингом:** Хук `pytest_runtest_call` может оборачивать выполнение теста в span
      трассировки (OpenTelemetry), отправляя метрики длительности, а `pytest_terminal_summary` — агрегировать и выводить
      перформанс-дайджест.
    - **Валидация тестового кода:** Плагин, использующий хук `pytest_collect_file` для анализа не только Python-файлов,
      но и конфигурационных (YAML, JSON) на соответствие схеме, и падения сборки тестов при нарушении.
    - **Кастомные фикстуры на уровне плагина:** Хук `pytest_fixture_setup` позволяет перехватывать создание фикстур и
      модифицировать их поведение глобально (например, добавлять автоматическое логирование для всех фикстур с
      определенным маркером).
    - **Интроспекция и отладка:** Плагин для отладки самого pytest, который регистрирует все хуки и логирует их вызовы с
      параметрами. Это помогает понять порядок выполнения и взаимодействие плагинов в сложных конфигурациях.
    - **Динамическое управление ресурсами:** Плагин, который в `pytest_sessionstart` резервирует виртуальную машину в
      облаке, в `pytest_collection_modifyitems` фильтрует тесты, которые могут на ней выполняться, а в
      `pytest_sessionfinish` — освобождает ее. Состояние VM хранится в `config` и доступно в фикстурах через
      `request.config`.
    - **Поддержка новых языков/форматов:** Через хук `pytest_pycollect_makeitem` можно научить pytest собирать тесты из
      файлов, написанных на другом языке (например, Gherkin для BDD), преобразуя их в Python-объекты на лету.

6. **Опасности и тонкости:**
    - **Циклические зависимости и порядок:** Неправильный порядок вызова хуков может привести к неожиданному поведению.
      Например, если плагин A зависит от данных, которые подготавливает плагин B, нужно обеспечить правильный порядок
      через `tryfirst`/`trylast` или явные зависимости плагинов.
    - **Производительность:** Слишком много хуков, особенно тех, которые выполняют тяжелые операции (сетевые запросы,
      анализ файлов), могут сильно замедлить выполнение тестов.
    - **Совместимость:** При обновлении pytest сигнатуры хуков могут меняться. Плагины должны быть готовы к обратной
      совместимости или явно указывать версию pytest, которую они поддерживают.
    - **Отладка:** Ошибки внутри хуков могут быть трудноотлаживаемыми, так как они происходят глубоко внутри механизма
      pytest. Необходимо тщательное логирование и обработка исключений.

7. **Расширенные объекты в хуках:**
    - **`PytestPluginManager`:** Позволяет программно управлять плагинами (включать/выключать).
    - **`Node`:** Базовый класс для `Item`, `Collector`. Хуки часто получают узлы дерева тестов и могут перемещаться по
      нему (родитель, дети).
    - **`CallInfo`:** Объект, содержащий информацию о вызове теста: результат, исключение, длительность. Доступен в
      `pytest_runtest_makereport`.

- [Содержание](#содержание)

---

# **Kubernetes**

## **Junior Level*

Kubernetes (K8s) — это система для автоматизации развертывания, масштабирования и управления контейнеризированными
приложениями. Представьте, что у вас есть много контейнеров (как изолированных пакетов с вашим приложением), и вам нужно
управлять ими на множестве серверов. Kubernetes берет на себя эту задачу: он сам решает, где запускать контейнеры, как
распределять между ними нагрузку, как перезапускать их при сбоях и как обновлять без простоев.

Для QA инженера Kubernetes важен по нескольким причинам:

1. **Тестовые окружения:** Можно быстро создавать изолированные окружения для тестирования, которые точно повторяют
   продакшен.
2. **Масштабирование тестов:** Запускать тысячи тестов параллельно, используя возможности Kubernetes по управлению
   ресурсами.
3. **Инфраструктура для тестов:** Сами тестовые фреймворки и системы отчетности можно развертывать в Kubernetes как
   микросервисы.
4. **Тестирование в реалистичных условиях:** Тестировать приложение в той же среде, где оно будет работать.

## **Middle Level**

С технической точки зрения, Kubernetes состоит из нескольких ключевых компонентов, которые взаимодействуют через API.

1. **Архитектура кластера:**
    - **Control Plane (Master):** Управляющая нода, содержащая API Server, Scheduler, Controller Manager, etcd (
      хранилище конфигурации).
    - **Worker Nodes:** Ноды, на которых запускаются контейнеры. Каждая содержит kubelet (агент), kube-proxy (сетевой
      прокси) и container runtime (например, Docker).

2. **Основные объекты Kubernetes:**
    - **Pod:** Минимальная единица развертывания. Это один или несколько контейнеров, которые разделяют сеть и
      хранилище.
    - **Deployment:** Описывает желаемое состояние приложения и управляет обновлением и откатом версий.
    - **Service:** Абстракция для доступа к группе подов (обычно через балансировку нагрузки).
    - **ConfigMap и Secret:** Для управления конфигурацией и секретами.
    - **Namespace:** Виртуальный кластер внутри физического, для изоляции ресурсов.

3. **Для AQA:**
    - **Тестовые среды:** Использование Namespaces для изоляции тестовых окружений. Можно создать namespace для каждого
      тестового прогона.
    - **Запуск тестов в Pod'ах:** Тесты могут запускаться в отдельных Pod'ах как Job или CronJob. Это позволяет легко
      масштабировать и управлять выполнением тестов.
    - **Доступ к приложению:** Использование Services для доступа к тестируемому приложению, развернутому в кластере.
    - **Конфигурация тестов:** Использование ConfigMaps для передачи конфигурации тестов (например, URL приложения,
      учетные данные).

4. **Инструменты:**
    - **kubectl:** CLI для управления кластером.
    - **Helm:** Менеджер пакетов для Kubernetes, упрощающий развертывание сложных приложений.
    - **Minikube и Kind:** Инструменты для запуска локального кластера Kubernetes на машине разработчика.

## **Senior Level**

На этом уровне мы рассматриваем Kubernetes как сложную распределенную систему, понимаем ее внутренние механизмы и
используем для построения продвинутых тестовых инфраструктур.

1. **Control Plane и распределенный консенсус:**
    - **etcd:** Распределенное key-value хранилище, основанное на Raft consensus алгоритме. Вся конфигурация и состояние
      кластера хранятся здесь. Для тестовых кластеров важно понимать, что etcd может стать узким местом при интенсивной
      нагрузке (много изменений конфигурации, например, частые создания/удаления Pod'ов для тестов).
    - **API Server:** Центральный управляющий компонент, который валидирует и обрабатывает запросы. Все взаимодействие с
      кластером происходит через его REST API. Можно интегрировать тестовые фреймворки напрямую с API Server для
      мониторинга состояния развертываний.

2. **Scheduler и распределение нагрузки:**
    - Scheduler решает, на какой ноде запустить Pod, на основе политик, ресурсов и ограничений. Для тестовых нагрузок
      важно правильно настраивать ресурсы (requests/limits) для Pod'ов с тестами, чтобы не перегружать ноды и не влиять
      на другие приложения.
    - Можно создавать custom schedulers для специализированных сценариев, например, для запуска тестов на нодах с
      определенными характеристиками (GPU, SSD).

3. **Сеть в Kubernetes:**
    - **CNI (Container Network Interface):** Плагины, обеспечивающие сетевое взаимодействие между Pod'ами. Для
      тестирования сетевых политик (Network Policies) важно понимать, какой CNI используется в кластере.
    - **Service Mesh (Istio, Linkerd):** Надстройка над сетью Kubernetes, предоставляющая продвинутые возможности:
      трафик, наблюдение, безопасность. В тестировании можно использовать Service Mesh для управления трафиком во время
      тестов (например, перенаправление части запросов на канареечную версию).

4. **Хранилище:**
    - **Persistent Volumes (PV) и Persistent Volume Claims (PVC):** Механизм предоставления постоянного хранилища
      Pod'ам. Для тестов, которые требуют сохранения состояния (например, тесты базы данных), важно правильно
      настраивать PVC.
    - **Storage Classes:** Позволяют динамически предоставлять хранилище. Можно создать отдельный storage class для
      тестов, который использует быстрые, но недолговечные диски.

5. **Для AQA:**
    - **Динамическое создание тестовых окружений:** Использование операторов Kubernetes (Kubernetes Operators) для
      автоматического развертывания и управления состоянием тестовых окружений. Оператор может отслеживать custom
      resources (CRD) и создавать namespace, deployment, service и другие ресурсы для каждого тестового прогона.
    - **Распределенное выполнение тестов:** Запуск тестов как Job с параллельными Pod'ами. Использование очередей (
      например, RabbitMQ, Kafka) для координации между Pod'ами. Каждый Pod берет задание из очереди, выполняет тест и
      отправляет результат обратно.
    - **Нагрузочное тестирование в K8s:** Развертывание системы нагрузочного тестирования (например, Apache JMeter в
      кластере) как Deployment. Управление количеством Pod'ов-генераторов нагрузки в зависимости от требований теста.
      Использование Horizontal Pod Autoscaler для автоматического масштабирования генераторов нагрузки.
    - **Мониторинг и observability:** Интеграция тестовых прогонов с системой мониторинга кластера (Prometheus,
      Grafana). Сбор метрик не только с тестируемого приложения, но и с самого кластера (использование ресурсов, ошибки
      kubelet). Настройка алертинга на аномалии во время тестов.
    - **Тестирование отказоустойчивости:** Использование chaos engineering инструментов (например, Chaos Mesh, Litmus)
      для внесения сбоев в кластер во время выполнения тестов. Проверка, как приложение и тесты себя ведут при отказе
      ноды, сети или хранилища.
    - **Безопасность (Security Testing):** Использование возможностей Kubernetes для тестирования безопасности: Security
      Context, Pod Security Policies (устарели), Pod Security Standards. Запуск тестов на уязвимости контейнеров (Trivy,
      Clair) непосредственно в CI/CD пайплайне, интегрированном с Kubernetes.
    - **Миграция состояния тестов:** При тестировании stateful приложений (базы данных) использование снапшотов (
      snapshots) Persistent Volumes для быстрого восстановления состояния перед каждым тестом. Настройка Init Containers
      в Pod'ах тестов для подготовки данных.
    - **Интеграция с внешними системами:** Использование Kubernetes API для автоматического создания тестовых данных во
      внешних системах (например, в облачных сервисах AWS, GCP) через механизм External Secrets и External DNS.
    - **Оптимизация затрат:** Настройка node autoscaling (Cluster Autoscaler) для автоматического добавления и удаления
      нод в зависимости от нагрузки тестов. Использование spot/preemptible инстансов для тестовых нод, чтобы снизить
      стоимость. Гибкое управление ресурсами через ResourceQuotas и LimitRanges в namespace тестов.

6. **Проблемы и решения:**
    - **Сетевые задержки:** В распределенной среде сетевые задержки могут влиять на результаты тестов, особенно на
      производительность. Необходимо учитывать расположение Pod'ов с тестами и тестируемым приложением (желательно в
      одной зоне доступности).
    - **Очистка ресурсов:** После тестового прогона важно удалить все созданные ресурсы (namespace, persistent volumes)
      чтобы избежать утечек. Использование механизма ttl для Job или написание собственных cleanup скриптов, которые
      вызываются через хуки (pre-stop, post-start).
    - **Доступ к результатам тестов:** Сбор артефактов тестов (логи, скриншоты, отчеты) из Pod'ов. Использование sidecar
      контейнеров для отправки артефактов в объектное хранилище (S3, GCS) или в специализированную систему (
      Elasticsearch).
    - **Воспроизводимость тестовых сред:** Использование Infrastructure as Code (IaC) инструментов (Terraform,
      Crossplane) для описания кластера и GitOps (ArgoCD, Flux) для управления конфигурацией приложений. Это позволяет
      воспроизводить окружения для отладки проблем.

7. **Тренды и будущее:**
    - **Serverless поверх Kubernetes (Knative):** Запуск тестов как serverless функций, что позволяет еще более
      эффективно использовать ресурсы.
    - **eBPF:** Использование расширенных возможностей ядра Linux для наблюдения и безопасности без модификации
      приложений. Инструменты типа Cilium предоставляют мощные возможности для тестирования сетевых взаимодействий.
    - **WebAssembly (Wasm) в Kubernetes:** Возможность запуска тестов, скомпилированных в Wasm, что обеспечивает лучшую
      изоляцию и производительность.

- [Содержание](#содержание)

---

# **Пирамида тестирования**

## **Junior Level*

Пирамида тестирования — это концепция, которая визуализирует оптимальное соотношение различных типов автоматизированных
тестов в проекте. Она состоит из трех основных уровней:

1. **Unit-тесты (нижний уровень, основание пирамиды):** Тестируют отдельные компоненты системы (функции, классы) в
   полной изоляции. Их должно быть больше всего — они быстрые, дешевые в поддержке и дают мгновенную обратную связь.

2. **Интеграционные тесты (средний уровень):** Проверяют взаимодействие нескольких компонентов (модулей, сервисов, баз
   данных). Их меньше, чем unit-тестов — они медленнее, сложнее в поддержке, но проверяют критически важные
   взаимодействия.

3. **UI/E2E-тесты (верхний уровень, вершина пирамиды):** Тестируют систему с точки зрения конечного пользователя,
   проверяя полные сценарии работы. Их должно быть меньше всего — они самые медленные, хрупкие и дорогие в поддержке, но
   дают уверенность в работе системы в целом.

Цель пирамиды — создать сбалансированную стратегию тестирования: много быстрых и стабильных тестов внизу, меньше
медленных и комплексных наверху. Для QA инженера понимание этой концепции помогает планировать усилия по автоматизации,
распределять ресурсы и строить эффективный процесс тестирования.

## **Middle Level**

С технической точки зрения реализация каждого уровня пирамиды в Python-экосистеме имеет свои особенности:

1. **Unit-тестирование:**
    - **Инструменты:** `pytest`, `unittest`, `nose2`. Pytest стал де-факто стандартом благодаря гибкости и богатой
      экосистеме.
    - **Изоляция:** Использование моков (`unittest.mock`) для замены зависимостей. Ключевые техники: патчинг (`patch`),
      подмены (`MagicMock`, `AsyncMock`).
    - **Покрытие кода:** Инструменты `coverage.py` и `pytest-cov` для измерения покрытия.
    - **Параметризация:** Декоратор `@pytest.mark.parametrize` для запуска одного теста с разными входными данными.
    - **Важно:** Хороший unit-тест не зависит от внешних систем (БД, файловая система, сеть).

2. **Интеграционное тестирование:**
    - **Тестирование API:** Библиотеки `requests` + `pytest` для HTTP-API. Для асинхронных API — `aiohttp` или `httpx`.
    - **Тестирование БД:** Использование тестовых баз данных (например, SQLite in-memory) или механизмов транзакций с
      откатом после каждого теста. Инструменты: `pytest-django`, `factory_boy` для генерации данных.
    - **Тестирование микросервисов:** Использование тестовых дублей (test doubles) — заглушек (stubs) и моков для
      зависимых сервисов. Контейнеризация зависимостей (Docker) для запуска реальных сервисов в тестовом окружении.
    - **Фикстуры с областью видимости:** В pytest использование `@pytest.fixture(scope="module")` или
      `@pytest.fixture(scope="session")` для создания дорогих ресурсов (например, соединение с БД), которые
      переиспользуются между тестами.

3. **UI/E2E-тестирование:**
    - **Инструменты:** `Selenium WebDriver`, `Playwright`, `Cypress` (через `pytest-playwright`).
    - **Page Object Pattern:** Организация тестового кода через абстракции страниц/компонентов для уменьшения хрупкости
      и повышения переиспользуемости.
    - **Управление состоянием:** Создание и очистка тестовых данных перед/после тестов. Использование API для
      предварительной настройки состояния системы.
    - **Параллельный запуск:** Инструменты `pytest-xdist` для параллельного выполнения тестов. Для UI-тестов важно
      изолировать сессии браузера.

4. **Для AQA:**
    - **Баланс уровней:** Практическое правило: 70% unit-тестов, 20% интеграционных, 10% E2E. Но пропорции зависят от
      проекта.
    - **CI/CD интеграция:** Размещение разных уровней тестов в разных стадиях пайплайна: unit-тесты запускаются на
      каждом коммите, интеграционные — на пулл-реквестах, E2E — на релизных кандидатах.
    - **Флаки-тесты:** UI-тесты часто нестабильны. Необходимы стратегии борьбы: retry механизмы, стабилизация ожиданий (
      explicit waits), изоляция окружения.

## **Senior Level**

Глубокий анализ пирамиды тестирования как архитектурного паттерна, его эволюции, ограничений и интеграции с современными
практиками разработки.

1. **Эволюция и критика классической пирамиды:**
    - **"Песочные часы" или "Ромб":** Современные подходы предлагают увеличивать средний уровень (
      интеграционные/сервисные тесты) для микросервисных архитектур. Вместо пирамиды — песочные часы: много unit-тестов,
      много E2E, но акцент на контрактных тестах между сервисами.
    - **Пирамида Майка Кона:** Дополнение пирамиды ручным тестированием (исследовательское, usability) и тестами
      производительности/безопасности.
    - **Критика:** В микросервисной архитектуре unit-тесты часто дают ложное чувство безопасности, так как не проверяют
      взаимодействие сервисов. Акцент смещается на контрактное тестирование (Pact) и тестирование потребителя (
      consumer-driven contracts).

2. **Архитектурные аспекты реализации каждого уровня:**
    - **Unit-тесты и чистая архитектура:** Unit-тесты должны тестировать бизнес-логику в изоляции от инфраструктуры.
      Достигается через Dependency Injection и следование принципам SOLID. Использование `Protocol` для абстракций
      позволяет создавать моки без наследования.
    - **Интеграционные тесты и транзакции:** Для тестов БД важно использовать механизмы отката транзакций. В Django —
      `@pytest.mark.django_db(transaction=True)`. В SQLAlchemy — `session.begin_nested()` для nested transactions. Для
      NoSQL БД — создание отдельной тестовой базы на каждый тестовый прогон.
    - **E2E тесты и идемпотентность:** Каждый E2E тест должен быть идемпотентным — его повторный запуск не должен
      зависеть от предыдущих запусков. Достигается через:
        - Глобальную уникальность тестовых данных (UUID, временные метки).
        - Паттерн Test Data Builder.
        - Автоматическую очистку через хуки (например, `pytest.fixture` с `autouse=True` и `yield`).

3. **Пирамида и CI/CD:**
    - **Стратификация выполнения:** Разделение тестов на "быстрые" и "медленные". Быстрые тесты запускаются на каждом
      коммите, медленные — по расписанию или по мере необходимости. В GitLab CI/CD — `rules: changes`, в GitHub
      Actions — `paths`.
    - **Канареечный деплоймент и тестирование:** E2E-тесты выполняются на канареечном окружении перед выкатом в прод.
      Использование feature flags для управления доступностью функциональности.
    - **Тестирование в продакшене:** Практики progressive delivery: A/B тестирование, мониторинг ошибок, трассировка
      запросов. Тесты в проде — это следующий уровень после пирамиды.

4. **Для AQA:**
    - **Динамическое определение уровня тестов:** Плагин для pytest, который анализирует зависимости теста (через
      интроспекцию фикстур) и автоматически определяет его уровень: если тест использует моки — unit; если реальную БД —
      интеграционный; если Selenium — E2E. Это позволяет автоматически распределять тесты по разным стадиям CI/CD.
    - **Автоматический баланс пирамиды:** Скрипт, который анализирует историю выполнения тестов (длительность,
      стабильность, покрытие кода) и рекомендует, какие тесты нужно перевести на другой уровень или переписать.
      Использование machine learning для предсказания хрупкости тестов.
    - **Контрактное тестирование для микросервисов:** Внедрение Pact или аналогичных инструментов. Проблема: поддержание
      актуальности контрактов. Решение: автоматическая генерация контрактов из аннотаций типов (TypeScript/OpenAPI для
      фронтенда, `pydantic` для бэкенда) и их валидация в CI.
    - **Тестирование event-driven архитектур:** Для систем на базе Kafka/RabbitMQ unit-тесты бессмысленны. Акцент на
      интеграционное тестирование с тестовым брокером (например, `testcontainers` для запуска Kafka в Docker) и проверка
      корректности обработки событий в различных сценариях.
    - **Перформанс-тестирование как часть пирамиды:** Интеграция нагрузочных тестов (Locust, k6) в CI/CD. Запуск базовых
      нагрузочных тестов на каждую сборку, расширенных — перед релизом. Анализ degradation по метрикам (памяти, CPU,
      latency).
    - **Security-тестирование:** Статический анализ (SAST) — на уровне unit-тестов (каждый коммит). Динамический
      анализ (DAST) — на уровне интеграционных тестов. Penetration testing — на уровне E2E.
    - **Метрики качества тестов:**
        - **Unit-уровень:** Code coverage (branch coverage > 80%), цикломатическая сложность, мутационное тестирование (
          mutmut).
        - **Интеграционный уровень:** Time to failure (как быстро тест обнаруживает регрессию), стабильность (percentage
          of flaky tests).
        - **E2E-уровень:** Business risk coverage (какие бизнес-сценарии покрыты), cost per test (время выполнения *
          стоимость инфраструктуры).
    - **Пирамида для ML-систем:** Особый вызов. Unit-тесты для функций предобработки данных, интеграционные для проверки
      цепочек пайплайнов, E2E для проверки качества модели на тестовых датасетах. Добавляется специфичный уровень —
      тестирование данных (data validation, drift detection).

5. **Антипаттерны и темная сторона:**
    - **Перевернутая пирамида:** Много медленных E2E-тестов, мало unit-тестов. Признак: CI/CD пайплайны выполняются
      часами, разработчики не запускают тесты локально.
    - **Ложные unit-тесты:** Тесты, которые используют реальную БД или сетевые вызовы, но называются unit-тестами.
      Следствие: медленные "unit-тесты", которые падают при отсутствии сети.
    - **Хрупкие интеграционные тесты:** Тесты, которые зависят от специфичного состояния данных. Падают при запуске в
      неправильном порядке или на неподготовленном окружении.
    - **Over-mocking:** Избыточное использование моков, когда тест проверяет не поведение системы, а то, как вызываются
      моки. Тест становится хрупким к рефакторингу.

6. **Будущее пирамиды:**
    - **Shift-left и shift-right:** Тестирование смещается влево (в процесс разработки) и вправо (в прод). Пирамида
      становится объемной фигурой.
    - **AI-assisted testing:** Использование LLM для генерации тестовых случаев, выявления паттернов в падающих тестах,
      предложения оптимизаций.
    - **Serverless и FaaS:** Для бессерверных архитектур классическая пирамида не подходит. Акцент на тестировании
      функций в изоляции (unit) и тестировании оркестрации (интеграционное).

- [Содержание](#содержание)

---

# **Виды тестирования**

## **Junior Level*

Виды тестирования — это различные подходы и методы проверки программного обеспечения, каждый из которых решает
конкретные задачи и имеет свою область применения. Основные виды:

1. **Функциональное тестирование** — проверяет, что система работает в соответствии с требованиями (что она делает).
2. **Нефункциональное тестирование** — проверяет, как система работает (производительность, безопасность, надежность).
3. **Модульное тестирование (Unit)** — тестирование отдельных компонентов кода (функций, классов) в изоляции.
4. **Интеграционное тестирование** — проверка взаимодействия между компонентами, модулями или системами.
5. **Системное тестирование (End-to-End)** — тестирование полного рабочего потока приложения от начала до конца.
6. **Регрессионное тестирование** — проверка, что новые изменения не сломали существующую функциональность.
7. **Дымовое тестирование (Smoke)** — быстрая проверка основных функций системы после сборки.
8. **Приемочное тестирование (Acceptance)** — проверка соответствия системы бизнес-требованиям.

Для QA инженера понимание этих видов помогает выбирать правильные подходы для разных ситуаций: что тестировать
автоматически, а что вручную, как распределять ресурсы и строить стратегию тестирования.

## **Middle Level**

С технической точки зрения каждый вид тестирования в Python-экосистеме реализуется через конкретные инструменты и
практики:

1. **Функциональное тестирование:**
    - **API-тестирование:** Использование `requests`, `httpx`, `aiohttp` для HTTP-запросов. Фреймворки: `pytest` с
      плагинами `pytest-httpx`, `pytest-asyncio`.
    - **UI-тестирование:** `Selenium WebDriver`, `Playwright`, `Cypress` через Python-биндинги. Паттерн Page Object для
      структурирования кода.
    - **Тестирование бизнес-логики:** Модульные и интеграционные тесты с использованием моков (`unittest.mock`) и
      стабов.

2. **Нефункциональное тестирование:**
    - **Нагрузочное тестирование:** `locust` (кодовая нагрузка), `k6` (через subprocess), `JMeter` (через
      `jmeter-python`).
    - **Тестирование безопасности:** Статические анализаторы (`bandit`, `safety`), динамические (`OWASP ZAP` API),
      проверка зависимостей (`dependabot`, `renovate`).
    - **Тестирование доступности (a11y):** `axe-core` через `selenium` или `playwright`.

3. **Модульное тестирование (Unit):**
    - **Изоляция:** Использование `unittest.mock.patch`, `MagicMock`, `AsyncMock` для подмены зависимостей.
    - **Параметризация:** `@pytest.mark.parametrize` для тестирования с разными входными данными.
    - **Property-based тестирование:** `hypothesis` для генерации тестовых данных и проверки инвариантов.

4. **Интеграционное тестирование:**
    - **Тестирование с БД:** Использование тестовых БД (SQLite in-memory), транзакций с откатом, фикстур для данных.
    - **Тестирование микросервисов:** `docker-compose` для поднятия зависимостей, `testcontainers` для управления
      контейнерами из кода.
    - **Контрактное тестирование:** `pact-python` для проверки совместимости между потребителем и поставщиком API.

5. **Регрессионное тестирование:**
    - **Тест-сьюты:** Организация тестов по тегам (`@pytest.mark.regression`) для выборочного запуска.
    - **Анализ покрытия:** `pytest-cov` для отслеживания покрытия измененного кода.

6. **Приемочное тестирование:**
    - **BDD-подход:** `behave`, `pytest-bdd` для тестирования на основе пользовательских сценариев (Gherkin).
    - **Автоматизация сценариев:** Комбинация API и UI-тестов для проверки полных пользовательских сценариев.

7. **Тестирование в CI/CD:**
    - **Стратификация тестов:** Разделение на быстрые (unit) и медленные (UI, нагрузочные) с разными триггерами запуска.
    - **Параллельный запуск:** `pytest-xdist` для ускорения выполнения.

## **Senior Level**

На этом уровне мы рассматриваем виды тестирования как систему взаимосвязанных практик, интегрированных в процесс
разработки и архитектуру приложения.

1. **Архитектурные аспекты тестирования:**
    - **Тестируемость как свойство архитектуры:** Принципы SOLID, Dependency Injection, использование абстракций (ABC,
      Protocol) для упрощения тестирования. Хорошо спроектированная система допускает легкую изоляцию компонентов для
      модульного тестирования и четкие контракты для интеграционного.
    - **Гексагональная архитектура (Ports & Adapters):** Позволяет тестировать бизнес-логику без инфраструктуры через
      in-memory адаптеры.
    - **CQRS и Event Sourcing:** Требует специализированных подходов к тестированию: проверка корректности обработки
      команд, проекций и событий.

2. **Продвинутые техники тестирования:**
    - **Мутационное тестирование:** `mutmut` для оценки качества unit-тестов. Мутанты (малые изменения в коде) должны
      обнаруживаться тестами.
    - **Фаззинг (Fuzz testing):** `atheris` (на основе libFuzzer) для поиска уязвимостей путем генерации случайных
      входных данных.
    - **Дифференциальное тестирование:** Сравнение поведения двух реализаций (старой и новой) на одном наборе данных для
      обнаружения регрессий.
    - **Тестирование вероятностных систем:** Системы с рандомным поведением или машинным обучением требуют
      статистических методов проверки (доверительные интервалы, p-value).

3. **Тестирование распределенных систем:**
    - **Тестирование в условиях сетевых проблем:** Использование инструментов chaos engineering (`chaostoolkit`,
      `pytest-chaos`) для внесения сбоев (задержки, обрывы соединений).
    - **Проверка идемпотентности и консистентности:** В системах с eventual consistency тесты должны учитывать временные
      задержки и возможные конфликты.
    - **Трассировка запросов:** Интеграция с OpenTelemetry для отслеживания выполнения запроса через несколько сервисов
      и проверки корректности работы цепочек.

4. **Мета-тестирование (тестирование тестов):**
    - **Валидация тестового кода:** Статический анализ тестов с помощью `pylint`, `flake8`, `mypy` для поддержания
      качества.
    - **Тестирование фикстур:** Проверка, что фикстуры корректно создают и очищают ресурсы, не оставляют побочных
      эффектов.
    - **Измерение эффективности тестов:** Метрики: процент обнаружения дефектов, время до обнаружения, стабильность
      тестов (flakiness score).

5. **Для AQA:**
    - **Динамическая генерация тестов:** Использование метаклассов и декораторов для создания тестов на лету на основе:
        - Конфигурационных файлов (YAML/JSON описывающих тест-кейсы).
        - Моделей данных (`pydantic` схемы для генерации тестовых данных).
        - Контрактов API (OpenAPI/Swagger спецификации для генерации тестов валидации запросов/ответов).
    - **Адаптивное тестирование:** Система, которая анализирует результаты предыдущих запусков и:
        - Повышает приоритет тестов, которые чаще ломались.
        - Динамически формирует регрессионный набор на основе анализа изменений кода (например, через `git diff` и
          анализ зависимостей).
    - **Тестирование времени выполнения (Runtime testing):** Использование декораторов, которые проверяют инварианты во
      время выполнения продакшен-кода (аналогично `assert`, но с сбором статистики и без прерывания работы). Например,
      проверка типов в рантайме с помощью `typeguard`, валидация бизнес-правил.
    - **Интеграция с мониторингом:** Тесты, которые проверяют не только функциональность, но и метрики мониторинга (
      Prometheus, Grafana). Например, после деплоя запускается тест, который создает нагрузку и проверяет, что latency
      не выросла, а ошибок нет.
    - **Тестирование миграций:** Автоматическая проверка миграций БД:
        - Тест на откат (rollback) миграции.
        - Проверка, что миграция не блокирует таблицы на продолжительное время в продакшене (через анализ плана
          выполнения).
        - Генерация тестовых данных, которые покрывают edge cases миграции.
    - **Тестирование resilience:** Автоматизированные сценарии восстановления после сбоев:
        - Симуляция отказов диска, сети, OOM killer.
        - Проверка, что система переключается на резервные сервера, перезапускает упавшие процессы, сохраняет данные.
    - **ML-тестирование:** Для систем с машинным обучением:
        - Тестирование качества моделей на валидационных датасетах.
        - Проверка на смещение (bias) и дрейф данных (data drift).
        - Интеграционные тесты для полного пайплайна ML: от данных до предсказания.

6. **Организационные и процессные аспекты:**
    - **Тестирование в условиях непрерывной поставки:** При частых деплоях (десятки в день) полный регресс невозможен.
      Стратегии:
        - Канареечный деплоймент с автоматическим откатом при падении ключевых тестов.
        - Feature flags для изоляции новой функциональности и её постепенного включения.
    - **Тестирование legacy систем:** Подходы для работы с кодом без тестов:
        - Characterization tests (тесты, которые описывают текущее поведение системы).
        - Golden master testing (сохранение выходных данных системы для будущего сравнения).
    - **Экономика тестирования:** Расчет ROI автоматизации, оптимизация стоимости тестов (инфраструктура, время
      выполнения, поддержка).

7. **Эволюция видов тестирования:**
    - **Shift-left:** Перенос тестирования на ранние этапы (статический анализ, проверка типов, линтинг в pre-commit).
    - **Shift-right:** Тестирование в продакшене (A/B тесты, мониторинг, canary releases).
    - **Тестирование как код (Testing as Code):** Инфраструктура тестирования управляется через код (тестовые сценарии,
      конфигурации, окружения), что позволяет применять практики разработки: ревью, версионирование, модульность.

- [Содержание](#содержание)

---

# **Техники тест дизайна**

## **Junior Level*

Техники проектирования тестов (Test Design Techniques) — это структурированные методы создания тестовых случаев, которые
помогают эффективно и полно проверить систему. Они отвечают на вопрос: "Как придумать хорошие тесты?" Вместо случайного
перебора тестовых данных эти техники предлагают системный подход.

Основные техники:

1. **Эквивалентное разделение (Equivalence Partitioning):** Разделение входных данных на группы (классы
   эквивалентности), в которых система должна вести себя одинаково. Например, для поля "возраст" можно выделить группы:
   отрицательные числа (невалидные), 0-17 (несовершеннолетние), 18-65 (взрослые), больше 65 (пенсионеры). Достаточно
   протестировать по одному значению из каждой группы.

2. **Анализ граничных значений (Boundary Value Analysis):** Тестирование на границах разделов. Ошибки часто возникают на
   краях допустимых диапазонов. Для возраста 18-65 граничные значения: 17, 18, 19 и 64, 65, 66.

3. **Таблица принятия решений (Decision Table Testing):** Используется, когда логика системы зависит от комбинации
   условий. Создается таблица, где перечислены все условия и соответствующие им действия. Каждая строка — это тестовый
   сценарий.

4. **Тестирование состояний и переходов (State Transition Testing):** Применяется для систем, которые имеют конечное
   число состояний и переходов между ними (например, банкомат: ввод карты -> ввод PIN -> выбор операции). Тестируются
   валидные и невалидные переходы.

5. **Тестирование сценариев использования (Use Case Testing):** Тестирование на основе пользовательских сценариев,
   описывающих взаимодействие пользователя с системой для достижения цели.

Для QA инженера владение этими техниками позволяет создавать тесты, которые с большей вероятностью найдут дефекты,
избегая избыточного тестирования.

## **Middle Level**

С технической точки зрения, применение этих техник в автоматизированном тестировании на Python имеет свои особенности.

1. **Эквивалентное разделение и анализ граничных значений:**
    - **Параметризация тестов:** В pytest с помощью `@pytest.mark.parametrize` легко реализовать проверку нескольких
      значений из одного класса эквивалентности.
    - **Генерация данных:** Для проверки граничных значений можно использовать `itertools.chain` для объединения списков
      значений или создавать фикстуры, возвращающие наборы данных.
    - **Пример:** Тестирование валидатора возраста:
      ```python
      @pytest.mark.parametrize('age, expected', [
          (-5, False),  # невалидный класс
          (0, False),   # граница
          (10, False),  # внутри класса
          (17, False),  # граница
          (18, True),   # граница
          (30, True),   # внутри класса
          (65, True),   # граница
          (66, False)   # граница
      ])
      def test_age_validation(age, expected):
          assert validate_age(age) == expected
      ```

2. **Таблица принятия решений:**
    - **Реализация через data-driven тесты:** Таблицу можно хранить в CSV, JSON или прямо в коде как список словарей.
      Затем итерироваться по строкам и запускать тест для каждой комбинации.
    - **Использование `pytest` и `pandas`:** Загрузить таблицу из Excel/CSV, преобразовать в список параметров.
    - **Пример:** Тестирование логики скидок, зависящей от статуса клиента и суммы покупки.

3. **Тестирование состояний и переходов:**
    - **Моделирование состояний:** Можно использовать библиотеку `state_machine` или реализовать простой конечный
      автомат своими силами.
    - **Проверка переходов:** Создавать тесты, которые имитируют последовательность событий и проверяют текущее
      состояние системы.
    - **Инструменты:** `pytest` с фикстурами для управления состоянием между шагами.

4. **Тестирование сценариев использования:**
    - **BDD-подход:** Инструменты `behave` или `pytest-bdd` позволяют описывать сценарии на языке, близком к
      естественному (Gherkin), и привязывать их к коду.
    - **Page Object Pattern:** Для UI-тестов Page Object хорошо ложится на сценарии использования, инкапсулируя
      взаимодействие с элементами страницы.

5. **Дополнительные техники:**
    - **Попарное тестирование (Pairwise Testing):** Использование инструментов `allpairspy` или `pict` для генерации
      комбинаций параметров, которые покрывают все пары значений. Это сильно сокращает количество тестов.
    - **Предугадывание ошибок (Error Guessing):** Опытный тестировщик на основе знаний о системе и типичных ошибках
      создает тесты. Автоматизировать сложно, но можно накапливать "шаблоны ошибок" и проверять их в регрессионных
      тестах.

6. **Для AQA:**
    - **Автоматизация техник:** Создание утилит, которые генерируют тестовые данные на основе техник эквивалентного
      разделения и граничных значений.
    - **Интеграция в CI/CD:** Параметризованные тесты могут запускаться на каждый коммит, обеспечивая быстрое получение
      обратной связи.
    - **Отчетность:** При использовании параметризации pytest генерирует отдельные записи для каждого набора данных, что
      упрощает анализ падений.

## **Senior Level**

На этом уровне техники проектирования тестов рассматриваются как фундаментальные принципы, интегрированные в процесс
разработки и архитектуру тестовой системы.

1. **Формальные основы и связь с теорией:**
    - **Теория множеств и логика:** Эквивалентное разделение и анализ граничных значений основаны на разбиении множества
      входных данных. Таблицы решений — это представление булевой логики.
    - **Конечные автоматы:** Тестирование переходов между состояниями основано на теории автоматов и может быть
      формально верифицировано с помощью model checking.
    - **Комбинаторика:** Попарное тестирование использует ортогональные массивы и покрывающие массивы (covering arrays)
      для минимизации количества комбинаций.

2. **Автоматическая генерация тестовых случаев:**
    - **Property-based testing:** Библиотека `hypothesis` не просто генерирует случайные данные, а использует техники
      эквивалентного разделения и граничных значений "под капотом". Она строит стратегии генерации данных (strategies),
      которые можно кастомизировать.
    - **Модельное тестирование (Model-based testing):** Создание формальной модели системы (например, на языке Python) и
      автоматическая генерация тестовых последовательностей, покрывающих все состояния и переходы. Инструменты:
      `GraphWalker` (можно интегрировать через API).
    - **Символьное выполнение (Symbolic execution):** Хотя в Python это сложно из-за динамической типизации, существуют
      инструменты, которые пытаются анализировать код и генерировать тесты, покрывающие все пути выполнения (например,
      `pyexz3` на основе Z3).

3. **Динамическое применение техник в зависимости от контекста:**
    - **Адаптивное тестирование:** Система, которая анализирует покрытие кода и результаты предыдущих тестов, чтобы
      решить, какие техники применять дальше. Например, если обнаруживается много ошибок на границах, автоматически
      увеличивается количество тестов на граничные значения.
    - **Машинное обучение для дизайна тестов:** Использование ML для предсказания, какие комбинации параметров с
      наибольшей вероятностью приведут к ошибке, на основе истории дефектов. Обучение модели на исторических данных и
      генерация тестовых случаев, которые "похожи" на те, что находили ошибки ранее.

4. **Интеграция техник в тестовые фреймворки:**
    - **Кастомные маркеры и хуки в pytest:** Создание плагина, который добавляет маркеры для различных техник (например,
      `@pytest.mark.boundary_values`). Хуки могут использоваться для сбора метрик: сколько тестов используют каждую
      технику, насколько они эффективны (количество найденных багов).
    - **Расширение pytest для поддержки таблиц решений:** Плагин, который позволяет загружать таблицы решений из файлов
      и преобразовывать их в параметризованные тесты автоматически.
    - **Генерация тестов из спецификаций:** Интеграция с инструментами спецификации (OpenAPI, Swagger) для
      автоматического создания тестов, использующих техники эквивалентного разделения (валидные/невалидные данные) для
      каждого поля.

5. **Для AQA:**
    - **Мета-тестирование техник:** Написание тестов, которые проверяют, что техники проектирования тестов применяются
      корректно. Например, тест, который проверяет, что для каждого параметризованного теста есть хотя бы один случай из
      каждого класса эквивалентности. Это можно сделать через анализ AST тестовых функций.
    - **Динамический анализ кода для определения границ:** Инструмент, который анализирует исходный код тестируемого
      приложения (через `ast` или `inspect`), находит условия (if-elif-else, сравнения) и автоматически определяет
      граничные значения для параметров. Затем генерирует тесты для этих границ.
    - **Фаззинг с обратной связью (coverage-guided fuzzing):** Инструменты типа `atheris` (на основе libFuzzer)
      используют информацию о покрытии кода для динамической корректировки входных данных, чтобы достичь новых ветвей
      кода. Это автоматическое применение анализа граничных значений и эквивалентного разделения в реальном времени.
    - **Тестирование вероятностных и недетерминированных систем:** Применение статистических методов для определения,
      достаточно ли тестовых случаев для обнаружения ошибок с заданной вероятностью. Использование техник, таких как
      Monte Carlo тестирование.
    - **Интеграция с формальной верификацией:** Для критичных систем (авионика, медицинское ПО) техники дизайна тестов
      могут комбинироваться с формальными методами. Например, использование контрактов (assertions, pre/post-conditions)
      и автоматической генерации тестов из формальных спецификаций с помощью инструментов, подобных `Daikon` (для Python
      можно использовать `icontract`).

6. **Проблемы и ограничения:**
    - **Проклятие размерности:** При большом количестве параметров даже попарное тестирование может генерировать много
      комбинаций. Необходимо использовать эвристики для приоритизации.
    - **Динамическая природа Python:** Статический анализ кода для определения границ сложен из-за динамической
      типизации и метапрограммирования. Часто приходится полагаться на аннотации типов и документацию.
    - **Сложность тестирования состояний:** В системах с большим количеством состояний и переходов полное покрытие может
      быть недостижимо. Приходится выбирать наиболее важные пути (например, на основе использования в продакшене).

7. **Будущее техник проектирования тестов:**
    - **AI-assisted test design:** Использование LLM (например, GPT) для предложения тестовых случаев на основе описания
      функциональности. Обучение моделей на больших наборах кода и тестов.
    - **Тестирование на основе моделей (MBT) в CI/CD:** Интеграция MBT в конвейер непрерывной интеграции, где модель
      системы постоянно обновляется и тесты генерируются автоматически при каждом изменении.
    - **Самоадаптирующиеся тестовые системы:** Фреймворки, которые обучаются на ходу, корректируя применяемые техники в
      зависимости от обнаруженных дефектов и изменений в системе.

- [Содержание](#содержание)

---

# **Метрики тестирования**

## **Junior Level*

Метрики тестирования — это количественные показатели, которые помогают измерить и оценить различные аспекты процесса
тестирования и качества продукта. Они отвечают на вопросы: "Насколько хорошо мы тестируем?", "Каково качество нашего
кода?", "Эффективны ли наши тесты?".

Основные метрики:

- **Покрытие кода (Code Coverage):** Какой процент кода выполняется во время тестов. Измеряется в процентах по строкам,
  ветвям, функциям.
- **Количество дефектов:** Сколько багов найдено, сколько исправлено, скорость их закрытия.
- **Время выполнения тестов:** Как долго работает тестовый набор.
- **Стабильность тестов (Flakiness):** Как часто тесты падают не из-за багов в коде, а по случайным причинам (например,
  проблемы с сетью).
- **Стоимость дефекта:** Сколько стоит найти и исправить баг на разных этапах (чем раньше, тем дешевле).

Метрики помогают принимать обоснованные решения: куда направить усилия по тестированию, когда можно выпускать релиз,
какие тесты нужно улучшить.

## **Middle Level**

С технической точки зрения метрики в Python-экосистеме тестирования собираются и анализируются с помощью конкретных
инструментов и практик.

1. **Метрики покрытия кода:**
    - **Инструменты:** `coverage.py` — стандартный инструмент для измерения покрытия. Интегрируется с pytest через
      `pytest-cov`.
    - **Типы покрытия:**
        - **Line coverage:** Процент выполненных строк.
        - **Branch coverage:** Процент пройденных ветвей в условиях (if/else).
        - **Function coverage:** Процент вызванных функций.
        - **Condition coverage:** Процент комбинаций условий в сложных булевых выражениях.
    - **Интеграция в CI/CD:** Генерация отчетов в формате XML/HTML, интеграция с сервисами (Codecov, Coveralls).

2. **Метрики качества тестов:**
    - **Mutation score (Мутационное тестирование):** `mutmut` внедряет мелкие изменения (мутации) в код и проверяет,
      обнаружат ли их тесты. Процент убитых мутаций — показатель эффективности тестов.
    - **Стабильность тестов (Flakiness):** Анализ истории запусков тестов. Если тест иногда проходит, иногда падает при
      тех же условиях — он нестабилен. Инструменты: `pytest-flakefinder`, кастомные скрипты анализа Jenkins/Allure
      отчетов.
    - **Время выполнения:** `pytest` с флагом `--durations` показывает самые медленные тесты. `pytest-xdist` для
      параллельного запуска, но нужно учитывать накладные расходы.

3. **Метрики дефектов:**
    - **Плотность дефектов (Defect Density):** Количество багов на тысячу строк кода (KLOC).
    - **Эффективность тестирования (Test Effectiveness):** Процент дефектов, найденных тестами, от общего числа
      дефектов (включая найденные пользователями).
    - **Время жизни дефекта (Defect Age):** Среднее время от создания бага до его закрытия.

4. **Метрики процесса:**
    - **Скорость выполнения тестов:** Сколько тестов выполняется в минуту/час.
    - **Автоматизация:** Процент автоматизированных тестов от общего числа.
    - **Стоимость:** Затраты на инфраструктуру тестирования (вычислительные ресурсы, лицензии инструментов).

5. **Инструменты для сбора метрик:**
    - **Allure TestOps / ReportPortal:** Системы для хранения результатов тестов, анализа метрик.
    - **Prometheus + Grafana:** Для мониторинга производительности тестовой инфраструктуры и самого приложения во время
      тестов.
    - **Кастомные скрипты на Python:** Анализ логов, парсинг отчетов, вычисление метрик.

## **Senior Level**

На этом уровне метрики рассматриваются как часть системы измерения качества (Quality Measurement System), которая
интегрирована в процесс разработки, архитектуру и бизнес-цели.

1. **Архитектурные метрики и тестируемость:**
    - **Связность (Cohesion) и связанность (Coupling):** Можно измерять через статические анализаторы (`radon`,
      `mccabe`). Высокая связность и низкая связанность упрощают тестирование.
    - **Цикломатическая сложность:** Количество независимых путей в коде. Высокая сложность (>10) указывает на код,
      трудный для тестирования. Инструменты: `mccabe`, `pylint`.
    - **Индекс поддерживаемости (Maintainability Index):** Комбинированная метрика, учитывающая сложность, объем кода,
      комментарии. Позволяет прогнозировать, насколько сложно будет изменять и тестировать код.

2. **Продвинутые метрики покрытия:**
    - **Semantic Coverage:** Покрытие не только синтаксических конструкций, но и семантики (например, все возможные
      состояния объекта). Частично достигается property-based тестированием (`hypothesis`).
    - **Data Coverage:** Покрытие различных комбинаций входных данных. Использование техник попарного тестирования (
      `allpairspy`) для оценки полноты тестовых данных.
    - **Temporal Coverage:** Для асинхронных и параллельных систем — покрытие различных порядков выполнения и состояний
      гонки. Инструменты: `pytest-asyncio` с детерминированным планировщиком.

3. **Мета-метрики (метрики метрик):**
    - **Полезность метрики:** Оценивается через корреляцию с бизнес-показателями (например, влияет ли покрытие кода на
      количество инцидентов в проде).
    - **Стоимость сбора:** Сколько ресурсов (время, вычисления) уходит на сбор и анализ метрики.
    - **Стабильность метрики:** Насколько метрика подвержена случайным колебаниям. Для важных метрик нужно вычислять
      доверительные интервалы.

4. **Прогнозные метрики и ML:**
    - **Предсказание дефектов:** Модели машинного обучения, которые на основе исторических данных (изменения кода,
      сложность, покрытие тестами) предсказывают, где могут появиться баги. Использование `scikit-learn` для построения
      моделей.
    - **Рекомендательные системы для тестов:** Анализ истории падений тестов и изменений кода для предложения, какие
      тесты нужно запустить при конкретном изменении (test impact analysis).
    - **Аномалии в метриках:** Автоматическое обнаружение аномалий в метриках (например, внезапное падение покрытия,
      рост времени выполнения тестов). Инструменты: `Prophet` от Facebook для временных рядов.

5. **Для AQA:**
    - **Динамическое вычисление метрик во время выполнения тестов:** Плагин для pytest, который с помощью хуков (
      `pytest_runtest_protocol`) собирает метрики в реальном времени: использование памяти, CPU, количество исключений,
      даже покрытие кода на лету (через `sys.settrace` или инструментирование байткода).
    - **Метрики для распределенных тестов:** При использовании `pytest-xdist` или запуске тестов в Kubernetes:
        - **Распределение тестов по нодам:** Равномерность загрузки.
        - **Сетевые задержки:** Время обмена данными между master и worker.
        - **Стоимость инфраструктуры:** Затраты на облачные инстансы во время тестового прогона.
    - **Метрики тестовых данных:**
        - **Разнообразие данных:** Энтропия тестовых данных (сколько различных сценариев покрывают).
        - **Реалистичность:** Насколько тестовые данные близки к продакшен-данным (можно измерять через статистические
          тесты — распределения, корреляции).
    - **Интеграция с бизнес-метриками:**
        - **Влияние тестирования на бизнес-показатели:** Корреляция между метриками тестирования (покрытие, количество
          найденных багов) и бизнес-метриками (коэффициент оттока пользователей, средний чек).
        - **ROI автоматизации:** Расчет возврата инвестиций: (экономия на ручном тестировании - затраты на разработку и
          поддержку автотестов) / затраты.
    - **Метрики для тестирования AI/ML систем:**
        - **Coverage для моделей:** Покрытие пространства признаков (feature space) тестовыми данными.
        - **Смещение (Bias) тестовых данных:** Насколько тестовые данные репрезентативны для всех групп пользователей.
        - **Дрейф данных (Data Drift):** Изменение распределения входных данных со временем и его влияние на качество
          модели.

6. **Визуализация и дашборды:**
    - **Grafana дашборды:** Отображение метрик в реальном времени: покрытие кода, скорость тестов, стабильность.
    - **Интерактивные отчеты:** `plotly` или `bokeh` для создания интерактивных графиков, позволяющих "копать" в данные.
    - **Heatmaps:** Визуализация покрытия кода на карте исходного кода — какие файлы/функции плохо покрыты.

7. **Ограничения и опасности метрик:**
    - **Охватка метрик (Goodhart's law):** "Когда метрика становится целью, она перестает быть хорошей метрикой".
      Пример: разработчики начинают писать бессмысленные тесты только чтобы повысить покрытие.
    - **Ложные корреляции:** Две метрики могут коррелировать, но не иметь причинно-следственной связи.
    - **Контекстная зависимость:** Одинаковые значения метрик могут означать разное в разных проектах (например, 80%
      покрытия для CRUD API и для ядра СУБД).
    - **Перегруженность метриками:** Слишком много метрик приводит к "аналитическому параличу". Нужно выбирать ключевые
      индикаторы (KPIs).

8. **Будущее метрик:**
    - **AI-ассистированный анализ:** LLM для генерации инсайтов из метрик, автоматического написания пояснений к
      изменениям.
    - **Прогнозное тестирование:** Системы, которые на основе метрик предсказывают, какие области кода станут
      проблемными в будущем, и предлагают превентивные тесты.
    - **Интеграция с DORA-метриками:** Связь метрик тестирования с DevOps Research and Assessment метриками (частота
      деплоев, время восстановления и т.д.).

- [Содержание](#содержание)

---

# **Техники тест-дизайна**

## **Junior Level**

Техники тест-дизайна — это систематические методы создания тестовых случаев, которые помогают эффективно проверять
программное обеспечение, находя максимальное количество дефектов при минимальном количестве тестов. Они основаны на
логике, математике и понимании работы системы.

Основные техники:

1. **Эквивалентное разбиение (Equivalence Partitioning):** Разделение входных данных на группы, которые система должна
   обрабатывать одинаково. Достаточно протестировать по одному значению из каждой группы.
2. **Анализ граничных значений (Boundary Value Analysis):** Тестирование значений на границах этих групп, где чаще всего
   возникают ошибки.
3. **Причина-Следствие (Cause-Effect Graphing):** Создание графа, связывающего входные условия (причины) с выходными
   действиями (следствиями), затем преобразование в таблицу решений.
4. **Таблица решений (Decision Table):** Систематическое представление комбинаций условий и соответствующих действий.
5. **Диаграмма переходов состояний (State Transition):** Тестирование систем, которые могут находиться в разных
   состояниях и переходить между ними.
6. **Попарное тестирование (Pairwise Testing):** Проверка всех возможных пар значений параметров вместо полного перебора
   комбинаций.

Для QA инженера владение этими техниками позволяет создавать осмысленные, полные и эффективные тесты вместо случайного
подбора проверок.

## **Middle Level**

С технической точки зрения применение этих техник в автоматизированном тестировании на Python требует специфических
подходов и инструментов:

1. **Эквивалентное разбиение и граничные значения:**
    - **Реализация в pytest:** Параметризация через `@pytest.mark.parametrize` с явным перечислением классов
      эквивалентности и граничных значений.
    - **Генерация данных:** Использование `itertools` или кастомных генераторов для создания тестовых данных.
    - **Пример:** Для функции, принимающей возраст от 18 до 65, тестируем значения: 17 (ниже границы), 18 (нижняя
      граница), 30 (внутри), 65 (верхняя граница), 66 (выше границы).

2. **Таблицы решений:**
    - **Data-driven подход:** Хранение таблиц в CSV, JSON, YAML или Excel. Использование `pytest` с фикстурой, которая
      загружает таблицу и преобразует в параметры.
    - **Библиотеки:** `pandas` для работы со сложными таблицами.
    - **Пример:** Таблица с условиями "статус пользователя" и "сумма заказа" и соответствующими скидками.

3. **Диаграмма переходов состояний:**
    - **Моделирование состояний:** Использование конечных автоматов (`transitions`, `automaton` библиотеки) или
      собственной реализации.
    - **Тестирование последовательностей:** Создание тестов, которые проверяют валидные и невалидные переходы между
      состояниями.
    - **Пример:** Тестирование жизненного цикла заказа: создан -> оплачен -> отправлен -> доставлен.

4. **Попарное тестирование:**
    - **Инструменты:** `allpairspy`, `pairwise` для генерации комбинаций.
    - **Интеграция с pytest:** Генерация параметров для тестов на лету.
    - **Пример:** Система с 10 параметрами, каждый из которых может принимать 10 значений. Полный перебор — 10^10
      комбинаций, попарное тестирование — порядка 100-200 комбинаций.

5. **Причина-Следствие:**
    - **Преобразование в таблицу решений:** После построения графа преобразование в таблицу и реализация как data-driven
      тесты.
    - **Инструменты:** Часто делается вручную или с помощью специализированных инструментов.

6. **Дополнительные техники:**
    - **Use Case Testing:** Реализация через BDD (`behave`, `pytest-bdd`).
    - **Error Guessing:** Накопление шаблонов ошибок и создание тестов на их основе.
    - **Exploratory Testing:** Хотя это ручная техника, можно автоматизировать некоторые аспекты через скрипты, которые
      выполняют случайные действия.

## **Senior Level**

На этом уровне техники тест-дизайна рассматриваются как формальные методы, интегрированные в процесс разработки, с
применением метапрограммирования, статического анализа и машинного обучения.

1. **Формальные методы и автоматическая дедукция:**
    - **Symbolic Execution:** Хотя в Python сложно реализовать полное символьное выполнение, инструменты вроде `pyexz3`
      пытаются анализировать пути выполнения и генерировать тестовые данные для покрытия всех ветвей.
    - **Теоретико-множественная основа:** Эквивалентное разбиение — это разбиение множества входных данных на классы
      эквивалентности (отношение эквивалентности). Граничные значения — это элементы на границах этих классов.
    - **Логическая полнота:** Таблицы решений обеспечивают проверку всех комбинаций условий, что соответствует полному
      покрытию по условиям (condition coverage).

2. **Автоматическая генерация тестов на основе техник:**
    - **Статический анализ кода для определения границ:** Инструмент, который анализирует AST кода, находит сравнения (
      `<`, `>`, `<=`, `>=`, `==`) и автоматически определяет граничные значения для параметров.
    - **Property-based тестирование как обобщение:** Библиотека `hypothesis` использует техники эквивалентного разбиения
      и граничных значений внутри стратегий генерации данных. Можно создавать кастомные стратегии, отражающие
      специфичные для домена разбиения.
    - **Модель-ориентированное тестирование (MBT):** Создание формальной модели системы (например, на языке `pydantic`
      или с помощью специализированных инструментов) и автоматическая генерация тестовых последовательностей,
      покрывающих все состояния и переходы.

3. **Динамическое применение техник в зависимости от контекста:**
    - **Адаптивный тест-дизайн:** Система, которая анализирует результаты предыдущих тестов (покрытие, найденные
      дефекты) и выбирает, какие техники применять дальше. Например, если в модуле много граничных ошибок — увеличивает
      количество тестов на граничные значения.
    - **Machine Learning для оптимизации комбинаций:** Использование ML для предсказания, какие комбинации параметров в
      попарном тестировании наиболее критичны, на основе исторических данных о дефектах.
    - **Генерация тестов на основе мутаций:** Инструменты мутационного тестирования (`mutmut`) могут быть расширены для
      генерации тестов, убивающих конкретные мутации, что косвенно применяет технику "предугадывания ошибок".

4. **Интеграция в CI/CD и DevOps:**
    - **Динамическое определение тестового набора:** На основе изменений кода (`git diff`) автоматическое определение,
      какие техники тест-дизайна применить к измененным модулям. Например, если изменено условие — генерировать тесты на
      граничные значения.
    - **Приоритизация тестов на основе риска:** Комбинирование техник тест-дизайна с анализом рисков. Более рискованные
      модули тестируются с использованием большего количества техник и более тщательно.
    - **Тестирование в продакшене (Production):** Применение техник тест-дизайна к данным из продакшена (
      анонимизированным) для создания более репрезентативных тестовых сценариев.

5. **Для AQA:**
    - **Мета-тестирование техник тест-дизайна:** Написание тестов, которые проверяют, что техники тест-дизайна
      применяются корректно. Например, тест, который анализирует параметризованные тесты и проверяет, что в них
      присутствуют значения из всех классов эквивалентности. Это можно сделать через анализ AST и декораторов pytest.
    - **Автоматическое построение диаграмм переходов состояний:** Инструмент, который путем статического анализа кода (
      поиск паттернов состояний) или динамического анализа (трассировка выполнения) строит диаграмму переходов, а затем
      генерирует тесты для покрытия всех переходов.
    - **Комбинаторные взрывы и их обход:** Для систем с огромным количеством параметров (например, конфигурация облачной
      инфраструктуры) использование продвинутых комбинаторных техник: ортогональные массивы, покрывающие массивы (
      covering arrays), t-wise тестирование (где t > 2). Интеграция с инструментами вроде `ACTS` (NIST).
    - **Тест-дизайн для вероятностных систем:** Для систем с недетерминированным поведением (например, рекомендательные
      системы, AI) использование статистических техник: проверка распределений выходных данных, доверительные интервалы,
      A/B тестирование в автоматическом режиме.
    - **Фаззинг с интеллектуальной генерацией данных:** Инструменты вроде `atheris` (на основе libFuzzer) могут быть
      дополнены знаниями о домене: подсказки о граничных значениях и классах эквивалентности для более эффективного
      поиска краевых случаев.
    - **Тест-дизайн для безопасности:** Применение техник тест-дизайна для поиска уязвимостей. Например, анализ
      граничных значений для буферов (переполнение), таблицы решений для контроля доступа.
    - **Визуализация и анализ покрытия техник:** Дашборды, которые показывают, какие техники тест-дизайна применялись к
      каждому модулю, и как это коррелирует с количеством найденных дефектов. Использование графовых баз данных для
      хранения отношений между тестами, техниками и дефектами.

6. **Ограничения и эволюция:**
    - **Контекстная зависимость:** Эффективность техник зависит от типа системы (например, для ML-моделей техники должны
      быть другими).
    - **Human-in-the-loop:** Некоторые техники (как Error Guessing) требуют экспертизы, которую сложно формализовать.
    - **Эволюция с развитием AI:** Возможность использования LLM для предложения тестовых случаев на основе
      естественного описания функциональности.

- [Содержание](#содержание)

---

# **Автоматизация тестирования**

## **Junior Level*

Автоматизация тестирования — это процесс использования специальных инструментов и скриптов для выполнения тестов,
проверки результатов и сравнения фактического поведения системы с ожидаемым без непосредственного участия человека.
Вместо того чтобы вручную кликать по интерфейсу или проверять API, мы пишем код, который делает это за нас.

Основные преимущества:

- **Скорость:** Автоматические тесты выполняются гораздо быстрее ручных.
- **Повторяемость:** Тесты можно запускать сколько угодно раз с одинаковой точностью.
- **Раннее обнаружение ошибок:** Автотесты можно запускать при каждом изменении кода, быстро находя регрессии.
- **Освобождение времени тестировщиков:** Позволяет сосредоточиться на сложных, исследовательских и нефункциональных
  тестах.

Для QA инженера автоматизация — это ключевой навык, который позволяет масштабировать тестирование, интегрировать его в
процесс разработки (CI/CD) и повышать общее качество продукта.

## **Middle Level**

С технической точки зрения автоматизация тестирования в Python-экосистеме охватывает несколько уровней и требует знания
конкретных инструментов и подходов.

1. **Уровни автоматизации (по пирамиде тестирования):**
    - **Unit-тесты:** Автоматизация с помощью `pytest`, `unittest`. Использование моков (`unittest.mock`) для изоляции.
    - **Интеграционные тесты:** Автоматизация проверки API (`requests`, `httpx`), взаимодействия с БД (транзакции,
      фикстуры), внешними сервисами.
    - **UI-тесты:** Автоматизация веб-интерфейсов через `Selenium WebDriver`, `Playwright`, `Cypress`. Использование
      паттерна Page Object.
    - **Нагрузочные тесты:** Автоматизация с помощью `locust`, `k6`, `JMeter`.

2. **Ключевые принципы автоматизации:**
    - **Поддерживаемость:** Код автотестов должен быть чистым, хорошо организованным и легко изменяемым.
    - **Стабильность:** Тесты должны быть надежными, не должны давать ложные срабатывания (flaky tests).
    - **Изоляция:** Тесты не должны зависеть друг от друга и от внешнего состояния.
    - **Информативность:** При падении тест должен давать четкое сообщение о том, что пошло не так.

3. **Инструменты и фреймворки:**
    - **Основной фреймворк:** `pytest` — де-факто стандарт для написания тестов в Python благодаря простоте, фикстурам и
      плагинам.
    - **Управление зависимостями:** `pip`, `poetry`, `pipenv`.
    - **CI/CD интеграция:** `Jenkins`, `GitLab CI`, `GitHub Actions`, `CircleCI` для автоматического запуска тестов.
    - **Управление тестовыми данными:** `factory_boy`, `Faker` для генерации данных.
    - **Отчетность:** `Allure`, `pytest-html`, `ReportPortal`.

4. **Паттерны автоматизации:**
    - **Page Object:** Для UI-тестов, инкапсулирует работу с элементами страницы.
    - **Screenplay:** Более современная альтернатива Page Object, фокусируется на взаимодействии пользователя с
      системой.
    - **Data-Driven Testing:** Отделение тестовых данных от логики теста (например, хранение данных в JSON, CSV).
    - **Keyword-Driven Testing:** Использование ключевых слов для описания тестовых шагов (часто используется в Robot
      Framework).

5. **Для AQA:**
    - **Выбор правильного уровня автоматизации:** Не все нужно автоматизировать. Критерии: частота выполнения, важность,
      стабильность функционала, сложность ручного тестирования.
    - **Баланс автоматизации:** Соотношение времени на написание автотестов и время на ручное тестирование. Обычно
      20-30% времени на автоматизацию, остальное — на исследовательское тестирование и анализ.
    - **Интеграция в процесс разработки:** Автотесты должны запускаться при каждом коммите (unit-тесты) и
      пулл-реквесте (интеграционные), а также ночью (полный регресс).

## **Senior Level**

На этом уровне автоматизация рассматривается как стратегическая дисциплина, интегрированная в архитектуру, процессы и
культуру разработки.

1. **Архитектура автоматизированной тестовой системы:**
    - **Многослойная архитектура:** Разделение на уровни: слой тестовых сценариев (тест-кейсы), слой бизнес-логики (Page
      Objects, API клиенты), слой утилит (хелперы, генераторы данных), слой драйверов (Selenium, requests).
    - **Принципы проектирования:** Применение SOLID, DRY, KISS к коду автотестов. Использование паттернов
      проектирования (Фабрика, Стратегия, Команда) для повышения гибкости и поддерживаемости.
    - **Микросервисная архитектура для тестов:** В больших распределенных системах сама тестовая система может быть
      построена как набор микросервисов: оркестратор тестов, генератор данных, сборщик отчетов, система нотификаций.

2. **Динамическая и адаптивная автоматизация:**
    - **Генерация тестов на лету:** Использование метапрограммирования (`type`, декораторы, метаклассы) для создания
      тестов в runtime на основе конфигурации или данных.
    - **Самоадаптирующиеся тесты:** Тесты, которые анализируют окружение и подстраиваются под него (например,
      определяют, какая версия API доступна, и выбирают соответствующую стратегию).
    - **Интеллектуальные тестовые данные:** Система, которая анализирует продакшен-данные (с соблюдением анонимности) и
      генерирует репрезентативные тестовые данные, сохраняя распределения и корреляции.

3. **Автоматизация в контексте DevOps и SRE:**
    - **Тестирование как часть инфраструктуры:** Инфраструктурные тесты (например, с помощью `terraform test` или
      `inspec`) для проверки конфигурации окружений.
    - **Тестирование надежности (Reliability Testing):** Автоматизация тестов на отказоустойчивость (chaos engineering)
      с помощью инструментов `chaostoolkit`, `Litmus`.
    - **Тестирование в продакшене:** Автоматизация canary-релизов, A/B тестов, проверки метрик мониторинга (Prometheus,
      Grafana) после деплоя.

4. **AI и ML в автоматизации тестирования:**
    - **Генерация тестовых сценариев:** Использование LLM (например, GPT) для создания тест-кейсов на основе
      пользовательских историй или документации.
    - **Визуальная валидация:** Компьютерное зрение (`opencv`, `pytesseract`) для сравнения скриншотов, поиска элементов
      на основе изображений.
    - **Предсказание падений тестов:** ML-модели, которые на основе истории выполнения предсказывают, какие тесты
      вероятнее упадут при данном изменении кода.
    - **Автоматический анализ root cause:** Система, которая анализирует падение теста, логи и изменения кода, и
      предлагает вероятную причину.

5. **Для AQA:**
    - **Мета-автоматизация:** Автоматизация процесса создания автотестов. Например, инструмент, который по OpenAPI
      спецификации генерирует базовые API-тесты, или по HTML-странице — скелет Page Object.
    - **Распределенное выполнение тестов в облаке:** Система, которая динамически запускает тесты в облачных средах (AWS
      Lambda, Google Cloud Functions) для экономии ресурсов и ускорения выполнения. Использование Kubernetes для
      оркестрации тысяч тестовых подов.
    - **Тестирование времени выполнения (Runtime Verification):** Внедрение агентов в продакшен-код, которые в реальном
      времени проверяют инварианты, контракты и бизнес-правила, отправляя алерты при нарушениях. Это форма "
      перманентного" автоматизированного тестирования.
    - **Эмуляция пользовательского поведения:** Создание "виртуальных пользователей", которые не просто следуют жестким
      сценариям, а используют reinforcement learning для изучения интерфейса и нахождения путей к ошибкам.
    - **Автоматизация исследования (Exploratory Testing Automation):** Инструменты, которые комбинируют случайные
      действия с эвристиками для исследования приложения и обнаружения неочевидных дефектов. Например, комбинация
      фаззинга для UI.
    - **Тестирование безопасности (Security Testing Automation):** Интеграция статических (SAST) и динамических (DAST)
      анализаторов в пайплайн, автоматическое сканирование зависимостей на уязвимости, автоматическое тестирование на
      инъекции и другие атаки.
    - **Перформанс-тестирование как код:** Описание нагрузочных тестов в виде кода (например, на Python с `locust`),
      который версионируется, ревьюится и запускается автоматически. Автоматический анализ результатов и сравнение с
      baseline.

6. **Экономика и ROI автоматизации:**
    - **Расчет ROI:** Формулы для оценки возврата инвестиций в автоматизацию. Учет факторов: время на разработку и
      поддержку, сокращение времени ручного тестирования, уменьшение стоимости дефектов, найденных позже.
    - **Оптимизация стоимости выполнения:** Использование spot-инстансов в облаке, отключение тестовых окружений когда
      они не используются, кэширование зависимостей.
    - **Приоритизация автоматизации:** Матрица принятия решений: что автоматизировать в первую очередь на основе частоты
      изменений, бизнес-критичности, сложности ручного тестирования.

7. **Культурные и организационные аспекты:**
    - **Shift-Left и Shift-Right:** Автоматизация тестирования на всех этапах: от статического анализа кода до
      мониторинга в проде.
    - **Collaboration между разработчиками и QA:** Разработчики пишут unit-тесты, QA фокусируется на интеграционных и
      E2E. Использование pull request ревью для кода автотестов.
    - **Обучение и менторинг:** Создание внутренних библиотек, шаблонов, проведение воркшопов для распространения лучших
      практик автоматизации.

8. **Будущее автоматизации:**
    - **Автономные тестовые системы:** Системы, которые сами определяют, что тестировать, генерируют тесты, выполняют
      их, анализируют результаты и вносят изменения в код (самоисцеляющиеся тесты).
    - **Тестирование в метавселенных и AR/VR:** Новые инструменты для автоматизации тестирования 3D-интерфейсов и
      иммерсивных сред.
    - **Квантовое тестирование:** С появлением квантовых компьютеров потребуются новые подходы к автоматизации
      тестирования квантовых алгоритмов и программ.

- [Содержание](#содержание)

---

Лягушка
