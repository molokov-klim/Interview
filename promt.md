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

- ABC (Abstract Base Classes)

====

Вот список вопросов:

# ООП

- Протоколы (Protocol)
- Паттерны проектирования
- SOLID
- Специфика ООП в Python
- Наследование и композиция (Композиция vs Наследование, Миксины, Diamond Problem)
- Метапрограммирование
- Подводные камни ООП в Python

# Типизация

- typing: Optional, Union, TypeVar, Generic
- Literal, TypedDict, Protocol
- Covariance/Contravariance (редко, но бывает)

# Инструменты

- Cubernetes
- pytest
- pytest hooks

# Теория тестирования

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
    - Page Object Pattern
    - Screenplay Pattern
