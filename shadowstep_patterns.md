## Найденные паттерны проектирования в проекте Shadowstep

### 1. Facade Pattern (Двухуровневый)

**Основной фасад - `Shadowstep`**:

- Скрывает сложность работы с Appium WebDriver
- Компонует подсистемы: Navigator, LocatorConverter, MobileCommands, Terminal, Logcat [1](#0-0)

**Вторичный фасад - `Element`**:

- Скрывает сложность операций с элементами
- Компонует специализированные обработчики: ElementDOM, ElementActions, ElementGestures, ElementProperties,
  ElementCoordinates, ElementScreenshots, ElementWaiting [2](#0-1)

### 2. Singleton Pattern

**`Shadowstep` как синглтон**:

- Реализован через `__new__()` с проверкой `_instance` [3](#0-2)
- Тесты подтверждают поведение синглтона [4](#0-3)

**`WebDriverSingleton` для сессии Appium**:

- Обеспечивает единую WebDriver сессию для всех компонентов
- Предотвращает конфликты множественных сессий

### 3. Page Object Model с графовой навигацией

**Базовый класс `PageBaseShadowstep`**:

- Абстрактный базовый класс для всех Page Objects
- Реализует синглтон для страниц [5](#0-4)
- Требует определения `edges` для навигации [6](#0-5)

**Пример Page Object**:

```python
class PageSettings(PageBaseShadowstep):
    @property
    def edges(self):
        return {
            "PageNetworkInternet": self.to_network_internet,
        }
``` 

### 4. Composition over inheritance

**Система Element**:

- Вместо наследования использует композицию специализированных классов
- Каждый обработчик отвечает за одну область:
    - `ElementDOM` - навигация по DOM
    - `ElementActions` - ввод текста
    - `ElementGestures` - жесты
    - `ElementProperties` - атрибуты и состояния [8](#0-7)

### 5. Strategy Pattern

**Стратегии жестов в `ElementGestures`**:

- `GestureStrategy.AUTO` - пробует W3C Actions, затем MobileCommands
- `GestureStrategy.W3C` - использует стандартный WebDriver Actions API
- `GestureStrategy.MOBILE_COMMANDS` - нативные UIAutomator2 команды

### 6. Template Method Pattern

**Генератор Page Objects**:

- `PageObjectGenerator` использует шаблоны Jinja2
- Определяет скелет генерации, позволяя настраивать детали [9](#0-8)

### 7. Factory Pattern

**Создание элементов**:

- `Shadowstep.get_element()` - фабричный метод для создания Element
- Поддерживает различные форматы локаторов через конвертер [10](#0-9)

### 8. Observer Pattern (потенциальный)

**Система логирования**:

- Использует Loguru для цветного вывода
- Компоненты могут логгировать события через общий интерфейс

## Notes

- Проект активно использует Clean Architecture принципы [11](#0-10)
- Все паттерны направлены на упрощение API для написания тестов
- Дополнительные паттерны могут присутствовать в модулях навигации и конвертеров локаторов

Wiki pages you might want to explore:

- [Overview (molokov-klim/Appium-Python-Client-Shadowstep)](/wiki/molokov-klim/Appium-Python-Client-Shadowstep#1)

### Citations

**File:** README.md (L207-212)

```markdown
class PageSettings(PageBaseShadowstep):
@property
def edges(self):
return {
"PageNetworkInternet": self.to_network_internet,
}
```

**File:** README.md (L263-270)

```markdown
```python
class Shadowstep(ShadowstepBase):
    """Main Facade for mobile automation."""

    def __init__(self):
        super().__init__()
        self.navigator = PageNavigator(self)
        self.converter = LocatorConverter()
```

**File:** README.md (L295-302)

```markdown
        self.properties = ElementProperties(self)
        self.dom = ElementDOM(self)
        self.actions = ElementActions(self)
        self.gestures = ElementGestures(self)
        self.coordinates = ElementCoordinates(self)
        self.screenshots = ElementScreenshots(self)
        self.waiting = ElementWaiting(self)
```

```

**File:** README.md (L306-313)
```markdown
- `ElementDOM` — finding related elements (parent, sibling, cousin)
- `ElementActions` — text input, clearing
- `ElementGestures` — tap, swipe, scroll, fling
- `ElementProperties` — attributes, states
- `ElementCoordinates` — coordinates, center
- `ElementScreenshots` — screenshots
- `ElementWaiting` — waits
- `ElementUtilities` — helper functions
```

**File:** README.md (L698-709)

```markdown
# Via Shadowstep

element = app.get_element({"text": "Settings"})

# Directly

from shadowstep.element import Element

element = Element(
locator={"text": "Settings"},
shadowstep=app,
timeout=30,
poll_frequency=0.5
)
```

**File:** README.md (L1585-1603)

```markdown
# 3. Generate Page Object

generator = PageObjectGenerator()
output_path, class_name = generator.generate(
ui_element_tree=ui_tree,
output_dir="./generated_pages",
filename_prefix="page_"
)

print(f"Generated: {output_path}")
print(f"Class: {class_name}")

# Result: page_settings.py

# class PageSettings(PageBaseShadowstep):

# @property

# def title(self) -> Element: ...

# @property

# def network_internet(self) -> Element: ...

# ...
```

```

**File:** README.md (L2056-2059)
```markdown
- **Clean Architecture** — separation of concerns
- **Clean Code** — readability and maintainability
- **Best Practices** — design patterns
- **Type Safety** — strict typing (Pyright strict mode)
```

**File:** shadowstep/page_base.py (L23-30)

```python
class PageBaseShadowstep(ABC):
    """Abstract shadowstep class for all pages in the Shadowstep framework.

    Implements singleton behavior and lazy initialization of the shadowstep context.
    """

    shadowstep: "Shadowstep"
    _instances: ClassVar[dict[type, "PageBaseShadowstep"]] = {}
```

**File:** shadowstep/page_base.py (L32-45)

```python
    def __new__(cls) -> Any:


"""Create a new instance or return existing singleton instance.

Returns:
    PageBaseShadowstep: The singleton instance of the page class.

"""
if cls not in cls._instances:
    from shadowstep.shadowstep import Shadowstep  # noqa: PLC0415

    instance = super().__new__(cls)
    instance.shadowstep = Shadowstep.get_instance()
    cls._instances[cls] = instance
return cls._instances[cls]
```

**File:** shadowstep/page_base.py (L62-70)

```python
    @property


@abstractmethod
def edges(self) -> dict[str, Callable[[], "PageBaseShadowstep"]]:
    """Each page must declare its dom edges.

    Returns:
        Dict[str, Callable]: Dictionary mapping page class names to dom methods.

    """
```

**File:** tests/test_unit/test_shadowstep_unit.py (L31-43)

```python
    def test_singleton_behavior(self):


"""Test that Shadowstep implements singleton pattern correctly."""
# Clear any existing instances
Shadowstep._instance = None

# Create first instance
instance1 = Shadowstep()
assert instance1 is not None

# Create second instance - should return the same instance
instance2 = Shadowstep()
assert instance1 is instance2

```
