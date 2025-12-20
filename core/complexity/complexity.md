# **Сложность кода (Asymptotic, Cyclomatic, Coupling, Maintainability)**

[Содержание](/CONTENTS.md#содержание)

## **Junior Level**

Сложность кода — это набор характеристик, определяющих, насколько программа понятна, эффективна и удобна для работы.
Выделяют четыре ключевых типа.

1. **Асимптотическая сложность (Asymptotic Complexity)** — показывает, как время выполнения или использование памяти
   программой зависит от объема входных данных (например, количества элементов в списке). Выражается в "О-нотации": O(
    1) — время постоянно, O(n) — растет линейно, O(n²) — растет квадратично (намного быстрее).

2. **Цикломатическая сложность (Cyclomatic Complexity)** — это мера количества независимых путей выполнения в коде (
   например, в функции). Чем больше операторов ветвления (`if/else`, `switch`) и циклов (`for`, `while`), тем она выше.
   Высокая цикломатическая сложность делает код трудным для понимания и тестирования.

3. **Сложность связей (Coupling Complexity)** — определяет, насколько сильно разные модули, классы или функции зависят
   друг от друга. Сильно связанный код сложно изменять, тестировать изолированно и повторно использовать.

4. **Сложность поддержки (Maintainability Complexity)** — общая оценка того, насколько легко анализировать, изменять и
   расширять код. Зависит от его читаемости, документированности, структуры и соответствия стандартам.

**Практическое значение для разработчика**:

- **Высокая асимптотическая сложность** → проверка производительности на больших данных.
- **Высокая цикломатическая сложность** → необходимость в большем количестве тестов для покрытия всех путей выполнения.
- **Высокая связанность** → сложности с изоляцией модулей для модульного тестирования.
- **Низкая поддерживаемость** → больше времени тратится на понимание кода перед внесением изменений или написанием
  тестов.

[Содержание](/CONTENTS.md#содержание)

## **Middle Level**

### **1. Асимптотическая сложность (Asymptotic Complexity)**

Это математическая оценка роста ресурсоемкости алгоритма.

- **Верхняя граница (O-нотация)**: `O(f(n))` означает, что время выполнения не превысит `c * f(n)` для всех достаточно
  больших `n`. Это наиболее часто используемая оценка.
- **Нижняя граница (Ω-нотация)**: `Ω(f(n))` означает, что время выполнения не меньше `c * f(n)`.
- **Точная граница (Θ-нотация)**: `Θ(f(n))` означает, что алгоритм является одновременно и `O(f(n))`, и `Ω(f(n))`.
- **Амортизированный анализ**: оценивает среднюю производительность операции в худшем случае за всю последовательность
  вызовов, что полезно для анализа структур данных.
- **Космическая сложность**: оценивает рост потребления памяти аналогично временной сложности.

**Практика для разработки и тестирования**:

- **Тестирование на разных объемах данных**: малые (пограничные случаи), средние, большие (нагрузочное тестирование).
- **Профилирование**: использование профилировщиков (cProfile, memory_profiler) для поиска узких мест.
- **Анализ в CI/CD**: интеграция проверки сложности в конвейер сборки с помощью инструментов статического анализа.

### **2. Цикломатическая сложность (Cyclomatic Complexity)**

Количественная метрика, основанная на графе потока управления программы.

- **Расчет**: `M = E - N + 2P`, где `E` — число рёбер в графе, `N` — число узлов, `P` — число компонентов связности (
  обычно 1). На практике рассчитывается автоматически.
- **Практические рекомендации** по значениям для функции/метода:
    - **1-10**: простая, низкий риск.
    - **11-20**: умеренная сложность.
    - **21-50**: высокая сложность, требует рассмотрения рефакторинга.
    - **51+**: очень высокая, тестирование затруднено, высок риск ошибок.
- **Инструменты**: radon, mccabe, pylint, а также комплексные платформы (SonarQube, Codacy), которые включают этот
  анализ.

**Связь с тестированием**:

- Минимальное количество тестов для покрытия всех путей должно быть не меньше цикломатической сложности.
- Помогает выявить функции с излишней логикой ветвления, которые являются кандидатами на упрощение.

### **3. Сложность связей (Coupling Complexity)**

Оценивает степень взаимозависимости между модулями. Цель — достичь слабой связанности.

**Типы связности (от худшего к лучшему)**:

- **Content coupling**: один модуль напрямую изменяет внутренние данные другого.
- **Common coupling**: модули используют общие глобальные данные.
- **Control coupling**: один модуль передает другому флаг или данные, управляющие его логикой.
- **Stamp coupling**: модулям передается большая структура данных, хотя нужна лишь ее часть.
- **Data coupling**: модули обмениваются только необходимыми данными через параметры (идеал).

**Метрики**:

- **CBO (Coupling Between Objects)**: количество классов, с которыми непосредственно связан данный класс.
- **Ca (Afferent Coupling)**: количество классов, которые зависят от данного (входящие связи).
- **Ce (Efferent Coupling)**: количество классов, от которых зависит данный (исходящие связи).

**Влияние на разработку**:

- Высокая связанность затрудняет модульное тестирование, требуя создания множества mock-объектов.
- Повышает риск каскадных изменений: правка в одном модуле ведет к необходимости правок во многих других.
- Современные инструменты анализа кода помогают визуализировать зависимости и выявлять проблемные узлы.

### **4. Сложность поддержки (Maintainability Complexity)**

Составная характеристика, прогнозирующая усилия по сопровождению кода.

**Основные компоненты**:

- **Анализируемость**: легкость понимания кода, отладки и локализации дефектов.
- **Изменяемость**: легкость внесения изменений и минимальный риск побочных эффектов.
- **Стабильность**: устойчивость к распространению ошибок при изменениях.
- **Тестируемость**: степень, в которой код облегчает создание и проведение тестов.

**Ключевые метрики**:

- **Индекс поддерживаемости (MI)**: комплексный показатель, рассчитываемый на основе цикломатической сложности,
  количества строк кода (LOC) и объема комментариев.
- **Соотношение комментарии/код**: показатель документированности.
- **Нарушения стандартов кода**: количество отступлений от соглашений (PEP8, Google Style и др.).
- **Дублирование кода**: процент строк, повторяющихся в кодовой базе.

**Инструменты**: SonarQube, CodeClimate, Codacy. Эти платформы агрегируют метрики, устанавливают пороговые значения ("
Quality Gate") и предоставляют сводные отчеты о здоровье кода, помогая командам проактивно работать над улучшением
сопровождаемости.

Надеюсь, это объяснение было полезным и полным. Если у вас есть вопросы по какому-то из видов сложности или инструментам
для их анализа — обращайтесь.

[Содержание](/CONTENTS.md#содержание)

## **Senior Level**

### **1. Асимптотическая сложность (Asymptotic Complexity) (Big O only)**

## **O(1) - Константная сложность**

```python
def get_first_element(arr):
    return arr[0]  # Всегда одна операция


def is_even(n):
    return n % 2 == 0  # Одна операция
```

## **O(log n) - Логарифмическая сложность**

```python
def binary_search(arr, target):
    left, right = 0, len(arr) - 1
    while left <= right:
        mid = (left + right) // 2
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    return -1
```

## **O(n) - Линейная сложность**

```python
def linear_search(arr, target):
    for i in range(len(arr)):  # Проход по всем элементам
        if arr[i] == target:
            return i
    return -1


def sum_array(arr):
    total = 0
    for num in arr:  # Один цикл по всем элементам
        total += num
    return total
```

## **O(n log n) - Линейно-логарифмическая сложность**

```python
def merge_sort(arr):
    if len(arr) <= 1:
        return arr

    mid = len(arr) // 2
    left = merge_sort(arr[:mid])
    right = merge_sort(arr[mid:])

    return merge(left, right)


def merge(left, right):
    result = []
    i = j = 0
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            result.append(left[i])
            i += 1
        else:
            result.append(right[j])
            j += 1
    result.extend(left[i:])
    result.extend(right[j:])
    return result
```

## **O(n²) - Квадратичная сложность**

```python
def bubble_sort(arr):
    n = len(arr)
    for i in range(n):  # Внешний цикл
        for j in range(0, n - i - 1):  # Внутренний цикл
            if arr[j] > arr[j + 1]:
                arr[j], arr[j + 1] = arr[j + 1], arr[j]
    return arr


def find_duplicates(arr):
    duplicates = []
    for i in range(len(arr)):  # Двойной цикл
        for j in range(i + 1, len(arr)):
            if arr[i] == arr[j]:
                duplicates.append(arr[i])
    return duplicates
```

## **O(n³) - Кубическая сложность**

```python
def find_triplets(arr):
    n = len(arr)
    triplets = []
    for i in range(n):  # Три вложенных цикла
        for j in range(i + 1, n):
            for k in range(j + 1, n):
                if arr[i] + arr[j] + arr[k] == 0:
                    triplets.append((arr[i], arr[j], arr[k]))
    return triplets
```

## **O(2ⁿ) - Экспоненциальная сложность**

```python
def fibonacci_recursive(n):
    if n <= 1:
        return n
    return fibonacci_recursive(n - 1) + fibonacci_recursive(n - 2)


def generate_subsets(arr):
    def backtrack(start, current):
        subsets.append(current.copy())
        for i in range(start, len(arr)):
            current.append(arr[i])
            backtrack(i + 1, current)
            current.pop()

    subsets = []
    backtrack(0, [])
    return subsets
```

## **O(n!) - Факториальная сложность**

```python
def generate_permutations(arr):
    def backtrack(path):
        if len(path) == len(arr):
            permutations.append(path.copy())
            return
        for num in arr:
            if num not in path:
                path.append(num)
                backtrack(path)
                path.pop()

    permutations = []
    backtrack([])
    return permutations

# Пример: для 3 элементов будет 3! = 6 перестановок
```

## **O(√n) - Сложность квадратного корня**

```python
def is_prime(n):
    if n < 2:
        return False
    for i in range(2, int(n ** 0.5) + 1):  # До квадратного корня
        if n % i == 0:
            return False
    return True
```

**Ключевые выводы:**

1. **O(1)** и **O(log n)** - самые эффективные
2. **O(n)** и **O(n log n)** - приемлемы для больших данных
3. **O(n²)**, **O(2ⁿ)**, **O(n!)** - становятся непрактичными при росте n
4. При выборе алгоритма важно оценивать ожидаемый размер входных данных

### **2. Цикломатическая сложность (Cyclomatic Complexity)**

### **V(G) = 1** - Простая функция

```python
def simple_greeting(name):
    """Цикломатическая сложность = 1"""
    return f"Hello, {name}!"

# Граф: один путь выполнения
```

### **V(G) = 2** - Одно условие

```python
def check_age(age):
    """Цикломатическая сложность = 2 (1 условие + 1)"""
    if age >= 18:
        return "Adult"
    else:
        return "Minor"

# Условия: 1
# Пути: 1) age >= 18, 2) age < 18
```

### **V(G) = 3** - Два условия (или if-elif)

```python
def grade_score(score):
    """Цикломатическая сложность = 3"""
    if score >= 90:
        return "A"
    elif score >= 70:
        return "B"
    else:
        return "C"

# Условия: 2 (score >= 90 и score >= 70)
# Пути: 3 возможных пути
```

### **V(G) = 4** - Несколько условий и цикл

```python
def categorize_temperature(temp):
    """Цикломатическая сложность = 4"""
    if temp < 0:
        category = "Freezing"
    elif temp < 15:
        category = "Cold"
    elif temp < 25:
        category = "Warm"
    else:
        category = "Hot"

    # Добавляем дополнительный путь
    if "Cold" in category or "Freezing" in category:
        return f"{category} - Wear jacket"
    return f"{category} - Light clothes"

# Условия: 4
# Пути: множество комбинаций
```

### **V(G) = 6** - Вложенные условия

```python
def check_access(user_role, subscription_type, age):
    """Цикломатическая сложность = 6"""
    if user_role == "admin":
        if subscription_type == "premium":
            return "Full access"
        else:
            return "Admin basic access"
    elif user_role == "user":
        if age >= 18:
            if subscription_type == "premium":
                return "Premium user access"
            else:
                return "Basic user access"
        else:
            return "Restricted access (minor)"
    else:
        return "No access"

# Граф:
# - 3 внешних условия (admin/user/else)
# - 2 вложенных условия в admin
# - 3 вложенных условия в user
```

### **V(G) = 8** - Сложная логика с циклами

```python
def process_items(items, threshold, max_attempts):
    """Цикломатическая сложность = 8"""
    result = []

    for item in items:  # Цикл добавляет 1 к сложности
        if item is None:
            continue  # Дополнительный путь

        attempts = 0
        success = False

        while attempts < max_attempts and not success:  # Еще +1
            if item["value"] > threshold:
                if item.get("valid", False):
                    success = True
                    result.append(item["value"] * 2)
                else:
                    result.append(item["value"])
            elif item["value"] > 0:
                result.append(item["value"] // 2)
            else:
                break  # Еще один путь

            attempts += 1

        if not success:
            result.append(-1)

    return result

# Составляющие сложности:
# - for loop: +1
# - while loop: +1  
# - if item is None: +1
# - if item["value"] > threshold: +1
# - if item.get("valid", False): +1
# - elif item["value"] > 0: +1
# - else в цикле while: +1
# - if not success: +1
# Итого: 8
```

### **V(G) = 15+** - "Плохой" код с высокой сложностью

```python
def monster_function(a, b, c, d, e, f):
    """Цикломатическая сложность > 15 - ТРЕБУЕТ РЕФАКТОРИНГА!"""
    result = 0

    if a > 0:
        if b < 10:
            result += 1
        elif b < 20:
            result += 2
        else:
            if c == "type1":
                result += 3
            elif c == "type2":
                result += 4
            else:
                result += 5
    elif a == 0:
        if d:
            for i in range(e):  # +1
                if i % 2 == 0:  # +1
                    result += i
                else:
                    if f:  # +1
                        result -= i
                    else:
                        result += i * 2
        else:
            result = -1
    else:
        if e > 100:
            result = e * 2
        elif e > 50:
            result = e
        else:
            if f:
                result = e // 2
            else:
                result = e // 4

    # Дополнительные вложенные условия
    if result > 1000:
        if a > 50:
            result *= 1.1
        elif a > 20:
            result *= 1.05

    return result

# Эта функция имеет очень высокую цикломатическую сложность
# из-за глубокой вложенности и множества условий
```

[Содержание](/CONTENTS.md#содержание)