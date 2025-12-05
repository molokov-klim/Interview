CONTEXT:

- Я готовлюсь к собеседованию на позицию Senior Python AQA. Не технический специалист будет задавать вопросы
  и сверять ответы с подготовленными эталонными.

RULES:

- Ответ должен быть развернутым и глубоким
- Раскрывать внутреннее устройство Python "под капотом"
- Не приводить примеры кода
- СТРОГО структурировать ответ: определение → внутренняя реализация (под капотом) → особенности → лучшие практики
- Python от 3.9 и выше

OBJECTIVE:

- На каждый мой вопрос давай развернутый ответ, как на собеседовании Senior Python AQA.

QUESTION:

- Декораторы

====

Вот список вопросов:

# 1. Основы

- Контекстные менеджеры (with)
- Генераторы и итераторы
- GIL (Global Interpreter Lock)

# 2. ООП

- __init__, __new__, __call__
- @property, @staticmethod, @classmethod
- Множественное наследование, MRO
- ABC (Abstract Base Classes)
- Протоколы (Protocol) — вы используете в Shadowstep

# 3. Типизация

- typing: Optional, Union, TypeVar, Generic
- Literal, TypedDict, Protocol
- Covariance/Contravariance (редко, но бывает)

Область видимости

Различие между copy() и глубоким копированием

Списковые выражения (List Comprehension)

Lambda функции

Асинхронность (Async/Await)

Изменение списка во время итерации

Встроенные функции map, filter, reduce, zip, enumerate

Garbage Collector

Сложность кода

Cubernetes

pytest

pytest hooks

паттерны проектирования

dataclass
enum

Теория тестирования

1. Пирамида тестирования
    - Unit → Integration → E2E
    - Почему больше unit-тестов?
    - Когда нарушать пирамиду?

2. Виды тестирования
    - Smoke, Sanity, Regression
    - Функциональное vs Нефункциональное
    - Black-box vs White-box vs Grey-box

3. Метрики
    - Code Coverage (Line, Branch, Path)
    - Mutation Testing
    - Flaky tests — как бороться?

4. Test Design Techniques
    - Equivalence Partitioning
    - Boundary Value Analysis
    - Decision Table
    - State Transition
    - Pairwise Testing

5. Автоматизация
    - Когда автоматизировать, когда нет?
    - ROI автотестов
    - Page Object Pattern (вы эксперт)
    - Screenplay Pattern




















