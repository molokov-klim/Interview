# **Garbage Collector (Сборщик мусора)**

[Содержание](/CONTENTS.md#содержание)

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

[Содержание](/CONTENTS.md#содержание)

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

[Содержание](/CONTENTS.md#содержание)

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

- [Содержание](/CONTENTS.md#содержание)