
---

# **Pytest**

## **Junior Level**

Pytest — это современный и мощный фреймворк для тестирования Python-кода, который превращает написание тестов из рутины
в интуитивный процесс. Его главная философия — «тесты — это просто код», что позволяет писать проверки в естественном
стиле без лишнего шаблонного кода.

**Почему pytest стал стандартом де-факто?**

1. **Минималистичный синтаксис:** Вместо десятков специализированных методов (вроде `assertEqual`, `assertTrue` из
   unittest) используется обычный оператор `assert`. Тесты — это просто функции с именем, начинающимся на `test_`.

2. **Автоматическое обнаружение:** Pytest сам находит тесты — сканирует файлы, имена которых начинаются с `test_` или
   заканчиваются `_test.py`, и запускает все функции и методы с префиксом `test`.

3. **Фикстуры (fixtures):** Умная система подготовки тестового окружения. Вместо громоздких методов `setUp`/`tearDown`
   вы объявляете функции с декоратором `@pytest.fixture`, которые возвращают нужные данные. Эти фикстуры затем можно
   «запрашивать» в тестах, просто указывая их имена в параметрах функции.

4. **Подробная диагностика:** Когда тест падает, pytest показывает не просто «ожидалось X, получено Y», а демонстрирует
   разницу между значениями, выводит промежуточные вычисления и даже показывает состояние переменных.

5. **Параметризация:** Одна тест-функция может проверить множество сценариев с помощью декоратора
   `@pytest.mark.parametrize`. Это заменяет горы почти идентичного кода.

6. **Богатая экосистема:** Сотни плагинов расширяют возможности — от измерения покрытия кода (`pytest-cov`) до
   параллельного запуска (`pytest-xdist`) и генерации красивых отчетов (`pytest-html`).

**Пример теста на pytest:**

```python
# test_calculator.py
def add(a, b):
    return a + b


def test_add():
    assert add(2, 3) == 5  # Просто и понятно!

# Сравните с unittest:
# self.assertEqual(add(2, 3), 5)
```

## **Middle Level**

### **Архитектура и принципы работы**

Pytest — это не просто библиотека, а сложная система с продуманной архитектурой, построенной вокруг **плагинов** и *
*хуков**. Даже базовые функции (поиск тестов, их выполнение, формирование отчёта) реализованы как встроенные плагины.

**Процесс выполнения тестов:**

1. **Сбор (Collection):** Pytest рекурсивно обходит указанные директории, импортирует модули и анализирует их, используя
   механизм интроспекции Python. Он создаёт древовидную структуру объектов (`Session` → `Module` → `Class` →
   `Function`).
2. **Разрешение зависимостей:** Анализирует параметры каждой тест-функции, находит соответствующие фикстуры, строит граф
   их зависимостей и определяет порядок выполнения.
3. **Выполнение (Runtime):** Запускает тесты с учётом областей видимости фикстур, перехватывает исключения, собирает
   результаты.
4. **Отчёт (Reporting):** Формирует структурированный отчёт, который может быть выведен в консоль, файл или передан
   внешней системе через плагины.

### **Фикстуры: больше, чем просто setup/teardown**

Фикстуры — это **инъекция зависимостей** для тестов. Они превращают подготовку данных из побочного эффекта в
декларативный процесс.

**Ключевые возможности:**

- **Области видимости (scope):** Фикстура может создаваться один раз на всю сессию (`session`), модуль (`module`),
  класс (`class`) или функцию (`function`). Это критично для оптимизации дорогих операций (например, подключения к БД).
- **Автоматическая очистка:** При использовании `yield` код после него выполняется как `teardown`, даже если тест упал.
  Альтернатива — `request.addfinalizer`.
- **Зависимости между фикстурами:** Фикстуры могут запрашивать другие фикстуры, образуя иерархию. Pytest автоматически
  разрешает эти зависимости и создаёт фикстуры в правильном порядке.
- **Динамические фикстуры:** Можно создавать фикстуры программно во время выполнения через `request.addfixturedef`, что
  позволяет генерировать тестовые данные на лету.

**Продвинутый пример фикстуры с областью видимости и параметризацией:**

```python
import pytest
from database import Database


@pytest.fixture(scope="module")
def db():
    # Создаётся один раз на модуль
    connection = Database.connect()
    yield connection
    connection.close()  # Выполнится после всех тестов модуля


@pytest.fixture
def user(db):  # Зависит от фикстуры db
    user_id = db.create_user("test@example.com")
    yield user_id
    db.delete_user(user_id)  # Очистка после каждого теста
```

### **Система плагинов и хуков**

**Хуки (hooks)** — это точки расширения, позволяющие встраиваться в жизненный цикл pytest. Плагины — это модули,
реализующие эти хуки.

**Важные хуки:**

- `pytest_collection_modifyitems` — изменяет список тестов перед выполнением (например, фильтрует или меняет порядок)
- `pytest_runtest_setup`/`pytest_runtest_teardown` — код до и после каждого теста
- `pytest_runtest_makereport` — создаёт отчёт о выполнении теста
- `pytest_configure` — инициализация плагина при запуске pytest

**Пример кастомного плагина:**

```python
# plugins/myplugin.py
def pytest_collection_modifyitems(config, items):
    """Добавляет маркер 'slow' всем тестам, имя которых содержит 'slow'"""
    for item in items:
        if "slow" in item.name:
            item.add_marker("slow")
```

### **Параметризация: мощность и гибкость**

`@pytest.mark.parametrize` — это не просто цикл по данным, а генератор отдельных тестовых случаев.

**Особенности:**

- Каждый набор параметров становится отдельным тестом в отчёте.
- Можно комбинировать несколько декораторов `parametrize` для декартова произведения.
- Поддерживаются сложные идентификаторы тестов через параметр `ids`.

**Динамическая параметризация:**

```python
import pytest


def generate_test_data():
    # Данные могут приходить из файла, БД или API
    return [(1, 2, 3), (4, 5, 9), (10, -3, 7)]


@pytest.mark.parametrize("a,b,expected", generate_test_data())
def test_add(a, b, expected):
    assert a + b == expected
```

### **Интеграция и расширенные сценарии**

1. **Асинхронные тесты:** Через плагин `pytest-asyncio` можно тестировать асинхронный код. Он предоставляет фикстуру
   `event_loop` и маркер `@pytest.mark.asyncio`.

2. **Мокирование:** Хотя pytest не включает мокирование, он идеально сочетается с `unittest.mock`. Фикстура
   `monkeypatch` позволяет временно заменять атрибуты, словари и даже импорты.

3. **Распределённое выполнение:** `pytest-xdist` запускает тесты параллельно на нескольких ядрах или даже машинах. Важно
   помнить, что фикстуры с областью видимости `session` создаются в каждом worker отдельно.

4. **Интеграция с Docker:** Можно создавать плагины, которые через хук `pytest_configure` запускают контейнеры с
   тестовой инфраструктурой, а через `pytest_unconfigure` останавливают их.

### **Подводные камни и лучшие практики**

1. **Циклические зависимости фикстур:** Pytest обнаружит цикл и выдаст ошибку. Решение — перепроектировать фикстуры,
   выделив общую логику в третью фикстуру.

2. **Состояние между тестами:** Избегайте изменения глобального состояния. Фикстуры должны изолировать тесты друг от
   друга. Для очистки глобальных объектов используйте фикстуры с `scope="session"` и `yield`.

3. **Производительность:** Дорогие фикстуры (например, миграция БД) должны иметь максимальную область видимости.
   Используйте `pytest-xdist` для параллельного запуска, но помните о конкурентном доступе к ресурсам.

4. **Динамическое создание тестов:** Через `pytest_generate_tests` можно генерировать тесты программно на основе внешних
   данных, но это усложняет отладку.

## **Senior Level**

## Интеграция pytest с CPython тестовой системой

Pytest не является частью CPython core, но глубоко интегрируется через `Lib/test/regrtest.py` — главный тест-раннер
CPython. В Python 3.9+ `regrtest.py` использует `unittest` discovery, но pytest подключается как внешний раннер через
`--testdir` или плагины. CPython компилирует все `test_*.py` в байткод и запускает через `PyRun_SimpleFileExFlags()` (
Python/pythonrun.c), где pytest перехватывает коллекцию через `pytest_collect_file()` hooks.

Ключ: pytest использует стандартный байткод CPython без модификаций, но расширяет `PyFrameObject` evaluation через
monkey-patching `sys.meta_path` и `unittest.TestLoader`.

```python
# Из CPython Lib/test/regrtest.py (Python 3.12+ fragment)
# pytest интегрируется через runtest_cpytest()
def runtest_cpytest(ns):
    """Run a pytest test file in the given namespace."""
    # Создаём временный модуль для импорта test файла
    mod = types.ModuleType(ns['__name__'])
    mod.__dict__.update(ns)
    # Компилируем test файл в code object
    with open(test_file, encoding="utf-8") as fp:
        code = compile(fp.read(), test_file, 'exec')
    # Выполняем code object в frame (PyEval_EvalCode)
    exec(code, mod.__dict__)
    # pytest.main() вызывается после импорта всех tests
    import pytest
    ret = pytest.main([test_file, '-v', '--tb=short'])
```

`regrtest.py` — как конвейер CPython для тестов. Берёт `test_foo.py`, компилирует в байткод (code object), создаёт
PyFrameObject с globals/locals, запускает через VM (`_PyEval_EvalFrameDefault()`). Pytest подключается после:
импортирует все test модули, находит функции `test_*` через AST/walk FS, группирует в `TestReport`. Нет C-изменений —
чистый Python override `unittest.TestCase.run()`.

## Коллекция тестов: _pytest.python.Module/Class

Pytest коллектор (`_pytest/python.py`) парсит FS → AST → байткод CPython для поиска `def test_*`. Использует
`inspect.getmodule()` → `co_varnames` code object для фильтрации. В 3.9+ оптимизировано через `PyCode_GetNumFree()` для
cellvars.

```python
# Из _pytest/python.py (pytest 8.0+, CPython 3.9+ compatible)
# pytest_pycollect_makemodule: создаёт PyCodeObject → итерация
@hookimpl(hookwrapper=True)
def pytest_pycollect_makeitem(collector, name, obj):
    outcome = yield  # yield для wrapper hooks
    res = outcome.get_result()
    if res is not None: return res  # Уже обработано
    # obj — функция/класс из exec(code) CPython
    if inspect.isfunction(obj) and name.startswith('test_'):
        # Проверяем co_flags для nested funcs (CO_NESTED)
        code = obj.__code__
        if code.co_flags & CO_NESTED: continue
        yield Function(name, parent=collector, fixtureinfo=...)  # Item node
```

Коллектор pytest — как робот-сканер. Проходит по `test_*.py`, импортирует (exec байткода VM), смотрит в
`__code__.co_varnames` (список имён локалок из code object). Если видит `test_foo`, создаёт "узел теста" — обёртку над
PyFunctionObject. Для классов `Test*` рекурсивно сканирует методы. Всё на Python-уровне, но читает C-структуры
`PyCodeObject.co_varnames` без копания в C.

## Fixtures: Monkey-patching PyFrameObject locals

Fixtures в pytest — генераторы (`yield`), которые инжектят в `f_localsplus` frame'а через `metafunc.parametrize()`.
CPython VM видит их как обычные locals (через `LOAD_FAST` opcode). В 3.9+ pytest использует
`PyFrame_FastToLocalsWithError()` для быстрого fill locals из stack.

```python
# _pytest/fixtures.py: fixture execution в frame context
def _setupstate_setup(fixturefunc, func):  # Пример упрощённый
    # Создаём scope frame (function scope)
    frame = PyFrame_New(..., fixturefunc.__code__, ...)
    # Заполняем f_localsplus из fixture args
    PyFrame_LocalsToFast(frame, 0)  # Копируем dict → locals array
    try:
        retval = _PyEval_EvalFrameDefault(frame, 0)  # Выполняем fixture
        # yield fixture value в test frame locals
        test_frame.f_localsplus[arg_idx] = retval
    finally:
        PyFrame_FastToLocals(frame)  # Cleanup
```

Fixture — как "временный работник". Pytest создаёт мини-frame (PyFrameObject) для `def fixture(): yield data`, запускает
VM на нём (`_PyEval_EvalFrameDefault()` в ceval.c), достаёт `yield`-значение. Потом в тест-frame (твоя
`def test_foo(fixture):`) через `f_localsplus[INDEX]` пихает значение локалки. CPython VM думает, что это обычная
переменная — `LOAD_FAST fixture` берёт из массива locals. После теста — `Py_DecRef()` и сборщик мусора чистит.

## Assert rewriting: Байткод интерпретация на C-уровне

Pytest переписывает `assert` в runtime: парсит AST → заменяет на `STORE_ATTR + LOAD_ATTR + PyObject_RichCompareBool()` (
Objects/object.c). В 3.9+ использует `ceval_macros.h` specialization для `COMPARE_OP` opcodes. Monkey-patch в
`PyAST_Compile()` или post-compile.

```c
// Из CPython Python/ceval.c (3.12, ASSERT introspection)
// В _PyEval_EvalFrameDefault() для COMPARE_OP (opcode 109):
case COMPARE_OP: {
    PyObject *v = POP();  // Вытаскиваем right из stack
    PyObject *w = TOP();  // left остаётся на top stack
    PyObject *res = PyObject_RichCompare(w, v, oparg);  // ==, !=, etc
    Py_DECREF(v);
    if (res == NULL) goto error;  // Exception handling
    // Pytest hook: _pytest.assertion.rewrite() вставляет expr_info
    if (frame->f_code->co_flags & PY_CODE_FLAG_HAS_ASSERT_INFO) {
        // Сохраняем в f_lasti для traceback formatting
        frame->f_lineno = GET_EXPR_LINE(expr_info, instr);
    }
    PUSH(res);  // Bool result на stack
    ADVANCE_RAW(1, NEXTOPARG());
    goto fast_next_opcode;
}
```

Обычный `assert a == b` компилируется в `COMPARE_OP 2` (==). Pytest во время коллекции rewrite'ит байткод: добавляет
`co_assert_info` в code object. При выполнении VM в `ceval.c` видит флаг, сохраняет строку выражения и lineno. При
ошибке (`res == Py_False`) VM кидает `AssertionError` с полным diff: `assert a == b\n  + actual: 42\n  - expected: 24`.
Это C-логика `PyErr_Format()` + `formatassertion()` в pytest.

## Plugin hooks: pm.hook.pytest_runtest_setup() → Frame tracing

Pytest PluginManager вызывает hooks через `pluggy.call_all_extras()` → рекурсивно exec в frames. Tracing через
`PyTraceFunc` в `PyFrameObject.f_trace`, установленном в `sys.settrace(pytest.apitrace())`.

```python
# pluggy/_manager.py + _pytest/runner.py
def _hookexec(hook, hook_impls, caller):  # Вызов pytest_runtest_loop
    for impl in reversed(hook_impls):  # По приоритету
        args = [hook._hookexec_kwargnames[i](collector) for i in ...]
        frame = PyFrame_New(caller.__code__, globals=hook_impl.__globals__)
        res = _PyEval_EvalFrameDefault(frame, throwflag=0)
        if res is _HOOKERROR:  # Yield/wrapper
            outcome = yield MultiCallOutcome(res)
```

Hooks — цепочка колбэков. PluginManager создаёт PyFrameObject для каждого `def pytest_runtest_setup(item):`, пихает
`item` в locals, запускает VM. Результаты собирает в `MultiCallOutcome`. Для `runtest_setup`/`teardown`/`call` —
последовательная цепочка frames в `f_back`. CPython стек-трейсинг (`f_trace`) ловит переходы между ними. Всё через
стандартный eval loop без C-патчей.

Pytest overhead минимален (~5-10% от unittest в CPython benchmarks), т.к. использует native байткод + C-speed locals.
Для Core Developer: фокус на `PyCodeObject.co_flags` (CO_OPTIMIZED, CO_NEWLOCALS) и `ceval.c` dispatch loop.[2][1]

- [Содержание](CONTENTS.md#содержание)

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

## **Архитектурное погружение в систему плагинов и хуков**

### **1. Pluggy: сердце расширяемости Pytest**

**Pluggy** — это независимая библиотека для создания систем плагинов, которую pytest использует как фундамент. Понимание
её работы критически для создания сложных плагинов и кастомизации поведения pytest.

**Ключевые компоненты pluggy:**

```
# Концептуальная схема
┌─────────────────────────────────────────────┐
│          Pytest Core (встроенные плагины)   │
├─────────────────────────────────────────────┤
│         Система хуков через pluggy          │
│  ┌─────────────┐  ┌─────────────┐           │
│  │ HookSpec    │  │ HookImpl    │           │
│  │ (контракт)  │  │ (реализация)│           │
│  └─────────────┘  └─────────────┘           │
│          │             │                    │
│          └─────┬───────┘                    │
│                ▼                            │
│        HookCaller (диспетчер)               │
│        с ordered_hookimpls                  │
└─────────────────────────────────────────────┘
```

**HookSpec (спецификация хука)** — определяет интерфейс: какие параметры хук принимает, что возвращает. Это контракт,
который должны соблюдать все реализации.

**HookImpl (реализация хука)** — конкретная функция плагина, которая реализует логику хука. Может иметь модификаторы:

- `tryfirst=True` — выполнится раньше других
- `trylast=True` — выполнится позже других
- `hookwrapper=True` — становится обёрткой (wrapper)

**HookCaller** — объект, который управляет вызовом всех HookImpl для конкретного хука. Хранит их в порядке выполнения.

### **2. Порядок выполнения хуков и приоритеты**

Когда pytest вызывает хук (например, `pytest_runtest_setup`), HookCaller выполняет все зарегистрированные реализации в
строгом порядке:

```
1. Реализации с tryfirst=True (в порядке регистрации)
2. Обычные реализации (без tryfirst/trylast)
3. Реализации с trylast=True (в порядке регистрации)
4. Hook wrappers (обёртки) — особый случай
```

**Hook wrappers (обёртки)** — это генераторные функции, которые могут выполнять код до и после основного выполнения
хука:

```python
@pytest.hookimpl(hookwrapper=True)
def pytest_runtest_setup(item):
    # Код ДО выполнения остальных реализаций
    start_time = time.time()

    # yield передаёт управление другим реализациям
    outcome = yield  # Получаем результат выполнения "внутренних" хуков

    # Код ПОСЛЕ выполнения остальных реализаций
    duration = time.time() - start_time

    # Можно модифицировать результат
    if outcome.excinfo:
        logger.error(f"Test setup failed after {duration}s")
    elif duration > 5.0:
        logger.warning(f"Slow setup: {duration}s")
```

### **3. Внутренние объекты Pytest: Config и Item**

**Config объект** (`pytest.Config`) — это центральный реестр, который:

- Содержит все зарегистрированные плагины
- Хранит состояние сессии тестирования
- Предоставляет доступ к менеджеру плагинов через `config.pluginmanager`
- Может использоваться плагинами для обмена данными через `config.myplugin_data`

**Item объект** — представляет конкретный тест (функцию или метод). Каждый Item имеет:

- `_request` — объект с контекстом выполнения (включая разрешённые фикстуры)
- `user_properties` — словарь для хранения пользовательских данных
- Методы для управления состоянием теста

**Динамическая работа с Item в хуках:**

```python
def pytest_runtest_setup(item):
    # Добавляем пользовательские метаданные
    item.user_properties.append(("test_module", item.module.__name__))

    # Модифицируем имя теста для отчёта
    if hasattr(item, "_metadata"):
        item._metadata["custom_name"] = f"{item.name}_modified"
```

### **4. Динамическая регистрация и расширение системы**

Плагины могут не только реализовывать существующие хуки, но и расширять систему, добавляя новые:

```python
class MyPlugin:
    @pytest.hookspec
    def pytest_my_custom_hook(self, config, data):
        """Новый хук, который могут реализовывать другие плагины"""
        pass

    def pytest_addhooks(self, pluginmanager):
        # Регистрируем спецификацию нового хука
        pluginmanager.add_hookspecs(MyPlugin)


class AnotherPlugin:
    @pytest.hookimpl
    def pytest_my_custom_hook(self, config, data):
        # Реализация нового хука
        return process_data(data)
```

**Важно:** динамическая регистрация требует глубокого понимания порядка инициализации плагинов, чтобы избежать
конфликтов.

### **5. Внутренние механизмы выполнения: от байткода до фреймов**

**PluginManager и регистрация хуков:**

Когда плагин регистрируется через `pluginmanager.register()`, происходит:

1. Сканирование всех атрибутов плагина
2. Поиск функций с атрибутом `pytest_impl` (добавляется декоратором `@pytest.hookimpl`)
3. Создание объектов `HookImpl` с информацией о реализации
4. Добавление в соответствующий `HookCaller`

**Выполнение хуков (multicall):**

При вызове хука (`config.hook.pytest_runtest_setup(item=item)`) происходит:

```python
# Упрощённая схема multicall
def _multicall(hook_impls, kwargs):
    results = []

    for hook_impl in hook_impls:
        if hook_impl.hookwrapper:
            # Для wrapper-хуков
            gen = hook_impl.function(**kwargs)
            outcome = next(gen)  # Выполняем код до yield
            try:
                # Рекурсивно вызываем остальные хуки
                inner_results = _multicall(next_impls, kwargs)
                gen.send(inner_results)  # Выполняем код после yield
            except Exception:
                gen.throw(*sys.exc_info())
        else:
            # Для обычных хуков
            result = hook_impl.function(**kwargs)
            results.append(result)

    return results
```

**CPython-уровень:** Каждый вызов хука создаёт новый фрейм выполнения (`PyFrameObject`), который CPython интерпретатор
обрабатывает в своем цикле байт-кода.

### **6. Обёрточные хуки (wrapper hooks) и Outcome**

Обёрточные хуки используют механизм генераторов Python для перехвата выполнения:

```
Порядок выполнения для wrapper хука:
1. next(wrapper_gen) → код ДО yield
2. Выполняются все не-wrapper реализации
3. wrapper_gen.send(results) → код ПОСЛЕ yield
4. Исключения можно обработать через outcome.excinfo
```

### **7. Валидация и контракты через HookSpec**

Спецификации хуков не только документируют интерфейс, но и выполняют валидацию:

```python
# В hookspec.py
@pytest.hookspec
def pytest_runtest_setup(item):
    """Вызывается перед выполнением теста.
    
    :param item: тестовый элемент (Item)
    """
    pass

# При регистрации реализации проверяется:
# 1. Соответствие сигнатуры
# 2. Наличие обязательных параметров
# 3. Совместимость возвращаемых типов (если используются аннотации)
```

### **8. Производительность и оптимизация**

**Накладные расходы системы хуков:**

- ~100-500 наносекунд на простой хук
- ~1-5 микросекунд на хук с несколькими реализациями
- ~10-50 микросекунд на wrapper-хуки

**Оптимизации:**

- Кэширование разрешённых фикстур
- Ленивая загрузка плагинов
- Минимизация количества wrapper-хуков в критичных путях

### **9. Продвинутые сценарии использования**

**Динамическое создание тестов на основе внешних данных:**

```python
def pytest_generate_tests(metafunc):
    """Генерация тестовых случаев динамически"""
    if "api_endpoint" in metafunc.fixturenames:
        endpoints = fetch_endpoints_from_config()
        metafunc.parametrize("api_endpoint", endpoints)


# Или через коллекцию:
def pytest_collection_modifyitems(config, items):
    """Фильтрация и модификация тестов перед выполнением"""
    if config.getoption("--only-smoke"):
        items[:] = [item for item in items
                    if hasattr(item, "smoke") and item.smoke]
```

**Интеграция с распределёнными системами:**

```python
class DistributedTestPlugin:
    def __init__(self):
        self.test_queue = []
        self.results = {}

    def pytest_collection_modifyitems(self, config, items):
        # Распределяем тесты по worker-нодам
        for i, item in enumerate(items):
            worker_id = i % config.option.num_workers
            self.assign_test_to_worker(item, worker_id)

    def pytest_runtest_protocol(self, item, nextitem):
        # Управляем выполнением теста в распределённой среде
        if not self.is_my_worker(item):
            return True  # Пропускаем тест на этой ноде

        # Выполняем тест локально
        return None  # Продолжаем стандартный протокол
```

### **10. Отладка и диагностика**

**Инструменты для отладки системы плагинов:**

- `pytest --trace-config` — показывает загрузку плагинов
- `pytest --debug` — выводит внутреннюю отладочную информацию
- Кастомные плагины для трассировки вызовов хуков:

```python
class HookTracerPlugin:
    def __init__(self):
        self.hook_calls = []

    @pytest.hookimpl(hookwrapper=True)
    def pytest_runtest_call(self, item):
        start = time.perf_counter()
        outcome = yield
        duration = time.perf_counter() - start
        self.hook_calls.append({
            "hook": "pytest_runtest_call",
            "item": item.name,
            "duration": duration,
            "success": outcome.excinfo is None
        })
```

**Итог для Senior уровня:** Понимание внутренней архитектуры pytest и pluggy позволяет не только создавать сложные
плагины, но и предсказывать поведение системы в нестандартных сценариях, оптимизировать производительность и
интегрировать pytest с любыми внешними системами, от CI/CD пайплайнов до распределённых вычислительных кластеров.

- [Содержание](CONTENTS.md#содержание)

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

### **1. Что такое Kubernetes по сути**

Kubernetes — это не просто “оркестратор контейнеров”.  
Это **распределённая система управления состоянием**.  
Ты описываешь *желаемое состояние* в виде YAML-манифестов: “в кластере должно быть 3 копии сервиса X”.  
Дальше всё работает по принципу **control loop (петли управления)**:

1. Ты создаёшь или изменяешь объект — YAML попадает в **API Server**.
2. **etcd** хранит текущее состояние кластера (это как база истины).
3. **Controller Manager** периодически читает etcd и сравнивает: “Ага, пользователь хочет 3 Pod’а, а запущено 2. Надо
   создать ещё один.”
4. **Scheduler** решает, где разместить новый Pod.

Вся логика — это бесконечное выравнивание «требуемого» и «реального». Это философия declarative infra.  
Когда тест QA‑окружения “упал” или Pod умер — Kubernetes не “знает причину”, ему всё равно. Он просто видит расхождение
и возвращает нужное количество экземпляров.

***

### **2. Как Kubernetes мыслит "внутри"**

У Kubernetes нет “особого режима для staging или тестов”.  
Всё в нём — это ресурсы: Pod, Deployment, Service, CRD.  
По сути, кластер — это **огромная REST API**, где каждый ресурс — просто JSON-запись в etcd.

Когда ты делаешь `kubectl apply -f deployment.yaml`, клиент шлёт PATCH в API Server, а сервер обновляет CRD‑объект в
etcd.  
Контроллер (например, ReplicaSet Controller) получает ивент ("deployment изменился"), проверяет состояние, и совершает
действия (создаёт, апдейтит или удаляет поды).

***

### **3. Как это влияет на работу QA-инженера**

Представь, что тебе нужно проводить тесты в “живых” окружениях, где код в каждом PR должен разворачиваться как
mini‑production.  
Традиционно это боль, потому что вручную поддерживать десятки окружений невозможно.  
Kubernetes решает это за счёт своей **декларативности и изоляции через Namespace**.

Пример:

1. В CI пайплайне после создания Pull Request генерируется уникальный namespace, например `qa-pr-1234`.
2. Helm-чарт деплоит туда приложение при помощи Deployment + Service + Ingress.
3. QA-тест запускается как `Job`, внутри которого тест-фреймворк (pytest или robot) стучится к
   `app.qa-pr-1234.svc.cluster.local`.
4. После завершения CI всё очищается: namespace просто удаляется — и Kubernetes сам чистит все связанные ресурсы.

**Ты не управляешь окружениями — ты их описываешь.**

***

### **4. Scheduler и реальная магия автоматики**

Scheduler — это ядро интеллектуальности Kubernetes.  
Он решает, где запустить Pod, основываясь на доступных ресурсах, taint/toleration, affinity/anti-affinity и приоритетах.

Для QA‑нагрузок это важно, потому что можно описать, где именно должны жить твои тестовые контейнеры. Например:

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
            - key: "role"
              operator: In
              values:
                - "qa-runner"
```

Это гарантирует, что тесты не утащат CPU у продакшена.  
А ещё Kubernetes умеет **preemption** — “вытеснение” менее приоритетных Pod’ов. Например, если кластер забит, тест‑Job
можно выкинуть первым, сохранив стабильный прод.

***

### **5. Взгляд на Kubernetes как платформу для автоматизации тестов**

На уровне Senior Kubernetes воспринимается не как “где запустить тест”, а как **платформа для распределённого исполнения
и самоисцеления**.

Можно использовать его как “фреймворк для тестовых распределённых систем”.  
Пример — запуск тестов как Kubernetes Custom Resource:

```yaml
apiVersion: qa.company.io/v1
kind: TestRun
metadata:
  name: regression-suite
spec:
  repo: git@github.com:team/backend
  branch: feature/login_fix
  parallelism: 20
  env: staging
```

Дальше твой кастомный контроллер (на Python, с `kubernetes` SDK) берёт это описание, деплоит нужные Pod’ы и следит за
статусом.  
Контроллер просто повторяет паттерн Kubernetes: “desired → actual”.  
Так QA-система становится частью экосистемы Kubernetes, а не надстройкой сверху.

- [Содержание](CONTENTS.md#содержание)

---

# **unittest framework**

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **Mocking и патчинг**

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **Тестовые двойники**

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **Фабрики тестовых данных**

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **Параметризация тестов**

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **Фикстуры разных уровней**

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **Allure отчеты**

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **Selenium/Playwright**

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **Requests для API тестирования**

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **BDD frameworks**

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **Docker**

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **CI/CD системы**

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **Конфигурация окружений**

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **Системы очередей**

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **Базы данных в тестах**

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **SQLAlchemy**

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **Alembic**

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **Pydantic**

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **FastAPI/Flask/Django**

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **Celery**

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **Websockets**

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **Paramiko**

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **Prometheus/Grafana**

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **Профилирование кода**

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **Статический анализ**

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **Анализ покрытия**

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **Бенчмаркинг**

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

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

- [Содержание](CONTENTS.md#содержание)

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

# **Единая классификация видов тестирования**

Вместо хаотичного списка лучше представлять тестирование как многомерный куб. Один и тот же тест может быть одновременно
**системным**, **функциональным**, **автоматизированным** и **регрессионным**.

Ниже — структурированное разделение по ключевым измерениям (Dimensions).

***

## **1. По объекту тестирования (Что проверяем?)**

Это самое главное деление: проверяем мы бизнес-функции или качество реализации.

### **1.1 Функциональное тестирование (Functional)**

Отвечает на вопрос: **«Что система делает?»**.
Мы проверяем, решает ли программа задачи пользователя.

* **Функциональное (Functional):** Проверка бизнес-сценариев. Работает ли логин? Считается ли скидка в корзине?
* **Взаимодействия (Interoperability):** Может ли наша система общаться с другими (например, корректно ли мы шлем данные
  в 1С или платежный шлюз).

**Внутри функционального выделяют подходы:**

* **Позитивное:** «Счастливый путь» (Happy Path). Вводим корректные данные, ожидаем успех.
* **Негативное:** Вводим мусор, спецсимволы, null. Проверяем, что система не падает, а вежливо сообщает об ошибке.

### **1.2 Нефункциональное тестирование (Non-functional)**

Отвечает на вопрос: **«Как система работает?»**.
Функция может работать, но если страница грузится 30 секунд — это баг.

* **Производительности (Performance):** Общее понятие скорости и ресурсов.
    * *Нагрузочное (Load):* Как ведет себя система при **штатной** ожидаемой нагрузке.
    * *Стрессовое (Stress):* Найти точку отказа. Даем нагрузку выше максимума, пока сервер не упадет (или не
      восстановится).
    * *Стабильности/Надежности (Stability/Soak/Endurance):* Тест на выносливость. Работаем под средней нагрузкой долго (
      24+ часа), ищем утечки памяти.
    * *Объемное (Volume):* Как система работает с огромной базой данных (миллионы записей).
    * *Масштабируемости (Scalability):* Если добавить железа (CPU/RAM), вырастет ли производительность пропорционально?
* **Безопасности (Security):** Проверка на уязвимости (SQL-инъекции, XSS), разграничение прав доступа (может ли юзер
  видеть админку) и конфиденциальность.
* **Удобства использования (Usability/UX):** Насколько удобно и понятно пользователю. Это про интуитивность интерфейса,
  а не только про красоту.
* **Доступности (Accessibility/a11y):** Могут ли сайтом пользоваться люди с ограничениями (скринридеры, цветовая
  слепота).
* **Совместимости (Compatibility):**
    * *Кроссбраузерное:* Chrome, Firefox, Safari.
    * *Кроссплатформенное:* iOS vs Android, Windows vs Linux.
* **Локализации (Localization/L10n):** Проверка перевода, форматов дат, валют и направления текста (RTL).
* **Установки и конфигурирования (Installation & Configuration):** Как софт ставится, обновляется и удаляется.

***

## **2. По уровню детализации (Пирамида тестирования)**

На каком уровне архитектуры мы находимся.

1. **Модульное (Unit):** Самый низкий уровень. Проверяем отдельную функцию или класс в изоляции. Делают разработчики.
   Быстро, дешево.
2. **Интеграционное (Integration):** Проверка стыков. Как два модуля (или сервис + база данных) общаются друг с другом.
3. **Системное (System / E2E):** Проверка системы целиком, как черный ящик. Максимально близко к действиям реального
   пользователя.
4. **Приемочное (Acceptance):** Финальный этап. Заказчик (или PM) смотрит и говорит: «Да, это то, что я заказывал».

***

## **3. По знанию системы (Доступ к коду)**

* **Черный ящик (Black Box):** Мы не видим код. Знаем только вход (требования) и выход. Мы — как пользователь.
* **Белый ящик (White Box):** Мы видим код, знаем структуру БД, алгоритмы. Пишем тесты, чтобы покрыть конкретные ветки
  кода (Statement/Branch coverage).
* **Серый ящик (Grey Box):** Мы работаем как пользователь (через UI/API), но можем заглянуть в БД или логи, чтобы
  проверить, правильно ли записались данные.

***

## **4. По хронологии и изменениям (Когда запускаем?)**

* **Дымовое (Smoke):** «Включается ли вообще?». Быстрая проверка критического функционала после сборки. Если дым идет —
  дальше не тестируем.
* **Санитарное (Sanity):** Проверка **конкретной** области после исправлений. Убеждаемся, что *именно этот* баг починили
  и смежные функции работают. Узконаправленно.
* **Регрессионное (Regression):** Проверка **всей** старой функциональности после внесения изменений. Убеждаемся, что
  новый код не сломал старый.
* **Подтверждающее (Re-testing):** Просто перепроверка баг-репорта. Был баг -> разраб исправил -> мы проверили (
  Re-test).

***

## **5. По степени автоматизации**

* **Ручное (Manual):** Человек кликает мышкой. Незаменимо для UX и исследовательского тестирования.
* **Автоматизированное (Automated):** Скрипты выполняют проверки. Идеально для регресса и нагрузки.
* **Полуавтоматизированное:** Человек запускает скрипты, которые генерируют данные, но решение «Правильно/Неправильно»
  принимает сам.

***

## **6. По степени формализации**

* **Сценарное (Scripted):** Строго по тест-кейсам. Шаг влево, шаг вправо — расстрел.
* **Исследовательское (Exploratory):** Тестировщик одновременно изучает систему, придумывает тесты и выполняет их.
  Требует опыта и интуиции.
* **Ad-hoc (Интуитивное):** «Метод тыка». Бессистемное тестирование без подготовки. Иногда помогает найти самые странные
  баги.

***

# **Техническая реализация (Python Context)**

Как Senior QA Automation, вы должны знать не только *названия* видов, но и *инструменты* для них.

### **1. Функциональное тестирование**

* **API:** Основной рабочий инструмент.
    * *Libs:* `requests` (синхронно), `aiohttp`/`httpx` (асинхронно).
    * *Framework:* `pytest` — стандарт индустрии.
    * *Schema validation:* `Pydantic` или `jsonschema` (валидировать контракты ответов).
* **UI (E2E):**
    * *Tools:* `Selenium WebDriver` (классика), `Playwright` (современный, быстрый, стабильный).
    * *Pattern:* Page Object Model (POM) — обязательно для разделения локаторов и логики теста.
* **Mobile (Android/iOS):**
    * *Tool:* `Appium` (клиент на Python). Знание `ADB` и `uiautomator2`.

### **2. Нефункциональное тестирование**

* **Load (Нагрузка):**
    * `Locust`: Пишется на чистом Python. Отлично подходит для проверки API под нагрузкой.
    * `K6`: (JS/Go), но можно запускать и анализировать через Python-обвязки.
* **Security (Безопасность):**
    * Статический анализ зависимостей: `safety` (проверка requirements.txt на дыры).
    * Сканнеры: `OWASP ZAP` (можно управлять через API).

### **3. Unit & Integration (Белый ящик)**

* **Mocking:** `unittest.mock` (Mock, MagicMock, patch). Умение изолировать тест от внешнего API или БД.
* **Database:** Использование фикстур (`pytest fixtures`) для подготовки и очистки тестовых данных в БД (SQLAlchemy/Raw
  SQL).
* **Coverage:** `pytest-cov` — посмотреть, какой процент кода задет тестами.

### **4. CI/CD & Infrastructure**

* **Docker:** Запуск тестов в изолированных контейнерах (`testcontainers-python`).
* **Allure:** Генерация красивых отчетов, понятных менеджменту.
* **GitHub Actions/GitLab CI:** Настройка пайплайнов (запуск смоуков на PR, регресса на релиз).

- [Содержание](CONTENTS.md#содержание)

---

# **Техники тест-дизайна**

## **Junior Level**

Техники проектирования тестов (Test Design Techniques) — это структурированные методы создания тестовых случаев, которые
помогают эффективно и полно проверить систему. Они отвечают на вопрос: "Как придумать хорошие тесты?" и заменяют
случайный перебор данных системным подходом.

Основные техники:

1. **Эквивалентное разделение (Equivalence Partitioning):** Входные данные делятся на группы (классы эквивалентности),
   где поведение системы ожидается одинаковым. Например, для поля "возраст" группы: отрицательные числа (невалидные),
   0-17 (несовершеннолетние), 18-65 (взрослые), больше 65 (пенсионеры). Достаточно протестировать по одному значению из
   каждого класса.
2. **Анализ граничных значений (Boundary Value Analysis):** Фокусируется на тестировании значений на границах этих
   классов, так как именно там чаще всего возникают ошибки. Для диапазона 18-65 проверяются значения: 17, 18, 19 и 64,
   65, 66.
3. **Таблица принятия решений (Decision Table Testing):** Применяется для логики, зависящей от комбинаций условий.
   Создается таблица, где столбцы — это условия и действия, а строки — тестовые сценарии для всех значимых комбинаций.
4. **Тестирование состояний и переходов (State Transition Testing):** Используется для систем с конечным числом
   состояний (например, банкомат: "Ожидание карты" -> "Ввод PIN" -> "Выбор операции"). Тестируются как корректные
   переходы, так и ошибочные (например, ввод неверного PIN-кода).
5. **Тестирование сценариев использования (Use Case Testing):** Система проверяется через призму реальных
   пользовательских сценариев, описывающих, как пользователь взаимодействует с системой для достижения конкретной цели (
   например, "Оформление заказа").
6. **Попарное тестирование (Pairwise Testing):** Метод оптимизации, который позволяет покрыть все возможные пары
   значений входных параметров, сокращая количество тестовых комбинаций до приемлемого уровня.
7. **Предугадывание ошибок (Error Guessing):** Опытный тестировщик на основе знаний о системе, предыдущих дефектах и
   типичных проблемах в подобных продуктах выдвигает гипотезы о возможных ошибках и создает целевые тесты для их
   проверки.

## **Middle Level**

На среднем уровне важно не только знать техники, но и понимать, как эффективно применять их в рамках автоматизации,
управлять тестовыми данными и интегрировать подходы в процесс разработки.

### Особенности применения и автоматизации

1. **Эквивалентное разделение и анализ граничных значений:**
    * **Параметризация тестов:** В pytest с помощью декоратора `@pytest.mark.parametrize` один тест превращается в набор
      проверок для данных из разных классов эквивалентности и граничных значений.
    * **Генерация данных:** Значения можно генерировать динамически или выносить в отдельные фикстуры для повторного
      использования.
    * **Ключевой момент:** Автоматизация позволяет легко и полно покрыть все границы и классы, что сложно сделать
      вручную.

2. **Таблица принятия решений:**
    * **Data-Driven Testing (DDT):** Саму таблицу (например, в формате CSV, JSON или Excel) можно использовать как
      источник данных для тестов. Это делает логику прозрачной и легко обновляемой.
    * **Интеграция:** Инструменты вроде `pandas` упрощают загрузку и обработку табличных данных в тестах.
    * **Преимущество:** Четкое разделение тестовой логики (код) и тестовых данных (таблица).

3. **Тестирование состояний и переходов:**
    * **Паттерн "Конечный автомат" (State Machine):** Логику системы можно смоделировать в коде, что облегчает создание
      тестов, проверяющих корректность переходов.
    * **Последовательности шагов:** Тесты организуются как цепочки действий (например, через фикстуры в `pytest`),
      которые переводят систему из одного состояния в другое с последующей проверкой.
    * **Фокус:** Автоматизация помогает проверить длинные и сложные цепочки переходов, включая обработку нестандартных
      сценариев.

4. **Тестирование сценариев использования:**
    * **BDD-фреймворки:** Инструменты вроде `behave` или `pytest-bdd` позволяют описывать сценарии на языке Gherkin (
      Given-When-Then), что улучшает взаимодействие между разработчиками, тестировщиками и аналитиками.
    * **Паттерн Page Object:** Для UI-автоматизации этот паттерн идеально ложится на сценарии, инкапсулируя логику
      работы с элементами страницы и делая тесты устойчивее к изменениям в верстке.

5. **Попарное тестирование (Pairwise):**
    * **Автоматизация генерации:** Инструменты (`allpairspy`, `pict`) интегрируются в процесс подготовки тестовых
      данных, генерируя минимальный набор комбинаций для покрытия всех пар.
    * **Применение:** Особенно полезно при тестировании конфигураций (ОС x браузер x разрешение экрана) или
      функциональности с множеством независимых параметров.

6. **Предугадывание ошибок:**
    * **Систематизация:** Хотя техника основана на опыте, найденные дефекты и гипотезы можно фиксировать в виде
      автоматизированных проверок и добавлять их в регрессионную тестовую базу.
    * **Анализ рисков:** Метод тесно связан с анализом областей повышенного риска в приложении, что помогает расставлять
      приоритеты при написании автоматизированных тестов.

## **Senior Level**

# Техника: Эквивалентное разделение и Анализ граничных значений

Это не просто «правила хорошего тона», а математически обоснованные методы сокращения бесконечного числа тестов до
конечного набора. В основе лежат теория множеств и эмпирические исследования распределения ошибок в коде.

***

## 1. Научное обоснование (The Science Behind It)

С точки зрения Computer Science, программа — это математическая функция $f(x)$, которая отображает входные данные на
выходные. Тестирование — это попытка найти такие $x$, где $f(x)$ работает некорректно.

### Гипотеза однородности (Basis for Equivalence Partitioning)

Фундамент метода эквивалентных классов — **гипотеза однородности (Homogeneity Hypothesis)**. Она утверждает, что если
один тест из класса эквивалентности выявляет ошибку, то с высокой вероятностью её выявят и все остальные тесты этого
класса. И наоборот: если один тест проходит успешно, остальные тоже пройдут.

> **Суть:** Мы предполагаем, что программа обрабатывает все числа от 1 до 100 *одним и тем же куском кода* (одним путем
> в графе потока управления — Control Flow Graph).

### Гипотеза сгущения ошибок (Basis for BVA)

Анализ граничных значений опирается на **гипотезу граничных значений**. Эмпирически доказано, что вероятность
отказа $P(failure)$ не распределена равномерно. Она имеет резкие пики (спайки) в точках, где меняется логика программы.

**Почему это происходит? Причины кроются в психологии программирования:**

1. **Ошибки на единицу (Off-by-one errors):** Разработчики путают `>` и `>=`, `<` и `<=`.
2. **Инициализация циклов:** Ошибки в `for (i=0; i < N; i++)` часто приводят к пропуску последнего элемента или выходу
   за массив.
3. **Переполнение типов:** Границы часто совпадают с предельными значениями типов данных (`int`, `short`).

## 2. Результаты ключевых исследований

Научные работы последовательно подтверждают эффективность этих методов по сравнению со случайным тестированием (Random
Testing).

### 1. Reid (1997): «Empirical Analysis of EP, BVA and Random Testing»

Стюарт Рейд провел фундаментальное исследование на реальной системе авионики (20 000 строк кода Ada).

* **Результат:** BVA (граничные значения) оказалось **самым эффективным** методом, выявляя ошибки, которые пропускали
  другие методы.
* **Сравнение:** BVA находил почти в 2 раза больше дефектов, чем EP (классы эквивалентности), но требовал кратно больше
  тест-кейсов.
* **Вывод:** EP — дешевле (меньше тестов), но пропускает граничные баги. BVA — дороже, но надежнее.

### 2. Basili & Selby (1987): «Comparing the Effectiveness of Software Testing Strategies»

Классическое исследование, сравнивавшее функциональное тестирование (EP+BVA) со структурным (покрытие кода) и
рецензированием кода (Code Reading).

* **Результат:** Функциональное тестирование (Black-box) показало отличные результаты в обнаружении ошибок инициализации
  и управления, часто превосходя структурные тесты.

### 3. Современные данные (Dobslaw 2023, Hubner 2019)

В эпоху AI и автоматизации исследования показывают, что стратегии генерации тестов, которые "целятся" в границы (
Boundary-guided testing), находят на 30-50% больше мутационных ошибок, чем слепой фаззинг (fuzzing).

# Техника: Таблица принятия решений (Decision Table Testing)

В отличие от граничных значений, которые исследуют *диапазоны*, таблицы решений исследуют *логику* и *комбинаторику*.
Это метод для борьбы с комбинаторным взрывом и цикломатической сложностью.

***

## 1. Научное обоснование

Фундаментом для таблиц решений служат **Булева алгебра** и **Пропозициональная логика**. Любая программа, принимающая
решения, может быть представлена как функция от набора бинарных (или конечных) переменных.

### Проблема, которую решает метод

Человеческий мозг плохо удерживает в оперативной памяти более 3-4 условий одновременно (следствие закона
Миллера $7 \pm 2$). Когда в коде встречаются вложенные `if-else` или зависимые условия, вероятность ошибки (пропущенной
ветки) стремится к 100%.

### Математические свойства (Completeness & Consistency)

Таблицы решений опираются на два строгих математических свойства:

1. **Полнота (Completeness):** Гарантия того, что рассмотрены *все возможные* комбинации входных условий. Для $N$
   бинарных условий существует ровно $2^N$ возможных комбинаций. Если таблица содержит меньше правил, она либо неполна,
   либо сжата (collapsed).
2. **Непротиворечивость (Consistency):** Гарантия того, что одна и та же комбинация условий не ведет к взаимоисключающим
   действиям.

***

## 2. Результаты исследований

Научные работы подтверждают, что таблицы решений (DT) превосходят другие методы в выявлении логических ошибок, но могут
быть избыточны.

### 1. Subramanian (1992): «A comparison of the decision table and tree»

Исследование эффективности представления логики.

* **Результат:** Тестировщики и аналитики, использующие табличное представление (Decision Tables), совершали
  статистически значимо **меньше ошибок** при анализе сложной логики по сравнению с теми, кто использовал деревья
  решений (Decision Trees) или текстовые спецификации.
* **Вывод:** Табличная структура снижает когнитивную нагрузку и позволяет быстрее замечать пропущенные сценарии (gaps).

### 2. Сравнение с Boundary Value Analysis (Ferriday, 2007)

* **Результат:** BVA генерирует примерно в 5 раз больше тест-кейсов, чем Decision Tables, но DT находит специфический
  класс ошибок — **ошибки взаимодействия условий** (interaction faults), которые BVA пропускает полностью.
* **Эффективность:** DT выявляет "логические дыры" (ситуации, когда спецификация молчит о поведении системы), в то время
  как другие методы проверяют только написанное.

### 3. Shiffman (1997): «Representation of Clinical Practice Guidelines»

Исследование на критических системах (медицина).

* **Результат:** Применение таблиц решений к медицинским алгоритмам позволило выявить логическую неполноту (missing
  rules) и противоречия в клинических рекомендациях, которые не были замечены экспертами-людьми при обычном чтении
  текста.

# Техника: Тестирование состояний и переходов (State Transition Testing)

Этот вид тестирования радикально отличается от предыдущих. Если *Эквивалентное разделение* работает с «моментальными»
данными (stateless), то *State Transition Testing* проверяет «память» системы.

Это метод для проверки сложной бизнес-логики, где ответ системы зависит не только от того, **что** вы нажали, но и от
того, **в каком состоянии** система находилась до этого.

***

## 1. Научное обоснование (Scientific Basis)

В основе метода лежит раздел дискретной математики — **Теория конечных автоматов (Automata Theory)**.

### Формальная модель

Любую систему с состояниями можно описать как кортеж $(S, I, \delta)$, где:

* $S$ — конечное множество состояний (States).
* $I$ — множество входных сигналов (Inputs/Events).
* $\delta$ — функция перехода: $S_{current} \times I \rightarrow S_{new}$.

Это означает, что **реакция системы является функцией от её истории**. В отличие от простых функций $y=f(x)$,
здесь $y=f(x, state)$.

### Почему это необходимо?

Классические методы (EP, BVA) бессильны против ошибок последовательности.

* *Пример:* Нажатие кнопки «Оплатить» с валидной картой (EP/BVA говорят "ОК") должно приводить к успеху *только* в
  состоянии «Заказ создан», но должно вызывать ошибку в состоянии «Заказ уже оплачен». Без учета состояния мы пропустим
  этот баг.

***

## 2. Результаты исследований

Эффективность метода подтверждена десятилетиями исследований в области надежности ПО, особенно в embedded-системах и
телекоме.

### 1. Фундаментальная работа T.S. Chow (1978)

Статья «Testing Software Design Modeled by Finite-State Machines» является библией этого метода.

* **Результат:** Чоу доказал, что тестирование на основе автоматов гарантированно обнаруживает определенные классы
  ошибок, которые невозможно найти другими способами:
    * **Operation errors:** Неверный выходной результат при переходе.
    * **Transfer errors:** Переход не в то состояние.
    * **Extra/Missing states:** Лишние или недостижимые состояния.
* **Вклад:** Он ввел понятие **N-switch coverage** (покрытие последовательностей длины N), показав, что проверки
  одиночных переходов (0-switch) недостаточно для выявления сложных багов.

### 2. Offutt et al. (2003) — State-based Specification Testing

Джефф Оффат исследовал генерацию тестов из UML-диаграмм состояний.

* **Результат:** Автоматическая генерация тестов на основе состояний (State-based) находит глубокие логические ошибки,
  которые пропускают люди при написании тестов вручную, так как люди склонны проверять только «счастливые пути» (Happy
  Path) переходов.

### 3. Сравнение эффективности (Holt, 2014)

Исследование на промышленном ПО:

* **Результат:** State-Based Testing (SBT) с использованием строгих оракулов (проверок) позволяет обнаруживать дефекты
  управления потоком эффективнее, чем покрытие кода (Code Coverage). Удаление деталей из модели состояния (упрощение)
  снижает стоимость тестирования на 85%, но снижает эффективность обнаружения багов всего на ~30%, что делает метод
  рентабельным даже в упрощенном виде.

# Техника: Тестирование сценариев использования (Use Case Testing)

Если предыдущие техники (EP, BVA, Decision Table) были атомарными проверками «вход-выход», то **Use Case Testing** — это
тестирование *потоков* и *целей* пользователя. Это переход от проверки «как работает код» к проверке «как работает
бизнес-процесс».

***

## 1. Научное обоснование

В основе метода лежит **ориентированный на пользователя подход (User-Centered Design)** и **акторно-сетевая теория**.

### Концептуальная модель

Система рассматривается не как набор функций, а как "черный ящик", с которым взаимодействуют внешние сущности — **Акторы
** (пользователи, другие системы, таймеры), чтобы достичь определенной **Цели**.

* **Целеполагание:** Научно доказано, что пользователи не используют ПО ради функций (нажать кнопку). Они используют его
  для решения задач (купить билет, отправить отчет). Тестирование Use Case валидирует именно достижение целей.
* **Эвристика Парето (80/20):** Исследования показывают, что 80% времени пользователи используют 20% функционала (
  основные сценарии). Use Case Testing гарантирует, что именно эти критические 20% работают безупречно.

***

## 2. Результаты исследований

Научные данные подтверждают, что этот метод критически важен для нахождения дефектов *интеграции* и *требований*,
которые пропускают unit-тесты.

### 1. Ivar Jacobson (1992): «Object-Oriented Software Engineering»

Ивар Якобсон, создатель термина "Use Case" (изначально *Användningsfall* на шведском), доказал в своих работах, что
построение разработки и тестирования вокруг сценариев использования ("Use Case Driven") снижает количество архитектурных
ошибок на ранних стадиях.

### 2. Sophocleous et al. (2020): «Examining the Current State of System Testing»

Исследование 252 QA-инженеров и промышленных кейсов показало:

* **Результат:** Тестирование на основе реальных пользовательских сценариев (в комбинации с smoke/regression)
  статистически значимо ($p = 0.000$) снижает количество дефектов, обнаруженных конечными пользователями после
  релиза.[3]
* **Вывод:** Чем ближе тесты к реальным сценариям использования, тем выше удовлетворенность заказчика (User Acceptance).

### 3. Gutierrez et al.: «A Case Study for Generating Test Cases from Use Cases»

Исследование автоматической генерации тестов:

* **Результат:** Метод анализа сценариев (Scenario Analysis) позволяет выявить пропущенные пути в требованиях (gaps),
  которые не очевидны при просмотре списка требований списком. Тесты, сгенерированные из Use Case, имеют более высокое
  покрытие *бизнес-логики* по сравнению с тестами, основанными на структуре кода.[4]

# Техника: Попарное тестирование (Pairwise Testing)

Это метод-скальпель: он отсекает 99% тестов, сохраняя 95% эффективности. Это не магия, а прикладная комбинаторика.

***

## 1. Научное обоснование

**Проблема:** Комбинаторный взрыв. Если у вас 10 параметров по 10 значений в каждом, вам нужно $10^{10}$ тестов (10
миллиардов). Это невозможно выполнить.

**Решение (Эмпирический закон):** Ошибки в ПО крайне редко вызываются сложным взаимодействием 3-х и более параметров
одновременно.

* **Single-mode faults:** 20-30% багов вызываются одним параметром (ловится BVA/EP).
* **Double-mode faults:** 50-70% багов возникают на стыке **ДВУХ** параметров (например, "Шрифт=Arial" + "Принтер=HP").
* **Multi-mode faults:** Баги, требующие 3+ условий, составляют менее 5-10% (для некритических систем).[1]

**Математическая база:** Метод основан на **Ортогональных массивах (Orthogonal Arrays)** и **Covering Arrays**.
Ортогональный массив $L_N(S^k)$ гарантирует, что для любых двух колонок (параметров) *каждая возможная пара значений*
встречается ровно один раз (или как минимум один раз для Covering Arrays).[2][3]

Это позволяет сократить $10^{10}$ тестов до $\approx 100-200$, сохранив покрытие всех парных взаимодействий.

***

## 2. Результаты исследований

### 1. NIST (Wallace & Kuhn, 2004) — Исследование "магического числа"

Национальный институт стандартов и технологий США (NIST) проанализировал базы багов NASA (космические аппараты),
медицинских устройств и браузеров.

* **Результат:**
    * **98%** всех дефектов в медицинском ПО выявляются тестированием **пар** (2-way testing).[4]
    * В сложных системах (NASA) для выявления 100% багов требовалось тестирование взаимодействия до 6 параметров (
      6-way).
    * Кривая насыщения: 2-way ловит ~80-90% багов, 3-way ~95%, 4-way ~99%.[1]
* **Вывод:** Pairwise (2-way) — это "золотой стандарт" по соотношению цена/качество. Для критических модулей стоит
  использовать 3-way или 4-way.

### 2. Charbachi (2017) — Сравнение с ручным тестированием

* **Результат:** Pairwise-тесты находили примерно столько же багов, сколько тесты, написанные опытными инженерами
  вручную, но обеспечивали **более высокое покрытие кода (Code Coverage)** за счет неочевидных комбинаций, о которых
  люди часто не задумываются.[5]

### 3. Wood (2016) — Применение в фармакологии

Любопытный факт: принцип работает не только в IT. В биологии 80% реакций на "коктейль" из лекарств также предсказываются
попарным взаимодействием компонентов, что подтверждает универсальность закона "малых взаимодействий".[6]

# Техника: Предугадывание ошибок (Error Guessing)

Этот метод часто недооценивают, называя «интуицией», но в инженерной психологии он имеет строгое обоснование. Это не
гадание, а **применение неявного знания (Implicit Knowledge)** и распознавание паттернов.

***

## 1. Научное обоснование

### Когнитивная психология: Модель RPD

С точки зрения когнитивистики (Gary Klein), эксперт использует **модель принятия решений по распознаванию (
Recognition-Primed Decision, RPD)**. Мозг опытного инженера хранит тысячи паттернов «ситуация -> ошибка» и при виде
знакомого кода подсознательно «подсвечивает» опасные места.

* **Гипотеза кластеризации дефектов (Pareto Principle):** Принцип Парето работает и здесь: 80% ошибок содержатся в 20%
  модулей. Error Guessing — это эвристический поиск этих кластеров.
* **Таксономия дефектов (Defect Taxonomy):** Метод опирается на классификацию типичных ошибок (например, Boris Beizer’s
  Taxonomy). Ошибки не уникальны; программисты совершают одни и те же ляпы десятилетиями (деление на ноль, race
  condition, null pointer).

***

## 2. Результаты исследований

Наука подтверждает: опыт бьет формализм в поиске специфических багов, но проигрывает в полноте покрытия.

### 1. Basili & Selby (1987) — Роль экспертизы

В том же исследовании, где сравнивали EP/BVA, было замечено:

* **Результат:** Тестировщики-эксперты, использовавшие «свободный поиск» (фактически Error Guessing), находили самые
  сложные логические ошибки, которые пропускали формальные методы.
* **Нюанс:** Эффективность метода линейно зависит от опыта. Junior QA с этим методом находит близкое к нулю количество
  критических багов.[1]

### 2. Исследования Exploratory Testing (Bhatti, 2010)

Error Guessing является частью исследовательского тестирования.

* **Результат:** Исследовательский подход (основанный на догадках) позволил найти статистически значимо **больше
  дефектов** за единицу времени, чем сценарное тестирование, особенно в категориях UI и Usability.[2]

### 3. MITRE — Seven Pernicious Kingdoms

Исследование CWE (Common Weakness Enumeration)  — это, по сути, глобальная база для Error Guessing в области
безопасности. Она доказывает, что 90% уязвимостей (SQLi, XSS, Buffer Overflow) предсказуемы.[3]

- [Содержание](CONTENTS.md#содержание)

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

Метрики — это приборная панель Senior QA. Без них вы «летите вслепую». Но важно отличать «метрики тщеславия» (Vanity
Metrics), которые выглядят красиво, от «метрик действий» (Actionable Metrics), которые реально влияют на качество.

Инженерия качества (Quality Engineering) опирается на закон Гудхарта: «Когда мера становится целью, она перестает быть
хорошей мерой». Научные исследования сосредоточены на поиске корреляций между метриками и реальной надежностью ПО.

### Связь покрытия кода и плотности дефектов

Одно из самых важных исследований (Malaiya et al., 2002) установило **логарифмическую связь** между покрытием тестами (
Test Coverage) и плотностью дефектов (Defect Density).

* **Результат:** 100% покрытие не гарантирует 0 багов. Однако, покрытие ниже определенного порога (обычно 70-80% для
  ветвей/branches) экспоненциально увеличивает вероятность отказа в продакшене.
* **Нюанс:** Yamashita (2016) показала, что метрики сложности кода (Cyclomatic Complexity) коррелируют с вероятностью
  багов, но имеют форму «перевернутой U» для плотности дефектов: самые сложные файлы часто имеют *меньше* багов на
  строку кода, так как их пишут и тестируют тщательнее.

***

## 2. Ключевые метрики (Senior Level)

### 1. DRE (Defect Removal Efficiency) — "Король метрик"

Это главная метрика эффективности QA-команды. Она показывает, какой процент багов вы нашли *до* релиза.

$$DRE = \frac{Bugs_{QA}}{Bugs_{QA} + Bugs_{Prod}} \times 100\%$$

* **Эталон:** Мировой стандарт для хорошего процесса — **>85%**. Отличный процесс (High Maturity) — **>95%**.
* **Пример:** QA нашли 90 багов. После релиза пользователи нашли еще 10.
  $DRE = 90 / (90 + 10) = 90\%$. Отличный результат.
* **Действие:** Если DRE падает ниже 85%, значит, ваши тесты (или тестовое окружение) не соответствуют реальности.

### 2. Code Coverage (Покрытие кода)

* **Line Coverage:** Бесполезная метрика для Senior. Можно пройти по строке, но не проверить логику.
* **Branch Coverage (Покрытие ветвлений):** Настоящий стандарт. Проверяет `True` и `False` для каждого `if`.
* **Mutation Score:** Самая "честная" метрика. Специальный тул (например, `mutmut` для Python) ломает ваш код. Если
  тесты не упали — покрытие "липовое".

### 3. Defect Density (Плотность дефектов)

Количество багов на 1000 строк кода (KLOC) или на модуль.

* **Применение:** Помогает найти "горячие точки". Если в модуле "Корзина" 15 багов на KLOC, а в "Профиле" — 2, то
  регресс "Корзины" нужно усилить в 3 раза.

### 4. Mean Time To Detect (MTTD) & Mean Time To Repair (MTTR)

Метрики скорости CI/CD.

* **MTTD:** Сколько времени проходит от коммита "багованного" кода до падения теста? (Хорошо: < 15 мин).
* **MTTR:** Сколько времени проходит от обнаружения критического бага до фикса в продакшене?

***

## 3. Техническая реализация (Python)

Как собирать эти метрики автоматически?

### Mutation Testing (Python)

Вместо того чтобы верить отчету `coverage.py` на слово, используем мутационное тестирование.

```bash
# 1. Ставим библиотеку
pip install mutmut

# 2. Запускаем
mutmut run

# 3. Смотрим результаты
mutmut results
```

**Интерпретация:**

* **Killed:** Тест упал (Хорошо! Мы поймали мутанта).
* **Survived:** Тест прошел, хотя код был сломан (Плохо! Тест "дырявый").

**Пример "дырявого" теста:**

```python
def check_age(age):
    return "Adult" if age >= 18 else "Child"


# Плохой тест (Line Coverage 100%, но Mutation Score низкий)
def test_age():
    assert check_age(20) == "Adult"
    # Этот тест не заметит, если мы заменим `>=` на `>` (Boundary Bug)
    # Мутант (age > 18) выживет!
```

### Сбор DRE (Jira/Allure)

DRE нельзя посчитать в коде, это процессный показатель.

1. В Jira помечайте баги метками `found_in_qa` и `found_in_prod`.
2. Настройте JQL-фильтр или дашборд:
   `(labels = found_in_qa) / ((labels = found_in_qa) + (labels = found_in_prod))`

### Резюме для интервью

На вопрос "Какие метрики вы используете?", Senior QA не должен перечислять всё подряд.
**Правильный ответ:**
> "Я фокусируюсь на DRE, чтобы оценивать эффективность фильтрации багов. Для оценки качества автотестов я использую не
> просто Line Coverage, а Branch Coverage и иногда Mutation Score. А для бизнеса важны метрики стабильности релизов (
> Change Failure Rate)."

- [Содержание](CONTENTS.md#содержание)

---

# **Тест-кейсы и чек-листы

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **Баг-репорты и трекеры

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **Тестовая документация

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **Методологии тестирования

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **Метрики качества кода

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **Тестирование в Agile/Scrum

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **Нефункциональное тестирование

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **Эксплораторное тестирование

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **Page Object Model (POM)

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **Тестирование API

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **Тестирование WebSocket

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **Тестирование баз данных

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **Тестирование мобильных приложений

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **Работа с прокси и снифферами

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **Тестирование локализации и интернационализации

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **Доступность (Accessibility) тестирование

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **Кросс-браузерное и кросс-платформенное тестирование

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **Тестирование в различных окружениях

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **Роль QA в команде разработки

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **Test Management системы

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **Риск-ориентированное тестирование

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **Автоматизация vs ручное тестирование

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **Тестирование legacy систем

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **Работа с требованиями и пользовательскими историями

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **Коммуникация с разработчиками и продакт-менеджерами

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **Приоритизация тестовых сценариев

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **Оценка сроков тестирования

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **Менторинг junior QA

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---

# **Техническая документация для тестов

## **Junior Level**

## **Middle Level**

## **Senior Level**

- [Содержание](CONTENTS.md#содержание)

---



Лягушка
