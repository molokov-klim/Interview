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

Срез `lst[1:3:2]` в памяти — это структура из 4 полей: 24-байт заголовок PyObject + 3
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

Интерпретатор снимает 2/3 значения со стека, вызывает PySlice_New (создаёт PySliceObject),
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

PySlice_New создаёт объект из 24 байт заголовка + 3 указателя. Каждый аргумент получает +1 к
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

Эта функция превращает `slice(-2:, None, 2)` в конкретные числа: для списка длины 10 даёт
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

`-1` всегда означает "последний элемент", `-2` — предпоследний. Функция переводит
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

Создаём новый список нужной длины, увеличиваем refcnt каждого элемента оригинала (+1), кладём
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

`lst[1:3] = [10,20]` удаляет 2 элемента, добавляет 2 новых, сдвигает хвост.
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

Все типы регистрируют таблицу `PySequenceMethods` со слотами. BINARY_SUBSCR выбирает
`sq_slice`/`mp_subscript` в зависимости от типа.

Срезы в CPython — это **PySliceObject** (3 указателя) + **PySlice_GetIndicesEx** (нормализация) + **тип-специфичные**
`sq_slice`/`sq_ass_slice`, вызываемые через байткоды BUILD_SLICE/BINARY_SUBSCR/STORE_SUBSCR.
