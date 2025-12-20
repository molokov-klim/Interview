# *Хеш-таблица*

[Содержание](/CONTENTS.md#содержание)

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

[Содержание](/CONTENTS.md#содержание)

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

[Содержание](/CONTENTS.md#содержание)

## **Senior Level**

В CPython 3.9+ **хеш-таблица** (`PyDictObject`) использует **split table** с **компактным представлением**: общие
`PyDictKeysObject` (shared keys) + массив значений `ma_values[]`. Поиск через `lookdict_*` с **робин-худ хешированием**
и **атомарными индексами**. `Objects/dictobject.c`,`Include/cpython/dictobject.h`

[Содержание](/CONTENTS.md#содержание)

## 1. PyDictObject (Python 3.9+)

```c
typedef struct {
    Py_ssize_t ma_used;            // Количество *реальных* пар ключ-значение
    uint64_t ma_version_tag;       // Версия для итераторов (атомарно)
    PyDictKeysObject *ma_keys;     // Общие ключи (может быть shared)
    PyObject **ma_values;          // Массив значений (split table)
} PyDictObject;
```

Словарь = счетчик занятых слотов + версия + общие ключи + массив значений. `ma_used` считает
пары, а не слоты хеш-таблицы.

[Содержание](/CONTENTS.md#содержание)

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

`PyDictKeysObject` содержит хеш-таблицу индексов + сами ключи. Может быть **общим** для многих
словарей одной структуры (экономия памяти на классах).

[Содержание](/CONTENTS.md#содержание)

## 3. Индексы в хеш-таблице (dk_indices)

```c
// dk_indices содержит значения:
// DKIX_EMPTY   (0-1)   - пустой слот
// DKIX_DUMMY   (0x8000)- удалённый слот (tombstone)
// DKIX_ACTIVE  (>0)    - индекс в entries/values (1..dk_size-1)
// 0xFFFF       - не используется
```

Каждый слот хеш-таблицы — это **8-битный индекс** (int8_t): 0=пусто, 255=удалено,
1-254=указатель на запись с ключом.

[Содержание](/CONTENTS.md#содержание)

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

Вычисляем `hash(key) % размер`. Если слот пустой — останавливаемся. Если занят — проверяем
ключ. Если не наш — делаем **робин-худ шаг** `(i*5 + 1 + perturb) % size`, пока не найдём или не упрёмся в пустой слот.

[Содержание](/CONTENTS.md#содержание)

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

Ищем место через lookdict. Если ключ есть — меняем значение. Если таблица заполнена на 2/3 —
удваиваем размер. Копируем ключ/значение (+refcnt).

[Содержание](/CONTENTS.md#содержание)

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

При 2/3 заполнении создаём новую таблицу вдвое больше, перехешируем **все** элементы заново,
атомарно меняем указатель `ma_keys`.

[Содержание](/CONTENTS.md#содержание)

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

В многопоточной среде (GIL released) индексы `dk_indices[]` обновляются атомарно. `release`
гарантирует видимость изменений.

[Содержание](/CONTENTS.md#содержание)

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

Все экземпляры класса `class C:` делят один `PyDictKeysObject` с одинаковыми именами
атрибутов. Значения хранятся отдельно в `ma_values[]`.

[Содержание](/CONTENTS.md#содержание)

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

`d1 |= d2` вызывает `_PyDict_MergeEx` (атомарное объединение с приоритетом правого словаря).

**Хеш-таблица** в CPython 3.9+ — **split table** (`PyDictKeysObject` + `ma_values[]`), **робин-худ хеширование**, *
*атомарные индексы**, **shared keys** для классов, resize при 2/3 load factor, vectorcall поддержка.

- [Содержание](/CONTENTS.md#содержание)
