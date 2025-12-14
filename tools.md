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
