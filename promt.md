CONTEXT:

- Я готовлюсь к собеседованию на позицию Senior Python AQA. Не технический специалист будет задавать вопросы
  и сверять ответы с подготовленными эталонными.

RULES:

- Ответ должен быть развернутым и глубоким
- Раскрывать внутреннее устройство Python "под капотом"
- Не приводить примеры кода
- СТРОГО структурировать ответ: junior level | middle level | senior level, где junior - описание простым человеческим
  языком; middle - углубление в технические детали; senior - кровь кишки и безумие, байткод, cpython и т.д (если
  применимо), в общем технарское порно
- Python от 3.9 и выше

OBJECTIVE:

- На каждый мой вопрос давай развернутый ответ, как на собеседовании Senior Python AQA.

QUESTION:

- Встроенные функции

====

Вот список вопросов:

# Базовые знания

- [Типы данных](#типы-данных)
- [*args и **kwargs](#args-и-kwargs)
- [Хеш-таблица](#хеш-таблица)
- Встроенные функции
- Контекстные менеджеры (with)
- Генераторы и итераторы
- Декораторы и замыкания
- GIL (Global Interpreter Lock)
- Изменение списка во время итерации
- Области видимости (scope)
- Lambda-функции
- Comprehensions и генераторные выражения
- copy() и deepcopy()
- Асинхронность (Async/Await)
- Dataclass
- Enum
- Garbage Collector (сборщик мусора)
- Виды сложности кода

# ООП

- Парадигмы ООП
- Абстракция (ООП)
- Инкапсуляция (ООП)
- Наследование (ООП)
- Полиморфизм (ООП)
- Магические о методы
- Инвариантность и ковариантность
- Декораторы классов и методов
- Множественное наследование и MRO
- ABC (Abstract Base Classes)
- Протокол (Protocol)
- Паттерны проектирования
- Композиция и агрегация
- Связность и связанность
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

- [Содержание](#содержание)
