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

[Содержание](/CONTENTS.md#содержание)

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

[Содержание](/CONTENTS.md#содержание)

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

- [Содержание](/CONTENTS.md#содержание)