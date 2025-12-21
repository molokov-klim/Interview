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

- [Содержание](/CONTENTS.md#содержание)