# **typing**

[Содержание](/CONTENTS.md#содержание)

## **Junior Level**

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

[Содержание](/CONTENTS.md#содержание)

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

[Содержание](/CONTENTS.md#содержание)

## **Senior Level**

## Хранение аннотаций в CPython

Аннотации типов в CPython (Python 3.9+) хранятся в специальном атрибуте `__annotations__` объектов (функций, классов,
модулей). Этот атрибут представляет собой словарь `dict[str, object]`, где ключи — имена переменных/параметров, а
значения — сами аннотации (типы или выражения). CPython не выполняет аннотации во время выполнения (runtime) — они
остаются "ленивыми" объектами для статических анализаторов вроде mypy.

В режиме `from __future__ import annotations` (по умолчанию с Python 3.10, рекомендуется в 3.9+), аннотации сохраняются
как строки без вычисления, что предотвращает ошибки вроде `NameError` при ссылке на неопределённые имена. Это
реализовано в парсере AST (Abstract Syntax Tree) CPython: во время компиляции в байткод аннотации парсятся как
`ast.AnnAssign` и сериализуются в строки.

```python
# Байткод примера: def func(x: str) -> int:
# dis.dis(func) покажет LOAD_BUILD_CLASS и MAKE_FUNCTION с флагом CO_NESTED
# Аннотации попадают в co_varnames и co_cellvars фрейма функции
import dis


def func(x: str) -> int:
    return x


print(func.__annotations__)  # {'x': <class 'str'>, 'return': <class 'int'>}
```

Представь, что CPython — это почтальон. Когда ты пишешь `x: str`, он не проверяет посылку (str) сразу, а просто
записывает адрес "x ожидает str" в специальную тетрадку `__annotations__`. Позже статический анализатор (mypy) открывает
тетрадку и проверяет, всё ли правильно. Если включишь `from __future__ import annotations`, он даже не смотрит на типы —
просто записывает как текст "str", чтоб не было ошибок, если имя типа ещё не определено.

## Реализация в модуле Lib/typing.py

Модуль `typing.py` (CPython sources: `Lib/typing.py`) — это чистый Python-код (~3000 строк в 3.12+), который создаёт
подклассы и прокси для типов. Он не трогает C-уровень CPython напрямую, но использует служебные классы вроде
`_GenericAlias`, `_SpecialForm`. Ключевые структуры: `_TypingEmpty`, `_TypingEllipsis` для специальных случаев (
Tuple[...], Callable).

Основная магия — в `__getitem__()` generic-типов. Когда пишешь `List[int]`, CPython вызывает `list.__getitem__(int)`, но
typing перехватывает через подкласс `_GenericAlias`:

```python
# Из CPython Lib/typing.py (Python 3.12+ fragment, упрощённо)
class _GenericAlias:
    def __getitem__(self, params):
        if not isinstance(params, tuple):
            params = (params,)
        # Проверяем, что параметры — валидные типы
        msg = "Parameters to generic types must be types."
        params = tuple(_type_check(p, msg) for p in params)
        # Собираем type variables для подстановки
        self.__parameters__ = _collect_type_vars(params)
        # Создаём новый alias с подстановкой
        return _subs_tvars(self, self.__parameters__, params)


# Пример работы:
List = _GenericAlias('list', inst=actual_list)  # actual_list из builtins
List[int]  # -> _GenericAlias(list, (int,), name='list[int]')
```

`_GenericAlias` — как фабрика по производству "типовых коробок". `List[int]` говорит: "Возьми коробку list и пометь её,
что внутри int". CPython проверяет, что `int` — настоящий тип (не число или строка), собирает переменные типов (
TypeVar), подставляет их и выдаёт новую "метку" вида list[int]. Это не настоящий list, а прокси, который mypy понимает.
На runtime ничего не проверяется — просто метка болтается в памяти.

## C-уровень: Обработка __annotations__ в Objects

На C-уровне (Objects/descrobject.c, Objects/functionobject.c) `__annotations__` — это дескриптор (data descriptor).
Доступ через `PyObject_GenericGetAttr` → проверка `__dict__` → fallback на слоты типа `tp_getattro`.

Для функций: при создании `PyFunctionObject` (Objects/funcobject.c) байткод компилятора (Python/compile.c) заполняет
`func_annotations` через `STORE_ANNOTATION` opcode (с Python 3.5+). В 3.9+ добавлена оптимизация: аннотации кэшируются в
`PyDictObject` и не пересчитываются.

```c
// Из CPython Objects/funcobject.c (Python 3.12, упрощённо)
// PyFunctionObject структура:
typedef struct {
    PyObject_HEAD
    PyObject *func_code;      /* код байткода */
    PyObject *func_globals;   /* глобальное пространство */
    PyObject *func_annotations; /* <-- НАШЕ dict[str, annotation] */
    // ... другие слоты
} PyFunctionObject;

// В func_new():
if (annotations != NULL) {
    // Копируем dict аннотаций из co_annotations код-объекта
    func->func_annotations = Py_NewRef(annotations);
    // PyDict_SetItem копирует пары key:value без вычисления
}
```

C-код CPython — это "склад товаров". Каждый объект (функция) имеет полку `func_annotations` — большой ящик-словарь.
Когда компилятор видит `: str` в коде, он кладёт ярлык "{'x': str}" в этот ящик, не открывая посылки. При
`func.__annotations__` C-функция просто достаёт ящик и отдаёт. Нет проверок типов — только хранение. В 3.9+ ящик
сделан "ленивым": если `from __future__ import annotations`, ярлыки — сырые строки вроде "<class 'str'>", чтоб не
ломалось при импорте.

## Байткод и компиляция аннотаций (Python 3.9+)

Компилятор (Python/compile.c → Python/symtable.c) парсит AST-узлы `AnnAssign`/`arg.annotation`. В 3.9+ добавлена
поддержка built-in generics (`list[int]` вместо `typing.List[int]`) через `symtable.c` анализ forward refs.

Ключевой opcode: `STORE_ANNOTATION` (opcode 95 в 3.12). Он сохраняет аннотацию в `co_annotations` код-объекта.

```python
# dis.dis для def func(a: int): pass
import dis


def func(a: int): pass


dis.dis(func)
# Вывод (Python 3.12):
#   2           0 RESUME                   0
#               1 LOAD_CONST               1 (<code object func>)
#               ...
# В co_annotations: {'a': <class 'int'>}
```

Байткод — инструкции для виртуальной машины CPython. `STORE_ANNOTATION` — команда "положи метку типа в специальный сейф
код-объекта". При запуске функции VM берёт сейф и копирует в `func_annotations`. В 3.9+ парсер стал умнее: распознаёт
`list[int]` на лету, без импорта typing, и сохраняет как `_GenericAlias`. Но на выполнении байткода типы игнорируются —
VM просто пропускает STORE_ANNOTATION.

## TypeVar и Generic: Внутренняя подстановка

`TypeVar` — `_Final` subclass с `__parameters__`. Generic использует `_collect_type_vars()` для рекурсивного сбора
переменных и `_subs_tvars()` для подстановки (из typing.py).

```python
# Из Lib/typing.py
def _subs_tvars(original, parameters, args):
    """Подстановка TypeVar в Generic"""
    new_args = []
    for arg in original.__args__:
        if isinstance(arg, _TypeVarLike):
            idx = parameters.index(arg)
            new_args.append(args[idx])
        else:
            new_args.append(arg)
    return type(original)(original.__origin__, tuple(new_args))
```

TypeVar — как пустая коробка с именем "T". Generic — шаблон "Коробка[T]". При `MyGeneric[str]` CPython находит все "T" в
шаблоне, заменяет на "str" и выдаёт новую коробку "Коробка[str]". Это рекурсивно: если внутри T есть другие переменные,
они тоже подставляются. Всё в Python-коде typing.py, без C, но быстро благодаря `@_tp_cache` (lru_cache).

Эти механизмы делают typing эффективным: ~0 overhead на runtime, полная поддержка в IDE/mypy. Для собеседования
акцентируй: нет enforcement в CPython, только storage/parsing.

- [Содержание](/CONTENTS.md#содержание)

---

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

- [Содержание](/CONTENTS.md#содержание)