# **Миксины**

[Содержание](/CONTENTS.md#содержание)

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

[Содержание](/CONTENTS.md#содержание)

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

[Содержание](/CONTENTS.md#содержание)

## **Senior Level**

## Миксины в CPython — множественное наследование + MRO

Миксины в CPython — это **обычные классы в множественном наследовании**, обрабатываемые через **C3 MRO** (Method
Resolution Order). Нет специальной поддержки — миксины работают как любой `class A(Base, Mixin1, Mixin2):`.

```c
// Objects/typeobject.c: mro_internal() — C3 алгоритм для миксинов
static PyObject *
mro_internal(PyTypeObject *type) {
    PyObject *bases = lookup_tp_bases(type);   // Кортеж баз: (Base, Mixin1, Mixin2)
    Py_ssize_t nbase = PyTuple_GET_SIZE(bases); // Кол-во родителей
    
    if (nbase == 0) {                          // Нет родителей → только type + object
        return mro_build([type, object]);
    }
    
    PyObject *result = NULL;                   // Результирующий MRO
    PyObject *sequences = PyTuple_New(nbase + 1); // Список MRO родителей
    PyTuple_SET_ITEM(sequences, 0, Py_NewRef((PyObject *)type)); // 1-й: сам класс
    
    // 2. Собираем MRO всех родителей (рекурсивно)
    for (Py_ssize_t i = 0; i < nbase; i++) {
        PyTypeObject *base = PyTuple_GET_ITEM(bases, i); // Base, Mixin1...
        PyObject *parent_mro = lookup_tp_mro(base);       // Их MRO
        if (parent_mro == NULL) goto error;
        PyTuple_SET_ITEM(sequences, i+1, Py_NewRef(parent_mro));
    }
    
    // 3. C3 линеаризация: миксины вставляются слева направо
    result = c3_merge(sequences, linearization_error);  // Алгоритм C3
    Py_DECREF(sequences);
    return result;
    
error:
    Py_DECREF(sequences);
    return NULL;
}
```

**Простыми словами для тупых**: `class A(Base, LoggingMixin, ValidationMixin):` → CPython строит MRO:
`[A, Base, LoggingMixin, ValidationMixin, object]`. **C3 алгоритм** гарантирует: 1) миксины идут **по порядку слева
направо**, 2) **НЕ дублируются**, 3) сохраняют **свой MRO**. При `A().method()` ищет:
`A.method → Base.method → LoggingMixin.method → ValidationMixin.method`.

## PyType_Ready() — наследование слотов миксинов

При создании класса CPython **копирует слоты** (методы) из миксинов по MRO, **разрешая конфликты**.

```c
// Objects/typeobject.c: inherit_special() — копирование слотов из миксинов
static int
inherit_special(PyTypeObject *type) {
    PyObject *mro = lookup_tp_mro(type);       // MRO: [A, Base, LoggingMixin...]
    
    // Идем по MRO, копируем слоты (tp_call, tp_new, tp_init...)
    for (Py_ssize_t i = 0; i < PyTuple_GET_SIZE(mro); i++) {
        PyTypeObject *base = PyTuple_GET_ITEM(mro, i); // LoggingMixin, ValidationMixin...
        
        // Копируем inheritable слоты (tp_doc, tp_new, tp_init, tp_call)
        if (base->tp_new && !type->tp_new)
            type->tp_new = base->tp_new;       // Берем из миксина
            
        if (base->tp_init && !type->tp_init)
            type->tp_init = base->tp_init;
            
        if (base->tp_call && !type->tp_call)
            type->tp_call = base->tp_call;
            
        // Пропускаем non-inheritable: tp_repr, tp_hash (нужны все или ничего)
    }
    
    // Разрешаем конфликты: последний по MRO побеждает
    resolve_slot_conflicts(type);
    return 0;
}
```

Каждый миксин имеет **таблицу слотов** (vtable): `tp_call`, `tp_new`, `tp_repr`... CPython идет по MRO и **копирует
слоты** в итоговую таблицу класса. **Последний миксин по MRO** переопределяет предыдущие. `A().__call__()` →
`A.tp_call` (из последнего миксина с `tp_call`).

## Конфликты слотов миксинов — разрешение по MRO

Если миксины определяют **одинаковые слоты**, побеждает **последний по MRO**.

```c
// Objects/typeobject.c: resolve_slots_inheritable()
static void
resolve_slots_inheritable(PyTypeObject *type) {
    PyObject *mro = lookup_tp_mro(type);
    Py_ssize_t nmro = PyTuple_GET_SIZE(mro);
    
    // Для каждого inheritable слота (tp_new, tp_init...)
    for (int slot = 0; slot < PyType_SlotCount; slot++) {
        if (!slot_is_inheritable(slot)) continue;
        
        void *last_value = NULL;               // Последнее значение по MRO
        
        // Ищем по MRO справа налево (последний побеждает)
        for (Py_ssize_t i = nmro - 1; i >= 0; i--) {
            PyTypeObject *base = PyTuple_GET_ITEM(mro, i);
            void *slot_value = *(void**)((char*)base + slot_offset[slot]);
            
            if (slot_value != NULL) {          // Найден непустой слот
                last_value = slot_value;       // Запоминаем последний
                break;
            }
        }
        
        // Записываем победивший слот
        if (last_value)
            *(void**)((char*)type + slot_offset[slot]) = last_value;
    }
}
```

`LoggingMixin.tp_repr`, `ValidationMixin.tp_repr` → **ValidationMixin** (правее в `bases`) дает финальный `tp_repr`. *
*MRO решает ВСЕ конфликты**: атрибуты, методы, слоты. Миксины должны быть **"тонкими"** (1 ответственность), иначе MRO
становится непредсказуемым.

## __slots__ в миксинах — запрещено множественное наследование

Миксины с `__slots__` **конфликтуют** из-за фиксированных смещений памяти.

```c
// Objects/typeobject.c: check_slots_multiple_bases()
static int
check_multiple_inheritance_slots(PyTypeObject *type) {
    PyObject *bases = lookup_tp_bases(type);   // (Base, LoggingSlotsMixin)
    Py_ssize_t nbase = PyTuple_GET_SIZE(bases);
    
    Py_ssize_t dictoffset = type->tp_dictoffset;  // Смещение __dict__
    
    // Считаем миксины с __slots__
    int slots_count = 0;
    for (Py_ssize_t i = 0; i < nbase; i++) {
        PyTypeObject *base = PyTuple_GET_ITEM(bases, i);
        if (base->tp_dictoffset != 0) {        // Есть __slots__
            slots_count++;
            if (base->tp_dictoffset != dictoffset) {
                PyErr_SetString(PyExc_TypeError,
                    "multiple bases with __slots__ have incompatible slots");
                return -1;                     // Конфликт смещений!
            }
        }
    }
    
    if (slots_count > 1) {
        PyErr_SetString(PyExc_TypeError,
            "can't derive from multiple bases with __slots__");
        return -1;
    }
    return 0;
}
```

`__slots__ = ['x']` → фиксирует `x` на `offsetof(PyObject_HEAD, x)`. Два миксина с `__slots__` → **разные смещения** →
дескрипторы "смотрят не туда". **Правило**: **только 1 родитель** с `__slots__`. Миксины должны использовать **обычные
атрибуты** или **композицию**.

## Кэш атрибутов миксинов — tp_version_tag

Поиск `A().method` кэшируется по `tp_version_tag`. Изменение миксинов **инвалидирует** весь поддерево.

```c
// Objects/typeobject.c: type_modified_unlocked() при изменении миксина
void PyType_Modified(PyTypeObject *mixin) {
    BEGIN_TYPE_LOCK();                         // Блокируем типы
    type_modified_unlocked(mixin);             // Рекурсивно по подклассам
    END_TYPE_LOCK();
}

static void
type_modified_unlocked(PyTypeObject *type) {
    ASSERT_TYPE_LOCK_HELD();
    
    // 1. Инвалидируем КЭШ этого типа
    set_version_unlocked(type, 0);             // tp_version_tag = 0
    
    // 2. Рекурсивно инвалидируем ПОДКЛАССЫ (использующие миксин!)
    PyObject *subclasses = lookup_tp_subclasses(type);
    if (subclasses) {
        Py_ssize_t pos = 0;
        PyObject *ref;
        while (PyDict_Next(subclasses, &pos, NULL, &ref)) {
            PyTypeObject *subclass = type_from_ref(ref);
            if (subclass) {
                type_modified_unlocked(subclass);  // A, B, C... все!
                Py_DECREF(subclass);
            }
        }
    }
}
```

Изменил `LoggingMixin.method()` → **ВСЕ классы** `class X(..., LoggingMixin):` получают `tp_version_tag=0` → **полная
инвалидация кэша**. `X().method` каждый раз ищет **заново** по MRO. **Дорого**, поэтому миксины **статичны**.

## super() в миксинах — MRO цепочка

`super().method()` в миксинах следует **MRO**, пропуская самого себя.

```c
// Objects/superobject.c: super_get() + метод разрешения
static PyObject *
super_descr_get(PyWrapperDescrObject *wrapper, PyObject *self, PyObject *instance) {
    PyTypeObject *type = wrapper->d_type;      // Класс миксина
    PyObject *mro = lookup_tp_mro(type);       // MRO миксина
    
    // Находим следующий класс ПОСЛЕ текущего в MRO
    Py_ssize_t i;
    for (i = 0; i < PyTuple_GET_SIZE(mro); i++) {
        if (PyTuple_GET_ITEM(mro, i) == (PyObject *)type) {
            break;                             // Найден LoggingMixin
        }
    }
    PyTypeObject *next_type = PyTuple_GET_ITEM(mro, i+1); // Следующий!
    
    // Создаем super(next_type, instance)
    return _PySuper_Create(next_type, instance);
}
```

В `LoggingMixin.method()` `super().method()` → **следующий** класс в MRO получает вызов. **Цепочка**:
`A.method() → LoggingMixin.method() → super() → ValidationMixin.method() → Base.method()`. **Правильный порядок** без
дублирования.

## Сравнение миксинов под капотом

| Аспект                 | Миксин (MI + MRO)       | Композиция            |
|------------------------|-------------------------|-----------------------|
| **Поиск методов**      | Авто по MRO             | `self.mixin.method()` |
| **Слоты**              | Копирование + конфликты | Отсутствуют           |
| **__slots__**          | Только 1 родитель       | Полная свобода        |
| **Изменения**          | Инвалидация поддерева   | Локально              |
| **super()**            | Автоматическая цепочка  | Ручная                |
| **Производительность** | Кэш + MRO поиск         | Прямой вызов          |

**Миксины** = **множественное наследование с дисциплиной**: слева направо, 1 ответственность, без `__slots__`. CPython
обрабатывает их через **универсальный MRO + слоты**, без специального кода.

- [Содержание](/CONTENTS.md#содержание)