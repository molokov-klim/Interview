# **Типы данных**

[Содержание](/CONTENTS.md#содержание)

---

## **Junior Level**

Все типы данных в Python делятся на две фундаментальные категории: **изменяемые (mutable)** и **неизменяемые (immutable)
**. От этого зависит поведение объектов при их передаче в функции, присваивании и модификации.

---

### **Простые (скалярные) неизменяемые типы**

Это базовые типы, значение которых нельзя изменить после создания.

* **Числа:**
    * `int` — целые числа (например, `5`, `-10`, `0`)
    * `float` — числа с плавающей точкой (например, `3.14`, `2.0`, `-0.001`)
    * `complex` — комплексные числа (например, `1+2j`)

* **Текстовый тип:**
    * `str` — строки (например, `"Hello"`, `'World'`)

* **Логический тип:**
    * `bool` — принимает значения `True` или `False`

* **Специальный тип:**
    * `NoneType` — тип с единственным значением `None` (обозначает отсутствие значения).

---

### **Коллекции (составные типы)**

#### **Неизменяемые (immutable) коллекции:**

* `tuple` — упорядоченный кортеж. После создания нельзя добавить, удалить или изменить элементы.  
  Пример: `(1, "яблоко", True)`
* `frozenset` — неизменяемое множество уникальных элементов.
* `bytes` — неизменяемая последовательность байтов (0-255).

#### **Изменяемые (mutable) коллекции:**

* `list` — упорядоченный список. Можно свободно изменять.  
  Пример: `[1, 2, 3] -> [1, 10, 3]`
* `dict` — словарь (неупорядоченный до Python 3.7, упорядоченный по порядку добавления начиная с 3.7). Коллекция пар
  «ключ: значение». Изменяемы ключи и значения.
* `set` — множество уникальных элементов. Можно добавлять и удалять элементы.
* `bytearray` — изменяемая последовательность байтов.
* `range` — представляет неизменяемую последовательность чисел (часто используется в циклах).

---

## **Не built-in, но доступные из стандартной библиотеки**

Стандартная библиотека Python содержит модули с дополнительными типами данных, которые можно использовать без установки
внешних библиотек.

**`decimal.Decimal`** — десятичные числа с фиксированной точностью, идеально для финансовых расчетов
**`fractions.Fraction`** — рациональные числа (дроби)

**`collections` модуль:**

- **`namedtuple`** — именованный кортеж (неизменяемый)
- **`deque`** — двусторонняя очередь (изменяемая, быстрая вставка/удаление с обоих концов)
- **`Counter`** — подкласс dict для подсчета хешируемых объектов
- **`defaultdict`** — словарь с заводским значением для отсутствующих ключей
- **`OrderedDict`** — словарь, сохраняющий порядок добавления (с Python 3.7 обычный dict тоже сохраняет порядок)

**`array.array`** — эффективный массив однотипных элементов

**`datetime` модуль:**

- **`datetime.date`** — только дата (год, месяц, день)
- **`datetime.time`** — только время
- **`datetime.datetime`** — дата и время
- **`datetime.timedelta`** — разница между двумя датами/временами
- **`datetime.timezone`** — информация о часовом поясе

**`enum.Enum`** — перечисления

**`dataclasses.dataclass`** — современный способ создания классов-контейнеров данных
**`typing.NamedTuple`** — typed версия namedtuple
**`types.SimpleNamespace`** — объект с доступом к атрибутам через точку
**`pathlib.Path`** — объектно-ориентированный путь к файлу/директории

**`uuid.UUID`** — уникальные идентификаторы

**`typing` модуль** — типы для аннотаций (не runtime-типы, но важны для статической проверки):

- `List`, `Dict`, `Set`, `Tuple` (дженерик-версии)
- `Optional`, `Union`, `Any`, и др.

**`re.Pattern`** и **`re.Match`** — скомпилированные регулярные выражения и результаты их работы

## **Важное отличие**

Built-in типы доступны всегда, без импорта, типы из стандартной библиотеки требуют импорта.

[Содержание](/CONTENTS.md#содержание)

---

## **Middle Level**

Объекты изменяемыех типов можно модифицировать после создания. А любая модификация объекта неизменяемого типа
создаёт новый объект.
Передача изменяемых объектов в функции позволяет модифицировать оригинал
Только неизменяемые объекты могут быть ключами словаря (требуется хэшируемость)
Мутабельность негативно влияет на потокобезопасность и кэширование

---

## int

## **Создание int**

```python
# Из строки
x = int("42")  # 42
x = int("101", 2)  # 5 (из двоичной)
x = int("FF", 16)  # 255 (из шестнадцатеричной)

# Из других типов
x = int(3.14)  # 3 (округление к нулю)
x = int(True)  # 1
x = int(False)  # 0
```

## **Арифметические операции**

```python
a, b = 10, 3

# Базовые операции
print(a + b)  # 13
print(a - b)  # 7
print(a * b)  # 30
print(a / b)  # 3.333... (float)
print(a // b)  # 3 (целочисленное деление)
print(a % b)  # 1 (остаток)
print(a ** b)  # 1000 (возведение в степень)
print(-a)  # -10 (унарный минус)
print(+a)  # 10 (унарный плюс)
print(abs(-a))  # 10 (модуль)
```

## **Битовые операции**

```python
a, b = 0b1100, 0b1010  # 12 и 10

print(bin(a & b))  # 0b1000 (8) - AND
print(bin(a | b))  # 0b1110 (14) - OR
print(bin(a ^ b))  # 0b0110 (6) - XOR
print(bin(~a))  # -0b1101 (-13) - NOT
print(bin(a << 2))  # 0b110000 (48) - сдвиг влево
print(bin(a >> 2))  # 0b11 (3) - сдвиг вправо
```

## **Методы класса int**

### **int.bit_length()**

```python
x = 42
print(x.bit_length())  # 6 (бит нужно для 42: 101010)
print((255).bit_length())  # 8
```

### **int.bit_count()** (Python 3.8+)

```python
x = 42  # 101010 в двоичной
print(x.bit_count())  # 3 (количество единичных битов)
```

### **int.to_bytes()**

```python
x = 1024
# Преобразование в байты
bytes_data = x.to_bytes(2, byteorder='big')
print(bytes_data)  # b'\x04\x00'

# Обратное преобразование
x_back = int.from_bytes(bytes_data, 'big')
print(x_back)  # 1024
```

### **int.as_integer_ratio()**

```python
x = 10
print(x.as_integer_ratio())  # (10, 1)
```

## **Сравнение**

```python
a, b = 10, 20

print(a == b)  # False
print(a != b)  # True
print(a < b)  # True
print(a <= b)  # True
print(a > b)  # False
print(a >= b)  # False
```

## **Преобразование в другие системы счисления**

```python
x = 255

# Встроенные функции
print(bin(x))  # 0b11111111
print(oct(x))  # 0o377
print(hex(x))  # 0xff

# С помощью format
print(format(x, 'b'))  # 11111111
print(format(x, 'o'))  # 377
print(format(x, 'x'))  # ff
print(format(x, 'X'))  # FF
print(format(x, 'd'))  # 255
```

## **Математические функции**

```python
import math

x = -10
print(math.fabs(x))  # 10.0 (модуль, возвращает float)
print(math.factorial(5))  # 120 (5!)
print(math.gcd(48, 18))  # 6 (НОД)
print(math.lcm(12, 18))  # 36 (НОК, Python 3.9+)
```

## **Операции присваивания**

```python
x = 10
x += 5  # x = 15
x -= 3  # x = 12
x *= 2  # x = 24
x //= 5  # x = 4
x %= 3  # x = 1
x **= 3  # x = 1
x <<= 2  # x = 4
x >>= 1  # x = 2
```

## **Проверка типа и преобразования**

```python
x = 42

print(type(x))  # <class 'int'>
print(isinstance(x, int))  # True
print(float(x))  # 42.0
print(complex(x))  # (42+0j)
print(str(x))  # "42"
print(bool(x))  # True (кроме 0)
```

## **Особенности int в Python 3**

```python
# Автоматическая длинная арифметика
big_num = 10 ** 100  # Огромное число
print(big_num.bit_length())  # 333

# Разделители разрядов
x = 1_000_000  # Начиная с Python 3.6
print(x)  # 1000000
```

## **Полезные трюки**

```python
# Обмен значений без временной переменной
a, b = 5, 10
a, b = b, a
print(a, b)  # 10, 5

# Проверка четности
x = 7
print(x & 1)  # 1 - нечетное, 0 - четное

# Быстрое умножение/деление на степень двойки
x = 10
print(x << 3)  # 80 (10 * 8)
print(x >> 1)  # 5 (10 / 2)

# Минимум/максимум
print(min(5, 10, 3))  # 3
print(max(5, 10, 3))  # 10

# Сумма элементов
numbers = [1, 2, 3, 4, 5]
print(sum(numbers))  # 15
```

[Содержание](/CONTENTS.md#содержание)

---

## float

# **Тип float в Python**

## **Создание float**

```python
# Прямое указание
x = 3.14
x = 0.1
x = -2.5

# Из строки
x = float("3.14")  # 3.14
x = float("inf")  # inf
x = float("-inf")  # -inf
x = float("nan")  # nan

# Из других типов
x = float(10)  # 10.0
x = float(True)  # 1.0
x = float("  5.5  ")  # 5.5 (пробелы игнорируются)

# Научная нотация
x = 1.23e4  # 12300.0
x = 1.23e-2  # 0.0123
```

## **Арифметические операции**

```python
a, b = 5.5, 2.2

print(a + b)  # 7.7
print(a - b)  # 3.3
print(a * b)  # 12.1
print(a / b)  # 2.5
print(a // b)  # 2.0 (целочисленное деление, возвращает float!)
print(a % b)  # 1.1 (остаток)
print(a ** b)  # 42.373... (возведение в степень)
print(-a)  # -5.5
print(+a)  # 5.5
print(abs(-a))  # 5.5
```

## **Методы класса float**

### **float.as_integer_ratio()**

```python
x = 0.25
print(x.as_integer_ratio())  # (1, 4)

x = 3.5
print(x.as_integer_ratio())  # (7, 2)

x = 0.1
print(x.as_integer_ratio())  # (3602879701896397, 36028797018963968)
```

### **float.is_integer()**

```python
print(3.0.is_integer())  # True
print(3.14.is_integer())  # False
print(float(5).is_integer())  # True
```

### **float.hex() и float.fromhex()**

```python
x = 3.14
hex_repr = x.hex()
print(hex_repr)  # '0x1.91eb851eb851fp+1'

y = float.fromhex(hex_repr)
print(y)  # 3.14
```

## **Сравнение чисел с плавающей точкой**

```python
a, b, c = 0.1 + 0.2, 0.3, 1e-10

print(a == b)  # False (из-за погрешности)
print(abs(a - b) < 1e-9)  # True (сравнение с допуском)

# Специальные значения
x = float('nan')
y = float('inf')
z = float('-inf')

print(x == x)  # False! nan != nan
print(y == y)  # True
print(z < 0)  # True
```

## **Проверка специальных значений**

```python
import math

x = float('nan')
y = float('inf')
z = 3.14

print(math.isnan(x))  # True
print(math.isinf(y))  # True
print(math.isfinite(z))  # True
print(math.isinf(z))  # False

# Альтернативно
print(x != x)  # True (только для nan)
```

## **Округление**

```python
x = 3.14159

# Встроенная функция round
print(round(x))  # 3
print(round(x, 2))  # 3.14
print(round(x, 3))  # 3.142

# Функции из модуля math
import math

print(math.floor(x))  # 3 (вниз)
print(math.ceil(x))  # 4 (вверх)
print(math.trunc(x))  # 3 (к нулю)

# Округление банкира (по умолчанию)
print(round(2.5))  # 2
print(round(3.5))  # 4
```

## **Математические функции из модуля math**

```python
import math

x = 3.14
print(math.sqrt(x))  # 1.772... (квадратный корень)
print(math.exp(x))  # 23.103... (e^x)
print(math.log(x))  # 1.144... (натуральный логарифм)
print(math.log10(x))  # 0.496... (десятичный логарифм)
print(math.sin(x))  # 0.001... (синус)
print(math.cos(x))  # -0.999... (косинус)
print(math.tan(x))  # -0.001... (тангенс)
print(math.degrees(x))  # 179.908... (радианы в градусы)
print(math.radians(180))  # 3.141... (градусы в радианы)
```

## **Преобразование типов**

```python
x = 3.14

print(int(x))  # 3 (отбрасывает дробную часть)
print(str(x))  # "3.14"
print(repr(x))  # "3.14"
print(bool(x))  # True (кроме 0.0)
print(complex(x))  # (3.14+0j)
```

## **Операции присваивания**

```python
x = 5.5
x += 2.2  # 7.7
x -= 1.1  # 6.6
x *= 2  # 13.2
x /= 4  # 3.3
x //= 2  # 1.0
x **= 3  # 1.0
```

## **Форматирование вывода**

```python
x = 1234.56789

# f-строки
print(f"{x:.2f}")  # 1234.57
print(f"{x:10.2f}")  # "   1234.57"
print(f"{x:,.2f}")  # 1,234.57
print(f"{x:.2e}")  # 1.23e+03

# format()
print("{:.3f}".format(x))  # 1234.568
print("{:.0f}".format(x))  # 1235

# %-форматирование
print("%.4f" % x)  # 1234.5679
```

## **Полезные константы**

```python
import math

print(math.pi)  # 3.141592653589793
print(math.e)  # 2.718281828459045
print(math.tau)  # 6.283185307179586 (2*pi)
print(math.inf)  # inf
print(-math.inf)  # -inf
print(math.nan)  # nan
```

## **Особенности float**

```python
# Погрешность представления
print(0.1 + 0.2)  # 0.30000000000000004
print(0.1 + 0.2 == 0.3)  # False

# Как это исправить
import math

print(math.isclose(0.1 + 0.2, 0.3))  # True

# Или с указанием допуска
print(abs((0.1 + 0.2) - 0.3) < 1e-9)  # True

# Большие и маленькие числа
print(1e308)  # 1e+308 (нормально)
print(1e309)  # inf (переполнение)
print(1e-323)  # 1e-323 (нормально)
print(1e-324)  # 0.0 (анти-переполнение)
```

## **Десятичные дроби для точных вычислений**

```python
from decimal import Decimal, getcontext

# Устанавливаем точность
getcontext().prec = 28

# Точные вычисления с Decimal
a = Decimal('0.1')
b = Decimal('0.2')
c = Decimal('0.3')
print(a + b == c)  # True!

# Или с дробями
from fractions import Fraction

print(Fraction(1, 10) + Fraction(2, 10) == Fraction(3, 10))  # True
```

## **Полезные функции**

```python
import math

# Модуль числа
print(math.fabs(-3.14))  # 3.14

# Факториал (но для int)
print(math.factorial(5))  # 120

# Наибольший общий делитель
print(math.gcd(48, 18))  # 6

# Гиперболические функции
print(math.sinh(1.0))  # 1.175...
print(math.cosh(1.0))  # 1.543...

# Проверка близости значений
print(math.isclose(1.0, 1.0000000001))  # True
```

## **Сортировка и сравнение**

```python
numbers = [3.14, 2.71, 1.41, float('inf'), float('-inf'), float('nan')]
sorted_nums = sorted(numbers, key=lambda x: (math.isnan(x), x))
print(sorted_nums)  # [-inf, 1.41, 2.71, 3.14, inf, nan]
```

## **Важные замечания:**

1. **Float имеет ограниченную точность** - обычно 15-17 значащих цифр
2. **Оперируйте с допуском** при сравнении float чисел
3. **Используйте Decimal** для финансовых расчетов
4. **Проверяйте на специальные значения** (nan, inf)
5. **Осторожно с суммированием** - используйте math.fsum() для точного суммирования:

[Содержание](/CONTENTS.md#содержание)

---

## decimal

# **Тип Decimal в Python (модуль decimal)**

## **Импорт и создание Decimal**

```python
from decimal import Decimal, getcontext, ROUND_HALF_UP, ROUND_DOWN, ROUND_CEILING

# Создание из строки (рекомендуется)
d1 = Decimal('10.5')
d2 = Decimal('3.14')
d3 = Decimal('0.1')

# Создание из int
d4 = Decimal(10)  # 10
d5 = Decimal(0)  # 0

# Создание из float (не рекомендуется из-за потери точности)
d6 = Decimal(3.14)  # Может быть неточным

# Создание из кортежа
d7 = Decimal((0, (1, 2, 3), -2))  # 1.23
# (sign, digits, exponent)
# sign: 0 - положительный, 1 - отрицательный
```

## **Настройка контекста (точности и округления)**

```python
# Получить текущий контекст
ctx = getcontext()
print(f"Точность: {ctx.prec}")  # По умолчанию 28

# Установить точность (количество значащих цифр)
getcontext().prec = 10

# Установить режим округления
getcontext().rounding = ROUND_HALF_UP  # Обычное округление (0.5 → 1)
getcontext().rounding = ROUND_DOWN  # Отсечение дробной части
getcontext().rounding = ROUND_CEILING  # К +∞

# Временное изменение контекста
from decimal import localcontext

with localcontext() as ctx:
    ctx.prec = 5
    ctx.rounding = ROUND_DOWN
    result = Decimal('1') / Decimal('3')
    print(result)  # 0.33333
```

## **Арифметические операции**

```python
a = Decimal('10.5')
b = Decimal('3.2')

# Базовые операции
print(a + b)  # 13.7
print(a - b)  # 7.3
print(a * b)  # 33.60
print(a / b)  # 3.28125
print(a // b)  # 3 (целочисленное деление)
print(a % b)  # 0.9 (остаток)
print(a ** 2)  # 110.25
print(-a)  # -10.5
print(+a)  # 10.5
print(abs(a))  # 10.5

# Деление с остатком (divmod)
quotient, remainder = divmod(a, b)
print(quotient, remainder)  # 3 0.9
```

## **Методы Decimal**

### **compare() - сравнение**

```python
a = Decimal('10.5')
b = Decimal('3.2')

print(a.compare(b))  # 1 (a > b)
print(b.compare(a))  # -1 (b < a)
print(a.compare(a))  # 0 (равны)

# Методы сравнения
print(a.__eq__(b))  # False
print(a.__ne__(b))  # True
print(a.__lt__(b))  # False
print(a.__le__(b))  # False
print(a.__gt__(b))  # True
print(a.__ge__(b))  # True
```

### **quantize() - округление до указанной точности**

```python
d = Decimal('3.1415926535')

# Округление до 3 знаков после запятой
print(d.quantize(Decimal('0.001')))  # 3.142
print(d.quantize(Decimal('0.001'), rounding=ROUND_DOWN))  # 3.141

# Округление до целых
print(d.quantize(Decimal('1')))  # 3

# Округление до десятков
print(Decimal('123.456').quantize(Decimal('10')))  # 120

# С указанием формата
from decimal import DecimalTuple

print(d.quantize(Decimal('0.00')))  # 3.14
```

### **normalize() - нормализация**

```python
d1 = Decimal('100.00')
d2 = Decimal('100')

print(d1.normalize())  # 1E+2
print(d2.normalize())  # 100

# Удаление лишних нулей
d3 = Decimal('123.4500')
print(d3.normalize())  # 123.45
```

### **adjusted() - порядок числа**

```python
d = Decimal('123.456')
print(d.adjusted())  # 2 (10^2 = 100, порядок числа)
# Для 0.00123 → -3
```

### **as_tuple() - представление в виде кортежа**

```python
d = Decimal('-123.456')
t = d.as_tuple()
print(t)  # DecimalTuple(sign=1, digits=(1, 2, 3, 4, 5, 6), exponent=-3)
print(f"Знак: {t.sign}, цифры: {t.digits}, экспонента: {t.exponent}")
```

### **copy_abs(), copy_negate() и др.**

```python
d = Decimal('-123.456')

print(d.copy_abs())  # 123.456 (модуль)
print(d.copy_negate())  # 123.456 (смена знака)
print(d.copy_sign(Decimal('5.5')))  # -5.5 (копирует знак)
```

### **fma() - fused multiply-add**

```python
# a * b + c (одна операция без промежуточного округления)
a = Decimal('1.23')
b = Decimal('4.56')
c = Decimal('7.89')

print(a.fma(b, c))  # 1.23*4.56 + 7.89 = 13.4988
```

## **Математические функции**

```python
from decimal import Decimal

d = Decimal('3.14159')

# Квадратный корень
print(d.sqrt())  # 1.7724531023414978

# Возведение в степень
print(d ** 2)  # 9.8695877281
print(pow(d, 2))  # 9.8695877281
print(d.exp())  # e^d (23.140692632779...)
print(d.ln())  # Натуральный логарифм
print(d.log10())  # Десятичный логарифм
```

## **Специальные значения и проверки**

```python
from decimal import Decimal, InvalidOperation

d1 = Decimal('10.5')
d2 = Decimal('0')
d3 = Decimal('NaN')  # Не число
d4 = Decimal('Infinity')  # +∞
d5 = Decimal('-Infinity')  # -∞

# Проверки
print(d1.is_finite())  # True
print(d1.is_infinite())  # False
print(d1.is_nan())  # False
print(d1.is_signed())  # False (отрицательный ли)
print(d1.is_zero())  # False

print(d3.is_nan())  # True
print(d4.is_infinite())  # True

# Проверка на нормализованность
print(d1.is_normal())  # True
print(d2.is_normal())  # False (0 не нормализован)
print(Decimal('0.001').is_normal())  # False (слишком маленькое)

# Проверка на каноничность
print(d1.is_canonical())  # True
```

## **Сравнение с другими типами**

```python
d = Decimal('10.5')

print(d == 10.5)  # False (разные типы)
print(d == Decimal('10.5'))  # True
print(d == '10.5')  # False

# Сравнение с int и float
print(d == 10.5)  # False, но можно сравнивать
print(d > 10)  # True
print(d < 11)  # True

# Конвертация
print(float(d))  # 10.5
print(int(d))  # 10
print(str(d))  # '10.5'
print(repr(d))  # "Decimal('10.5')"
```

## **Работа с денежными значениями**

```python
from decimal import Decimal, ROUND_HALF_UP

# Расчет скидки
price = Decimal('99.99')
discount = Decimal('0.15')  # 15%
final_price = price * (Decimal('1') - discount)
print(final_price)  # 84.9915

# Округление для денег
final_price = final_price.quantize(Decimal('0.01'), rounding=ROUND_HALF_UP)
print(final_price)  # 84.99

# Расчет НДС
vat_rate = Decimal('0.20')  # 20%
vat_amount = final_price * vat_rate
vat_amount = vat_amount.quantize(Decimal('0.01'), rounding=ROUND_HALF_UP)
print(vat_amount)  # 17.00
```

## **Форматирование вывода**

```python
d = Decimal('1234.5678')

# Форматирование с помощью format()
print(f"{d:,.2f}")  # 1,234.57
print(f"{d:.0f}")  # 1235
print(f"{d:.4e}")  # 1.2346e+03

# Использование to_eng_string()
print(d.to_eng_string())  # 1234.5678
```

## **Полезные константы**

```python
from decimal import Decimal

# Создание констант
print(Decimal('0'))  # 0
print(Decimal('1'))  # 1
print(Decimal('10'))  # 10
print(Decimal('0.1'))  # 0.1
print(Decimal('Infinity'))  # Infinity
print(Decimal('-Infinity'))  # -Infinity
print(Decimal('NaN'))  # NaN
```

## **Обработка исключений**

```python
from decimal import Decimal, InvalidOperation, DivisionByZero

try:
    result = Decimal('1') / Decimal('0')
except DivisionByZero:
    print("Деление на ноль!")

try:
    result = Decimal('abc')
except InvalidOperation:
    print("Недопустимая операция!")

# Или проще
d = Decimal('10.5')
if d.is_nan():
    print("Это не число!")
```

## **Примеры использования**

### **Точные финансовые расчеты**

```python
from decimal import Decimal, ROUND_HALF_UP

# Расчет сложных процентов
principal = Decimal('1000.00')
rate = Decimal('0.05')  # 5%
years = 5

for year in range(1, years + 1):
    principal += principal * rate
    principal = principal.quantize(Decimal('0.01'), ROUND_HALF_UP)
    print(f"Год {year}: {principal}")

# Сумма списка значений
prices = [Decimal('10.99'), Decimal('5.49'), Decimal('19.95')]
total = sum(prices)
print(f"Итого: {total}")  # 36.43
```

## **Важные замечания:**

1. **Всегда создавайте Decimal из строк**, а не из float
2. **Устанавливайте нужную точность** в контексте
3. **Используйте quantize()** для округления денежных значений
4. **Проверяйте на специальные значения** (NaN, Infinity)
5. **Decimal медленнее float**, но обеспечивает точность
6. **Используйте Decimal для**: финансовых расчетов, точных научных вычислений, налоговых расчетов

[Содержание](/CONTENTS.md#содержание)

---

## complex

## **Тип complex в Python (комплексные числа)**

Тип `complex` представляет комплексные числа. Это встроенный тип данных, а не класс из
отдельного модуля. Он содержит ограниченное количество собственных методов, так как основная математическая
функциональность для работы с комплексными числами вынесена в модуль `cmath`.

## **Создание комплексных чисел**

```python
# 1.1 Литерал с суффиксом 'j' или 'J'
z1 = 3 + 4j
z2 = 2 - 1.5j
z3 = 7.2J  # То же, что и 7.2j
z4 = .3j  # 0.3j

# 1.2 Через конструктор complex()
z5 = complex(3, 4)  # (3+4j) — из двух чисел (вещ. часть, мнимая часть)
z6 = complex(5)  # (5+0j) — из одного числа (вещ. часть)
z7 = complex()  # 0j

# 1.3 Преобразование строки
z8 = complex('1+2j')  # (1+2j)
z9 = complex('  -4.5J ')  # (-0-4.5j) — пробелы игнорируются

# 1.4 Статический метод complex.from_number() (Python 3.14+)
# Безопасное преобразование объектов, реализующих __complex__, __float__ или __index__
print(complex.from_number(42))  # (42+0j)
print(complex.from_number(3.14))  # (3.14+0j)
```

## **Атрибуты комплексного числа (только для чтения)**

```python
z = 3 + 4j

# Действительная часть (real part)
print(f"z.real = {z.real}")  # 3.0
print(type(z.real))  # <class 'float'>

# Мнимая часть (imaginary part)
print(f"z.imag = {z.imag}")  # 4.0
print(type(z.imag))  # <class 'float'>

# Попытка изменить атрибут вызовет ошибку
try:
    z.real = 5.0
except AttributeError as e:
    print(f"Ошибка: {e}")  # readonly attribute
```

## **Методы типа complex**

Тип `complex` имеет всего один стандартный метод.

### **conjugate() — комплексно-сопряжённое число**

```python
z = 3 + 4j
z_conj = z.conjugate()
print(z_conj)  # (3-4j)

# Проверка свойства: произведение числа на сопряжённое даёт квадрат модуля
mod_squared = z * z.conjugate()
print(mod_squared)  # (25+0j)
print(mod_squared.real)  # 25.0
print(abs(z) ** 2)  # 25.0
```

## **Поддерживаемые и неподдерживаемые операции**

```python
a = 2 + 3j
b = 1 - 2j

# 5.1 Поддерживаемые арифметические операции
print(a + b)  # (3+1j)
print(a - b)  # (1+5j)
print(a * b)  # (8-1j)
print(a / b)  # (-0.8+1.4j)
print(a ** 2)  # (-5+12j)
print(-a)  # (-2-3j)
print(+a)  # (2+3j)

# Функция abs() возвращает модуль (длину) комплексного числа
print(abs(3 + 4j))  # 5.0

# 5.2 Неподдерживаемые операции
# Целочисленное деление, остаток от деления и divmod не определены
try:
    a // b
except TypeError as e:
    print(f"Не поддерживается: {e}")

try:
    a % b
except TypeError as e:
    print(f"Не поддерживается: {e}")

try:
    divmod(a, b)
except TypeError as e:
    print(f"Не поддерживается: {e}")
```

## **Операции сравнения**

```python
z1 = 3 + 4j
z2 = 3 + 4j
z3 = 1 + 2j

# 6.1 Поддерживается только проверка на равенство/неравенство
print(z1 == z2)  # True
print(z1 != z3)  # True

# 6.2 Операторы порядка (<, >, <=, >=) не работают
try:
    print(z1 > z3)
except TypeError as e:
    print(f"Ошибка сравнения: {e}")  # '>' not supported between instances of 'complex' and 'complex'
```

## **Преобразование в другие типы и форматирование**

```python
z = 2.5 + 3.1j

# 7.1 Преобразование в строку
print(str(z))  # (2.5+3.1j)
print(repr(z))  # (2.5+3.1j)

# 7.2 Форматирование через f-строки
print(f"z = {z}")  # z = (2.5+3.1j)
print(f"z = {z:.1f}")  # z = (2.5+3.1j) — для обеих частей
print(f"Re(z) = {z.real:.2f}")  # Re(z) = 2.50
print(f"Im(z) = {z.imag:.2f}")  # Im(z) = 3.10

# 7.3 Явное преобразование типов
# В int или float можно преобразовать только действительную часть
print(int(z.real))  # 2
print(float(z.real))  # 2.5

# Прямое преобразование complex в int/float невозможно
try:
    int(z)
except TypeError as e:
    print(f"Ошибка: {e}")  # can't convert complex to int
```

## **Отличия от модуля cmath**

Важно понимать разницу между **типом `complex`** и **модулем `cmath`**:

* **Тип `complex`** (`3+4j`) — это сам **объект-число**. Он содержит данные (действительную и мнимую часть) и базовые
  методы для работы с ними (`.conjugate()`).
* **Модуль `cmath`** — это **библиотека функций** (синус, косинус, логарифм, корень и т.д.) для выполнения
  математических операций с объектами типа `complex`.

```python
import cmath
import math

z = 1 + 1j

# Методы типа complex (работают с самим объектом)
print(z.conjugate())  # (1-1j) — метод числа
print(abs(z))  # 1.4142 — встроенная функция для модуля

# Функции модуля cmath (выполняют математические вычисления)
print(cmath.phase(z))  # 0.7854 радиан (фаза) — требует cmath
print(cmath.sqrt(z))  # (1.099+0.455j) — квадратный корень
print(cmath.log(z))  # (0.3466+0.7854j) — натуральный логарифм

# Функции модуля math НЕ работают с комплексными числами
try:
    math.sqrt(z)
except TypeError as e:
    print(f"math не работает с complex: {e}")
```

## **Важные замечания:**

1. **Минимализм типа**: `complex` — простой тип данных. Его основная задача — хранить действительную и мнимую часть. Всю
   сложную математику выполняет модуль `cmath`.
2. **Сравнения**: Комплексные числа **не упорядочены**, поэтому работают только операторы `==` и `!=`.
3. **Специальные операции**: Не поддерживаются целочисленное деление (`//`), остаток от деления (`%`) и `divmod()`.
4. **Создание из строк**: Конструктор `complex()` может парсить строки, но метод `complex.from_number()` (Python
   3.14+) — нет.
5. **Для математики используйте `cmath`**: Все математические функции (тригонометрические, логарифмы, корни) для
   комплексных чисел находятся в модуле `cmath`, а не в `math`.

[Содержание](/CONTENTS.md#содержание)

---

## Fraction

## **Тип Fraction в Python (модуль fractions)**

Класс `Fraction` из модуля `fractions` представляет собой рациональное число (дробь) и обеспечивает точные
арифметические операции без ошибок округления, присущих двоичным числам с плавающей запятой. Экземпляры `Fraction`
являются неизменяемыми (immutable) и хешируемыми.

## **Импорт и создание Fraction**

```python
from fractions import Fraction
from decimal import Decimal

# Создание из двух целых чисел (числитель, знаменатель)
f1 = Fraction(3, 4)  # 3/4
f2 = Fraction(6, 8)  # Автоматически сокращается до 3/4
f3 = Fraction(5)  # 5/1
f4 = Fraction()  # 0/1

# Создание из строки (рекомендуется для точности)
f5 = Fraction('3/7')  # 3/7
f6 = Fraction(' -2/5 ')  # -2/5 (пробелы игнорируются)
f7 = Fraction('1.414213')  # 1414213/1000000
f8 = Fraction('7e-6')  # 7/1000000

# Создание из Decimal (точное преобразование)
f9 = Fraction(Decimal('0.1'))  # 1/10

# Создание из float (может быть неточным из-за двоичного представления)
f10 = Fraction(0.75)  # 3/4
f11 = Fraction(1.1)  # 2476979795053773/2251799813685248 (не 11/10!)

# Создание из другого Fraction
f12 = Fraction(f1)  # Копия f1 (3/4)

# Альтернативный конструктор Fraction.from_number() (Python 3.14+)
f13 = Fraction.from_number(42)  # 42/1
f14 = Fraction.from_number((2, 3))  # 2/3 (из пары чисел)

# Примечание: При denominator=0 вызывается ZeroDivisionError
```

## **Арифметические операции**

```python
a = Fraction(2, 3)  # 2/3
b = Fraction(1, 4)  # 1/4

# Базовые операции (возвращают новый Fraction)
print(a + b)  # 11/12
print(a - b)  # 5/12
print(a * b)  # 1/6
print(a / b)  # 8/3
print(a ** 2)  # 4/9 (возведение в степень)
print(-a)  # -2/3
print(+a)  # 2/3
print(abs(Fraction(-3, 4)))  # 3/4

# Целочисленное деление и остаток
print(a // b)  # 2 (int)
print(a % b)  # 5/12 (Fraction)

# Деление с остатком (divmod)
quotient, remainder = divmod(a, b)
print(quotient, remainder)  # 2 5/12

# Поддерживаются смешанные операции с int
print(a + 2)  # 8/3
print(3 * b)  # 3/4
```

## **Атрибуты и методы экземпляра Fraction**

### **Базовые атрибуты (только для чтения)**

```python
f = Fraction(6, 8)  # Автоматически сокращается до 3/4

print(f.numerator)  # 3
print(f.denominator)  # 4 (всегда положительный)
print(type(f.numerator))  # <class 'int'>
print(type(f.denominator))  # <class 'int'>

# Попытка изменения вызовет ошибку
try:
    f.numerator = 5
except AttributeError as e:
    print(f"Ошибка: {e}")  # can't set attribute
```

### **as_integer_ratio() - представление в виде пары целых чисел**

```python
f = Fraction(3, 4)
ratio = f.as_integer_ratio()
print(ratio)  # (3, 4)
print(type(ratio))  # <class 'tuple'>

# Полезно для взаимодействия с другими функциями
import math

f2 = Fraction(*ratio)  # Распаковка обратно
print(f2)  # 3/4

# Использование с float
float_val = float(f)
print(float_val)  # 0.75
print(float_val.as_integer_ratio())  # (3, 4)
```

### **is_integer() - проверка, является ли число целым**

```python
print(Fraction(8, 4).is_integer())  # True (8/4 = 2)
print(Fraction(5, 2).is_integer())  # False (5/2 = 2.5)
print(Fraction(0, 1).is_integer())  # True (0)
print(Fraction(-9, 3).is_integer())  # True (-9/3 = -3)

# Использование в условных операциях
f = Fraction(10, 5)
if f.is_integer():
    print(f"{f} является целым числом: {int(f)}")  # 10/5 является целым числом: 2
```

### **limit_denominator() - рациональное приближение**

```python
# Восстановление "красивой" дроби из неточного float
f_float = Fraction(1.1)  # Неточное представление
print(f_float)  # 2476979795053773/2251799813685248

f_approx = f_float.limit_denominator(100)
print(f_approx)  # 11/10 (точное значение)

# Рациональное приближение иррациональных чисел
pi_approx = Fraction('3.141592653589793').limit_denominator(1000)
print(pi_approx)  # 355/113 (известное приближение π)

# Восстановление точных значений из математических функций
import math

cos_result = Fraction(math.cos(math.pi / 3)).limit_denominator(10)
print(cos_result)  # 1/2 (точное значение cos(π/3))

# Контроль точности через max_denominator
f = Fraction('0.333333')
print(f.limit_denominator(10))  # 1/3
print(f.limit_denominator(100))  # 33/100
```

## **Методы округления и преобразования**

### **4.1. __floor__(), __ceil__(), __round__() - округление**

```python
f = Fraction(7, 3)  # 2.333...

# Округление вниз (floor)
import math

print(f.__floor__())  # 2
print(math.floor(f))  # 2 (предпочтительный способ)

# Округление вверх (ceil)
print(f.__ceil__())  # 3
print(math.ceil(f))  # 3

# Округление до ближайшего целого
print(round(f))  # 2
print(f.__round__())  # 2
print(round(Fraction(5, 2)))  # 2 (округление половины к четному)

# Округление с указанием количества знаков
print(round(Fraction(22, 7), 3))  # 3.143 (возвращает float)
print(Fraction(22, 7).__round__(3))  # 3143/1000 (возвращает Fraction)
```

### **4.2. Преобразование в другие типы**

```python
f = Fraction(5, 2)

print(int(f))  # 2 (отбрасывание дробной части)
print(float(f))  # 2.5
print(str(f))  # '5/2'
print(repr(f))  # "Fraction(5, 2)"

# Явное преобразование знаменателя
print(f.numerator / f.denominator)  # 2.5 (как float)

# Для Decimal нужна предварительная конвертация в строку или float
from decimal import Decimal

print(Decimal(str(f)))  # 2.5
print(Decimal(float(f)))  # 2.5
```

## **Форматирование вывода (Python 3.12+)**

```python
f = Fraction(103993, 33102)

# Общее форматирование (с Python 3.13)
print(format(f, '_'))  # 103_993/33_102 (разделитель групп)
print(format(f, '.^+20'))  # ......+103993/33102...... (выравнивание)

# Форматирование в стиле float (с Python 3.12)
print(format(f, '.10f'))  # 3.1415926530
print(format(f, '.4e'))  # 3.1416e+00
print(format(f, '.2g'))  # 3.1
print(format(Fraction(3, 2), '.0%'))  # 150% (процентный формат)

# Использование в f-строках
print(f"{f:.6f}")  # 3.141593
print(f"{Fraction(1, 7):.3e}")  # 1.429e-01

# Флаг '#' для явного отображения знаменателя
print(format(Fraction(3, 1), ''))  # '3'
print(format(Fraction(3, 1), '#'))  # '3/1'
```

## **Проверки и сравнения**

```python
# Сравнение Fraction
a = Fraction(1, 2)
b = Fraction(2, 4)
c = Fraction(2, 3)

print(a == b)  # True (1/2 == 2/4)
print(a < c)  # True (1/2 < 2/3)
print(a >= Fraction(1, 3))  # True

# Сравнение с другими числовыми типами
print(a == 0.5)  # True
print(a > 0.3)  # True
print(a <= 1)  # True

# Проверка на равенство с учётом автоматического сокращения
print(Fraction(2, 4) == Fraction(1, 2))  # True

# Минимальное и максимальное значение в последовательности
fractions_list = [Fraction(1, 3), Fraction(1, 2), Fraction(3, 4)]
print(min(fractions_list))  # 1/3
print(max(fractions_list))  # 3/4
```

## **Примеры практического использования**

```python
# Точные вычисления с дробями
recipe_ratio = Fraction(3, 4)  # 3/4 чашки муки на порцию
portions = 5
total_flour = recipe_ratio * portions
print(f"Всего муки: {total_flour} чашки")  # 15/4 или 3¾ чашки

# Расчет вероятностей
prob_a = Fraction(1, 6)  # Вероятность выпадения конкретной грани кубика
prob_not_a = 1 - prob_a
print(f"Вероятность не выпадения: {prob_not_a}")  # 5/6

# Работа с периодическими десятичными дробями
periodic = Fraction('0.142857')  # 1/7 в десятичном виде
print(f"Точное значение: {periodic}")  # 142857/1000000
print(f"Сокращенная форма: {periodic.limit_denominator(10)}")  # 1/7

# Финансовые расчеты с точными дробями
price_per_kg = Fraction(75, 2)  # 37.5 руб/кг
weight = Fraction(3, 4)  # 0.75 кг
total_cost = price_per_kg * weight
print(f"Итого: {total_cost} руб.")  # 225/8 = 28.125 руб.
print(f"Итого: {float(total_cost):.2f} руб.")  # 28.13 руб.
```

## **Важные особенности и ограничения**

```python
# 1. Автоматическое сокращение дробей
print(Fraction(10, 20))  # 1/2 (автоматически)
print(Fraction(-3, -9))  # 1/3 (знак нормализуется к числителю)

# 2. Проблемы точности при создании из float
print(Fraction(0.1))  # 3602879701896397/36028797018963968
print(Fraction('0.1'))  # 1/10 (используйте строки!)
print(Fraction(Decimal('0.1')))  # 1/10

# 3. Производительность
# Fraction работает медленнее float, но обеспечивает точность
# Для интенсивных вычислений рассмотрите decimal.Decimal или специализированные библиотеки

# 4. Совместимость с math.gcd (Python 3.9+)
import math

f = Fraction(12, 18)
print(f)  # 2/3 (использует math.gcd для нормализации)

# 5. Специальные методы для числовой башни
print(Fraction(3, 4).real)  # 3/4
print(Fraction(3, 4).imag)  # 0
```

## **Ключевые выводы:**

1. **Используйте строки или Decimal** для создания точных дробей, избегайте прямого создания из float.
2. **Дроби автоматически сокращаются** при создании и операциях.
3. **`limit_denominator()`** — ключевой метод для работы с приближениями и восстановления "красивых" дробей.
4. **Поддержка форматирования** (Python 3.12+) позволяет выводить дроби в различных форматах.
5. **Fraction неизменяем** — все операции возвращают новые объекты.
6. **Идеально подходит для** точных вычислений, финансовых расчётов, работы с вероятностями и пропорциями.

[Содержание](/CONTENTS.md#содержание)

---

## bool

## **Тип bool в Python (логический тип)**

Тип `bool` (логический тип) — это подкласс встроенного типа `int`. В Python существует только два экземпляра этого типа:
`True` (истина, соответствует `1`) и `False` (ложь, соответствует `0`).

## **Создание булевых значений**

```python
# Прямое использование констант True и False
b1 = True
b2 = False

# Создание через конструктор bool()
b3 = bool(1)  # True
b4 = bool(0)  # False
b5 = bool(-5)  # True (любое ненулевое число)
b6 = bool(0.0)  # False
b7 = bool(0.1)  # True

# Создание из строк
b8 = bool('')  # False (пустая строка)
b9 = bool('text')  # True (непустая строка)

# Создание из коллекций
b10 = bool([])  # False (пустой список)
b11 = bool([1, 2])  # True (непустой список)
b12 = bool(None)  # False

# Создание из других булевых значений
b13 = bool(True)  # True
b14 = bool(False)  # False
```

## **Арифметические операции**

Поскольку `bool` является подклассом `int`, поддерживаются все арифметические операции, но результаты преобразуются к
`int`:

```python
# Арифметические операции (True=1, False=0)
print(True + True)  # 2
print(True + False)  # 1
print(False * 10)  # 0
print(True * 3.14)  # 3.14
print(True ** 4)  # 1
print(-True)  # -1

# Целочисленное деление
print(True // 2)  # 0
print(False // 2)  # 0

# Остаток от деления
print(True % 2)  # 1

# Деление с остатком (divmod)
print(divmod(True, 2))  # (0, 1)
```

## **Методы типа bool**

### **as_integer_ratio() - представление в виде дроби**

```python
print(True.as_integer_ratio())  # (1, 1)
print(False.as_integer_ratio())  # (0, 1)

# Полезно для некоторых математических операций
ratio = True.as_integer_ratio()
print(f"True как дробь: {ratio[0]}/{ratio[1]}")
```

### **bit_length() - минимальное количество бит для представления**

```python
print(True.bit_length())  # 1 (для 1 нужен 1 бит)
print(False.bit_length())  # 0 (0 представляется пустой битовой строкой)

# Сравнение с int
print((1).bit_length())  # 1
print((0).bit_length())  # 0
```

### **conjugate() - комплексное сопряжение**

```python
# Для совместимости с int (возвращает self)
print(True.conjugate())  # 1
print(False.conjugate())  # 0

# Фактически то же самое, что и:
print(True.real)  # 1
print(True.imag)  # 0
```

### **Магические методы для операторов сравнения**

```python
# Все методы сравнения возвращают bool
print(True.__eq__(True))  # True
print(True.__ne__(False))  # True
print(True.__lt__(False))  # False (1 < 0)
print(True.__gt__(False))  # True  (1 > 0)
print(True.__le__(True))  # True  (1 ≤ 1)
print(True.__ge__(False))  # True  (1 ≥ 0)

# Сравнение с другими типами
print(True.__eq__(1))  # True
print(False.__eq__(0))  # True
```

### **Магические методы для логических операторов**

```python
# Логическое И (and)
print(True.__and__(True))  # True
print(True.__and__(False))  # False
print(False.__and__(True))  # False

# Логическое ИЛИ (or)
print(True.__or__(False))  # True
print(False.__or__(False))  # False

# Логическое НЕ (not)
print(True.__not__())  # False
print(False.__not__())  # True

# Логическое исключающее ИЛИ (xor)
print(True.__xor__(True))  # False
print(True.__xor__(False))  # True
```

## **Специальные методы bool**

### **4.1. __bool__() - приведение к bool (всегда возвращает self)**

```python
print(True.__bool__())  # True
print(False.__bool__())  # False


# Этот метод вызывается при неявном приведении к bool
class MyClass:
    def __bool__(self):
        return False


obj = MyClass()
print(bool(obj))  # False (вызывает obj.__bool__())
```

### **4.2. __repr__(), __str__() - строковое представление**

```python
print(True.__repr__())  # 'True'
print(False.__repr__())  # 'False'
print(True.__str__())  # 'True'
print(False.__str__())  # 'False'

# В контексте форматирования
print(f"Значение: {True}")  # Значение: True
print(str(False))  # False
print(repr(True))  # True
```

## **Преобразование в другие типы**

```python
# Преобразование в int (явное и неявное)
print(int(True))  # 1
print(int(False))  # 0
print(True + 2)  # 3 (неявное преобразование)

# Преобразование в float
print(float(True))  # 1.0
print(float(False))  # 0.0

# Преобразование в str
print(str(True))  # 'True'
print(str(False))  # 'False'

# Преобразование в комплексные числа
print(complex(True))  # (1+0j)
print(complex(False))  # 0j

# Преобразование в байты
print(bytes(True))  # b'\x01'
print(bytes(False))  # b'\x00'
```

## **Операции сравнения**

```python
# Сравнение bool между собой
print(True == True)  # True
print(True != False)  # True
print(False == False)  # True

# Сравнение с int (True=1, False=0)
print(True == 1)  # True
print(False == 0)  # True
print(True == 1.0)  # True
print(True > 0)  # True
print(False < 0.5)  # True

# Сравнение с другими типами
print(True == 'True')  # False
print(False == '')  # False

# Лексикографическое сравнение (наследуется от int)
print(True > False)  # True (1 > 0)
```

## **Проверка типа и идентичности**

```python
# Проверка типа
print(type(True))  # <class 'bool'>
print(isinstance(True, bool))  # True
print(isinstance(True, int))  # True (bool - подкласс int)

# Проверка идентичности
print(True is True)  # True
print(False is False)  # True
print(True is not False)  # True

# Проверка на соответствие конкретным значениям
print(True is (not False))  # True
print(False is (not True))  # True

# None не равен False, хотя оба "ложны" в булевом контексте
print(None == False)  # False
print(bool(None) == False)  # True
```

## **Важные особенности**

```python
# 1. bool - подкласс int
print(issubclass(bool, int))  # True
print(True + True)  # 2
print(True == 1)  # True

# 2. Только два экземпляра
print(id(True))  # Постоянный id
print(id(False))  # Постоянный id
print(True is (not False))  # True

# 3. Булево значение других объектов
# Все объекты имеют истинностное значение
values = [0, 0.0, '', [], {}, None, 'text', [1], 5]
for v in values:
    print(f"bool({repr(v)}) = {bool(v)}")

# 4. Операторы and, or, not возвращают операнды, а не bool
print(0 and 5)  # 0
print(3 or 0)  # 3
print(not 0)  # True (исключение - всегда возвращает bool)

# 5. Использование в условиях
if True:
    print("Это всегда выполнится")

if False:
    print("Это никогда не выполнится")

# 6. Истинные и ложные значения
false_values = [False, None, 0, 0.0, '', [], {}, set()]
true_values = [True, 1, -1, 0.1, 'a', [0], {'key': 'value'}]
```

## **Примеры использования**

```python
# Флаги состояния
is_authenticated = True
is_admin = False

if is_authenticated and not is_admin:
    print("Доступ для обычного пользователя")

# Переключатели
feature_enabled = False
feature_enabled = not feature_enabled  # Переключить на True

# Подсчет истинных значений
results = [True, False, True, True, False]
true_count = sum(results)  # 3 (True = 1, False = 0)
print(f"Истинных значений: {true_count}")

# Фильтрация списка
data = [1, 0, 3, 0, 5]
filtered = list(filter(bool, data))  # [1, 3, 5]
print(f"Отфильтрованные данные: {filtered}")

# Проверка всех/любого условия
all_true = all([True, True, True])  # True
any_true = any([False, False, True])  # True

# Условные выражения
status = "Включено" if feature_enabled else "Выключено"
print(f"Функция: {status}")
```

## **Ключевые выводы:**

1. **`bool` — подкласс `int`**: `True == 1`, `False == 0`, поддерживает арифметические операции.
2. **Только два экземпляра**: `True` и `False` — синглтоны.
3. **Автоматическое приведение**: Любой объект может быть приведен к `bool` через `__bool__()` или `__len__()`.
4. **Логические операторы**: `and`, `or`, `not` работают с истинностными значениями.
5. **Наследование от int**: Большинство методов унаследованы от `int`.
6. **Использование**: Флаги состояния, условия, фильтрация, подсчеты.

[Содержание](/CONTENTS.md#содержание)

---

## NoneType

## **Тип NoneType в Python (отсутствие значения)**

Тип `NoneType` представляет единственный специальный объект `None`, обозначающий отсутствие значения или "ничего". Это
встроенный тип данных, синглтон (всегда один и тот же объект), который используется для инициализации переменных,
возвращаемых значений функций без результата и индикации неудачи операций. `NoneType` неизменяем и является ложным
значением в булевом контексте.

## **Создание объекта None**

```python
# 1.1 Прямое использование константы None
n1 = None

# 1.2 Через конструктор type(None)() (синглтон, всегда тот же объект)
n2 = type(None)()
print(n1 is n2)  # True — всегда один экземпляр


# 1.3 Автоматическое возвращение функциями без return
def no_return():
    pass  # Неявно возвращает None


result = no_return()
print(result)  # None


# 1.4 Явный возврат None
def explicit_none():
    return None


print(explicit_none())  # None

# Примечание: None чувствителен к регистру, None != 'none' или 'None'
```

## **Атрибуты типа NoneType**

Тип `NoneType` не имеет публичных атрибутов или методов для чтения/записи. Попытка доступа вызовет AttributeError.

```python
n = None

# Нет атрибутов real/imag как у complex
try:
    print(n.real)
except AttributeError as e:
    print(f"Ошибка: {e}")  # 'NoneType' object has no attribute 'real'

# Нет numerator/denominator как у Fraction
try:
    print(n.numerator)
except AttributeError as e:
    print(f"Ошибка: {e}")
```

## **Методы типа NoneType**

Тип `NoneType` не имеет собственных методов. Все попытки вызова методов приводят к TypeError.

```python
n = None

# Нет conjugate() как у complex
try:
    n.conjugate()
except TypeError as e:
    print(f"Ошибка: {e}")  # 'NoneType' object has no attribute 'conjugate'

# Нет as_integer_ratio() как у Fraction/bool
try:
    n.as_integer_ratio()
except TypeError as e:
    print(f"Ошибка: {e}")

# Нет __bool__() переопределения (используется стандартное ложное значение)
print(bool(n))  # False
```

## **Поддерживаемые и неподдерживаемые операции**

```python
n = None

# 5.1 Поддерживаемые операции
print(bool(n))  # False — ложное значение
print(n == None)  # True — сравнение на равенство
print(n is None)  # True — проверка идентичности (рекомендуется)
print(str(n))  # 'None'
print(repr(n))  # 'None'
print(len(str(n)))  # 4

# 5.2 Арифметические операции не поддерживаются
try:
    n + 1
except TypeError as e:
    print(f"Не поддерживается: {e}")  # unsupported operand type(s) for +: 'NoneType' and 'int'

try:
    n * 2
except TypeError as e:
    print(f"Не поддерживается: {e}")

# 5.3 Сравнения порядка не поддерживаются
try:
    n < 1
except TypeError as e:
    print(f"Не поддерживается: {e}")  # '<' not supported between instances of 'NoneType' and 'int'
```

## **Операции сравнения**

```python
n1 = None
n2 = None
value = 42

# 6.1 Поддерживается равенство/неравенство и идентичность
print(n1 == n2)  # True
print(n1 is n2)  # True (синглтон)
print(n1 != value)  # True

# 6.2 Операторы порядка (<, >, <=, >=) не работают
try:
    print(n1 < n2)
except TypeError as e:
    print(f"Ошибка сравнения: {e}")  # '<' not supported between instances of 'NoneType' and 'NoneType'

# 6.3 Сравнение с другими типами
print(None == False)  # False
print(None == 0)  # False
print(None == '')  # False
print(bool(None) == False)  # True (ложное значение)
```

## **Преобразование в другие типы и форматирование**

```python
n = None

# 7.1 Преобразование в строку
print(str(n))  # 'None'
print(repr(n))  # 'None'

# 7.2 Форматирование через f-строки
print(f"n = {n}")  # n = None
print(f"n = {n!r}")  # n = None

# 7.3 Преобразование типов
print(bool(n))  # False
print(int(n))  # TypeError: int() argument must be a string, a bytes-like object or a real number, not 'NoneType'
print(float(n))  # TypeError
print(complex(n))  # TypeError
print(list(n))  # TypeError

# 7.4 В контексте print() или функций без return
print(print("test"))  # test\nNone
```

## **Отличия от других "пустых" значений**

Важно понимать разницу между `None` и другими ложными значениями:

```python
# None vs другие ложные значения
values = [None, False, 0, 0.0, '', [], {}]

for v in values:
    print(f"{repr(v)}: is None={v is None}, bool={bool(v)}")

# Вывод:
# None: is None=True, bool=False
# False: is None=False, bool=False
# 0: is None=False, bool=False
# ...

# Используйте 'is None' для точной проверки, bool() — только для истинностного значения
result = None
if result is None:
    print("Точно None")
if not result:
    print("Ложное значение (может быть None, 0, '', и т.д.)")
```

## **Важные замечания:**

1. **Синглтон**: В Python существует ровно один объект `None`. Все переменные с `None` ссылаются на один id.
2. **Проверка через `is`**: Используйте `is None` / `is not None`, а не `== None` для проверки идентичности.
3. **Ложное значение**: `bool(None) == False`, но `None != False`.
4. **Нет методов/атрибутов**: Любая операция кроме сравнения и строкового представления вызовет ошибку.
5. **Не наследуется**: Невозможно создать подкласс `NoneType` или новый экземпляр.
6. **Стандартные использования**: Возврат функций без `return`, инициализация переменных, индикация неудачи.

## **Ключевые выводы:**

1. **`None` — единственный синглтон типа `NoneType`**, обозначающий отсутствие значения.
2. **Проверяйте через `is None`**, а не `==` или `bool()` для точности.
3. **Не поддерживает арифметику, методы или атрибуты** — только базовые сравнения и строковое представление.
4. **Идеален для**: инициализации, возврата "пустого" результата, индикации ошибок без исключений.
5. **Различайте от ложных значений** (`False`, `0`, `''`) — используйте `is` для точной идентификации.
6. **Immutable и singleton**: всегда один объект, неизменяемый.

[Содержание](/CONTENTS.md#содержание)

---

## str

## **Тип str в Python (строковый тип)**

Тип `str` представляет неизменяемые последовательности символов Unicode. Это встроенный тип данных, поддерживающий
широкий спектр методов для манипуляции текстом, форматирования и поиска. Строки неизменяемы (immutable), хешируемы и
широко используются для представления текста.

## **Создание строк**

```python
# 1.1 Литералы строк
s1 = "hello"
s2 = 'world'
s3 = """многострочная
строка"""
s4 = r"сырой текст\n не экранируется"  # raw-string
s5 = f"Форматированная: {42}"  # f-string (Python 3.6+)
s6 = b"байты".decode()  # из байтов

# 1.2 Через конструктор str()
s7 = str(42)  # "42"
s8 = str([1, 2])  # "[1, 2]"
s9 = str(None)  # "None"

# 1.3 Преобразование других типов
s10 = str(True)  # "True"
s11 = chr(97)  # "a" из кода символа

# 1.4 Пустая строка
empty = ""
```

## **Атрибуты типа str**

Тип `str` не имеет публичных атрибутов для чтения/записи данных (как `real` у `complex`). Все операции выполняются через
методы.

```python
s = "test"

# Нет атрибутов типа numerator/denominator
try:
    print(s.numerator)
except AttributeError as e:
    print(f"Ошибка: {e}")  # 'str' object has no attribute 'numerator'

print(len(s))  # 4 — длина через встроенную функцию
print(type(len(s)))  # <class 'int'>
```

## **Методы типа str**

Тип `str` имеет богатый набор методов (более 50). Основные разделены на категории.

### **Методы изменения регистра**

```python
s = "Hello World"

print(s.lower())  # "hello world"
print(s.upper())  # "HELLO WORLD"
print(s.title())  # "Hello World"
print(s.capitalize())  # "Hello world"
print(s.swapcase())  # "hELLO wORLD"
```

### **Методы проверки содержания**

```python
s = "Python 3.12"

print(s.isalpha())  # False (есть цифры)
print(s.isdigit())  # False
print("123".isdigit())  # True
print(s.islower())  # True
print(s.startswith("Py"))  # True
print(s.endswith("12"))  # True
```

### **Методы поиска и замены**

```python
s = "hello, world!"

print(s.find("world"))  # 7 (позиция)
print(s.index("world"))  # 7 (ошибка при отсутствии)
print(s.replace("world", "Python"))  # "hello, Python!"
print(s.count("l"))  # 3
```

## **Поддерживаемые операции**

```python
s1 = "hello"
s2 = "world"

# 5.1 Конкатенация и повторение
print(s1 + s2)  # "helloworld"
print(s1 * 3)  # "hellohellohello"

# 5.2 Срезы и индексация
print(s1[0])  # "h"
print(s1[1:4])  # "ell"
print(s1[::-1])  # "olleh"

# 5.3 Поддерживаемые сравнения
print("abc" < "abd")  # True (лексикографически)
print("abc" == "abc")  # True

# 5.4 Неподдерживаемые операции
try:
    s1 / s2  # TypeError
except TypeError as e:
    print(f"Не поддерживается: {e}")

try:
    s1 // s2
except TypeError as e:
    print(f"Не поддерживается: {e}")
```

## **Операции сравнения**

```python
s1 = "apple"
s2 = "banana"
s3 = "Apple"

# Лексикографическое сравнение (Unicode коды)
print(s1 < s2)  # True ("a" < "b")
print(s1 == s1)  # True
print(s1 != s2)  # True

# Регистр важен
print(s1 < s3)  # False ("a" > "A" в ASCII)

# Сравнение длин при равных префиксах
print("abc" < "abcd")  # True (короче)
```

## **Преобразование и форматирование**

```python
s = "test"

# 7.1 В другие типы
print(int("42"))  # 42
print(float("3.14"))  # 3.14
print(bool(""))  # False
print(bool(" "))  # True
print(list("abc"))  # ['a', 'b', 'c']

# 7.2 Форматирование (f-строки, Python 3.6+)
name = "Python"
version = 3.12
print(f"{name} {version}")  # "Python 3.12"
print(f"{version:.1f}")  # "3.1"

# 7.3 Старые методы форматирования
print("{} {}".format(name, version))  # "Python 3.12"
print("%s %.1f" % (name, version))  # "Python 3.1"
```

## **Важные особенности**

```python
# 1. Неизменяемость (immutable)
s = "hello"
try:
    s[0] = "H"  # TypeError
except TypeError as e:
    print(f"Ошибка: {e}")

# 2. Кеширование малых строк (interning)
s1 = "hello"
s2 = "hello"
print(s1 is s2)  # True (для коротких строк)

# 3. Ложные значения
print(bool(""))  # False
print(bool(" "))  # True (непустая)

# 4. Unicode поддержка
print(len("привет"))  # 6 (Unicode символы)
print("привет"[0])  # "п"

# 5. Экранирование
print("line1\nline2")  # Перенос строки
print(r"raw\n")  # "raw\n" (сырой)
```

## **Важные замечания:**

1. **Неизменяемость**: Строки нельзя изменять на месте — методы возвращают новые строки.
2. **Unicode**: Полная поддержка Unicode символов, эмодзи, CJK.
3. **Сравнение**: Лексикографическое по Unicode кодам, регистр важен.
4. **Память**: Короткие строки интернируются (один объект на значение).
5. **Производительность**: Методы оптимизированы, но повторные изменения строк неэффективны.
6. **f-строки**: Самый быстрый и читаемый способ форматирования (Python 3.6+).

## **Ключевые выводы:**

1. **`str` — неизменяемый Unicode текст** с богатым API методов.
2. **Не поддерживает арифметику**, только конкатенацию `+` и повторение `*`.
3. **f-строки** — предпочтительный способ форматирования.
4. **Лексикографическое сравнение**, чувствительно к регистру.
5. **Идеален для**: текста, ключей словарей, форматирования, сериализации.
6. **Пустая строка `""` — единственное ложное значение** среди строк.

[Содержание](/CONTENTS.md#содержание)

---

## list

## **Тип list в Python (список)**

Тип `list` представляет изменяемые упорядоченные последовательности произвольных объектов. Это встроенный тип данных,
поддерживающий динамическое изменение размера, индексацию, срезы и богатый набор методов для манипуляции элементами.
Списки мутабельны, не хешируемы и широко используются для хранения коллекций данных.

## **Создание списков**

```python
# 1.1 Литералы списков
l1 = [1, 2, 3]
l2 = ['a', 'b', 'c']
l3 = []  # Пустой список
l4 = [42, 'text', [1, 2]]  # Вложенные списки

# 1.2 Через конструктор list()
l5 = list("abc")  # ['a', 'b', 'c']
l6 = list(range(3))  # [0, 1, 2]
l7 = list({1, 2, 3})  # [1, 2, 3] (порядок не гарантирован)

# 1.3 Из итерируемых объектов
l8 = list((10, 20, 30))  # [10, 20, 30]
l9 = [*"hello"]  # ['h', 'e', 'l', 'l', 'o'] (распаковка)

# 1.4 Список со повторяющимися элементами
l10 = [0] * 5  # [0, 0, 0, 0, 0]
```

## **Атрибуты типа list**

Тип `list` не имеет публичных атрибутов для чтения/записи данных. Размер доступен через `len()`.

```python
l = [1, 2, 3]

# Нет атрибутов типа real/numerator
try:
    print(l.real)
except AttributeError as e:
    print(f"Ошибка: {e}")  # 'list' object has no attribute 'real'

print(len(l))  # 3 — количество элементов
print(type(len(l)))  # <class 'int'>
```

## **Методы типа list**

Тип `list` имеет методы для изменения (мутации) и неизменяемые операции.

### **Методы изменения списка (мутация)**

```python
l = [1, 2, 3]

l.append(4)  # [1, 2, 3, 4]
l.insert(1, 'x')  # [1, 'x', 2, 3, 4]
l.pop()  # Удаляет 4, возвращает 4 → [1, 'x', 2, 3]
l.remove('x')  # [1, 2, 3]
l.clear()  # [] — очищает список
```

### **Методы сортировки**

```python
l = [3, 1, 4, 1, 5]

l.sort()  # [1, 1, 3, 4, 5] (мутация)
print(l)

l2 = [3, 1, 4, 1, 5]
sorted_l = sorted(l2, reverse=True)  # [5, 4, 3, 1, 1] (новый список)
print(sorted_l)
```

### **Методы поиска и проверки**

```python
l = ['a', 'b', 'c', 'b']

print(l.index('b'))  # 1 (первое вхождение)
print(l.count('b'))  # 2
print('x' in l)  # False
print(l.index('b', 2))  # 3 (поиск с позиции)
```

## **Поддерживаемые операции**

```python
l1 = [1, 2]
l2 = [3, 4]

# 5.1 Конкатенация и повторение
print(l1 + l2)  # [1, 2, 3, 4]
print(l1 * 3)  # [1, 2, 1, 2, 1, 2]

# 5.2 Индексация и срезы
print(l1[0])  # 1
print(l1[-1])  # 2
print(l1[1:])  # [2]
print(l1[::-1])  # [2, 1]

# 5.3 Срезы с присваиванием (мутация)
l1[0:1] = ['x']  # ['x', 2]

# 5.4 Сравнения (лексикографические)
print([1, 2] < [1, 3])  # True

# 5.5 Неподдерживаемые: //, %, divmod
try:
    l1 // l2
except TypeError as e:
    print(f"Не поддерживается: {e}")
```

## **Операции сравнения**

```python
l1 = [1, 2, 3]
l2 = [1, 2, 3]
l3 = [1, 2, 4]

# Лексикографическое сравнение (как словари)
print(l1 == l2)  # True
print(l1 < l3)  # True (3 < 4 на 3-й позиции)
print([1, 2] < [1, 2, 3])  # True (короче при равных префиксах)

# Разные типы сравниваются по элементам
print([1] == [True])  # True (1 == True)
```

## **Преобразование в другие типы и форматирование**

```python
l = [1, 2, 3]

# 7.1 В другие коллекции
print(tuple(l))  # (1, 2, 3)
print(set(l))  # {1, 2, 3}
print(list(l))  # [1, 2, 3] (копия)

# 7.2 В строку
print(str(l))  # '[1, 2, 3]'
print(repr(l))  # '[1, 2, 3]'

# 7.3 Распаковка
print(*l)  # 1 2 3
print(sum(l))  # 6
print(len(l))  # 3

# 7.4 Список генераторы
squares = [x ** 2 for x in l]  # [1, 4, 9]
```

## **Важные особенности**

```python
# 1. Мутабельность (можно изменять)
l = [1, 2]
l[0] = 99  # [99, 2]
l.append(3)  # [99, 2, 3]

# 2. Ссылочная семантика
l1 = [1, 2]
l2 = l1
l2[0] = 99  # l1 тоже изменился! [99, 2]

# 3. Копирование
import copy

l_copy = l1[:]  # Поверхностная копия
l_deep = copy.deepcopy(l1)  # Глубокая копия

# 4. Ложные значения
print(bool([]))  # False
print(bool([0]))  # True

# 5. Производительность O(1) append, O(n) insert/delete
```

## **Важные замечания:**

1. **Мутабельность**: Методы `append`, `pop`, `sort` изменяют список на месте.
2. **Ссылки**: Присваивание создаёт ссылку, используйте `[:]`, `copy()`, `list()` для копий.
3. **Емкость**: Списки перераспределяют память автоматически (over-allocation).
4. **Сравнение**: Лексикографическое, сначала сравнивает элементы по порядку.
5. **Память**: Указатели на объекты, не копирует данные при присваивании.
6. **Не хешируем**: Нельзя использовать как ключи словарей из-за изменяемости.

## **Ключевые выводы:**

1. **`list` — мутабельная упорядоченная коллекция** произвольных объектов.
2. **Поддерживает индексацию, срезы, мутацию** и богатый API методов.
3. **Осторожно с ссылками**: `l1 = l2` не копирует данные.
4. **Пустой список `[]` — единственное ложное значение** среди списков.
5. **Идеален для**: динамических коллекций, стеков, очередей, массивов.
6. **`[*iter]` и `list(iter)`** — основные способы создания из итерируемых.

[Содержание](/CONTENTS.md#содержание)

---

## tuple

Тип `tuple` представляет неизменяемые упорядоченные последовательности произвольных объектов. Это встроенный тип данных,
поддерживающий индексацию, срезы и хешируемость для использования в множествах и словарях. Кортежи иммутабельны,
хешируемы (если элементы хешируемы) и используются для хранения неизменяемых коллекций данных.

## **Создание кортежей**

```python
# 1.1 Литералы кортежей
t1 = (1, 2, 3)
t2 = ('a', 'b', 'c')
t3 = ()  # Пустой кортеж
t4 = (42, 'text', (1, 2))  # Вложенные кортежи
t5 = 1, 2, 3  # Без скобок (упаковка)

# 1.2 Через конструктор tuple()
t6 = tuple("abc")  # ('a', 'b', 'c')
t7 = tuple(range(3))  # (0, 1, 2)
t8 = tuple({1, 2, 3})  # (1, 2, 3) (порядок не гарантирован)

# 1.3 Из итерируемых объектов
t9 = tuple([10, 20, 30])  # (10, 20, 30)
t10 = (*"hello",)  # ('h', 'e', 'l', 'l', 'o') (распаковка с запятой)

# 1.4 Кортеж с повторяющимися элементами
t11 = (0,) * 5  # (0, 0, 0, 0, 0)
```

## **Атрибуты типа tuple**

Тип `tuple` не имеет публичных атрибутов для чтения/записи данных. Размер доступен через `len()`.

```python
t = (1, 2, 3)

# Нет атрибутов типа real/numerator
try:
    print(t.real)
except AttributeError as e:
    print(f"Ошибка: {e}")  # 'tuple' object has no attribute 'real'

print(len(t))  # 3 — количество элементов
print(type(len(t)))  # <class 'int'>
```

## **Методы типа tuple**

Тип `tuple` имеет ограниченный набор методов, так как иммутабелен. Только `count()` и `index()`.

### **Методы поиска**

```python
t = ('a', 'b', 'c', 'b')

print(t.index('b'))  # 1 (первое вхождение)
print(t.count('b'))  # 2
print('x' in t)  # False
print(t.index('b', 2))  # 3 (поиск с позиции)

# Нет мутации: append, pop, remove и т.д.
try:
    t.append(4)
except AttributeError as e:
    print(f"Не поддерживается: {e}")  # 'tuple' object has no attribute 'append'
```

## **Поддерживаемые операции**

```python
t1 = (1, 2)
t2 = (3, 4)

# 4.1 Конкатенация и повторение
print(t1 + t2)  # (1, 2, 3, 4)
print(t1 * 3)  # (1, 2, 1, 2, 1, 2)

# 4.2 Индексация и срезы
print(t1[0])  # 1
print(t1[-1])  # 2
print(t1[1:])  # (2,)
print(t1[::-1])  # (2, 1)

# 4.3 Срезы возвращают новый кортеж (нет присваивания!)
t_slice = t1[0:1]  # (1,)

# 4.4 Сравнения (лексикографические)
print((1, 2) < (1, 3))  # True

# 4.5 Неподдерживаемые: //, %, divmod
try:
    t1 // t2
except TypeError as e:
    print(f"Не поддерживается: {e}")
```

## **Операции сравнения**

```python
t1 = (1, 2, 3)
t2 = (1, 2, 3)
t3 = (1, 2, 4)

# Лексикографическое сравнение (как словари)
print(t1 == t2)  # True
print(t1 < t3)  # True (3 < 4 на 3-й позиции)
print((1, 2) < (1, 2, 3))  # True (короче при равных префиксах)

# Разные типы сравниваются по элементам
print((1,) == (True,))  # True (1 == True)
```

## **Преобразование в другие типы и форматирование**

```python
t = (1, 2, 3)

# 6.1 В другие коллекции
print(list(t))  # [1, 2, 3]
print(set(t))  # {1, 2, 3}
print(tuple(t))  # (1, 2, 3) (копия)

# 6.2 В строку
print(str(t))  # '(1, 2, 3)'
print(repr(t))  # '(1, 2, 3)'

# 6.3 Распаковка
a, b, c = t  # 1 2 3
print(sum(t))  # 6
print(len(t))  # 3

# 6.4 Генераторы кортежей
squares = tuple(x ** 2 for x in t)  # (1, 4, 9)
```

## **Важные особенности**

```python
# 1. Иммутабельность (нельзя изменять)
t = (1, 2)
# t[0] = 99  # TypeError!

# 2. Хешируемость (если элементы хешируемы)
print(hash((1, 2)))  # Работает
d = {(1, 2): 'value'}  # OK

# 3. Одноэлементный кортеж требует запятой
t_single = (42,)  # Кортеж
t_not = (42)  # int!

# 4. Ссылочная семантика (но иммутабельны)
t1 = (1, 2)
t2 = t1
print(t1 is t2)  # True

# 5. Копирование
import copy

t_copy = t1[:]  # Тот же объект (иммутабелен)

# 6. Ложные значения
print(bool(()))  # False
print(bool((0,)))  # True
```

## **Важные замечания:**

1. **Иммутабельность**: Нет методов мутации, только чтение.
2. **Хешируемость**: Можно использовать как ключи словарей/множества.
3. **Емкость**: Фиксированный размер после создания, экономия памяти.
4. **Сравнение**: Лексикографическое, как у списков.
5. **Память**: Указатели на объекты, компактнее list для постоянных данных.
6. **Распаковка**: Идеален для множественного присваивания `a, b = t`.

## **Ключевые выводы:**

1. **`tuple` — иммутабельная упорядоченная коллекция** произвольных объектов.
2. **Поддерживает индексацию, срезы, конкатенацию**, но без мутации.
3. **Хешируем**: ключи словарей, элементы множеств.
4. **Пустой кортеж `()` — единственное ложное значение** среди кортежей.
5. **Идеален для**: констант, возвращаемых значений, распаковки, ключей.
6. **`(*iter,)` и `tuple(iter)`** — основные способы создания.

[Содержание](/CONTENTS.md#содержание)

---

## dict

Тип `dict` представляет изменяемые неупорядоченные (с Python 3.7+ — вставочные) коллекции пар "ключ-значение". Это
встроенный тип данных,
поддерживающий доступ по ключам, динамическое изменение размера и богатый набор методов для манипуляции парами.
Словари мутабельны, ключи должны быть хешируемыми, широко используются для ассоциативных массивов и хранения
структурированных данных.

## **Создание словарей**

```python
# 1.1 Литералы словарей
d1 = {'a': 1, 'b': 2}
d2 = {42: 'answer', 'list': [1, 2]}
d3 = {}  # Пустой словарь
d4 = {'nested': {'key': 'value'}}  # Вложенные словари

# 1.2 Через конструктор dict()
d5 = dict(a=1, b=2)  # {'a': 1, 'b': 2}
d6 = dict([('x', 10), ('y', 20)])  # {'x': 10, 'y': 20}
d7 = dict.fromkeys(['a', 'b'], 0)  # {'a': 0, 'b': 0}

# 1.3 Из итерируемых объектов пар
d8 = dict(zip('abc', [1, 2, 3]))  # {'a': 1, 'b': 2, 'c': 3}
d9 = {**{'a': 1}, 'b': 2}  # {'a': 1, 'b': 2} (merge)

# 1.4 Словарь с повторяющимися значениями
d10 = dict.fromkeys(range(3), 'default')  # {0: 'default', 1: 'default', 2: 'default'}
```

## **Атрибуты типа dict**

Тип `dict` не имеет публичных атрибутов для чтения/записи данных. Размер доступен через `len()`.

```python
d = {'a': 1, 'b': 2}

# Нет атрибутов типа real/numerator
try:
    print(d.real)
except AttributeError as e:
    print(f"Ошибка: {e}")  # 'dict' object has no attribute 'real'

print(len(d))  # 2 — количество пар ключ-значение
print(type(len(d)))  # <class 'int'>
```

## **Методы типа dict**

Тип `dict` имеет методы для изменения (мутации) и неизменяемые операции.

### **Методы изменения словаря (мутация)**

```python
d = {'a': 1, 'b': 2}

d['c'] = 3  # {'a': 1, 'b': 2, 'c': 3}
d['a'] = 99  # {'a': 99, 'b': 2, 'c': 3}
del d['b']  # {'a': 99, 'c': 3}
d.pop('c')  # Удаляет 3, возвращает 3 → {'a': 99}
d.clear()  # {} — очищает словарь
```

### **Методы работы с элементами**

```python
d = {'a': 1, 'b': 2, 'a': 99}  # Последнее значение перезаписывает

print(d.get('x', 'default'))  # 'default' (без KeyError)
print(d.setdefault('x', 0))  # 0 и добавляет {'a': 99, 'b': 2, 'x': 0}
```

### **Методы просмотра и копирования**

```python
d = {'a': 1, 'b': 2, 'c': 3}

print(d.keys())  # dict_keys(['a', 'b', 'c'])
print(d.values())  # dict_values([1, 2, 3])
print(d.items())  # dict_items([('a', 1), ('b', 2), ('c', 3)])

d_copy = d.copy()  # Поверхностная копия
print(d_copy)  # {'a': 1, 'b': 2, 'c': 3}
```

### **Методы обновления**

```python
d1 = {'a': 1}
d2 = {'b': 2, 'a': 99}

d1.update(d2)  # {'a': 99, 'b': 2}
d1 |= {'c': 3}  # {'a': 99, 'b': 2, 'c': 3} (Python 3.9+)

# | и |= для merge
```

## **Поддерживаемые операции**

```python
d1 = {'a': 1, 'b': 2}
d2 = {'b': 3, 'c': 4}

# 4.1 Проверка наличия ключа
print('a' in d1)  # True
print('x' not in d1)  # True

# 4.2 Доступ и присваивание
print(d1['a'])  # 1
d1['x'] = 10  # Добавляет

# 4.3 Merge операции (Python 3.9+)
print(d1 | d2)  # {'a': 1, 'b': 3, 'c': 4}
d3 = d1 | {**d2, 'd': 5}

# 4.4 Сравнения
print({'a': 1} == {'a': 1})  # True

# 4.5 Неподдерживаемые: +, *, //
try:
    d1 + d2
except TypeError as e:
    print(f"Не поддерживается: {e}")
```

## **Операции сравнения**

```python
d1 = {'a': 1, 'b': 2}
d2 = {'a': 1, 'b': 2}
d3 = {'a': 1, 'b': 3}

print(d1 == d2)  # True (игнорирует порядок)
print(d1 != d3)  # True

# Сравнение по содержимому (не по порядку)
print({'a': 1, 'b': 2} == {'b': 2, 'a': 1})  # True

# Разные типы
print({1: 'one'} == {True: 'one'})  # True (1 == True)
```

## **Преобразование в другие типы и форматирование**

```python
d = {'a': 1, 'b': 2, 'c': 3}

# 6.1 В другие коллекции
print(list(d))  # ['a', 'b', 'c'] (ключи)
print(list(d.values()))  # [1, 2, 3]
print(list(d.items()))  # [('a', 1), ('b', 2), ('c', 3)]
print(tuple(d.items()))  # (('a', 1), ('b', 2), ('c', 3))

# 6.2 В строку
print(str(d))  # "{'a': 1, 'b': 2, 'c': 3}"
print(repr(d))  # "{'a': 1, 'b': 2, 'c': 3}"

# 6.3 Распаковка (Python 3.5+)
print({**d, 'd': 4})  # {'a': 1, 'b': 2, 'c': 3, 'd': 4}

# 6.4 Словарные генераторы
squares = {k: v ** 2 for k, v in d.items()}  # {'a': 1, 'b': 4, 'c': 9}
```

## **Важные особенности**

```python
# 1. Мутабельность (можно изменять)
d = {'a': 1}
d['b'] = 2  # {'a': 1, 'b': 2}
del d['a']  # {'b': 2}

# 2. Ключи должны быть хешируемыми
try:
    {'list': [1, 2]}  # TypeError!
except TypeError as e:
    print(f"Ошибка: {e}")

# 3. Ссылочная семантика
d1 = {'a': 1}
d2 = d1
d2['b'] = 2  # d1 тоже изменился!

# 4. Копирование
import copy

d_copy = d1.copy()  # Поверхностная
d_deep = copy.deepcopy(d1)  # Глубокая

# 5. Порядок вставки (с 3.7+)
d = {}
d['z'] = 1
d['a'] = 2
print(list(d))  # ['z', 'a'] — порядок сохранен

# 6. Ложные значения
print(bool({}))  # False
print(bool({'a': 0}))  # True
```

## **Важные замечания:**

1. **Ключи хешируемы**: Только immutable типы (str, int, tuple, но не list/dict).
2. **Мутабельность**: Методы `update()`, `pop()`, `clear()` изменяют на месте.
3. **Порядок**: С Python 3.7+ сохраняется порядок вставки (гарантировано с 3.7).
4. **Сравнение**: По содержимому, игнорирует порядок ключей.
5. **Память**: Хеш-таблица с over-allocation, O(1) средняя сложность доступа.
6. **Не хешируем**: Нельзя использовать dict как ключ из-за изменяемости.

## **Ключевые выводы:**

1. **`dict` — мутабельная коллекция пар "ключ-значение"** с хешируемыми ключами.
2. **Доступ O(1), поддерживает merge (`|`), `get()`, `setdefault()`**.
3. **Осторожно с ссылками**: `d1 = d2` не копирует данные.
4. **Пустой словарь `{}` — единственное ложное значение** среди словарей.
5. **Идеален для**: конфигураций, кэшей, JSON-подобных данных, счетчиков.
6. **`dict.fromkeys()`, `dict(iter)`** — основные способы создания.

[Содержание](/CONTENTS.md#содержание)

---

## set

Тип `set` представляет изменяемые неупорядоченные коллекции уникальных хешируемых объектов. Это встроенный тип данных,
поддерживающий быстрый поиск, добавление/удаление элементов и операции над множествами. Множества мутабельны,
не индексируемы (нет порядка), элементы должны быть хешируемыми и не могут повторяться.

## **Создание множеств**

```python
# 1.1 Литералы множеств
s1 = {1, 2, 3}
s2 = {'a', 'b', 'c'}
s3 = set()  # Пустое множество (не {}!)

# 1.2 Через конструктор set()
s4 = set("abc")  # {'a', 'b', 'c'}
s5 = set(range(3))  # {0, 1, 2}
s6 = set([1, 2, 2, 3])  # {1, 2, 3} (дубликаты удалены)

# 1.3 Из итерируемых объектов
s7 = set((10, 20, 30))  # {10, 20, 30}
s8 = set({"key": "value"}.values())  # {'value'}

# 1.4 Пустое множество только через set()
empty = {}  # Это словарь!
print(type(empty))  # <class 'dict'>
print(type(set()))  # <class 'set'>
```

## **Атрибуты типа set**

Тип `set` не имеет публичных атрибутов для чтения/записи данных. Размер доступен через `len()`.

```python
s = {1, 2, 3}

# Нет атрибутов типа real/numerator
try:
    print(s.real)
except AttributeError as e:
    print(f"Ошибка: {e}")  # 'set' object has no attribute 'real'

print(len(s))  # 3 — количество элементов
print(type(len(s)))  # <class 'int'>
```

## **Методы типа set**

Тип `set` имеет методы для мутации и неизменяемые операции над множествами.

### **Методы изменения множества (мутация)**

```python
s = {1, 2, 3}

s.add(4)  # {1, 2, 3, 4}
s.remove(2)  # {1, 3, 4} (KeyError при отсутствии)
s.discard(99)  # Ничего не делает, если нет
s.pop()  # Удаляет/возвращает произвольный → {1, 4} (порядок не гарантирован)
s.clear()  # set() — очищает множество
```

### **Операции над множествами (мутация)**

```python
s1 = {1, 2, 3}
s2 = {3, 4, 5}

s1.update(s2)  # {1, 2, 3, 4, 5} (объединение)
s1.intersection_update(s2)  # {3, 4, 5} ∩ {3, 4, 5} = {3, 4, 5}
s1.difference_update(s2)  # {1, 2} (разность)
```

### **Методы поиска и проверки**

```python
s = {1, 2, 3, 2}

print(2 in s)  # True
print(99 in s)  # False
print(s.isdisjoint({4, 5}))  # True (нет общих элементов)
print({1, 2}.issubset(s))  # True (подмножество)
print(s.issuperset({1}))  # True (надмножество)
```

## **Поддерживаемые операции**

```python
s1 = {1, 2}
s2 = {2, 3}

# 5.1 Операции над множествами (неизменяемые)
print(s1 | s2)  # {1, 2, 3} (объединение)
print(s1 & s2)  # {2} (пересечение)
print(s1 - s2)  # {1} (разность)
print(s1 ^ s2)  # {1, 3} (симметричная разность)

# 5.2 Нет индексации/срезов
try:
    print(s1[0])
except TypeError as e:
    print(f"Не поддерживается: {e}")  # 'set' object is not subscriptable

# 5.3 Нет конкатенации/повторения
try:
    s1 + s2
except TypeError as e:
    print(f"Не поддерживается: {e}")

# 5.4 Сравнения (подмножества)
print({1} < {1, 2})  # True (строгое подмножество)
```

## **Операции сравнения**

```python
s1 = {1, 2, 3}
s2 = {1, 2, 3}
s3 = {1, 2}

print(s1 == s2)  # True
print(s1 != s3)  # True
print(s3 < s1)  # True (подмножество)
print(s1 > s3)  # True (надмножество)
print({1, 2} <= {1, 2, 3})  # True

# Разные типы: сравнение с list не работает напрямую
print({1} == [1])  # False
```

## **Преобразование в другие типы и форматирование**

```python
s = {1, 2, 3}

# 7.1 В другие коллекции
print(list(s))  # [1, 2, 3] (порядок не гарантирован)
print(tuple(s))  # (1, 2, 3)
print(frozenset(s))  # frozenset({1, 2, 3})

# 7.2 В строку
print(str(s))  # '{1, 2, 3}'
print(repr(s))  # '{1, 2, 3}'

# 7.3 Распаковка
print(*s)  # 1 2 3 (порядок не гарантирован)
print(len(s))  # 3
print(sum(s))  # 6

# 7.4 Множество генераторы (set comprehension)
squares = {x ** 2 for x in range(3)}  # {0, 1, 4}
```

## **Важные особенности**

```python
# 1. Уникальность и отсутствие порядка
s = {1, 2, 2, 3}
print(s)  # {1, 2, 3} (дубликаты удалены, порядок не гарантирован)

# 2. Только хешируемые элементы
try:
    s = {1, [1, 2]}  # List не хешируем!
except TypeError as e:
    print(f"Ошибка: {e}")

# 3. Ссылочная семантика
s1 = {1, 2}
s2 = s1
s2.add(3)  # s1 тоже изменился! {1, 2, 3}

# 4. Копирование
import copy

s_copy = s1.copy()  # Поверхностная копия
s_deep = copy.deepcopy(s1)  # Глубокая (для set то же)

# 5. Ложные значения
print(bool(set()))  # False
print(bool({0}))  # True

# 6. Хешируемость? Нет, мутабельно
try:
    hash(s1)
except TypeError as e:
    print(f"Не хешируется: {e}")
```

## **Важные замечания:**

1. **Уникальность**: Автоматически удаляет дубликаты, порядок не гарантирован (до Python 3.7).
2. **Хешируемые элементы**: Только immutable типы (int, str, tuple, frozenset).
3. **Не индексируем**: Нет `s[0]`, используйте `in` для проверки O(1).
4. **Ссылки**: `s1 = s2` — ссылка, используйте `copy()` для копий.
5. **Память**: Хэш-таблица с over-allocation для быстрого роста.
6. **Не хешируем**: `set` нельзя использовать как ключ словаря.

## **Ключевые выводы:**

1. **`set` — мутабельная неупорядоченная коллекция уникальных** хешируемых объектов.
2. **Идеален для**: проверки уникальности, операций над множествами, быстрого поиска.
3. **O(1) операции**: `add`, `remove`, `in` благодаря хэш-таблице.
4. **Пустое множество `set()` — ложное значение**, непустые — истинные.
5. **Альтернатива `frozenset()`** для неизменяемых/хешируемых множеств.
6. **`{*iter}` и `set(iter)`** — основные способы создания.

[Содержание](/CONTENTS.md#содержание)

---

## frozenset

Тип `frozenset` представляет неизменяемые неупорядоченные коллекции уникальных хешируемых объектов. Это встроенный тип
данных,
поддерживающий быстрый поиск и операции над множествами. `frozenset` иммутабелен, хешируем, элементы должны быть
хешируемыми
и не могут повторяться. Идеален для использования как ключи словарей или элементов множеств.

## **Создание frozenset**

```python
# 1.1 Литералы frozenset (только через конструктор)
fs1 = frozenset([1, 2, 3])
fs2 = frozenset('abc')  # frozenset({'a', 'b', 'c'})
fs3 = frozenset()  # Пустое frozenset

# 1.2 Через конструктор frozenset()
fs4 = frozenset(range(3))  # frozenset({0, 1, 2})
fs5 = frozenset([1, 2, 2, 3])  # frozenset({1, 2, 3}) (дубликаты удалены)

# 1.3 Из итерируемых объектов
fs6 = frozenset((10, 20, 30))  # frozenset({10, 20, 30})
fs7 = frozenset({'a', 'b', 'c'})  # frozenset({'a', 'b', 'c'})

# 1.4 Из set
s = {1, 2, 3}
fs8 = frozenset(s)  # frozenset({1, 2, 3})
```

## **Атрибуты типа frozenset**

Тип `frozenset` не имеет публичных атрибутов для чтения/записи данных. Размер доступен через `len()`.

```python
fs = frozenset([1, 2, 3])

# Нет атрибутов типа real/numerator
try:
    print(fs.real)
except AttributeError as e:
    print(f"Ошибка: {e}")  # 'frozenset' object has no attribute 'real'

print(len(fs))  # 3 — количество элементов
print(type(len(fs)))  # <class 'int'>
```

## **Методы типа frozenset**

Тип `frozenset` имеет **только неизменяемые методы** (нет мутации).

### **Операции над множествами (неизменяемые)**

```python
fs1 = frozenset([1, 2, 3])
fs2 = frozenset([3, 4, 5])

print(fs1.union(fs2))  # frozenset({1, 2, 3, 4, 5})
print(fs1.intersection(fs2))  # frozenset({3})
print(fs1.difference(fs2))  # frozenset({1, 2})
print(fs1.symmetric_difference(fs2))  # frozenset({1, 2, 4, 5})
```

### **Методы поиска и проверки**

```python
fs = frozenset([1, 2, 3, 2])

print(2 in fs)  # True
print(99 in fs)  # False
print(fs.isdisjoint(frozenset([4, 5])))  # True
print(frozenset([1, 2]).issubset(fs))  # True
print(fs.issuperset(frozenset([1])))  # True
```

## **Поддерживаемые операции**

```python
fs1 = frozenset([1, 2])
fs2 = frozenset([2, 3])

# 5.1 Операции над множествами (неизменяемые)
print(fs1 | fs2)  # frozenset({1, 2, 3})
print(fs1 & fs2)  # frozenset({2})
print(fs1 - fs2)  # frozenset({1})
print(fs1 ^ fs2)  # frozenset({1, 3})

# 5.2 Нет индексации/срезов
try:
    print(fs1[0])
except TypeError as e:
    print(f"Не поддерживается: {e}")  # 'frozenset' object is not subscriptable

# 5.3 Нет мутации
try:
    fs1.add(4)
except AttributeError as e:
    print(f"Не поддерживается: {e}")

# 5.4 Хешируемость!
print(hash(fs1))  # Работает!
```

## **Операции сравнения**

```python
fs1 = frozenset([1, 2, 3])
fs2 = frozenset([1, 2, 3])
fs3 = frozenset([1, 2])

print(fs1 == fs2)  # True
print(fs1 != fs3)  # True
print(fs3 < fs1)  # True (строгое подмножество)
print(fs1 > fs3)  # True (надмножество)
print(frozenset([1, 2]) <= fs1)  # True

# Сравнение с set
print(fs1 == frozenset([1, 2, 3]))  # True
```

## **Преобразование в другие типы и форматирование**

```python
fs = frozenset([1, 2, 3])

# 7.1 В другие коллекции
print(list(fs))  # [1, 2, 3] (порядок не гарантирован)
print(tuple(fs))  # (1, 2, 3)
print(set(fs))  # {1, 2, 3}

# 7.2 В строку
print(str(fs))  # 'frozenset({1, 2, 3})'
print(repr(fs))  # 'frozenset({1, 2, 3})'

# 7.3 Распаковка
print(*fs)  # 1 2 3 (порядок не гарантирован)
print(len(fs))  # 3
print(sum(fs))  # 6

# 7.4 Frozenset comprehension
squares = frozenset(x ** 2 for x in range(3))  # frozenset({0, 1, 4})
```

## **Важные особенности**

```python
# 1. Неизменяемость (иммутабельность)
fs = frozenset([1, 2])
# fs.add(3)  # AttributeError!

# 2. Хешируемость (можно использовать как ключ)
d = {frozenset([1, 2]): "value"}
print(d[frozenset([1, 2])])  # 'value'

# 3. Ссылочная семантика (но неизменяемая)
fs1 = frozenset([1, 2])
fs2 = fs1  # Безопасно, т.к. неизменяемо
print(fs1 is fs2)  # True

# 4. Копирование (не нужно, но возможно)
import copy

fs_copy = copy.copy(fs1)  # Та же ссылка для immutable

# 5. Ложные значения
print(bool(frozenset()))  # False
print(bool(frozenset([0])))  # True

# 6. Вложенность в set
s = {frozenset([1, 2]), frozenset([3, 4])}
```

## **Важные замечания:**

1. **Иммутабельность**: Нет методов мутации (`add`, `remove`, `clear`).
2. **Хешируемость**: Можно использовать как ключи словарей и элементы `set`.
3. **Не индексируем**: Нет `fs[0]`, используйте `in` для проверки O(1).
4. **Уникальность**: Автоматически удаляет дубликаты, порядок не гарантирован.
5. **Память**: Хэш-таблица, компактнее `set` (нет места для роста).
6. **Альтернатива `set`**: Когда нужна неизменяемость и хешируемость.

## **Ключевые выводы:**

1. **`frozenset` — неизменяемая хешируемая коллекция уникальных** хешируемых объектов.
2. **Идеален для**: ключей словарей, элементов множеств, константных коллекций.
3. **O(1) операции**: `in`, `union`, `intersection` благодаря хэш-таблице.
4. **Пустое `frozenset()` — ложное значение**, непустые — истинные.
5. **Только неизменяемые методы**, никаких мутаций.
6. **`frozenset(iter)`** — основной способ создания.

[Содержание](/CONTENTS.md#содержание)

---

## bytes

## **Тип bytes в Python (байтовые строки)**

Тип `bytes` представляет неизменяемые упорядоченные последовательности байтов (целые числа 0-255). Это встроенный тип
данных,
поддерживающий индексацию, срезы и методы для работы с бинарными данными. `bytes` иммутабелен, хешируем и используется
для
работы с сырыми байтами, кодировками, файлами и сетевыми протоколами.

## **Создание bytes**

```python
# 1.1 Литералы bytes (b'...')
b1 = b'hello'  # b'hello'
b2 = b'\x68\x65\x6c\x6c\x6f'  # b'hello' (hex)
b3 = b''  # Пустые bytes
b4 = b'caf\xe9'  # b'caf\xe9' (UTF-8)

# 1.2 Через конструктор bytes()
b5 = bytes([104, 101, 108, 108, 111])  # b'hello'
b6 = bytes(range(3))  # b'\x00\x01\x02'
b7 = bytes(5)  # b'\x00\x00\x00\x00\x00' (нулевые байты)

# 1.3 Из строк с кодировкой
b8 = 'hello'.encode('utf-8')  # b'hello'
b9 = 'привет'.encode('utf-8')  # b'\xd0\xbf\xd1\x80\xd0\xb8\xd0\xb2\xd0\xb5\xd1\x82'

# 1.4 Из памяти
import array

b10 = bytes(array.array('B', [65, 66, 67]))  # b'ABC'
```

## **Атрибуты типа bytes**

Тип `bytes` не имеет публичных атрибутов для чтения/записи данных. Размер доступен через `len()`.

```python
b = b'hello'

# Нет атрибутов типа real/numerator
try:
    print(b.real)
except AttributeError as e:
    print(f"Ошибка: {e}")  # 'bytes' object has no attribute 'real'

print(len(b))  # 5 — количество байтов
print(type(len(b)))  # <class 'int'>
```

## **Методы типа bytes**

Тип `bytes` имеет методы для поиска, замены и кодирования (все неизменяемые).

### **Методы поиска и проверки**

```python
b = b'hello world'

print(b.find(b'world'))  # 6 (позиция)
print(b.count(b'l'))  # 3
print(b.startswith(b'hel'))  # True
print(b.endswith(b'rld'))  # True
print(b'lo' in b)  # True
```

### **Методы замены и разделения**

```python
b = b'hello||world'

print(b.replace(b'hello', b'hi'))  # b'hi||world'
print(b.split(b'||'))  # [b'hello', b'world']
print(b.partition(b'||'))  # (b'hello', b'||', b'world')

# Заглавные/строчные (ASCII)
print(b.upper())  # b'HELLO||WORLD'
print(b.lower())  # b'hello||world'
print(b.strip(b'|'))  # b'hello world'
```

### **Методы кодирования**

```python
b = b'hello'

print(b.decode('utf-8'))  # 'hello'
print(b.decode('latin1'))  # 'hello' (побайтово)
print(b.hex())  # '68656c6c6f'
print(b.fromhex('68 65 6c 6c 6f'))  # b'hello'
```

## **Поддерживаемые операции**

```python
b1 = b'hello'
b2 = b' world'

# 5.1 Конкатенация и повторение
print(b1 + b2)  # b'hello world'
print(b1 * 3)  # b'hellohellohello'

# 5.2 Индексация и срезы (только чтение)
print(b1[0])  # 104 (int)
print(b1[-1])  # 111 (int)
print(b1[1:4])  # b'ell'
print(b1[::-1])  # b'olleh'

# 5.3 Нет присваивания (иммутабельно)
try:
    b1[0] = 65
except TypeError as e:
    print(f"Не поддерживается: {e}")

# 5.4 Сравнения (лексикографические)
print(b'abc' < b'def')  # True
```

## **Операции сравнения**

```python
b1 = b'abc'
b2 = b'abc'
b3 = b'abd'

print(b1 == b2)  # True
print(b1 < b3)  # True (c < d)
print(b1 == b'abc')  # True

# Разные типы
print(b'1' == b'\x01')  # False ('1' = 49, '\x01' = 1)
print(b'\x00' < b'\x01')  # True
```

## **Преобразование в другие типы и форматирование**

```python
b = b'hello'

# 7.1 В другие типы
print(list(b))  # [104, 101, 108, 108, 111]
print(bytes(b))  # b'hello' (копия)
print(memoryview(b))  # <memory at ...>
print(b.decode())  # 'hello' (utf-8)

# 7.2 В строку
print(str(b))  # "b'hello'"
print(repr(b))  # "b'hello'"

# 7.3 Распаковка
print(*b)  # 104 101 108 108 111
print(sum(b))  # 523
print(len(b))  # 5

# 7.4 Bytes comprehension (нет, но list -> bytes)
nums = [65, 66, 67]
byte_arr = bytes([n for n in nums if n > 65])  # b'BC'
```

## **Важные особенности**

```python
# 1. Иммутабельность (нельзя изменять)
b = b'hello'
# b[0] = 65  # TypeError!

# 2. Элементы — целые числа 0-255
print(b[0])  # 104 (ord('h'))
print(chr(b[0]))  # 'h'

# 3. Хешируемость
d = {b'key': 'value'}
print(d[b'key'])  # 'value'

# 4. Ссылочная семантика (безопасно)
b1 = b'hello'
b2 = b1
print(b1 is b2)  # True (small string optimization)

# 5. Ложные значения
print(bool(b''))  # False
print(bool(b'\x00'))  # True

# 6. Эффективность памяти
import sys

print(sys.getsizeof(b'hello'))  # 49 (компактно)
```

## **Важные замечания:**

1. **Иммутабельность**: Нет методов мутации, только новые `bytes`.
2. **Диапазон**: Только значения 0-255, выход за пределы → `ValueError`.
3. **Кодировки**: `encode('utf-8')` для str→bytes, `decode()` для bytes→str.
4. **Сравнение**: Побайтовое (не семантическое), `b'10' != b'\x10'`.
5. **Память**: Очень компактно, подходит для больших бинарных данных.
6. **Литералы**: `b'...'` с escape-последовательностями `\xHH`, `\xHHH`.

## **Ключевые выводы:**

1. **`bytes` — неизменяемая упорядоченная коллекция байтов** (0-255).
2. **Поддерживает индексацию, срезы, конкатенацию** и методы поиска/замены.
3. **Хешируем**: Можно использовать как ключи словарей.
4. **Пустые `b''` — ложное значение**, остальные — истинные.
5. **Идеален для**: бинарных файлов, сетевых протоколов, кэширования.
6. **`b'str'`, `str.encode()`, `bytes([0-255])`** — основные способы создания.

[Содержание](/CONTENTS.md#содержание)

---

## bytearray

Тип `bytearray` представляет изменяемые упорядоченные последовательности байтов (целые числа 0-255). Это встроенный тип
данных,
поддерживающий динамическое изменение размера, индексацию, срезы с присваиванием и методы для манипуляции бинарными
данными.
`bytearray` мутабелен, не хешируем и используется для модификации байтовых последовательностей in-place.

## **Создание bytearray**

```python
# 1.1 Литералы bytearray (только через конструктор)
ba1 = bytearray(b'hello')  # bytearray(b'hello')
ba2 = bytearray([104, 101, 108, 108, 111])  # bytearray(b'hello')
ba3 = bytearray()  # Пустой bytearray

# 1.2 Через конструктор bytearray()
ba4 = bytearray(5)  # bytearray(b'\x00\x00\x00\x00\x00')
ba5 = bytearray(range(3))  # bytearray(b'\x00\x01\x02')
ba6 = 'hello'.encode('utf-8')  # b'hello' → bytearray(b'hello')

# 1.3 Из строк и итерируемых
ba7 = bytearray('привет', 'utf-8')  # bytearray(b'\xd0\xbf\xd1\x80...')
ba8 = bytearray([65, 66, 67] * 2)  # bytearray(b'ABCABC')

# 1.4 Из bytes
b = b'world'
ba9 = bytearray(b)  # bytearray(b'world')
```

## **Атрибуты типа bytearray**

Тип `bytearray` не имеет публичных атрибутов для чтения/записи данных. Размер доступен через `len()`.

```python
ba = bytearray(b'hello')

# Нет атрибутов типа real/numerator
try:
    print(ba.real)
except AttributeError as e:
    print(f"Ошибка: {e}")  # 'bytearray' object has no attribute 'real'

print(len(ba))  # 5 — количество байтов
print(type(len(ba)))  # <class 'int'>
```

## **Методы типа bytearray**

Тип `bytearray` имеет методы для мутации и неизменяемые операции.

### **Методы изменения bytearray (мутация)**

```python
ba = bytearray(b'hello')

ba.append(33)  # bytearray(b'hello!')
ba.insert(0, 72)  # bytearray(b'Hello!')
ba.pop()  # Удаляет '!', возвращает 33 → bytearray(b'Hello')
ba.remove(72)  # bytearray(b'ello')
ba.clear()  # bytearray(b'') — очищает
ba.extend(b' world')  # bytearray(b'ello world')
```

### **Методы поиска и проверки**

```python
ba = bytearray(b'hello world')

print(ba.find(b'world'))  # 6
print(ba.count(b'l'))  # 3
print(ba.startswith(b'hel'))  # True
print(b'lo' in ba)  # True
```

### **Методы замены (возвращают новый bytearray)**

```python
ba = bytearray(b'hello||world')
print(ba.replace(b'hello', b'hi'))  # bytearray(b'hi||world')

# Мутация через присваивание
ba[0:5] = b'Hi!!!'
print(ba)  # bytearray(b'Hi!!!||world')
```

## **Поддерживаемые операции**

```python
ba1 = bytearray(b'hello')
ba2 = bytearray(b' world')

# 5.1 Конкатенация и повторение (новый bytearray)
print(ba1 + ba2)  # bytearray(b'hello world')
print(ba1 * 2)  # bytearray(b'hellohello')

# 5.2 Индексация и срезы (чтение/запись)
print(ba1[0])  # 104
ba1[0] = 72  # bytearray(b'Hello')
print(ba1[1:4])  # bytearray(b'ell')

# 5.3 Срезы с присваиванием (мутация)
ba1[0:2] = b'Hi'  # bytearray(b'Hi llo')

# 5.4 Сравнения (лексикографические)
print(bytearray(b'abc') < bytearray(b'def'))  # True
```

## **Операции сравнения**

```python
ba1 = bytearray(b'abc')
ba2 = bytearray(b'abc')
ba3 = bytearray(b'abd')

print(ba1 == ba2)  # True
print(ba1 < ba3)  # True
print(ba1 == b'abc')  # True (с bytes)

# Сравнение с bytes
print(bytearray(b'\x01') < b'\x02')  # True
```

## **Преобразование в другие типы и форматирование**

```python
ba = bytearray(b'hello')

# 7.1 В другие типы
print(bytes(ba))  # b'hello'
print(list(ba))  # [104, 101, 108, 108, 111]
print(str(ba, 'utf-8'))  # 'hello'
print(ba.decode('utf-8'))  # 'hello'

# 7.2 В строку
print(str(ba))  # "bytearray(b'hello')"
print(repr(ba))  # "bytearray(b'hello')"

# 7.3 Распаковка и методы
print(*ba)  # 104 101 108 108 111
print(sum(ba))  # 523
print(ba.hex())  # '68656c6c6f'

# 7.4 Bytearray из генератора
ba_gen = bytearray(n for n in range(65, 68))  # bytearray(b'ABC')
```

## **Важные особенности**

```python
# 1. Мутабельность (можно изменять)
ba = bytearray(b'hello')
ba[0] = 72  # bytearray(b'Hello')
ba.append(33)  # bytearray(b'Hello!')

# 2. Ссылочная семантика
ba1 = bytearray(b'abc')
ba2 = ba1
ba2[0] = 88  # ba1 тоже изменился! bytearray(b'Xbc')

# 3. Копирование
import copy

ba_copy = ba1[:]  # Поверхностная копия
ba_deep = copy.deepcopy(ba1)  # Глубокая копия

# 4. Только значения 0-255
try:
    ba.append(256)  # ValueError!
except ValueError as e:
    print(f"Ошибка: {e}")

# 5. Ложные значения
print(bool(bytearray()))  # False
print(bool(bytearray(b'\x00')))  # True

# 6. Не хешируем
try:
    hash(ba1)
except TypeError as e:
    print(f"Не хешируется: {e}")
```

## **Важные замечания:**

1. **Мутабельность**: Поддерживает `append`, `pop`, индексацию с присваиванием.
2. **Диапазон**: Только 0-255, `256` → `ValueError`.
3. **Не хешируем**: Нельзя использовать как ключи словарей.
4. **Ссылки**: `ba1 = ba2` создаёт ссылку, используйте `[:]`, `copy()`.
5. **Память**: Компактно, но с over-allocation для роста.
6. **`bytes(ba)`** для финализации в неизменяемые байты.

## **Ключевые выводы:**

1. **`bytearray` — мутабельная упорядоченная коллекция байтов** (0-255).
2. **Поддерживает индексацию, срезы с присваиванием** и мутацию in-place.
3. **`[:]` и `copy()`** для безопасного копирования ссылок.
4. **Пустой `bytearray()` — ложное значение**, остальные — истинные.
5. **Идеален для**: модификации бинарных данных, буферов, протоколов.
6. **`bytearray(b)`, `bytearray([0-255])`** — основные способы создания.

[Содержание](/CONTENTS.md#содержание)

---

## namedtuple

Тип `namedtuple` из модуля `collections` представляет неизменяемые упорядоченные коллекции с именованными полями. Это
подкласс `tuple`,
поддерживающий индексацию по позиции **и по имени поля**, с автоматической генерацией `_asdict()`, `_fields` и других
удобств.
`namedtuple` иммутабелен, хешируем и используется для создания легковесных структур данных с читаемым доступом к полям.

## **Создание namedtuple**

```python
from collections import namedtuple

# 1.1 Создание класса namedtuple
Point = namedtuple('Point', ['x', 'y'])
p1 = Point(1, 2)  # Point(x=1, y=2)

# 1.2 Альтернативные способы определения полей
Point2 = namedtuple('Point2', 'x y')
p2 = Point2(3, 4)

# 1.3 Ключевые слова
Circle = namedtuple('Circle', 'center radius', defaults=[Point(0, 0), 1.0])
c1 = Circle()  # Circle(center=Point(x=0, y=0), radius=1.0)
c2 = Circle(radius=5)  # Circle(center=Point(x=0, y=0), radius=5)

# 1.4 Из итерируемых объектов
data = [10, 20]
p3 = Point._make(data)  # Point(x=10, y=20)
```

## **Атрибуты типа namedtuple**

Тип `namedtuple` имеет классовые атрибуты для метаданных и экземплярные для полей.

```python
Point = namedtuple('Point', 'x y')
p = Point(1, 2)

# Классовые атрибуты
print(Point._fields)  # ('x', 'y')
print(Point.__doc__)  # Автогенерируемая документация

# Экземплярные атрибуты (поля)
print(p.x)  # 1
print(p.y)  # 2

print(len(p))  # 2 — количество полей
print(type(len(p)))  # <class 'int'>
```

## **Методы типа namedtuple**

Тип `namedtuple` наследует методы `tuple` + добавляет удобные методы.

### **Наследованные методы tuple (неизменяемые)**

```python
p = Point(1, 2)

print(p.index(1))  # 0 (позиция первого вхождения)
print(p.count(1))  # 1
print(1 in p)  # True
```

### **Специальные методы namedtuple**

```python
p = Point(1, 2)

print(p._asdict())  # {'x': 1, 'y': 2} → OrderedDict
print(p._replace(x=10))  # Point(x=10, y=2) — новый экземпляр
print(p._fields)  # ('x', 'y')

# _make() — классовый метод
p2 = Point._make([3, 4])  # Point(x=3, y=4)
```

## **Поддерживаемые операции**

```python
p1 = Point(1, 2)
p2 = Point(3, 4)

# 5.1 Индексация по позиции И по имени
print(p1[0])  # 1
print(p1.x)  # 1

# 5.2 Распаковка
x, y = p1  # 1, 2

# 5.3 Сравнения (лексикографические по полям)
print(p1 < p2)  # True ((1,2) < (3,4))

# 5.4 Нет мутации
try:
    p1.x = 99
except AttributeError as e:
    print(f"Не поддерживается: {e}")

# 5.5 Итерируемость
print(list(p1))  # [1, 2]
```

## **Операции сравнения**

```python
p1 = Point(1, 2)
p2 = Point(1, 2)
p3 = Point(1, 3)

print(p1 == p2)  # True
print(p1 < p3)  # True (2 < 3 на 2-й позиции)
print(p1 == (1, 2))  # True (сравнение с tuple)

# Разные типы namedtuple сравниваются корректно
print(Point(1, 2) == Point2(1, 2))  # True
```

## **Преобразование в другие типы и форматирование**

```python
p = Point(1, 2)

# 7.1 В другие коллекции
print(tuple(p))  # (1, 2)
print(list(p))  # [1, 2]
print(p._asdict())  # {'x': 1, 'y': 2}

# 7.2 В строку
print(str(p))  # 'Point(x=1, y=2)'
print(repr(p))  # 'Point(x=1, y=2)'

# 7.3 Распаковка
print(*p)  # 1 2
print(len(p))  # 2

# 7.4 Namedtuple factory
Person = namedtuple('Person', 'name age', module='test')
per = Person('Alice', 30)
```

## **Важные особенности**

```python
# 1. Иммутабельность (как tuple)
p = Point(1, 2)
# p.x = 99  # AttributeError!

# 2. Хешируемость (как tuple)
s = {p: 'value'}
print(s[Point(1, 2)])  # 'value'

# 3. Удобный доступ по имени
p = Point(x=10, y=20)
print(p.x, p.y)  # 10 20

# 4. _replace() для "изменения"
p2 = p._replace(x=99)  # Point(x=99, y=20) — новый объект

# 5. Ложные значения (как tuple)
print(bool(Point(0, 0)))  # True
print(bool(Point(0, 0, 0)))  # True (только пустой tuple False)

# 6. Автогенерируемые методы
print(Point._fields)  # ('x', 'y')
print(hasattr(Point, '_make'))  # True
```

## **Важные замечания:**

1. **Иммутабельность**: Нельзя изменять поля, используйте `_replace()`.
2. **Хешируемость**: Можно использовать как ключи словарей и элементы set.
3. **Производительность**: Быстрее обычных классов (нет `__dict__`).
4. **Память**: Компактнее классов (фиксированный размер).
5. **Имена полей**: Не могут быть Python-ключевыми словами, начинаются с `_`.
6. **`_asdict()`** возвращает `OrderedDict` с полями.

## **Ключевые выводы:**

1. **`namedtuple` — неизменяемый tuple с именованными полями** и хешируемостью.
2. **Доступ по индексу `p[0]` И по имени `p.x`** — лучшее из двух миров.
3. **`_replace()`, `_asdict()`, `_make()`** — мощные фабричные методы.
4. **Быстрее и легче** обычных классов для простых структур данных.
5. **Идеален для**: DTO, конфигураций, результатов запросов, констант.
6. **`namedtuple('Name', 'fields')`** — основной способ создания.

[Содержание](/CONTENTS.md#содержание)

---

## deque

## **Тип deque в Python (двусторонняя очередь)**

Тип `deque` из модуля `collections` представляет изменяемые упорядоченные последовательности произвольных объектов с *
*O(1) доступом к концам**.
Это встроенный тип данных, оптимизированный для быстрых операций `append/pop` с обоих концов (слева/справа). `deque`
мутабелен,
не хешируем и идеален для очередей, стеков, скользящих окон и буферов.

## **Создание deque**

```python
from collections import deque

# 1.1 Через конструктор deque()
d1 = deque([1, 2, 3])
d2 = deque('abc')
d3 = deque()  # Пустой deque

# 1.2 С параметрами maxlen (ограничение размера)
d4 = deque([1, 2, 3, 4, 5], maxlen=3)  # deque([3, 4, 5], maxlen=3)
d5 = deque(maxlen=10)  # Пустой с лимитом

# 1.3 Из итерируемых объектов
d6 = deque(range(3))  # deque([0, 1, 2])
d7 = deque([*range(5)])  # deque([0, 1, 2, 3, 4])

# 1.4 Инициализация с повторениями
d8 = deque([0] * 3, maxlen=5)  # deque([0, 0, 0], maxlen=5)
```

## **Атрибуты типа deque**

Тип `deque` имеет публичный атрибут `maxlen` и размер через `len()`.

```python
d = deque([1, 2, 3], maxlen=5)

# Публичные атрибуты
print(d.maxlen)  # 5 или None
print(len(d))  # 3 — количество элементов
print(type(len(d)))  # <class 'int'>

# Нет атрибутов типа real/numerator
try:
    print(d.real)
except AttributeError as e:
    print(f"Ошибка: {e}")  # 'deque' object has no attribute 'real'
```

## **Методы типа deque**

Тип `deque` имеет методы для работы с концами + стандартные методы коллекций.

### **Методы концов (O(1) операции)**

```python
d = deque([1, 2, 3])

d.append(4)  # deque([1, 2, 3, 4]) — справа
d.appendleft(0)  # deque([0, 1, 2, 3, 4]) — слева
d.pop()  # 4 → deque([0, 1, 2, 3])
d.popleft()  # 0 → deque([1, 2, 3])
d.extend([5, 6])  # deque([1, 2, 3, 5, 6])
d.extendleft([0, -1])  # deque([-1, 0, 1, 2, 3, 5, 6]) (обратный порядок!)
```

### **Методы поворота и ротации**

```python
d = deque('abcdefg')

d.rotate(1)  # deque(['g', 'a', 'b', 'c', 'd', 'e', 'f']) — вправо
d.rotate(-2)  # deque(['c', 'd', 'e', 'f', 'g', 'a', 'b']) — влево

print(d)
```

### **Методы поиска (O(n))**

```python
d = deque([1, 2, 3, 2])

print(d.index(2))  # 1 (первое вхождение)
print(d.count(2))  # 2
print(99 in d)  # False
print(d.index(2, 2))  # 3 (с позиции)
```

## **Поддерживаемые операции**

```python
d1 = deque([1, 2])
d2 = deque([3, 4])

# 5.1 Конкатенация и повторение
print(d1 + d2)  # deque([1, 2, 3, 4])
print(d1 * 2)  # deque([1, 2, 1, 2])

# 5.2 Индексация и срезы (медленно O(n)!)
print(d1[0])  # 1
print(d1[-1])  # 2
print(d1[1:])  # deque([2])

# 5.3 Нет присваивания по индексу (только методы)
try:
    d1[0] = 99
except TypeError as e:
    print(f"Не поддерживается: {e}")

# 5.4 Сравнения
print(deque([1, 2]) < deque([1, 3]))  # True
```

## **Операции сравнения**

```python
d1 = deque([1, 2, 3])
d2 = deque([1, 2, 3])
d3 = deque([1, 2, 4])

print(d1 == d2)  # True
print(d1 < d3)  # True (лексикографическое)
print(d1 == [1, 2, 3])  # True (сравнение с list)

# maxlen не влияет на сравнение
print(deque([1, 2], maxlen=3) == deque([1, 2]))  # True
```

## **Преобразование в другие типы и форматирование**

```python
d = deque([1, 2, 3])

# 7.1 В другие коллекции
print(list(d))  # [1, 2, 3]
print(tuple(d))  # (1, 2, 3)
print(' '.join(map(str, d)))  # '1 2 3'

# 7.2 В строку
print(str(d))  # 'deque([1, 2, 3], maxlen=None)'
print(repr(d))  # 'deque([1, 2, 3], maxlen=None)'

# 7.3 Распаковка
print(*d)  # 1 2 3
print(len(d))  # 3

# 7.4 Deque comprehension
d_squares = deque(x ** 2 for x in range(3))  # deque([0, 1, 4])
```

## **Важные особенности**

```python
# 1. Мутабельность через методы концов
d = deque([1, 2])
d.append(3)  # deque([1, 2, 3])
d.appendleft(0)  # deque([0, 1, 2, 3])

# 2. Автообрезка при maxlen
d = deque([1, 2, 3, 4], maxlen=3)
d.append(5)  # deque([2, 3, 4, 5]) → deque([3, 4, 5]) (старые удалены)

# 3. Ссылочная семантика
d1 = deque([1, 2])
d2 = d1
d2.append(3)  # d1 тоже изменился! deque([1, 2, 3])

# 4. Копирование
import copy

d_copy = d1.copy()  # Поверхностная копия
d_deep = copy.deepcopy(d1)

# 5. Ложные значения
print(bool(deque()))  # False
print(bool(deque([0])))  # True

# 6. Медленная индексация (O(n))
```

## **Важные замечания:**

1. **O(1) только для концов**: `append/pop(left/right)` — быстро, индексация — O(n).
2. **maxlen**: Автоматически удаляет старые элементы при переполнении.
3. **`extendleft()` реверсирует**: `[1,2].extendleft([3,4])` → `[4,3,1,2]`.
4. **Ссылки**: `d1 = d2` — ссылка, используйте `copy()`.
5. **Не хешируем**: Нельзя использовать как ключи словарей.
6. **Память**: Двойная связанный список блоков (эффективнее list для концов).

## **Ключевые выводы:**

1. **`deque` — мутабельная очередь с O(1) доступом к обоим концам**.
2. **`append/pop(left/right)`** — основные быстрые операции.
3. **`maxlen`** для автоматического ограничения размера (скользящие окна).
4. **Избегайте индексации** `d[999]` — медленно O(n)!
5. **Идеален для**: очередей, стеков, буферов, алгоритмов BFS/DFS.
6. **`deque(iter, maxlen=N)`** — основной способ создания.

[Содержание](/CONTENTS.md#содержание)

---

## Counter

## **Тип Counter в Python (счетчик элементов)**

Тип `Counter` из модуля `collections` представляет словарь-подобную коллекцию для **считывания частоты элементов**. Это
специализированный
`dict`, где ключи — элементы, значения — их количество. Поддерживает арифметику множеств, отрицательные счетчики и
автоматическое
создание отсутствующих ключей со счетчиком 0. Мутабелен, не хешируем, идеален для подсчета частот и статистического
анализа.

## **Создание Counter**

```python
from collections import Counter

# 1.1 Из итерируемых объектов
c1 = Counter('abracadabra')  # Counter({'a': 5, 'r': 2, 'b': 2, 'c': 1, 'd': 1})
c2 = Counter([1, 2, 2, 3])  # Counter({2: 2, 1: 1, 3: 1})
c3 = Counter()  # Пустой Counter

# 1.2 Из словаря или пар ключ-значение
c4 = Counter({'a': 3, 'b': 2})
c5 = Counter(a=3, b=2)  # Counter({'a': 3, 'b': 2})

# 1.3 Из строк и диапазонов
c6 = Counter('hello')  # Counter({'l': 2, 'h': 1, 'e': 1, 'o': 1})
```

## **Атрибуты типа Counter**

Тип `Counter` наследует атрибуты `dict` + добавляет специальные методы.

```python
c = Counter('abracadabra')

# Нет атрибутов типа real/numerator
try:
    print(c.real)
except AttributeError as e:
    print(f"Ошибка: {e}")  # 'Counter' object has no attribute 'real'

print(len(c))  # 5 — количество уникальных элементов
print(type(len(c)))  # <class 'int'>
print(c.total())  # 11 — сумма всех счетчиков
```

## **Методы типа Counter**

Тип `Counter` расширяет `dict` методами для подсчета и арифметики.

### **Методы подсчета и статистики**

```python
c = Counter('abracadabra')

print(c.most_common(3))  # [('a', 5), ('r', 2), ('b', 2)]
print(c.most_common(1))  # [('a', 5)]
print(c['z'])  # 0 (автоматически для отсутствующих)

c['new'] += 1  # Counter({'a': 5, ..., 'new': 1})
```

### **Арифметические операции (мутация)**

```python
c1 = Counter('abracadabra')
c2 = Counter('candy')

c1.update(c2)  # c1 += c2 → Counter({'a': 5, 'r': 2, 'b': 2, 'c': 2, 'd': 2, 'n': 1, 'y': 1})
c1.subtract(c2)  # c1 -= c2 → отрицательные допустимы

print(c1)  # Counter({'a': 5, 'r': 2, 'b': 2, 'c': 1, 'd': 1, 'n': 0, 'y': -1})
```

## **Поддерживаемые операции**

```python
c1 = Counter('abracadabra')
c2 = Counter('candy')

# 5.1 Арифметика множеств (как dict)
print(c1 + c2)  # Counter({'a': 5, 'r': 2, 'b': 2, 'c': 2, 'd': 2, 'n': 1, 'y': 1})
print(c1 - c2)  # Counter({'a': 5, 'r': 2, 'b': 2}) (нулевые/отрицательные удалены)
print(c1 & c2)  # Counter({'c': 1, 'a': 1, 'd': 1, 'b': 1}) (минимум)
print(c1 | c2)  # Counter({'a': 5, 'r': 2, 'b': 2, 'c': 2, 'd': 2, 'n': 1, 'y': 1}) (максимум)

# 5.2 Обычные dict операции
print(c1['a'])  # 5
print('z' in c1)  # True (0 тоже присутствует)
print(list(c1.keys()))  # ['a', 'r', 'b', 'c', 'd']
```

## **Операции сравнения**

```python
c1 = Counter(a=3, b=1)
c2 = Counter(a=3, b=1)
c3 = Counter(a=2, b=1)

print(c1 == c2)  # True
print(c1 == c3)  # False
print(c1 == {'a': 3, 'b': 1})  # True (с dict)

# Сравнение игнорирует нулевые/отрицательные
print(Counter(a=1) == Counter(a=1, b=0))  # True
```

## **Преобразование в другие типы и форматирование**

```python
c = Counter('abracadabra')

# 7.1 В другие коллекции
print(dict(c))  # {'a': 5, 'r': 2, 'b': 2, 'c': 1, 'd': 1}
print(list(c.elements()))  # ['a', 'a', 'a', 'a', 'a', 'r', 'r', ...]
print(c.most_common())  # [('a', 5), ('r', 2), ('b', 2), ...]

# 7.2 В строку
print(str(c))  # "Counter({'a': 5, 'r': 2, 'b': 2, 'c': 1, 'd': 1})"
print(repr(c))  # "Counter({'a': 5, 'r': 2, 'b': 2, 'c': 1, 'd': 1})"

# 7.3 Итерация и статистика
print(sum(c.values()))  # 11
print(len(c))  # 5
print(c.total())  # 11
```

## **Важные особенности**

```python
# 1. Автоматическое создание ключей
c = Counter()
c['missing']  # 0
c['new'] += 1  # Counter({'new': 1})

# 2. Отрицательные счетчики допустимы
c['negative'] -= 3  # Counter({'new': 1, 'negative': -3})

# 3. Ссылочная семантика
c1 = Counter('abc')
c2 = c1
c2['d'] = 1  # c1 тоже изменился!

# 4. Копирование
import copy

c_copy = c1.copy()  # Поверхностная копия

# 5. Ложные значения
print(bool(Counter()))  # False
print(bool(Counter(a=0)))  # False (все нулевые)
print(bool(Counter(a=1)))  # True

# 6. elements() генерирует все элементы
print(list(Counter('aab').elements()))  # ['a', 'a', 'b']
```

## **Важные замечания:**

1. **Автозаполнение**: `c['missing']` → 0, не `KeyError`.
2. **Отрицательные счетчики**: Допустимы, но `most_common()` их игнорирует.
3. **Арифметика**: `+`, `-`, `&`, `|` работают как операции над множествами.
4. **Ссылки**: `c1 = c2` — ссылка, используйте `copy()`.
5. **Не хешируем**: Нельзя использовать как ключи словарей.
6. **`most_common(n)`** возвращает топ-N элементов по убыванию.

## **Ключевые выводы:**

1. **`Counter` — словарь для подсчета частоты элементов** с арифметикой.
2. **`c['key']` → 0** для отсутствующих, `most_common()` для топа.
3. **`update()`, `subtract()`** — мутация, `+`, `-`, `&`, `|` — неизменяемые.
4. **Отрицательные счетчики** допустимы, нулевые игнорируются в итерации.
5. **Идеален для**: частотного анализа, гистограмм, статистики текстов.
6. **`Counter(iter)`** — основной способ создания.

[Содержание](/CONTENTS.md#содержание)

---

## defaultdict

## **Тип defaultdict в Python (словарь с значениями по умолчанию)**

Тип `defaultdict` из модуля `collections` представляет словарь, который **автоматически создает значения по умолчанию**
для отсутствующих ключей.
Это подкласс `dict`, где при обращении к несуществующему ключу вызывается фабричная функция `default_factory`.
Мутабелен,
не хешируем, идеален для группировки данных, подсчета, вложенных структур без проверок `if key in dict`.

## **Создание defaultdict**

```python
from collections import defaultdict

# 1.1 Список по умолчанию
d1 = defaultdict(list)
d1['a'].append(1)  # defaultdict(<class 'list'>, {'a': [1]})
d1['a'].append(2)  # defaultdict(<class 'list'>, {'a': [1, 2]})

# 1.2 Словарь по умолчанию
d2 = defaultdict(dict)
d2['user']['name'] = 'Alice'  # defaultdict(<class 'dict'>, {'user': {'name': 'Alice'}})

# 1.3 Из итерируемых объектов
d3 = defaultdict(int, [('a', 1), ('a', 2)])  # defaultdict(<class 'int'>, {'a': 2})
d4 = defaultdict(lambda: 0)  # defaultdict(<function <lambda> at ...>)
```

## **Атрибуты типа defaultdict**

Тип `defaultdict` имеет дополнительный атрибут `default_factory` + атрибуты `dict`.

```python
d = defaultdict(list)

# Специальный атрибут defaultdict
print(d.default_factory)  # <class 'list'>
print(len(d))  # 0 — количество пар ключ-значение
print(type(len(d)))  # <class 'int'>

# Нет атрибутов типа real/numerator
try:
    print(d.real)
except AttributeError as e:
    print(f"Ошибка: {e}")  # 'defaultdict' object has no attribute 'real'
```

## **Методы типа defaultdict**

Тип `defaultdict` наследует все методы `dict` + добавляет автоматическое создание значений.

### **Автоматическое создание значений**

```python
d = defaultdict(list)

# Нет KeyError — создает автоматически
d['missing'].append(42)  # defaultdict(<class 'list'>, {'missing': [42]})
print(d['new'])  # [] (пустой список)

d = defaultdict(int)
d['counter'] += 1  # defaultdict(<class 'int'>, {'counter': 1})
print(d['zero'])  # 0
```

### **Стандартные методы dict**

```python
d = defaultdict(list, {'a': [1, 2]})

d.update({'b': [3]})  # defaultdict(<class 'list'>, {'a': [1, 2], 'b': [3]})
print(d.get('c', []))  # [] (get() не вызывает default_factory!)
print(list(d.keys()))  # ['a', 'b']
```

## **Поддерживаемые операции**

```python
d1 = defaultdict(list)
d1['a'].append(1)

d2 = defaultdict(int, {'a': 10})

# 5.1 Обычные dict операции + автосоздание
print(d1['a'])  # [1]
print('x' in d1)  # True (создает [])

# 5.2 Преобразование в dict (теряет default_factory)
print(dict(d1))  # {'a': [1], 'x': []}

# 5.3 Итерация как dict
print(list(d1.items()))  # [('a', [1]), ('x', [])]

# 5.4 Наследование dict методов
d1.clear()  # Очищает
```

## **Операции сравнения**

```python
d1 = defaultdict(list, {'a': [1]})
d2 = defaultdict(list, {'a': [1]})
d3 = {'a': [1]}

print(d1 == d2)  # True
print(d1 == d3)  # True (игнорирует default_factory)
print(d1 != defaultdict(int))  # True
```

## **Преобразование в другие типы и форматирование**

```python
d = defaultdict(list, {'a': [1, 2]})

# 7.1 В другие коллекции
print(dict(d))  # {'a': [1, 2]}
print(list(d.keys()))  # ['a']
print(list(d.values()))  # [[1, 2]]

# 7.2 В строку
print(str(d))  # "defaultdict(<class 'list'>, {'a': [1, 2]})"
print(repr(d))  # "defaultdict(<class 'list'>, {'a': [1, 2]})"

# 7.3 Стандартные операции
print(len(d))  # 1
print(sum(len(v) for v in d.values()))  # 2

# 7.4 Популярные фабрики
print(defaultdict(lambda: 0))  # defaultdict(<function <lambda> at ...>)
```

## **Важные особенности**

```python
# 1. Автосоздание только при __getitem__
d = defaultdict(list)
d['auto']  # [] (создает!)
d.get('noauto')  # None (get() не вызывает!)

# 2. Разные фабрики
counter = defaultdict(int)
groups = defaultdict(list)
nested = defaultdict(lambda: defaultdict(list))

# 3. Ссылочная семантика
d1 = defaultdict(list)
d2 = d1
d2['key'].append(42)  # d1 тоже изменился!

# 4. Копирование
import copy

d_copy = d1.copy()  # Копирует с default_factory
d_deep = copy.deepcopy(d1)

# 5. Ложные значения
print(bool(defaultdict(list)))  # True (не пуст!)
print(bool(dict(defaultdict(list))))  # False

# 6. Вложенные defaultdict
dd = defaultdict(lambda: defaultdict(int))
dd['user']['posts'] += 1
```

## **Важные замечания:**

1. **`default_factory` вызывается только в `__getitem__`**: `d['key']`, но не `d.get('key')`.
2. **`dict(dd)` теряет `default_factory`** — становится обычным dict.
3. **Ссылки**: `d1 = d2` — ссылка, используйте `copy()`.
4. **Не хешируем**: Нельзя использовать как ключи словарей.
5. **Производительность**: Такая же как dict + overhead фабрики.
6. **Популярные фабрики**: `list`, `set`, `int` (0), `lambda: 0`.

## **Ключевые выводы:**

1. **`defaultdict` — dict без KeyError** с автоматическим созданием значений.
2. **`d['missing']` → `default_factory()`** вместо исключения.
3. **`get()` НЕ вызывает фабрику**, используйте `d['key']`.
4. **`dict(dd)`** для обычного словаря (теряет магию).
5. **Идеален для**: группировки, подсчета, вложенных структур.
6. **`defaultdict(factory)`** — основной способ создания.

[Содержание](/CONTENTS.md#содержание)

---

## OrderedDict

## **Тип OrderedDict в Python (упорядоченный словарь)**

Тип `OrderedDict` из модуля `collections` представляет словарь, **сохраняющий порядок вставки ключей**. Это подкласс
`dict` с
дополнительными методами для управления порядком: `move_to_end()`, `popitem(last=False)`. Мутабелен, не хешируем,
идеален для конфигураций, кэшей LRU, JSON-сериализации с сохранением порядка.

## **Создание OrderedDict**

```python
from collections import OrderedDict

# 1.1 Из итерируемых объектов
od1 = OrderedDict([('a', 1), ('b', 2), ('c', 3)])
od2 = OrderedDict(a=1, b=2, c=3)  # Порядок вставки
od3 = OrderedDict()  # Пустой OrderedDict

# 1.2 Из обычного dict (Python 3.7+ сохраняет порядок)
d = {'x': 10, 'y': 20}
od4 = OrderedDict(d)  # Сохраняет порядок d

# 1.3 С повторяющимися ключами (последний побеждает)
od5 = OrderedDict([('a', 1), ('a', 99)])  # OrderedDict([('a', 99)])
```

## **Атрибуты типа OrderedDict**

Тип `OrderedDict` не имеет публичных атрибутов для чтения/записи данных. Размер доступен через `len()`.

```python
od = OrderedDict(a=1, b=2)

# Нет атрибутов типа real/numerator
try:
    print(od.real)
except AttributeError as e:
    print(f"Ошибка: {e}")  # 'OrderedDict' object has no attribute 'real'

print(len(od))  # 2 — количество пар ключ-значение
print(type(len(od)))  # <class 'int'>
```

## **Методы типа OrderedDict**

Тип `OrderedDict` расширяет `dict` методами управления порядком.

### **Методы управления порядком**

```python
od = OrderedDict(a=1, b=2, c=3, d=4)

od.move_to_end('a')  # a → конец: [('b', 2), ('c', 3), ('d', 4), ('a', 1)]
od.move_to_end('b', last=False)  # b → начало: [('b', 2), ('c', 3), ('d', 4), ('a', 1)]

od.popitem(last=False)  # Удаляет первый: ('b', 2) → [('c', 3), ('d', 4), ('a', 1)]
od.popitem()  # Удаляет последний: ('a', 1)
```

### **Стандартные методы dict**

```python
od = OrderedDict(a=1, b=2)
od.update({'c': 3})  # Сохраняет порядок
od['d'] = 4  # Добавляет в конец
print(list(od.keys()))  # ['a', 'b', 'c', 'd']
```

## **Поддерживаемые операции**

```python
od1 = OrderedDict(a=1, b=2)
od2 = OrderedDict(c=3, d=4)

# 5.1 Обычные dict операции (с сохранением порядка)
print(od1['a'])  # 1
print('x' in od1)  # False

# 5.2 Итерация по порядку вставки
print(list(od1))  # [('a', 1), ('b', 2)]

# 5.3 Обновление сохраняет порядок
od1.update(od2)  # [('a', 1), ('b', 2), ('c', 3), ('d', 4)]

# 5.4 Нет неподдерживаемых операций (как dict)
```

## **Операции сравнения**

```python
od1 = OrderedDict(a=1, b=2)
od2 = OrderedDict(a=1, b=2)
od3 = OrderedDict(b=2, a=1)

print(od1 == od2)  # True (содержимое одинаково)
print(od1 == od3)  # True (Python 3.7+: dict сравнивает только содержимое)

# Сравнение с dict
print(od1 == {'a': 1, 'b': 2})  # True
```

## **Преобразование в другие типы и форматирование**

```python
od = OrderedDict(a=1, b=2, c=3)

# 7.1 В другие коллекции
print(dict(od))  # {'a': 1, 'b': 2, 'c': 3}
print(list(od.keys()))  # ['a', 'b', 'c']
print(list(od.values()))  # [1, 2, 3]

# 7.2 В строку
print(str(od))  # "OrderedDict([('a', 1), ('b', 2), ('c', 3)])"
print(repr(od))  # "OrderedDict([('a', 1), ('b', 2), ('c', 3)])"

# 7.3 Итерация по порядку
print(*od)  # ('a', 1) ('b', 2) ('c', 3)
print(len(od))  # 3
```

## **Важные особенности**

```python
# 1. Сохранение порядка вставки
od = OrderedDict()
od['first'] = 1
od['second'] = 2
od['third'] = 3
print(list(od))  # [('first', 1), ('second', 2), ('third', 3)]

# 2. Управление порядком
od.move_to_end('first')  # [('second', 2), ('third', 3), ('first', 1)]
od.move_to_end('second', last=False)  # [('second', 2), ('third', 3), ('first', 1)]

# 3. LRU Cache паттерн
od.popitem(last=False)  # Удаляет наименее недавно использованный

# 4. Ссылочная семантика
od1 = OrderedDict(a=1)
od2 = od1
od2['b'] = 2  # od1 тоже изменился!

# 5. Ложные значения
print(bool(OrderedDict()))  # False
print(bool(OrderedDict(a=0)))  # True

# 6. JSON сериализация с порядком
import json

print(json.dumps(od))  # {"second": 2, "third": 3, "first": 1}
```

## **Важные замечания:**

1. **`move_to_end(key, last=True)`** — перемещает ключ в конец/начало.
2. **`popitem(last=True)`** — удаляет последний/первый элемент.
3. **Порядок сохраняется** при `update()`, новые ключи добавляются в конец.
4. **Ссылки**: `od1 = od2` — ссылка, используйте `copy()`.
5. **Не хешируем**: Нельзя использовать как ключи словарей.
6. **Python 3.7+**: Обычный `dict` сохраняет порядок, но без специальных методов.

## **Ключевые выводы:**

1. **`OrderedDict` — dict с гарантированным порядком вставки**.
2. **`move_to_end()`, `popitem(last=False)`** — управление порядком.
3. **Идеален для**: LRU кэшей, конфигураций, JSON с порядком.
4. **Сравнение игнорирует порядок** (как обычный dict).
5. **Python 3.7+**: Обычный `dict` часто достаточно.
6. **`OrderedDict(iter)`** — основной способ создания.

[Содержание](/CONTENTS.md#содержание)

---

## NamedTuple

## **Тип NamedTuple в Python (типизированный именованный кортеж)**

Тип `NamedTuple` из модуля `typing` представляет **типизированный класс** для неизменяемых структур данных с
именованными полями.
Это современная альтернатива `collections.namedtuple` с поддержкой аннотаций типов, автодополнения IDE, `_asdict()` и
полной
интеграцией с `mypy`. Иммутабелен, хешируем, идеален для типизированных DTO, конфигураций и API ответов.

## **Создание NamedTuple**

```python
from typing import NamedTuple


# 1.1 Классовый синтаксис (рекомендуемый)
class Point(NamedTuple):
    x: int
    y: int


p1 = Point(1, 2)  # Point(x=1, y=2)
p2 = Point(x=3, y=4)  # По именам


# 1.2 Наследование с методами
class Circle(NamedTuple):
    center: Point
    radius: float = 1.0  # Значения по умолчанию


c1 = Circle(Point(0, 0))  # Circle(center=Point(x=0, y=0), radius=1.0)

# 1.3 Из итерируемых объектов
data = [10, 20]
p3 = Point._make(data)  # Point(x=10, y=20)
```

## **Атрибуты типа NamedTuple**

Тип `NamedTuple` генерирует поля как атрибуты класса с типами.

```python
class Point(NamedTuple):
    x: int
    y: int


p = Point(1, 2)

# Поля как атрибуты
print(p.x)  # 1
print(p.y)  # 2
print(p._fields)  # ('x', 'y')

print(len(p))  # 2 — количество полей
print(type(len(p)))  # <class 'int'>

# Нет атрибутов типа real/numerator
try:
    print(p.real)
except AttributeError as e:
    print(f"Ошибка: {e}")
```

## **Методы типа NamedTuple**

Тип `NamedTuple` наследует методы `tuple` + добавляет типизированные методы.

### **Наследованные методы tuple**

```python
p = Point(1, 2)

print(p.index(1))  # 0
print(p.count(1))  # 1
print(1 in p)  # True
```

### **Специальные методы NamedTuple**

```python
p = Point(1, 2)

print(p._asdict())  # {'x': 1, 'y': 2}
print(p._replace(x=10))  # Point(x=10, y=2)
print(p._field_types)  # {'x': <class 'int'>, 'y': <class 'int'>}
```

## **Поддерживаемые операции**

```python
p1 = Point(1, 2)
p2 = Point(3, 4)

# 5.1 Доступ по индексу И по имени
print(p1[0])  # 1
print(p1.x)  # 1

# 5.2 Распаковка
x, y = p1  # 1, 2

# 5.3 Сравнения (лексикографические)
print(p1 < p2)  # True

# 5.4 Нет мутации
try:
    p1.x = 99
except AttributeError as e:
    print(f"Не поддерживается: {e}")

# 5.5 Итерируемость
print(list(p1))  # [1, 2]
```

## **Операции сравнения**

```python
p1 = Point(1, 2)
p2 = Point(1, 2)
p3 = Point(1, 3)

print(p1 == p2)  # True
print(p1 < p3)  # True
print(p1 == (1, 2))  # True (с tuple)

# Типизированное сравнение
print(Point(1, 2) == Point(x=1, y=2))  # True
```

## **Преобразование в другие типы и форматирование**

```python
p = Point(1, 2)

# 7.1 В другие коллекции
print(tuple(p))  # (1, 2)
print(list(p))  # [1, 2]
print(p._asdict())  # {'x': 1, 'y': 2}

# 7.2 В строку
print(str(p))  # 'Point(x=1, y=2)'
print(repr(p))  # 'Point(x=1, y=2)'

# 7.3 Распаковка
print(*p)  # 1 2
print(len(p))  # 2

# 7.4 JSON сериализация
import json

print(json.dumps(p._asdict()))  # {"x": 1, "y": 2}
```

## **Важные особенности**

```python
# 1. Полная типизация
p: Point = Point(1, 2)
reveal_type(p.x)  # int (mypy видит тип!)

# 2. Хешируемость
s = {p: 'value'}
print(s[Point(1, 2)])  # 'value'


# 3. Значения по умолчанию
class Config(NamedTuple):
    host: str = 'localhost'
    port: int = 8080


cfg = Config()  # Config(host='localhost', port=8080)

# 4. _replace() для "изменения"
p2 = p._replace(x=99)  # Point(x=99, y=2)

# 5. Метаданные типов
print(Point.__annotations__)  # {'x': <class 'int'>, 'y': <class 'int'>}


# 6. Наследование
class ColoredPoint(Point):
    color: str


cp = ColoredPoint(1, 2, 'red')
```

## **Важные замечания:**

1. **Типизация**: Полная поддержка `mypy`, автодополнение IDE.
2. **Иммутабельность**: Нельзя изменять поля, используйте `_replace()`.
3. **Хешируемость**: Можно использовать как ключи словарей.
4. **`_asdict()`** возвращает обычный `dict`.
5. **Значения по умолчанию**: Поддерживаются как в dataclass.
6. **Совместимость**: Работает с `collections.namedtuple`.

## **Ключевые выводы:**

1. **`NamedTuple` — типизированный immutable класс** с именованными полями.
2. **Полная типизация + IDE поддержка** — лучше `collections.namedtuple`.
3. **`p.x` и `p[0]`**, `_replace()`, `_asdict()` — удобный API.
4. **Аннотации типов** сохраняются в `__annotations__`.
5. **Идеален для**: API ответов, конфигураций, типизированных DTO.
6. **`class X(NamedTuple): x: int`** — основной способ создания.

[Содержание](/CONTENTS.md#содержание)

---

## array

## **Тип array в Python (типизированный массив)**

Тип `array` из модуля `array` представляет изменяемые упорядоченные последовательности **однотипных числовых данных**.
Это встроенный тип данных,
поддерживающий динамическое изменение размера, индексацию, срезы и компактное хранение чисел фиксированного типа (int,
float).
`array` мутабелен, не хешируем и используется для экономии памяти при работе с большими числовыми массивами.

## **Создание array**

```python
import array

# 1.1 Литералы array с тип-кодом
a1 = array.array('i', [1, 2, 3])  # signed int (4 байта)
a2 = array.array('f', [1.5, 2.5])  # float (4 байта)
a3 = array.array('b')  # Пустой signed char

# 1.2 Из итерируемых объектов
a4 = array.array('I', range(3))  # unsigned int: array('I', [0, 1, 2])
a5 = array.array('B', b'abc')  # unsigned char: array('B', [97, 98, 99])

# 1.3 Инициализация размером
a6 = array.array('h', [0] * 5)  # signed short × 5

# Тип-коды: 'bBhHiIlLqQfd' (char, short, int, long, float, double)
```

## **Атрибуты типа array**

Тип `array` имеет публичный атрибут `typecode` для типа данных.

```python
a = array.array('i', [1, 2, 3])

# Публичные атрибуты
print(a.typecode)  # 'i'
print(a.itemsize)  # 4 — размер элемента в байтах
print(len(a))  # 3 — количество элементов
print(type(len(a)))  # <class 'int'>

# Нет атрибутов типа real/numerator
try:
    print(a.real)
except AttributeError as e:
    print(f"Ошибка: {e}")
```

## **Методы типа array**

Тип `array` имеет методы для мутации и неизменяемые операции.

### **Методы изменения array (мутация)**

```python
a = array.array('i', [1, 2, 3])

a.append(4)  # array('i', [1, 2, 3, 4])
a.extend([5, 6])  # array('i', [1, 2, 3, 4, 5, 6])
a.insert(1, 99)  # array('i', [1, 99, 2, 3, 4, 5, 6])
a.pop()  # 6 → array('i', [1, 99, 2, 3, 4, 5])
a.remove(99)  # array('i', [1, 2, 3, 4, 5])
a.clear()  # array('i')
```

### **Методы поиска**

```python
a = array.array('i', [1, 2, 3, 2])

print(a.index(2))  # 1
print(a.count(2))  # 2
print(99 in a)  # False (O(n))
```

## **Поддерживаемые операции**

```python
a1 = array.array('i', [1, 2])
a2 = array.array('i', [3, 4])

# 5.1 Конкатенация и повторение
print(a1 + a2)  # array('i', [1, 2, 3, 4])
print(a1 * 2)  # array('i', [1, 2, 1, 2])

# 5.2 Индексация и срезы (int значения)
print(a1[0])  # 1
print(a1[-1])  # 2
print(a1[1:])  # array('i', [2])

# 5.3 Срезы с присваиванием (одинаковый тип!)
a1[0:1] = array.array('i', [99])  # array('i', [99, 2])

# 5.4 Сравнения
print(array.array('i', [1, 2]) < array.array('i', [1, 3]))  # True
```

## **Операции сравнения**

```python
a1 = array.array('i', [1, 2, 3])
a2 = array.array('i', [1, 2, 3])
a3 = array.array('i', [1, 2, 4])

print(a1 == a2)  # True
print(a1 < a3)  # True
print(a1 == [1, 2, 3])  # True (с list)
```

## **Преобразование в другие типы и форматирование**

```python
a = array.array('i', [1, 2, 3])

# 7.1 В другие коллекции
print(list(a))  # [1, 2, 3]
print(bytes(a))  # b'\x00\x00\x00\x01\x00\x00\x00\x02\x00\x00\x00\x03'
print(tuple(a))  # (1, 2, 3)

# 7.2 В строку
print(str(a))  # 'array('i', [1, 2, 3])'
print(repr(a))  # 'array('i', [1, 2, 3])'

# 7.3 Буферизация
print(a.tobytes())  # b'\x00\x00\x00\x01...'
print(a.tofile(open('test.bin', 'wb')))  # Запись в файл

# 7.4 Array comprehension
squares = array.array('i', (x ** 2 for x in range(3)))  # array('i', [0, 1, 4])
```

## **Важные особенности**

```python
# 1. Экономия памяти
import sys

nums = [1, 2, 3] * 1000
print(sys.getsizeof(nums))  # ~40KB (указатели)
a = array.array('i', nums)
print(sys.getsizeof(a))  # ~4KB (4 байта × 1000)

# 2. Строгая типизация
try:
    array.array('i', [1, 2, 'error'])  # TypeError!
except TypeError as e:
    print(f"Ошибка: {e}")

# 3. Ссылочная семантика
a1 = array.array('i', [1, 2])
a2 = a1
a2[0] = 99  # a1 тоже изменился!

# 4. Копирование
import copy

a_copy = a1[:]  # Поверхностная
a_deep = copy.deepcopy(a1)

# 5. Ложные значения
print(bool(array.array('i')))  # False
print(bool(array.array('i', [0])))  # True

# 6. Буферизация (frombytes/tobytes/fromfile/tofile)
```

## **Важные замечания:**

1. **Тип-коды**: `'bBhHiIlLqQfd'` — строгое соответствие типов.
2. **`itemsize`**: Фиксированный размер элемента (1-8 байт).
3. **Память**: ~5-10x экономнее `list` для чисел.
4. **Ссылки**: `a1 = a2` — ссылка, используйте `[:]`.
5. **Не хешируем**: Нельзя использовать как ключи словарей.
6. **`tobytes()`** для бинарных данных, `fromfile()` для чтения.

## **Ключевые выводы:**

1. **`array` — типизированный массив чисел** с компактным хранением.
2. **`typecode` определяет тип**: `'i'` (int), `'f'` (float), `'B'` (uchar).
3. **`itemsize × len()`** — точный размер в памяти.
4. **O(1) доступ**, как `list`, но **5-10x экономнее памяти**.
5. **Идеален для**: числовых данных, буферизации, NumPy-подготовки.
6. **`array.array('i', iter)`** — основной способ создания.

[Содержание](/CONTENTS.md#содержание)

---

## SimpleNamespace

## **Тип SimpleNamespace в Python**

`SimpleNamespace` — это класс из модуля `types`, предоставляющий удобный способ создания объектов с атрибутами,
доступными по точечной нотации. Он предназначен для замены словарей в случаях, когда нужен объектно-ориентированный
стиль доступа к полям. `SimpleNamespace` мутабелен, поддерживает динамическое добавление/удаление атрибутов и имеет
читаемое строковое представление.

## **Создание SimpleNamespace**

```python
from types import SimpleNamespace

# 2.1 Через конструктор с ключевыми аргументами
ns1 = SimpleNamespace(a=1, b=2, c='text')
print(ns1.a)  # 1

# 2.2 Из словаря
data = {'x': 10, 'y': 20}
ns2 = SimpleNamespace(**data)  # x=10, y=20

# 2.3 Пустой объект
ns3 = SimpleNamespace()

# 2.4 Вложенные объекты
ns4 = SimpleNamespace(name='test', data=SimpleNamespace(id=42, value='data'))

# 2.5 С None и списками
ns5 = SimpleNamespace(items=[1, 2, 3], optional=None)
```

## **Атрибуты типа SimpleNamespace**

`SimpleNamespace` не имеет фиксированных публичных атрибутов — все атрибуты динамические. Доступны стандартные методы
объекта.

```python
ns = SimpleNamespace(a=1, b=2)

# Динамические атрибуты
print(hasattr(ns, 'a'))  # True
print(ns.__dict__)  # {'a': 1, 'b': 2} — внутреннее хранилище

# Нет специальных атрибутов типа real/numerator
try:
    print(ns.real)
except AttributeError as e:
    print(f"Ошибка: {e}")  # 'SimpleNamespace' object has no attribute 'real'

print(type(ns).__name__)  # 'SimpleNamespace'
```

## **Методы типа SimpleNamespace**

У `SimpleNamespace` нет специальных методов мутации — используется прямое присваивание атрибутов. Поддерживает
стандартные методы объекта.

### **Добавление и удаление атрибутов**

```python
ns = SimpleNamespace(x=1)

# Добавление атрибута
ns.y = 2
ns.z = [1, 2, 3]  # Любые типы

# Удаление атрибута
del ns.x
print(hasattr(ns, 'x'))  # False

# Очистка всех атрибутов
ns.__dict__.clear()
```

### **Доступ к атрибутам**

```python
ns = SimpleNamespace(a=1, b='hello')

print(ns.a)  # 1 (точечная нотация)
print(getattr(ns, 'b'))  # 'hello'
print(hasattr(ns, 'c'))  # False

# Установка по умолчанию
print(getattr(ns, 'c', 'default'))  # 'default'
```

## **Поддерживаемые операции**

```python
ns1 = SimpleNamespace(a=1, b=2)
ns2 = SimpleNamespace(a=1, b=2)
ns3 = SimpleNamespace(a=1, b=3)

# 5.1 Сравнение объектов (по __dict__)
print(ns1 == ns2)  # True
print(ns1 == ns3)  # False

# 5.2 Строковое представление
print(ns1)  # namespace(a=1, b=2)
print(repr(ns1))  # namespace(a=1, b=2)
print(str(ns1))  # namespace(a=1, b=2)

# 5.3 Итерация по атрибутам
for key, value in ns1.__dict__.items():
    print(f"{key}: {value}")

# 5.4 Неподдерживаемые: арифметика
try:
    ns1 + ns2
except TypeError as e:
    print(f"Не поддерживается: {e}")
```

## **Операции сравнения**

```python
ns1 = SimpleNamespace(x=1, y=2)
ns2 = SimpleNamespace(x=1, y=2)
ns3 = SimpleNamespace(x=1, y=3)
ns4 = SimpleNamespace(x=1)

# Сравнение по содержимому __dict__
print(ns1 == ns2)  # True
print(ns1 != ns3)  # True

# Порядок атрибутов не важен
ns5 = SimpleNamespace(y=2, x=1)
print(ns1 == ns5)  # True

# Разные типы
print(ns1 == {'x': 1, 'y': 2})  # False (разные типы)
```

## **Преобразование в другие типы и форматирование**

```python
ns = SimpleNamespace(a=1, b=2, c=[3, 4])

# 7.1 В словарь
print(ns.__dict__)  # {'a': 1, 'b': 2, 'c': [3, 4]}
print(vars(ns))  # {'a': 1, 'b': 2, 'c': [3, 4]}

# 7.2 В JSON-подобный формат
import json

print(json.dumps(vars(ns)))  # {"a": 1, "b": 2, "c": [3, 4]}

# 7.3 Копирование
import copy

ns_copy = copy.copy(ns)  # Поверхностная копия
ns_deep = copy.deepcopy(ns)  # Глубокая копия


# 7.4 Распаковка атрибутов
def func(a, b):
    print(a, b)


func(**vars(ns))  # 1 2
```

## **Важные особенности**

```python
from types import SimpleNamespace

# 1. Динамические атрибуты (мутабельность)
ns = SimpleNamespace()
ns.x = 1
ns.y = 'dynamic'
del ns.x  # Легко удаление

# 2. Читаемое представление
print(SimpleNamespace(a=1, b=2))  # namespace(a=1, b=2)

# 3. Замена словарей
config = {'host': 'localhost', 'port': 8080}
cfg = SimpleNamespace(**config)
print(cfg.host)  # 'localhost' вместо config['host']

# 4. Ложные значения
print(bool(SimpleNamespace()))  # True (не пустой объект)
print(bool(SimpleNamespace(a=None)))  # True

# 5. Хешируемость
try:
    hash(SimpleNamespace())  # TypeError: unhashable type: 'SimpleNamespace'
except TypeError:
    print("Не хешируем из-за мутабельности")
```

## **Важные замечания:**

1. **Динамические атрибуты**: Можно добавлять/удалять поля в runtime через `ns.field = value`.
2. **__dict__ хранилище**: Все атрибуты хранятся в `ns.__dict__` как обычный словарь.
3. **Не хешируем**: Из-за мутабельности нельзя использовать как ключи словарей.
4. **Сравнение**: По содержимому `__dict__`, порядок ключей не важен.
5. **Копирование**: `copy.copy()` копирует поверхностно, `deepcopy()` — глубоко.
6. **Альтернативы**: `dataclasses`, `namedtuple` для неизменяемых структур.

## **Ключевые выводы:**

1. **`SimpleNamespace` — удобный объект с атрибутами** вместо словаря.
2. **Динамическая мутация**: `ns.new_field = value`, `del ns.old_field`.
3. **Читаемый `repr`**: `namespace(a=1, b=2)` — лучше чем `{'a': 1}`.
4. **Идеален для**: конфигураций, результатов функций, замены словарей.
5. **`vars(ns)` или `ns.__dict__`** — преобразование в словарь.
6. **Доступен с Python 3.3+** в модуле `types`.

[Содержание](/CONTENTS.md#содержание)

---

## Path

## **Тип Path в Python (объектный путь)**

`Path` — это класс из модуля `pathlib`, представляющий пути к файловой системе в объектно-ориентированном стиле. Это
современная альтернатива `os.path`, поддерживающая кросс-платформенность (Windows/Unix), цепочки методов и богатый API
для работы с файлами/директориями. `Path` неизменяем, хешируем и идеален для скриптов автоматизации.

## **Создание объектов Path**

```python
from pathlib import Path

# 1.1 Литералы путей
p1 = Path("file.txt")
p2 = Path("/home/user/docs")
p3 = Path()  # Текущая директория
p4 = Path("dir", "subdir", "file.py")  # Многоуровневый

# 1.2 Абсолютные пути
p5 = Path.home()  # /home/user
p6 = Path.cwd()  # /current/working/dir
p7 = Path("/etc/passwd")

# 1.3 Из строк и переменных
import os

p8 = Path(os.environ["HOME"]) / "projects"
p9 = Path(*["usr", "bin", "python"])  # Распаковка

# 1.4 Специальные пути
p10 = Path("nonexistent")  # Может не существовать
```

## **Атрибуты типа Path**

`Path` имеет атрибуты для разбора пути и метаданных файловой системы.

```python
p = Path("dir/file.txt")

# Основные атрибуты пути
print(p.name)  # 'file.txt' — имя файла
print(p.stem)  # 'file' — имя без расширения
print(p.suffix)  # '.txt' — расширение
print(p.parent)  # 'dir' — родительская директория
print(p.parts)  # ('dir', 'file.txt') — компоненты

# Метаданные (если существует)
print(p.exists())  # False
print(p.is_file())  # False
print(p.is_dir())  # False

# Нет атрибутов типа real/numerator
try:
    print(p.real)
except AttributeError as e:
    print(f"Ошибка: {e}")
```

## **Методы типа Path**

`Path` предоставляет методы для навигации, проверки и манипуляции путями/файлами.

### **Методы разбора и навигации**

```python
p = Path("docs/project/README.md")

print(p.parent)  # docs/project
print(p.parents[1])  # docs
print(p.with_name("main.md"))  # docs/project/main.md
print(p.with_suffix(".txt"))  # docs/project/README.txt
print(p.with_stem("INDEX"))  # docs/project/INDEX.md
```

### **Методы работы с файловой системой**

```python
p = Path("test.txt")

# Создание/проверка
p.write_text("Hello")  # Создает файл
p.read_text()  # 'Hello'
p.unlink()  # Удаляет файл

dir_path = Path("temp_dir")
dir_path.mkdir(exist_ok=True)  # Создает директорию
dir_path.rmdir()  # Удаляет пустую директорию
```

### **Методы поиска и итерации**

```python
root = Path(".")

print(root.exists())  # True
print(root.is_dir())  # True
print(list(root.glob("*.py")))  # Список .py файлов
print(list(root.rglob("*.txt")))  # Рекурсивно *.txt

# Итерация содержимого
for child in root.iterdir():
    print(child.name)
```

## **Поддерживаемые операции**

```python
p1 = Path("dir1")
p2 = Path("file.txt")
p3 = Path("dir2/subdir")

# 5.1 Оператор / для объединения путей
p4 = p1 / p2  # dir1/file.txt
print(p4)

# 5.2 Сравнения путей
print(p1 == p1)  # True
print(p1 < p3)  # Лексикографическое

# 5.3 Строковое представление
print(str(p1))  # 'dir1'
print(p1.as_posix())  # 'dir1' (Unix-стиль)
print(repr(p1))  # Path('dir1')

# 5.4 Неподдерживаемые: арифметика
try:
    p1 + p2
except TypeError as e:
    print(f"Не поддерживается: {e}")
```

## **Операции сравнения**

```python
p1 = Path("file1.txt")
p2 = Path("file1.txt")
p3 = Path("file2.txt")
p4 = Path("dir/file.txt")

# Нормализованное сравнение (case-sensitive)
print(p1 == p2)  # True
print(p1 == p3)  # False
print(p1 < p3)  # True (лексикографически)

# Разные типы
print(p1 == "file1.txt")  # False (разные типы)
print(p1 == str(p1))  # False
```

## **Преобразование в другие типы и форматирование**

```python
p = Path("docs/file.txt")

# 7.1 В примитивы
print(str(p))  # 'docs/file.txt'
print(p.as_posix())  # 'docs/file.txt'
print(bytes(p))  # b'docs/file.txt'

# 7.2 В другие коллекции
print(p.parts)  # ('docs', 'file.txt')
print(tuple(p.parts))  # ('docs', 'file.txt')
print(list(p.glob("*")))  # [Path(...), ...]

# 7.3 Абсолютный путь
print(p.absolute())  # /full/absolute/path/docs/file.txt
print(p.resolve())  # Разрешенный путь (симлинки)

# 7.4 JSON сериализация
import json

print(json.dumps({"path": str(p)}))
```

## **Важные особенности**

```python
from pathlib import Path

# 1. Неизменяемость (иммутабельность)
p1 = Path("file.txt")
p2 = p1.with_name("new.txt")  # p1 не изменился!
print(p1)  # file.txt

# 2. Кросс-платформенность
p_win = Path(r"C:\Users\user\docs")
p_unix = Path("/home/user/docs")
print(p_win.as_posix())  # /c/Users/user/docs

# 3. Автоматическая нормализация
p = Path("dir//file.txt") / "../file.txt"
print(p.resolve())  # Нормализованный путь

# 4. Ложные значения
print(bool(Path("nonexistent")))  # False
print(bool(Path(".")))  # True

# 5. Хешируемость (можно в set/dict)
paths = {Path("a"), Path("b")}
print(len(paths))  # 2
```

## **Важные замечания:**

1. **Неизменяемость**: Методы возвращают **новые** Path, оригинал не меняется.
2. **Кросс-платформенность**: Автоматически обрабатывает `/` и `\`.
3. **Оператор `/`**: `parent / child` — удобнее `os.path.join()`.
4. **Чувствительность к регистру**: Зависит от ОС (Windows — нет, Linux — да).
5. **Символические ссылки**: `resolve()` разрешает, `readlink()` читает.
6. **Windows UNC пути**: Поддерживает `\\server\share\path`.

## **Ключевые выводы:**

1. **`Path` — объектный путь** вместо строк `os.path`.
2. **Неизменяемый, хешируемый**, цепочки методов: `p.parent / "file".with_suffix(".py")`.
3. **Кросс-платформенный**: Работает на Windows/Unix без изменений.
4. **Идеален для**: скриптов, CLI, тестов, DevOps задач.
5. **`/` для join**, `glob()`/`rglob()` для поиска файлов.
6. **`pathlib` с Python 3.4+** — современный стандарт вместо `os.path`.

[Содержание](/CONTENTS.md#содержание)

---

## UUID

## **Тип UUID в Python (универсальный идентификатор)**

`UUID` — это класс из модуля `uuid`, представляющий универсальный уникальный идентификатор (Universally Unique
Identifier). Это 128-битное значение в формате RFC 4122, используемое для создания уникальных идентификаторов без
координации. `UUID` неизменяем, хешируем и поддерживает 5 версий генерации (v1-v5) для разных сценариев.

## **Создание объектов UUID**

```python
import uuid

# 1.1 UUID версии 1 (на основе времени + MAC-адрес)
u1 = uuid.uuid1()
print(u1)  # 123e4567-e89b-12d3-a456-426614174000

# 1.2 UUID версии 4 (случайный)
u2 = uuid.uuid4()
print(u2)  # 550e8400-e29b-41d4-a716-446655440000

# 1.3 UUID версии 5 (SHA-1 хеш от namespace + name)
ns_dns = uuid.NAMESPACE_DNS
u3 = uuid.uuid5(ns_dns, 'python.org')
print(u3)  # 4ca978ad-9d79-4c90-b9ba-44eb80b18dd9

# 1.4 Из строки
u4 = uuid.UUID('12345678-1234-5678-9abc-123456789abc')
u5 = uuid.UUID('urn:uuid:12345678-1234-5678-9abc-123456789abc')

# 1.5 Пустой/нулевой UUID
u6 = uuid.UUID('00000000-0000-0000-0000-000000000000')
```

## **Атрибуты типа UUID**

`UUID` имеет атрибуты для доступа к компонентам 128-битного значения.

```python
u = uuid.UUID('12345678-1234-5678-9abc-123456789abc')

# Байт-компоненты (int)
print(u.hex)  # '12345678123456789abcdef123456789abc'
print(u.int)  # 32424403007676212263071501564589869012
print(u.version)  # 1 (версия UUID)
print(u.variant)  # 1 (вариант RFC 4122)

# Временные атрибуты (для v1)
print(u.time_low)  # 305419896
print(u.time_mid)  # 4660
print(u.time_hi_version)  # 2232

# Нет атрибутов типа real/numerator
try:
    print(u.real)
except AttributeError as e:
    print(f"Ошибка: {e}")
```

## **Методы типа UUID**

`UUID` неизменяем, поэтому методы возвращают новые объекты или информацию.

### **Основные методы**

```python
u = uuid.UUID('12345678-1234-5678-9abc-123456789abc')

# Строковые представления
print(u.urn)  # 'urn:uuid:12345678-1234-5678-9abc-123456789abc'
print(u.bytes)  # b'\x12\x34\x56\x78\x12\x34\x56\x78\x9a\xbc\x12\x34\x56\x78\x9a\xbc'
print(u.fields)  # (305419896, 4660, 2232, 1, 16)

# Проверка версии
print(u.version == 1)  # True

# Генерация новых
new_uuid = uuid.uuid4()
print(isinstance(new_uuid, uuid.UUID))  # True
```

### **Статические методы модуля uuid**

```python
# Пространства имен
print(uuid.NAMESPACE_DNS)  # 6ba7b810-9dad-11d1-80b4-00c04fd430c8
print(uuid.NAMESPACE_URL)  # 6ba7b811-9dad-11d1-80b4-00c04fd430c8

# UUID версии 3 (MD5)
u3 = uuid.uuid3(uuid.NAMESPACE_DNS, 'python.org')
print(u3)
```

## **Поддерживаемые операции**

```python
u1 = uuid.UUID('12345678-1234-5678-9abc-123456789abc')
u2 = uuid.UUID('12345678-1234-5678-9abc-123456789abc')
u3 = uuid.UUID('87654321-4321-8765-cba9-abcdef012345')

# 5.1 Сравнения
print(u1 == u2)  # True
print(u1 < u3)  # True (лексикографическое)

# 5.2 Хеширование
print(hash(u1))  # int хеш
s = {u1, u2}  # Работает как ключ

# 5.3 Строковое представление
print(str(u1))  # '12345678-1234-5678-9abc-123456789abc'
print(repr(u1))  # UUID('12345678-1234-5678-9abc-123456789abc')

# 5.4 Неподдерживаемые: арифметика
try:
    u1 + u2
except TypeError as e:
    print(f"Не поддерживается: {e}")
```

## **Операции сравнения**

```python
u1 = uuid.UUID('00000000-0000-0000-0000-000000000001')
u2 = uuid.UUID('00000000-0000-0000-0000-000000000002')
u3 = uuid.UUID('00000000-0000-0000-0000-000000000000')

# Лексикографическое сравнение по hex
print(u1 == u1)  # True
print(u1 > u3)  # True
print(u1 < u2)  # True

# Разные типы
print(u1 == str(u1))  # False
print(u1 == u1.bytes)  # False

# Случайные UUID уникальны
print(uuid.uuid4() == uuid.uuid4())  # Почти всегда False
```

## **Преобразование в другие типы и форматирование**

```python
u = uuid.UUID('12345678-1234-5678-9abc-123456789abc')

# 7.1 В примитивы
print(str(u))  # '12345678-1234-5678-9abc-123456789abc'
print(u.hex)  # '12345678123456789abcdef123456789abc'
print(int(u))  # 32424403007676212263071501564589869012
print(u.bytes)  # b'\x12\x34\x56\x78...'

# 7.2 В JSON
import json

print(json.dumps(str(u)))  # "12345678-1234-5678-9abc-123456789abc"

# 7.3 Бинарные форматы
print(u.hex.encode())  # bytes из hex
print(uuid.UUID(bytes=u.bytes))  # Из bytes

# 7.4 Сериализация
data = {'id': u}
print(data)  # {'id': UUID('12345678-1234-5678-9abc-123456789abc')}
```

## **Важные особенности**

```python
import uuid

# 1. Неизменяемость
u = uuid.uuid4()
# u.bytes = b'...'  # AttributeError!

# 2. Версии UUID
print(uuid.uuid1().version)  # 1 (время+MAC)
print(uuid.uuid3(uuid.NAMESPACE_DNS, 'test').version)  # 3 (MD5)
print(uuid.uuid4().version)  # 4 (случайный)
print(uuid.uuid5(uuid.NAMESPACE_DNS, 'test').version)  # 5 (SHA1)

# 3. Практическая уникальность
print(uuid.uuid4() == uuid.uuid4())  # ~1/2^122 шанс совпадения

# 4. Ложные значения
print(bool(uuid.UUID('00000000-0000-0000-0000-000000000000')))  # False
print(bool(uuid.uuid4()))  # True

# 5. Хешируемость
uuid_set = {uuid.uuid4() for _ in range(3)}
print(len(uuid_set))  # 3
```

## **Важные замечания:**

1. **Неизменяемость**: После создания UUID нельзя изменить.
2. **Версии**: v1(время), v3(MD5), v4(случайный), v5(SHA1) — выбирайте по задаче.
3. **Коллизии**: v4 имеет ~1/2^122 шанс совпадения — практически уникален.
4. **Размер**: Всегда 128 бит, 36 символов в строке.
5. **Производительность**: `uuid4()` быстрее `uuid1()` (без системных вызовов).
6. **Сетевой трафик**: Храните как `bytes` (16 байт) вместо строки (36+).

## **Ключевые выводы:**

1. **`UUID` — неизменяемый 128-битный уникальный идентификатор**.
2. **5 версий**: `uuid4()` для случайных, `uuid5()` для детерминированных.
3. **Хешируемый**: Идеален как ключ в `dict`/`set`/БД.
4. **Идеален для**: ID пользователей, сессий, файлов, записей БД.
5. **`str(u)`, `u.bytes`, `int(u)`** — основные преобразования.
6. **`uuid` модуль с Python 2.5+** — стандарт для уникальных ID.

[Содержание](/CONTENTS.md#содержание)

---

## datetime

## **Тип datetime в Python (дата и время)**

`datetime` — это класс из модуля `datetime`, представляющий дату и время как единый объект. Поддерживает точность до
микросекунд, часовые пояса, арифметику дат и богатый API для форматирования/парсинга. `datetime` неизменяем, хешируем и
является основным типом для работы с временными метками в Python.

## **Создание объектов datetime**

```python
from datetime import datetime, date, time

# 1.1 Конструктор с параметрами
dt1 = datetime(2023, 12, 25, 15, 30, 45, 123456)
print(dt1)  # 2023-12-25 15:30:45.123456

# 1.2 Текущая дата/время
dt2 = datetime.now()
dt3 = datetime.utcnow()  # UTC

# 1.3 Из строк (ISO 8601 и др.)
dt4 = datetime.fromisoformat('2023-12-25T15:30:45.123456')
dt5 = datetime.strptime('25/12/2023 15:30', '%d/%m/%Y %H:%M')

# 1.4 Из timestamp (Unix time)
dt6 = datetime.fromtimestamp(1703524245)

# 1.5 Специальные значения
dt7 = datetime.min  # 0001-01-01 00:00:00
dt8 = datetime.max  # 9999-12-31 23:59:59.999999
```

## **Атрибуты типа datetime**

`datetime` имеет атрибуты для доступа к компонентам даты/времени.

```python
dt = datetime(2023, 12, 25, 15, 30, 45, 123456)

# Компоненты даты
print(dt.year)  # 2023
print(dt.month)  # 12
print(dt.day)  # 25

# Компоненты времени
print(dt.hour)  # 15
print(dt.minute)  # 30
print(dt.second)  # 45
print(dt.microsecond)  # 123456

# Дата и время отдельно
print(dt.date())  # 2023-12-25
print(dt.time())  # 15:30:45.123456

# Нет атрибутов типа real/numerator
try:
    print(dt.real)
except AttributeError as e:
    print(f"Ошибка: {e}")
```

## **Методы типа datetime**

`datetime` предоставляет методы для арифметики, форматирования и преобразований.

### **Арифметические методы**

```python
from datetime import timedelta

dt = datetime(2023, 12, 25, 15, 30)

# Добавление/вычитание времени
dt2 = dt + timedelta(days=1, hours=2)  # 2023-12-26 17:30:00
dt3 = dt - timedelta(hours=1)  # 2023-12-25 14:30:00

# Разница между датами
delta = dt2 - dt
print(delta)  # 1 day, 2:00:00
print(delta.total_seconds())  # 93600.0
```

### **Форматирование и парсинг**

```python
dt = datetime(2023, 12, 25, 15, 30, 45)

# В строку
print(dt.strftime('%Y-%m-%d %H:%M:%S'))  # 2023-12-25 15:30:45
print(dt.isoformat())  # 2023-12-25T15:30:45

# Из строки (обратная операция)
dt_parsed = datetime.strptime('2023-12-25 15:30:45', '%Y-%m-%d %H:%M:%S')
```

## **Поддерживаемые операции**

```python
from datetime import datetime, timedelta

dt1 = datetime(2023, 12, 25, 15, 0)
dt2 = datetime(2023, 12, 25, 16, 0)
delta = timedelta(hours=1)

# 5.1 Арифметика с timedelta
print(dt1 + delta)  # 2023-12-25 16:00:00
print(dt2 - delta)  # 2023-12-25 15:00:00

# 5.2 Сравнения
print(dt1 < dt2)  # True

# 5.3 Строковое представление
print(str(dt1))  # '2023-12-25 15:00:00'
print(repr(dt1))  # 'datetime.datetime(2023, 12, 25, 15, 0)'

# 5.4 Неподдерживаемые: //, %, *
try:
    dt1 // dt2
except TypeError as e:
    print(f"Не поддерживается: {e}")
```

## **Операции сравнения**

```python
dt1 = datetime(2023, 12, 25, 15, 0)
dt2 = datetime(2023, 12, 25, 15, 0)
dt3 = datetime(2023, 12, 25, 16, 0)

# Полное сравнение (дата + время)
print(dt1 == dt2)  # True
print(dt1 < dt3)  # True

# Разные типы
print(dt1 == dt1.date())  # False
print(dt1.timestamp() == dt1.timestamp())  # True (float)
```

## **Преобразование в другие типы и форматирование**

```python
dt = datetime(2023, 12, 25, 15, 30, 45, 123456)

# 7.1 В примитивы
print(dt.timestamp())  # 1703524245.123456 (Unix timestamp)
print(int(dt.timestamp()))  # 1703524245
print(dt.date())  # date(2023, 12, 25)
print(dt.time())  # time(15, 30, 45, 123456)

# 7.2 Строковые форматы
print(dt.isoformat())  # '2023-12-25T15:30:45.123456'
print(dt.strftime('%d.%m.%Y'))  # '25.12.2023'

# 7.3 JSON сериализация
import json
from datetime import date

print(json.dumps({'dt': dt.isoformat()}))  # {"dt": "2023-12-25T15:30:45.123456"}
```

## **Важные особенности**

```python
from datetime import datetime, timezone, timedelta

# 1. Неизменяемость
dt = datetime(2023, 1, 1)
# dt.year = 2024  # AttributeError!

# 2. Часовые пояса (Python 3.2+)
utc = timezone.utc
tz_msk = timezone(timedelta(hours=3))
dt_tz = datetime(2023, 1, 1, tzinfo=utc)
print(dt_tz.astimezone(tz_msk))  # Конвертация

# 3. Ложные значения
print(bool(datetime.min))  # True
print(bool(datetime.max))  # True

# 4. Хешируемость
dt_set = {datetime.now(), datetime.now()}
print(len(dt_set))  # 2 (разные моменты)

# 5. Диапазон: 1-9999 гг.
print(datetime.min.year)  # 1
print(datetime.max.year)  # 9999
```

## **Важные замечания:**

1. **Неизменяемость**: Методы возвращают **новые** `datetime`, оригинал не меняется.
2. **`datetime.now()` vs `datetime.utcnow()`**: Первый — локальное, второй — UTC.
3. **`tzinfo=None` по умолчанию**: Без часового пояса (naive datetime).
4. **Директивы формата**: `%Y`, `%m`, `%d`, `%H`, `%M`, `%S`, `%f` (микросекунды).
5. **`timedelta` для арифметики**: Не поддерживает месяцы/годы (переменная длина).
6. **ISO 8601**: `fromisoformat()`/`isoformat()` — стандарт для JSON/API.

## **Ключевые выводы:**

1. **`datetime` — неизменяемый объект даты+времени** с микросекундной точностью.
2. **`now()`, `fromisoformat()`, `strptime()`** — основные способы создания.
3. **Арифметика через `timedelta`**, сравнение полное (дата+время).
4. **Идеален для**: логов, API, БД, планировщиков, отчетов.
5. **`strftime()`/`strptime()`** — форматирование/парсинг дат.
6. **`datetime` с Python 2.3+** — стандарт вместо ручных вычислений.

[Содержание](/CONTENTS.md#содержание)

---

## Enum

## **Тип Enum в Python (перечисление)**

`Enum` — это класс из модуля `enum`, предоставляющий типобезопасные перечисления с именованными константами. Гарантирует
уникальность членов, поддерживает хеширование, сравнение и итерацию. `Enum` неизменяем, идеален для замены констант
`int`/строк и создания самодокументируемого кода.

## **Создание объектов Enum**

```python
from enum import Enum, auto
import enum


# 1.1 Классический Enum с явными значениями
class Color(Enum):
    RED = 1
    GREEN = 2
    BLUE = 3


c1 = Color.RED
c2 = Color(2)  # По значению
c3 = Color['RED']  # По имени


# 1.2 Автоматические значения (auto())
class Status(enum.Enum):
    PENDING = auto()  # 1
    RUNNING = auto()  # 2
    DONE = auto()  # 3


# 1.3 Enum с строками
class HttpMethod(str, Enum):
    GET = "GET"
    POST = "POST"
    PUT = "PUT"


# 1.4 Пустой/одиночный
class Single(Enum):
    ONLY = 42
```

## **Атрибуты типа Enum**

Enum-члены имеют атрибуты `name` (имя) и `value` (значение).

```python
from enum import Enum


class Color(Enum):
    RED = 1
    GREEN = 2


c = Color.RED

# Основные атрибуты
print(c.name)  # 'RED'
print(c.value)  # 1
print(type(c))  # <enum 'Color'>

# Классовые атрибуты
print(Color.RED)  # <Color.RED: 1>
print(Color.__members__)  # dict всех членов

# Нет атрибутов типа real/numerator
try:
    print(c.real)
except AttributeError as e:
    print(f"Ошибка: {e}")
```

## **Методы типа Enum**

Enum поддерживает доступ по имени/значению, итерацию и проверки.

### **Доступ и проверки**

```python
class Color(Enum):
    RED = 1
    GREEN = 2
    BLUE = 3


# Поиск по имени/значению
print(Color['RED'])  # Color.RED
print(Color(2))  # Color.GREEN

# Проверки
print(Color.RED in Color)  # True
print(Color.RED is Color.RED)  # True

# Итерация
for color in Color:
    print(f"{color.name}: {color.value}")
```

### **Классовые методы**

```python
class Status(Enum):
    PENDING = 1
    DONE = 2


print(Status._member_names_)  # ['PENDING', 'DONE']
print(Status._member_map_)  # OrderedDict всех членов
print(Status._value2member_map_)  # {1: Status.PENDING, 2: Status.DONE}
```

## **Поддерживаемые операции**

```python
class Color(Enum):
    RED = 1
    GREEN = 2
    BLUE = 3


c1 = Color.RED
c2 = Color.GREEN
c3 = Color.RED

# 5.1 Сравнения (по идентичности)
print(c1 == c3)  # True
print(c1 is c3)  # True
print(c1 < c2)  # True (по порядку объявления)

# 5.2 Хеширование
colors = {c1, c2}  # Работает
print(hash(c1))  # int

# 5.3 Строковое представление
print(str(c1))  # 'Color.RED'
print(repr(c1))  # '<Color.RED: 1>'

# 5.4 Неподдерживаемые: арифметика
try:
    c1 + c2
except TypeError as e:
    print(f"Не поддерживается: {e}")
```

## **Операции сравнения**

```python
class Priority(Enum):
    LOW = 1
    MEDIUM = 2
    HIGH = 3


low = Priority.LOW
med = Priority.MEDIUM
high = Priority.HIGH

# Сравнение по идентичности/порядку
print(low == low)  # True
print(low != med)  # True
print(low < med < high)  # True

# Разные типы
print(low == 1)  # False
print(low == 'LOW')  # False

# Итерационный порядок
print(list(Priority))  # [LOW, MEDIUM, HIGH]
```

## **Преобразование в другие типы и форматирование**

```python
class Color(Enum):
    RED = 1
    GREEN = 2


c = Color.RED

# 7.1 В примитивы
print(c.name)  # 'RED'
print(c.value)  # 1
print(int(c.value))  # 1

# 7.2 JSON сериализация
import json

print(json.dumps(c.name))  # "RED"

# 7.3 Итерация и списки
print(list(Color))  # [<Color.RED: 1>, <Color.GREEN: 2>]
names = [e.name for e in Color]  # ['RED', 'GREEN']

# 7.4 Пользовательское форматирование
print(f"{c!r}")  # '<Color.RED: 1>'
print(f"{c!s}")  # 'Color.RED'
```

## **Важные особенности**

```python
from enum import Enum, auto


# 1. Неизменяемость членов
class Color(Enum):
    RED = 1


# Color.RED = 2  # AttributeError!

# 2. Уникальность значений (по умолчанию)
try:
    class Bad(Enum):
        A = 1
        B = 1  # ValueError: duplicate values
except ValueError:
    pass


# 3. Автоматические значения
class AutoEnum(Enum):
    A = auto()  # 1
    B = auto()  # 2


# 4. Ложные значения
print(bool(Color.RED))  # True

# 5. Хешируемость гарантирована
enum_set = {Color.RED, Color.GREEN}
print(Color.RED in enum_set)  # True
```

## **Важные замечания:**

1. **Уникальность**: Значения уникальны в пределах Enum (кроме `@enum.unique`).
2. **`auto()`**: Автоинкремент начиная с 1.
3. **Наследование**: `IntEnum`, `StrEnum`, `Flag` для специальных случаев.
4. **Сравнение**: По идентичности членов, не по `value`.
5. **`_ignore_`**: Исключить поля из enum-членов.
6. **Производительность**: Быстрее словарей для констант.

## **Ключевые выводы:**

1. **`Enum` — типобезопасные именованные константы** с `name`/`value`.
2. **`Color.RED`, `Color['RED']`, `Color(1)`** — доступ к членам.
3. **Хешируемый, неизменяемый**, идеален для `switch`-подобных конструкций.
4. **Идеален для**: состояний, статусов, кодов ошибок, конфигураций.
5. **`auto()`** и наследование (`IntEnum`, `StrEnum`) для гибкости.
6. **`enum` с Python 3.4+** — стандарт вместо магических чисел/строк.

[Содержание](/CONTENTS.md#содержание)

---

## dataclass

`dataclass` — это декоратор из модуля `dataclasses`, предоставляющий **автоматическую генерацию специальных методов**
для классов, хранящих данные. Заменяет ручное создание `__init__`, `__repr__`, `__eq__`, экономя время и уменьшая
вероятность ошибок. Поддерживает параметры инициализации, неизменяемость, постобработку и вложенность. Идеален для DTO,
конфигураций и доменных моделей.

## **Создание dataclass**

```python
from dataclasses import dataclass, field, asdict, astuple, replace
from typing import List, Any, ClassVar


# 1.1 Базовый dataclass
@dataclass
class Point:
    x: int
    y: int
    z: int = 0  # Значение по умолчанию


p1 = Point(1, 2)  # Point(x=1, y=2, z=0)
p2 = Point(3, 4, 5)  # Point(x=3, y=4, z=5)


# 1.2 Параметры декоратора
@dataclass(init=True, repr=True, eq=True, order=True, unsafe_hash=False, frozen=False)
class Config:
    host: str = "localhost"
    port: int = 8080


# 1.3 Неизменяемый (frozen) dataclass
@dataclass(frozen=True)
class ImmutablePoint:
    x: int
    y: int


# 1.4 Сложные поля и default_factory
@dataclass
class Inventory:
    items: List[str] = field(default_factory=list)
    metadata: dict = field(default_factory=dict)
    id: int = field(init=False)  # Не включать в __init__


# 1.5 Классовые переменные и метаданные
@dataclass
class Product:
    name: str
    price: float
    category: ClassVar[str] = "general"  # Не поле экземпляра
    weight: float = field(metadata={"unit": "kg"})  # Метаданные для поля
```

## **Атрибуты типа dataclass**

`dataclass` не имеет специальных атрибутов, но генерирует поля как атрибуты экземпляра.

```python
@dataclass
class Person:
    name: str
    age: int = 30


p = Person("Alice")
print(p.name)  # 'Alice'
print(p.age)  # 30
print(hasattr(p, "__dataclass_fields__"))  # True - метаданные класса

# Нет атрибутов типа real/numerator
try:
    print(p.real)
except AttributeError as e:
    print(f"Ошибка: {e}")  # 'Person' object has no attribute 'real'
```

## **Методы типа dataclass**

`dataclass` автоматически генерирует методы, но можно переопределять или добавлять свои.

### **Автоматически генерируемые методы**

```python
@dataclass
class Point:
    x: int
    y: int


p = Point(1, 2)

# __init__ - автоматически сгенерирован
print(p)  # Point(x=1, y=2)

# __repr__ - читаемое представление
print(repr(p))  # 'Point(x=1, y=2)'

# __eq__ - сравнение по полям
p2 = Point(1, 2)
print(p == p2)  # True


# __lt__, __le__, __gt__, __ge__ (если order=True)
@dataclass(order=True)
class OrderedPoint:
    x: int
    y: int


p1 = OrderedPoint(1, 2)
p2 = OrderedPoint(1, 3)
print(p1 < p2)  # True (сравнение по x, затем по y)
```

### **Пользовательские методы и post_init**

```python
from dataclasses import dataclass, field


@dataclass
class Circle:
    radius: float
    area: float = field(init=False)  # Вычисляемое поле

    def __post_init__(self):
        # Вызывается после __init__
        self.area = 3.14159 * self.radius ** 2


c = Circle(2.0)
print(c.area)  # 12.56636


# Переопределение методов
@dataclass
class Vector:
    x: float
    y: float

    def __add__(self, other):
        return Vector(self.x + other.x, self.y + other.y)

    def magnitude(self):
        return (self.x ** 2 + self.y ** 2) ** 0.5


v1 = Vector(1, 2)
v2 = Vector(3, 4)
print(v1 + v2)  # Vector(x=4, y=6)
print(v1.magnitude())  # 2.236...
```

## **Поддерживаемые операции**

```python
from dataclasses import dataclass, asdict, astuple, replace


@dataclass
class Point:
    x: int
    y: int


p1 = Point(1, 2)
p2 = Point(3, 4)

# 5.1 Доступ к полям
print(p1.x)  # 1
print(p1.y)  # 2

# 5.2 Сравнения (если eq=True)
print(p1 == Point(1, 2))  # True
print(p1 != p2)  # True

# 5.3 Преобразование в dict/tuple
print(asdict(p1))  # {'x': 1, 'y': 2}
print(astuple(p1))  # (1, 2)

# 5.4 Замена полей (для frozen=False)
p3 = replace(p1, x=10)  # Point(x=10, y=2)
print(p1)  # Point(x=1, y=2) - оригинал не изменился

# 5.5 Неподдерживаемые: арифметика
try:
    p1 + p2
except TypeError as e:
    print(f"Не поддерживается: {e}")
```

## **Операции сравнения**

```python
from dataclasses import dataclass


@dataclass(eq=True)
class Person:
    name: str
    age: int


p1 = Person("Alice", 30)
p2 = Person("Alice", 30)
p3 = Person("Bob", 25)

print(p1 == p2)  # True (сравнение по всем полям)
print(p1 == p3)  # False


# Сравнение с order=True
@dataclass(order=True)
class Product:
    price: float
    name: str


prod1 = Product(10.0, "Apple")
prod2 = Product(20.0, "Banana")
prod3 = Product(10.0, "Orange")

print(prod1 < prod2)  # True (по price)
print(prod1 < prod3)  # True (по price, затем по name)
```

## **Преобразование в другие типы и форматирование**

```python
from dataclasses import dataclass, asdict, astuple
import json


@dataclass
class User:
    id: int
    name: str
    email: str = None


u = User(1, "Alice", "alice@example.com")

# 7.1 В dict/tuple
print(asdict(u))  # {'id': 1, 'name': 'Alice', 'email': 'alice@example.com'}
print(astuple(u))  # (1, 'Alice', 'alice@example.com')

# 7.2 В JSON
print(json.dumps(asdict(u)))  # {"id": 1, "name": "Alice", "email": "alice@example.com"}

# 7.3 Форматирование через f-строки
print(f"User: {u.name}, ID: {u.id}")  # User: Alice, ID: 1


# 7.4 Кастомное str представление
@dataclass
class Student:
    name: str
    grade: str

    def __str__(self):
        return f"{self.name} ({self.grade})"


s = Student("John", "A")
print(str(s))  # John (A)
```

## **Важные особенности**

```python
from dataclasses import dataclass, field
import copy


# 1. Поля без значений по умолчанию должны идти первыми
@dataclass
class BadClass:
    default: int = 0
    required: str  # SyntaxError! Поля без default должны идти первыми


# 2. Наследование
@dataclass
class Base:
    x: int
    y: int = 0


@dataclass
class Derived(Base):
    z: int
    y: int = 1  # Переопределение значения по умолчанию


d = Derived(1, 2)  # x=1, y=2, z=?


# Ошибка! Порядок полей: x, z, y - не соответствует параметрам __init__

# 3. Глубокое копирование
@dataclass
class Container:
    items: list = field(default_factory=list)


c1 = Container([1, 2, 3])
c2 = copy.deepcopy(c1)
c2.items.append(4)
print(c1.items)  # [1, 2, 3] - оригинал не изменился


# 4. Frozen dataclass (неизменяемый)
@dataclass(frozen=True)
class FrozenPoint:
    x: int
    y: int


fp = FrozenPoint(1, 2)
try:
    fp.x = 10  # FrozenInstanceError!
except dataclasses.FrozenInstanceError as e:
    print(f"Ошибка: {e}")


# 5. Хешируемость (если unsafe_hash=True или frozen=True)
@dataclass(frozen=True)
class HashablePoint:
    x: int
    y: int


hp = HashablePoint(1, 2)
print(hash(hp))  # Работает!
s = {hp}
print(hp in s)  # True
```

## **Важные замечания:**

1. **Порядок полей**: Поля без значений по умолчанию должны идти **перед** полями со значениями по умолчанию.
2. **Наследование**: При наследовании порядок полей определяется MRO, что может привести к неожиданному порядку в
   `__init__`.
3. **default_factory**: Всегда используйте `default_factory` для изменяемых значений по умолчанию (списки, словари).
4. **Производительность**: `dataclass` компилируется в обычный класс, поэтому накладные расходы минимальны.
5. **Frozen vs Hashable**: `frozen=True` автоматически делает класс хешируемым, `unsafe_hash=True` позволяет хешировать
   изменяемые классы.
6. **Метаданные полей**: `field(metadata=...)` позволяет хранить дополнительную информацию о полях для рефлексии.

## **Ключевые выводы:**

1. **`dataclass` — автоматическая генерация методов** для классов-контейнеров данных.
2. **`@dataclass`** заменяет ручное написание `__init__`, `__repr__`, `__eq__`.
3. **`field()`** для сложных настроек полей (изменяемые default, исключение из init).
4. **`frozen=True`** создает неизменяемые классы с поддержкой хеширования.
5. **`asdict()`, `astuple()`, `replace()`** — утилиты для преобразования и копирования.
6. **`__post_init__()`** для постобработки после инициализации.
7. **Идеален для**: DTO, конфигураций, доменных моделей, иммутабельных структур данных.

[Содержание](/CONTENTS.md#содержание)

---

## re.Pattern & re.Match

`re.Pattern` и `re.Match` — это типы из модуля `re`, представляющие **скомпилированные регулярные выражения** и 
**результаты их поиска** соответственно. `Pattern` предоставляет интерфейс для поиска по строкам, а `Match` содержит
информацию о найденных совпадениях, включая группы, позиции и именованные захваты. Оба типа неизменяемы, оптимизированы
для повторного использования и поддерживают мощные операции с текстом.

## **Создание Pattern и Match**

```python
import re
from typing import Pattern, Match

# 1.1 Скомпилированный Pattern
pattern = re.compile(r'\d+')  # re.Pattern объект
print(type(pattern))  # <class 're.Pattern'>

# 1.2 Match через search()
match = pattern.search("abc123def")  # re.Match объект
print(type(match))  # <class 're.Match'>

# 1.3 Прямое создание без компиляции
match2 = re.search(r'\d+', "abc123def")  # Автоматически компилирует

# 1.4 С флагами
pattern_with_flags = re.compile(r'python', re.IGNORECASE | re.MULTILINE)

# 1.5 Именованные группы
pattern_named = re.compile(r'(?P<year>\d{4})-(?P<month>\d{2})-(?P<day>\d{2})')
match_named = pattern_named.search("2023-12-25")
```

## **Атрибуты типа re.Pattern**

`Pattern` имеет атрибуты для доступа к исходному шаблону и метаданным.

```python
pattern = re.compile(r'\w+@\w+\.\w+', re.IGNORECASE)

# Основные атрибуты
print(pattern.pattern)  # '\\w+@\\w+\\.\\w+' (исходный шаблон)
print(pattern.flags)  # 2 (re.IGNORECASE = 2)
print(pattern.groups)  # 0 (количество групп без имен)
print(pattern.groupindex)  # {} (именованные группы)

# Pattern с именованными группами
pattern_named = re.compile(r'(?P<year>\d{4})-(?P<month>\d{2})')
print(pattern_named.groups)  # 2
print(pattern_named.groupindex)  # {'year': 1, 'month': 2}

# Нет атрибутов типа real/numerator
try:
    print(pattern.real)
except AttributeError as e:
    print(f"Ошибка: {e}")  # 're.Pattern' object has no attribute 'real'
```

## **Методы типа re.Pattern**

`Pattern` предоставляет методы для поиска и модификации строк.

### **Методы поиска**

```python
pattern = re.compile(r'\d+')

# search() - первое совпадение в любой позиции
text = "abc123def456"
match1 = pattern.search(text)
print(match1)  # <re.Match object; span=(3, 6), match='123'>

# match() - совпадение только с начала строки
match2 = pattern.match(text)  # None (нет цифр в начале)
match3 = pattern.match("123abc")  # <re.Match object; span=(0, 3), match='123'>

# fullmatch() - полное совпадение всей строки
print(pattern.fullmatch("123"))  # <re.Match object; span=(0, 3), match='123'>
print(pattern.fullmatch("123abc"))  # None
```

### **Методы поиска всех совпадений**

```python
pattern = re.compile(r'\d+')
text = "123 abc 456 def 789"

# findall() - список всех совпадений
print(pattern.findall(text))  # ['123', '456', '789']

# findall() с группами
pattern_groups = re.compile(r'(\d+)([a-z]+)')
print(pattern_groups.findall(text))  # []

# finditer() - итератор по Match объектам
for match in pattern.finditer(text):
    print(f"Найдено: {match.group()} на позициях {match.start()}-{match.end()}")
# Найдено: 123 на позициях 0-3
# Найдено: 456 на позициях 8-11
# Найдено: 789 на позициях 16-19
```

### **Методы замены и разделения**

```python
pattern = re.compile(r'\s+')  # Пробельные символы

# split() - разделение строки
text = "a  b   c    d"
print(pattern.split(text))  # ['a', 'b', 'c', 'd']

# sub() - замена совпадений
email_pattern = re.compile(r'\b[\w.]+@[\w]+\.[a-z]{2,4}\b')
text = "Contact john@example.com and jane@domain.com"
print(email_pattern.sub("[EMAIL]", text))


# Contact [EMAIL] and [EMAIL]

# sub() с функцией замены
def replace_email(match):
    email = match.group()
    user, domain = email.split('@')
    return f"[{user[:2]}***@{domain}]"


print(email_pattern.sub(replace_email, text))
# Contact [jo***@example.com] and [ja***@domain.com]

# subn() - замена с подсчетом
new_text, count = email_pattern.subn("[EMAIL]", text)
print(f"Заменено {count} email адресов")  # Заменено 2 email адресов
```

## **Атрибуты и методы типа re.Match**

`Match` содержит информацию о найденном совпадении.

### **Основные атрибуты**

```python
pattern = re.compile(r'(\d{4})-(\d{2})-(\d{2})')
match = pattern.search("Date: 2023-12-25")

# Основная информация
print(match.string)  # 'Date: 2023-12-25' (исходная строка)
print(match.re)  # re.compile('(\\d{4})-(\\d{2})-(\\d{2})') (Pattern объект)
print(match.pos)  # 0 (начальная позиция поиска)
print(match.endpos)  # 19 (конечная позиция поиска)
print(match.lastindex)  # 3 (индекс последней захваченной группы)
print(match.lastgroup)  # None (имя последней группы, если есть)

# С позиционными группами
print(match.group())  # '2023-12-25' (все совпадение)
print(match.group(0))  # '2023-12-25' (тоже все совпадение)
print(match.group(1))  # '2023' (первая группа)
print(match.group(2))  # '12' (вторая группа)
print(match.group(3))  # '25' (третья группа)

# С именованными группами
pattern_named = re.compile(r'(?P<year>\d{4})-(?P<month>\d{2})-(?P<day>\d{2})')
match_named = pattern_named.search("Date: 2023-12-25")
print(match_named.group('year'))  # '2023'
print(match_named.group('month'))  # '12'
print(match_named.group('day'))  # '25'
```

### **Позиции совпадений**

```python
pattern = re.compile(r'(\d{4})-(\d{2})')
match = pattern.search("Date: 2023-12-25")

# Позиции групп
print(match.start())  # 6 (начало всего совпадения)
print(match.end())  # 13 (конец всего совпадения)
print(match.span())  # (6, 13) (кортеж start, end)

# Позиции отдельных групп
print(match.start(1))  # 6 (начало первой группы)
print(match.end(1))  # 10 (конец первой группы)
print(match.span(1))  # (6, 10)

print(match.start(2))  # 11 (начало второй группы)
print(match.end(2))  # 13 (конец второй группы)
print(match.span(2))  # (11, 13)

# Именованные группы
print(match_named.start('year'))  # 6
print(match_named.span('month'))  # (11, 13)
```

### **Группы и словари**

```python
pattern = re.compile(r'(?P<year>\d{4})-(?P<month>\d{2})-(?P<day>\d{2})')
match = pattern.search("2023-12-25")

# groups() - кортеж всех групп
print(match.groups())  # ('2023', '12', '25')

# groupdict() - словарь именованных групп
print(match.groupdict())  # {'year': '2023', 'month': '12', 'day': '25'}

# expand() - форматирование с заменой групп
print(match.expand(r"Year: \1, Month: \2"))  # Year: 2023, Month: 12
print(match.expand(r"Month-Day: \g<month>-\g<day>"))  # Month-Day: 12-25
```

## **Поддерживаемые операции**

```python
pattern = re.compile(r'\d+')
match = pattern.search("abc123def")

# 5.1 Проверка наличия совпадения
if match:
    print("Совпадение найдено")
else:
    print("Совпадений нет")

# 5.2 Доступ к совпадению
print(match[0])  # '123' (аналог group(0))
print(match[1])  # IndexError если нет групп

# 5.3 Итерация по группам
for i in range(len(match.groups()) + 1):
    print(f"Группа {i}: {match.group(i)}")

# 5.4 Логическое значение
print(bool(match))  # True (если совпадение есть)
print(bool(None))  # False (если совпадений нет)

# 5.5 Неподдерживаемые: арифметика
try:
    match + match
except TypeError as e:
    print(f"Не поддерживается: {e}")
```

## **Операции сравнения**

```python
pattern1 = re.compile(r'\d+')
pattern2 = re.compile(r'\d+')
pattern3 = re.compile(r'[0-9]+')

match1 = pattern1.search("123")
match2 = pattern2.search("123")
match3 = pattern3.search("123")

# Сравнение Pattern объектов
print(pattern1 == pattern2)  # False (разные объекты)
print(pattern1.pattern == pattern2.pattern)  # True (одинаковые шаблоны)

# Сравнение Match объектов
print(match1 == match2)  # False (разные объекты)
print(match1.group() == match2.group())  # True (одинаковое содержимое)

# Сравнение с другими типами
print(match1 == "123")  # False
print(match1.group() == "123")  # True
```

## **Преобразование в другие типы и форматирование**

```python
pattern = re.compile(r'(\d{4})-(\d{2})-(\d{2})')
match = pattern.search("2023-12-25")

# 7.1 В строку
print(str(match))  # "<re.Match object; span=(0, 10), match='2023-12-25'>"
print(repr(match))  # "<re.Match object; span=(0, 10), match='2023-12-25'>"

# 7.2 Преобразование групп
print(int(match.group(1)))  # 2023
print(list(match.groups()))  # ['2023', '12', '25']

# 7.3 Форматирование результатов
print(f"Дата: {match.group(1)}-{match.group(2)}-{match.group(3)}")
# Дата: 2023-12-25

# 7.4 Сериализация
import json

match_data = {
    'match': match.group(),
    'groups': match.groups(),
    'span': match.span(),
    'groupdict': match.groupdict()
}
print(json.dumps(match_data))
# {"match": "2023-12-25", "groups": ["2023", "12", "25"], "span": [0, 10], "groupdict": {"year": "2023", "month": "12", "day": "25"}}
```

## **Важные особенности**

```python
import re

# 1. Кэширование скомпилированных шаблонов
# re.compile() кэширует последние 512 шаблонов
re.search(r'\d+', "123")  # Автоматическая компиляция
re.search(r'\d+', "456")  # Использует кэш

# 2. Небезопасные шаблоны (ReDoS)
# Осторожно с вложенными квантификаторами!
pattern = re.compile(r'(a+)+b')
# pattern.search('a' * 30 + 'c')  # Может зависнуть!

# 3. Флаги компиляции
pattern_multiline = re.compile(r'^line', re.MULTILINE)
text = "line1\nline2\nline3"
print(pattern_multiline.findall(text))  # ['line', 'line', 'line']

# 4. Юникод и локаль
pattern_unicode = re.compile(r'\w+', re.UNICODE)
print(pattern_unicode.findall("привет world"))  # ['привет', 'world']

# 5. Отладка шаблонов
pattern_debug = re.compile(r'(\d{4})-(\d{2})', re.VERBOSE)
# re.VERBOSE позволяет добавлять комментарии и пробелы
pattern_debug = re.compile(r"""
    (\d{4})   # Год
    -         # Разделитель
    (\d{2})   # Месяц
""", re.VERBOSE)

# 6. Разница между search() и match()
print(re.match(r'\d+', "abc123"))  # None
print(re.search(r'\d+', "abc123"))  # <re.Match object; span=(3, 6), match='123'>
```

## **Важные замечания:**

1. **Компиляция vs прямой вызов**: `re.compile()` для повторного использования, `re.search()` для однократных операций.
2. **Флаги компиляции**: `re.IGNORECASE`, `re.MULTILINE`, `re.DOTALL`, `re.UNICODE`, `re.VERBOSE` меняют поведение
   шаблона.
3. **Производительность**: Скомпилированные шаблоны быстрее, особенно при многократном использовании.
4. **Безопасность**: Избегайте шаблонов с экспоненциальной сложностью (ReDoS-атаки).
5. **Юникод**: Используйте `re.UNICODE` для корректной работы с не-ASCII символами (по умолчанию в Python 3).
6. **Именованные группы**: `(?P<name>...)` делают код более читаемым и устойчивым к изменениям шаблона.

## **Ключевые выводы:**

1. **`re.Pattern` — скомпилированное регулярное выражение**, `re.Match` — результат поиска.
2. **`compile()`** для оптимизации повторных поисков, **`search()/match()/fullmatch()`** для поиска.
3. **`group()/groups()/groupdict()`** для доступа к захваченным группам, **`start()/end()/span()`** для позиций.
4. **`findall()/finditer()`** для поиска всех совпадений, **`sub()/split()`** для замены и разделения.
5. **Флаги (`re.IGNORECASE` и др.)** расширяют возможности шаблонов.
6. **`(?P<name>...)`** для именованных групп — лучшая практика для сложных шаблонов.
7. **Идеален для**: валидации данных, парсинга логов, извлечения информации, реформатирования текста.

[Содержание](/CONTENTS.md#содержание)

---

## **Senior Level**

В CPython **все типы данных** наследуют от базовой структуры `PyObject`, которая содержит refcount и указатель на тип.
Каждый тип описывается массивом **слотов** в `PyTypeObject`.
`Include/object.h`, `Include/cpython/object.h`

## Базовый объект (PyObject)

```c
typedef struct _object {
    _PyObject_HEAD_EXTRA       // Платформо-зависимые поля для отладки (PyDebug)
    Py_ssize_t ob_refcnt;      // Счётчик ссылок - при 0 вызывается tp_dealloc
    struct _typeobject *ob_type; // Указатель на PyTypeObject описывающий тип
} PyObject;
```

### Общее

Это **фундаментальная C-структура из ядра CPython**, которая представляет любой объект в Python на самом низком уровне.
Каждый объект в Python — будь то число, строка, список или ваш собственный класс — в памяти выглядит как экземпляр этой
структуры. Она содержит три ключевых компонента: счётчик ссылок (для управления памятью), указатель на тип объекта и
служебные поля для отладки.

### Описание

**Отладочная информация:**

- `_PyObject_HEAD_EXTRA` — это не одно поле, а макрос, который раскрывается в дополнительные платформо-зависимые поля *
  *только в отладочных сборках Python** (когда интерпретатор скомпилирован с флагом `--with-pydebug`). В обычных сборках
  это ничего не добавляет. Используется для отслеживания проблем с памятью.

**Счётчик ссылок:**

- `ob_refcnt` — целое число, которое **считает, сколько переменных или структур указывают на этот объект**. Когда
  счётчик становится 0, Python вызывает функцию `tp_dealloc` (из `PyTypeObject`) для освобождения памяти. Это основной
  механизм управления памятью в CPython (garbage collection через reference counting).

**Указатель на тип:**

- `ob_type` — указатель на структуру `PyTypeObject`, которая описывает тип этого объекта. Это то же самое, что
  возвращает встроенная функция `type(obj)` в Python. Через этот указатель интерпретатор узнаёт, как работать с
  объектом: какие операции он поддерживает, как получить его атрибуты, как его удалить.

### Уточнения

- **Reference counting — это не threading-safe по умолчанию**: В CPython счётчик ссылок работает за счёт глобальной
  блокировки (GIL — Global Interpreter Lock), поэтому многопоточность имеет ограничения. В Python 3.13+ появился режим
  free-threaded, где это иначе.

- **Минимальный размер**: Даже самый простой объект в памяти занимает как минимум столько байт, сколько занимает
  `PyObject` (обычно 16-24 байта на 64-битной системе в зависимости от конфигурации). Это почему в Python много памяти
  уходит на служебную информацию, даже для маленьких объектов.

- **ob_refcnt увеличивается и уменьшается постоянно**: Каждый раз, когда вы присваиваете объект переменной, счётчик
  увеличивается на 1. Когда переменная удаляется или переназначается, счётчик уменьшается на 1. Вы можете проверить это
  функцией `sys.getrefcount(obj)`.

- **Связь с PyTypeObject**: `PyObject` и `PyTypeObject` работают вместе: `PyObject` — это экземпляр, `PyTypeObject` —
  это описание класса. Например, объект `5` имеет `ob_type`, указывающий на `PyTypeObject` для типа `int`.

[Содержание](/CONTENTS.md#содержание)

---

## "Паспорт" каждого типа (PyTypeObject)

```c
typedef struct _typeobject {
    PyVarObject ob_base;       // Тип сам является объектом (можно наследовать)
    
    const char *tp_name;       // Имя типа ("list", "dict", "int")
    Py_ssize_t tp_basicsize;   // Размер в байтах без переменной части
    Py_ssize_t tp_itemsize;    // Размер одного элемента переменной части
    
    destructor tp_dealloc;     // Функция уничтожения (list_dealloc)
    printfunc tp_print;        // Для print()
    reprfunc tp_repr;          // Для repr()
    
    // Протокол чисел
    PyNumberMethods *tp_as_number;  // nb_add, nb_sub, nb_multiply...
    
    // Протокол последовательностей
    PySequenceMethods *tp_as_sequence; // sq_item, sq_ass_slice...
    
    // Протокол маппингов
    PyMappingMethods *tp_as_mapping;  // mp_subscript, mp_ass_subscript
    
    // Поиск атрибутов
    getattrofunc tp_getattro;      // obj.attr
    setattrofunc tp_setattro;      // obj.attr = value
    
    // Дескрипторы (property, method)
    descrgetfunc tp_descr_get;     // __get__
    descrsetfunc tp_descr_set;     // __set__
    
    Py_ssize_t tp_dictoffset;      // Смещение __dict__ (или -1)
    Py_ssize_t tp_weaklistoffset;  // Смещение weakref списка
    
    PyObject *tp_mro;              // Method Resolution Order (tuple типов)
    PyObject *tp_cache;            // Кеш атрибутов (free-threaded)
    unsigned int tp_subclasses;    // Количество живых подклассов
    
    PyObject *tp_dict;             // __dict__ класса
    // ... 100+ слотов
} PyTypeObject;
```

### Общее

Это **внутренняя C-структура из ядра Python**, которая определяет тип объекта на самом низком уровне. Каждый класс и
встроенный тип в Python (list, dict, int, str) представлены в памяти именно такой структурой. Она содержит всю
метаинформацию о типе: имя, размер, методы обработки операций (сложение, индексирование), управление атрибутами и многое
другое. Это фундамент, на котором работает весь механизм объектно-ориентированного программирования в Python.

### Описание

**Базовая информация:**

- `ob_base` — каждый тип сам является объектом (в Python всё является объектом, включая сами типы)
- `tp_name` — строка с именем типа, которая выводится, например, при вызове `type(obj).__name__`
- `tp_basicsize` и `tp_itemsize` — память: базовый размер структуры плюс размер на один элемент для переменных
  контейнеров (список с 10 элементами занимает `basicsize + 10 * itemsize` байт)

**Управление жизненным циклом:**

- `tp_dealloc` — функция уничтожения объекта, вызывается при удалении (соответствует `__del__` на уровне C)
- `tp_print`, `tp_repr` — старый способ для вывода и представления (в Python 3.9+ `tp_print` устарел)

**Протоколы операций** — указатели на структуры с функциями для конкретных операций:

- `tp_as_number` — арифметические операции: `__add__`, `__sub__`, `__mul__` и т.д.
- `tp_as_sequence` — операции последовательностей: `obj[i]`, `obj[i:j]`, `len(obj)`
- `tp_as_mapping` — операции словарей: `obj[key]`, `len(obj)` для словарей

**Управление атрибутами:**

- `tp_getattro` / `tp_setattro` — низкоуровневые функции для доступа к атрибутам (`obj.attr` и `obj.attr = value`)

**Дескрипторы:**

- `tp_descr_get` / `tp_descr_set` — механизм дескрипторов, на котором работают `property`, методы класса, статические
  методы

**Служебная информация:**

- `tp_dictoffset` — смещение в памяти, где находится `__dict__` объекта (словарь атрибутов экземпляра)
- `tp_mro` — кортеж типов в порядке разрешения методов (Method Resolution Order) — порядок поиска атрибутов в иерархии
  наследования
- `tp_dict` — `__dict__` самого класса, где хранятся методы и атрибуты класса

### Уточнения

1. **C-структура, но видна из Python**: Вы не работаете с этой структурой напрямую в коде на Python, но когда пишете
   класс с методами `__add__`, `__getitem__` или `__get__`, Python переводит эти методы в соответствующие C-функции в
   этой структуре для производительности.

2. **tp_print устарел**: В Python 3.9+ поле `tp_print` более не используется. Вывод объектов управляется через `tp_repr`
   и `tp_str`.

3. **100+ слотов**: Код содержит комментарий «... 100+ слотов», что означает в реальной структуре есть множество
   дополнительных полей для специальных случаев, обработки исключений, сериализации и прочего. Здесь показаны только
   самые важные.

4. **Зачем это нужно знать**: Если вы пишете расширения на C для Python или оптимизируете критичные места, понимание
   этой структуры помогает правильно определить поведение типа. Для обычного Python-кода достаточно знать, что она
   существует.

5. **Про free-threaded**: Поле `tp_cache` относится к улучшениям для параллельного выполнения (free-threaded Python),
   актуально для Python 3.13+, для Python 3.9-3.12 на него можно не обращать внимание.

[Содержание](/CONTENTS.md#содержание)

---

## Инициализация типов: PyType_Ready

```c
int PyType_Ready(PyTypeObject *type) {
    if (type->tp_flags & Py_TPFLAGS_READY)  // Уже инициализирован
        return 0;
    
    // Наследуем слоты от базовых классов
    if (type->tp_bases) {
        Py_ssize_t i, nbase = PyTuple_GET_SIZE(type->tp_bases);
        for (i = 0; i < nbase; i++) {
            PyTypeObject *base = (PyTypeObject *)PyTuple_GET_ITEM(type->tp_bases, i);
            if (PyType_Ready(base) < 0)
                return -1;
            
            // Наследуем слоты (tp_as_number, tp_as_sequence...)
            inherit_special(base, type);
        }
    }
    
    // Вычисляем MRO
    if (mro_internal(type) < 0)
        return -1;
        
    // Инициализируем tp_dict
    if (type->tp_dict == NULL) {
        if (PyType_AllocDict(type) < 0)
            return -1;
    }
    
    type->tp_flags |= Py_TPFLAGS_READY;  // Отмечаем готовым
    return 0;
}
```

### Общее

`PyType_Ready` — это ключевая функция CPython, которая **финализирует тип** (класс) перед его использованием.  
Вызывается автоматически при определении класса (`class MyClass:`) или импорте модуля.  
Подготавливает все слоты, наследование и MRO (порядок разрешения методов).

### Описание

- `if (type->tp_flags & Py_TPFLAGS_READY)` — проверяет, уже ли тип инициализирован (избегает повторной работы).

- `if (type->tp_bases)` — цикл по базовым классам:
    - Рекурсивно вызывает `PyType_Ready(base)` для каждого родителя.
    - `inherit_special(base, type)` — копирует слоты (`tp_as_number`, `tp_as_sequence` и др.) из базового класса.

- `mro_internal(type)` — вычисляет **Method Resolution Order** (порядок поиска методов при множественном наследовании).

- `PyType_AllocDict(type)` — создаёт `__dict__` класса, если его нет (словарь атрибутов и методов).

- `type->tp_flags |= Py_TPFLAGS_READY` — помечает тип как полностью готовый к использованию.

### Уточнения

- **Рекурсивная инициализация** гарантирует, что все предки готовы перед дочерним классом.
- `inherit_special()` заполняет протоколы операций (арифметика, индексирование) из ближайшего базового класса.
- MRO критичен для **множественного наследования** — определяет порядок `super()`.
- Вызывается **один раз** за тип — повторные вызовы игнорируются (`Py_TPFLAGS_READY`).
- В Python 3.9+ улучшена производительность за счёт кэширования MRO и оптимизации слотов.

[Содержание](/CONTENTS.md#содержание)

---

## База коллекций (PyVarObject)

```c
typedef struct {
    PyObject ob_base;          // Встроенный PyObject (refcnt + type)
    Py_ssize_t ob_size;        // Количество элементов (длина строки/списка)
} PyVarObject;
```

### Общее

- Это структура данных из исходного кода CPython (стандартной реализации Python, написанной на C).
- Она представляет собой расширенный заголовок для объектов, которые имеют переменную длину (количество элементов).
- Такие типы данных, как списки (`list`), кортежи (`tuple`) и строки (`str`), в памяти начинаются именно с этой
  структуры.

### Описание

- `typedef struct { ... } PyVarObject;` — конструкция языка C, создающая новый тип данных `PyVarObject`, описывающий,
  как объект лежит в оперативной памяти.
- `PyObject ob_base` — поле, внедряющее базовый объект внутрь. Оно содержит два критически важных элемента: счетчик
  ссылок (для сборщика мусора) и указатель на тип данных (класс объекта).
- `Py_ssize_t ob_size` — поле, хранящее количество элементов в объекте (значение, которое возвращает функция `len()`).
  Для списка это число слотов, для строки — количество символов.

### Уточнения

- Благодаря тому, что `ob_base` стоит первым, любой указатель на `PyVarObject` можно безопасно рассматривать как
  указатель на `PyObject`. Это позволяет функциям интерпретатора работать с любыми объектами (фиксированными и
  переменными) одинаково.
- Целые числа (`int`) в Python 3 также реализованы через `PyVarObject`, так как они поддерживают длинную арифметику и
  могут состоять из произвольного количества «цифр» в памяти.
- Доступ к длине объекта происходит за время O(1), так как интерпретатор просто читает значение `ob_size`, не
  пересчитывая элементы каждый раз.

[Содержание](/CONTENTS.md#содержание)

---

## int (PyLongObject)

```c
typedef uint32_t digit;        // 30-битная цифра (2^30 = ~1e9)

typedef struct _longobject {
    PyObject_VAR_HEAD          // PyObject + ob_size (кол-во цифр)
    digit ob_digit[1];         // Массив цифр (размер ob_size)
} PyLongObject;
```

### Общее

`PyLongObject` — это внутренняя структура CPython, представляющая **все целые числа (`int`)** в Python.  
Даже обычное число `42` или огромный `10**1000` — это один и тот же тип объекта `PyLongObject`.  
Главная идея — хранить число не как одно значение, а как **массив “цифр”** в 30 бит, что позволяет поддерживать целые
числа любой длины (длинная арифметика).

### Описание

- `typedef uint32_t digit;` — определяет тип для хранения одной “цифры” числа. Каждая цифра занимает 32 бита, но реально
  используются только 30. Это оставляет запас для удобных вычислений без переполнений.  
  Например, большое число хранится как массив таких 30-битных частей.

- `PyObject_VAR_HEAD` — макрос, который вставляет в структуру стандартные поля из `PyVarObject`:
    - `ob_refcnt` — счётчик ссылок (для управления памятью).
    - `ob_type` — указатель на тип (`int`).
    - `ob_size` — количество “цифр” (`digit`) в числе и знак (если отрицательное, `ob_size` < 0).

- `digit ob_digit[1];` — первый элемент массива, в котором реально хранятся “цифры” числа.  
  Несмотря на размер `[1]`, массив в памяти может быть длиннее — его конечная длина задается `ob_size`.  
  Например, число `12345678901234567890` будет храниться как несколько элементов в этом массиве.

### Уточнения

- Такой подход позволяет Python-числам быть **неограниченно большими**, в отличие от фиксированной разрядности в C (
  `int`, `long`).
- Младшая цифра хранится первой (наименьший порядок) — это **младший разрядный порядок (little-endian)**.
- “30 бит на цифру” выбрано, чтобы при арифметических операциях с двумя цифрами не происходило переполнения 32‑битного
  регистра.
- Для маленьких чисел Python использует **кэш** заранее созданных объектов (от -5 до 256), чтобы не пересоздавать их в
  памяти.

[Содержание](/CONTENTS.md#содержание)

---

## Создание int (PyLongObject)

```c
PyObject *_PyLong_New(Py_ssize_t size) {
    PyLongObject *result;
    
    size = Py_ABS(size);  // Берем модуль размера
    
    // Выделяем память под заголовок + массив цифр
    result = PyObject_MALLOC(sizeof(PyLongObject) + 
                             (size-1) * sizeof(digit));
    if (!result) {
        return PyErr_NoMemory();
    }
    
    // Инициализируем как PyObject
    PyObject_INIT(result, &PyLong_Type);
    Py_SET_SIZE(result, size);  // Устанавливаем ob_size
    result->ob_digit[0] = 0;    // Нулевая цифра
    
    return (PyObject *)result;
}
```

### Общее

`_PyLong_New` — это внутренняя функция CPython, которая **создаёт объект целого числа (`int`) в памяти**.  
Она не вычисляет значение, а лишь выделяет и подготавливает структуру `PyLongObject`, чтобы Python мог потом записать
туда цифры числа.  
Используется внутри интерпретатора при создании или копировании больших чисел.

### Описание

- `PyObject *_PyLong_New(Py_ssize_t size)` — функция принимает количество “цифр” (`digit`), которые нужно
  зарезервировать для будущего числа.  
  Если `size` отрицательное, берётся модуль (`Py_ABS(size)`), т.к. знак хранится отдельно через `ob_size`.

- `PyObject_MALLOC(sizeof(PyLongObject) + (size - 1) * sizeof(digit))` — выделяет память под заголовок (`PyLongObject`)
  и нужное количество “цифр”.  
  В структуре уже есть один элемент `ob_digit[1]`, поэтому добавляется `(size - 1)`.

- `if (!result) return PyErr_NoMemory();` — если память не выделилась, Python выбрасывает ошибку `MemoryError`.

- `PyObject_INIT(result, &PyLong_Type);` — макрос инициализирует базовую часть как `PyObject`, указывая, что это объект
  типа `int`.

- `Py_SET_SIZE(result, size);` — записывает длину числа (количество “цифр”) в `ob_size`.

- `result->ob_digit[0] = 0;` — устанавливает первую “цифру” числа равной нулю (инициализация).

- `return (PyObject *)result;` — возвращает созданный объект, приведённый к общему типу `PyObject *`, как это принято
  для всех Python-объектов.

### Уточнения

- Эта функция только выделяет память — **значение числа формируется позже**, другими функциями (например, при парсинге
  литерала или в арифметике).
- Функция — **внутренняя** (начинается с подчёркивания) и не предназначена для использования из Python‑C API напрямую.
- Используется стандартный **аллокатор Python** (`PyObject_MALLOC`), чтобы система могла отслеживать память GC и
  профилировать выделения.
- Знак числа не устанавливается здесь — он задаётся отдельно через `Py_SET_SIZE()` (положительный или отрицательный).

[Содержание](/CONTENTS.md#содержание)

---

## float (PyFloatObject)

```c
typedef struct {
    PyObject ob_base;      // PyObject (refcnt + type)
    double ob_fval;        // IEEE 754 double-precision значение
} PyFloatObject;
```

### Общее

`PyFloatObject` — это внутренняя структура CPython, представляющая **числа с плавающей запятой (`float`)**.  
Любое число вроде `3.14` или `math.pi` в памяти хранится именно так.  
Простая структура — всего **одно поле со значением** в формате IEEE 754 double.

### Описание

- `PyObject ob_base` — стандартный заголовок:
    - `ob_refcnt` — счётчик ссылок для управления памятью.
    - `ob_type` — указатель на `PyFloat_Type` (тип `float`).

- `double ob_fval` — **реальное значение** числа в формате double (64 бита, ~15 значащих цифр).  
  Это стандарт IEEE 754, который используется во всех современных языках программирования.

### Уточнения

- **Фиксированный размер** — всегда 24 байта на 64-битной системе (PyObject + double).
- **Кэш маленьких float** отсутствует (в отличие от `int`), каждый `3.14` создаёт новый объект.
- **NaN, Inf, -Inf** поддерживаются стандартно через IEEE 754 (`float('nan')`, `float('inf')`).
- Арифметика (`+`, `-`, `*`, `/`) реализована через `tp_as_number` в `PyFloat_Type`.

[Содержание](/CONTENTS.md#содержание)

---

## Создание float (PyFloatObject)

```c
PyObject *PyFloat_FromDouble(double fval) {
    PyFloatObject *op = _PyFloat_New();  // Выделяем PyFloatObject
    if (!op)
        return NULL;
    op->ob_fval = fval;                  // Записываем значение
    return (PyObject *)op;
}

static PyObject *_PyFloat_New(void) {
    PyFloatObject *op = (PyFloatObject*)PyObject_MALLOC(sizeof(PyFloatObject));
    if (!op) {
        return PyErr_NoMemory();
    }
    PyObject_INIT(op, &PyFloat_Type);    // Инициализируем как float
    return (PyObject *)op;
}
```

### Общее

`PyFloat_FromDouble` — основная функция CPython для **создания объекта `float` из C double**.  
Вызывается интерпретатором при литералах `3.14`, вызовах `float()`, математических операциях.  
Создаёт готовый Python-объект с заданным значением.

### Описание

- `PyFloatObject *op = _PyFloat_New();` — вызывает внутреннюю функцию выделения памяти под `PyFloatObject`.
- `_PyFloat_New()` — выделяет память через `PyObject_MALLOC` и инициализирует:
    - `PyObject_INIT(op, &PyFloat_Type)` — устанавливает `ob_refcnt=1`, `ob_type=&PyFloat_Type`.
- `op->ob_fval = fval;` — записывает **значение double** в поле объекта.
- `return (PyObject *)op;` — возвращает как универсальный `PyObject *`.

### Уточнения

- **Нет кэша** для float (в отличие от малых int) — каждый `3.14` создаёт новый объект.
- `PyObject_MALLOC` — специальный аллокатор Python для отслеживания GC.
- Функция **thread-safe** благодаря GIL (Python 3.9+).
- Используется в байткоде (`BINARY_OP` для `+`, `-`, `*`, `/` с float).

[Содержание](/CONTENTS.md#содержание)

---

## decimal (PyDecimalObject)

```c
// decimal.Decimal - пользовательский класс Python, НЕ встроенный тип CPython
// Реализован в Objects/decimalmodule.c как PyTypeObject с C-ускорением

typedef struct {
    PyObject ob_base;           // Стандартный заголовок
    PyObject *digits;           // Массив цифр (список int)
    PyObject *exp;              // Показатель степени 10 (int)
    PyObject *prec;             // Точность (int)
    PyObject *rounding;         // Режим округления
    PyObject *context;          // Контекст вычислений
} PyDecimalObject;
```

### Общее

`decimal.Decimal` — **НЕ встроенный тип CPython**, а **модульный класс** из `decimal` с C-реализацией.  
Предоставляет **десятичную арифметику фиксированной точности** (банковские расчёты, финансы).  
В отличие от `float`, хранит числа как массив цифр + показатель степени.

### Описание

- `PyObject ob_base` — стандартный заголовок (`refcnt`, `type=PyDecimal_Type`).
- `PyObject *digits` — **массив цифр** в десятичной системе (список `int`).
- `PyObject *exp` — **показатель степени** (например, `123E-2` = 1.23).
- `PyObject *prec` — **точность** (максимум цифр для операций).
- `PyObject *rounding` — режим округления (`ROUND_HALF_UP` и др.).
- `PyObject *context` — глобальный контекст вычислений (`getcontext()`).

### Уточнения

- **Полностью управляемый** через `decimal.getcontext()` — точность, округление, исключения.
- Арифметика **десятичная**, не двоичная (никаких ошибок округления как в `float`).
- В CPython ускорен **C-кодом** (`Objects/decimalmodule.c`), но это всё равно Python-класс.
- Используется для **финансовых расчётов** (`0.1 + 0.2 == 0.3` работает правильно).

[Содержание](/CONTENTS.md#содержание)

---

## Создание decimal (PyDecimalObject)

```c
PyObject *PyDecimal_FromString(PyObject *s) {
    PyDecimalObject *result = (PyDecimalObject *)PyObject_MALLOC(sizeof(PyDecimalObject));
    if (!result)
        return PyErr_NoMemory();
    
    PyObject_INIT(result, &PyDecimal_Type);
    
    // Парсим строку в цифры + экспоненту
    result->digits = parse_decimal_digits(s);
    result->exp = parse_exponent(s);
    result->prec = getcontext_prec();
    result->rounding = getcontext_rounding();
    result->context = getcontext_ref();
    
    return (PyObject *)result;
}
```

### Общее

Создание `PyDecimalObject` происходит через **функции модуля `decimal`** (например, `decimal.Decimal('1.23')`).  
Парсит строку в **десятичное представление** (цифры + экспонента) и связывает с глобальным контекстом точности.  
НЕ использует стандартный `_PyXXX_New()` — это модульная логика.

### Описание

- `PyObject_MALLOC(sizeof(PyDecimalObject))` — выделяет память под структуру.
- `PyObject_INIT(result, &PyDecimal_Type)` — инициализирует как `decimal.Decimal`.
- `result->digits = parse_decimal_digits(s)` — **разбирает цифры** из строки (`['1', '2', '3']`).
- `result->exp = parse_exponent(s)` — извлекает **экспоненту** (например, `E-2`).
- `result->prec = getcontext_prec()` — берёт **точность** из `decimal.getcontext()`.
- `result->rounding/context` — копирует настройки контекста округления и вычислений.

### Уточнения

- **Парсинг строковый** — `Decimal('1.23')` разбирается посимвольно, без потери точности.
- **Глобальный контекст** (`getcontext()`) определяет поведение всех операций.
- Создание **медленнее float/int** из-за парсинга и инициализации Python-объектов (`digits`, `exp`).
- В CPython 3.9+ **ускорено C-парсингом**, но остаётся модульным (не встроенный тип).

[Содержание](/CONTENTS.md#содержание)

---

## str (PyUnicodeObject)

```c
typedef struct {
    PyObject_VAR_HEAD       // PyObject + ob_size (длина в символах)
    Py_UCS4 *ob_sval;       // Массив Unicode символов (UCS-4)
    // Или Py_UCS2 *ob_sval для компактных строк
    // Или char *ob_sval для latin-1 строк
} PyUnicodeObject;
```

### Общее

`PyUnicodeObject` — внутренняя структура CPython для **строк (`str`)**.  
Любая строка `s = "hello"` или `s = "привет"` хранится именно так.  
Поддерживает **три формата** хранения (compact, 1-byte, 2-byte, 4-byte) для экономии памяти.

### Описание

- `PyObject_VAR_HEAD` — стандартный заголовок:
    - `ob_refcnt` — счётчик ссылок.
    - `ob_type` — `&PyUnicode_Type`.
    - `ob_size` — **длина строки** в символах (не байтах).

- `Py_UCS4 *ob_sval` — **массив символов Unicode**:
    - `Py_UCS4` (32-bit) — полный Unicode (эмодзи, редкие символы).
    - `Py_UCS2` (16-bit) — BMP символы (большинство текста).
    - `char` (8-bit) — только ASCII/Latin-1.  
      Формат выбирается автоматически по содержимому.

### Уточнения

- **Автоматическая компрессия** — короткие ASCII строки занимают минимум памяти.
- **Interning** — одинаковые строки (`"hello"`) кэшируются как синглтоны (`sys.intern()`).
- **Immutable** — строки неизменяемы, операции `+`, `replace()` создают новые объекты.
- В Python 3.9+ улучшена **кодировка** (PEEPHOLER оптимизирует конкатенацию строк).

[Содержание](/CONTENTS.md#содержание)

---

# Создание str (PyUnicodeObject)

```c
// Include/cpython/unicodeobject.h (Python 3.9+)
typedef struct {
    PyObject_VAR_HEAD
    Py_ssize_t length;      // Длина строки в символах (obsize)
    Py_hash_t hash;         // Кэшированный хэш (0 = не вычислен)
    struct {
        // Компактные Unicode варианты (Python 3.9+)
        Py_UCS1 *str;       // 1-byte ASCII/Latin-1
        Py_UCS2 *str2;      // 2-byte BMP
        Py_UCS4 *str4;      // 4-byte полный Unicode
    } data;
} PyUnicodeObject;

// Создание строки из C-строки (ASCII/Latin-1)
PyObject* PyUnicode_FromString(const char *u) {
    return PyUnicode_DecodeUTF8(u, strlen(u), NULL);
}

// Внутренняя функция создания (упрощённо)
static PyObject* unicode_new(Py_ssize_t length, int kind) {
    PyUnicodeObject *u;
    Py_ssize_t size;
    
    size = length * PyUnicode_KIND_SIZE(kind) + sizeof(PyUnicodeObject);
    u = PyObject_MALLOC(size);
    if (!u) return PyErr_NoMemory();
    
    PyObject_INIT_VAR(u, &PyUnicode_Type, length);
    u->hash = -1;  // Хэш не вычислен
    u->data.any = PyUnicode_DATA(u);  // Указатель на данные
    
    return (PyObject*)u;
}
```

### Общее

PyUnicodeObject — это C-структура CPython для хранения строк str в Python 3.9+. Использует **компактное представление
** (compact unicode): 1/2/4 байта на символ в зависимости от диапазона Unicode. Создание происходит через
PyUnicode_FromString() или внутренние функции вроде unicode_new().

### Описание

- **PyObject_VAR_HEAD**: Наследует refcnt, obtype (PyUnicode_Type), obsize (длина в символах).
- **length**: Количество символов (не байт!).
- **hash**: Кэшированный хэш для быстрого dict/set (-1 = не вычислен).
- **data.any**: Умный union — ASCII (1байт), BMP (2байт), полный Unicode (4байт).
- **unicode_new()**: Выделяет память под PyUnicodeObject + данные, инициализирует через PyObject_INIT_VAR.

### Уточнения

- **Компактность Python 3.9+**: "hello" → 1-byte (Py_UCS1), "привет" → 2-byte (Py_UCS2).
- **PyUnicode_FromString()**: Автоматически выбирает UTF-8 → PyUnicodeObject.
- **Interning**: sys.intern("hello") сохраняет один PyUnicodeObject для одинаковых строк.
- **Immutable**: После создания str нельзя изменить (no setattr!).
- **Memory layout**: PyObject (24байт) + данные сразу следом (O(1) доступ).

[Содержание](/CONTENTS.md#содержание)

---

## dict (PyDictObject)

```c
typedef struct {
    Py_ssize_t ma_used;        // Кол-во ключей (не слотов!)
    uint64_t ma_version_tag;   // Версия для итераторов
    PyDictKeysObject *ma_keys; // Общие ключи
    PyObject **ma_values;      // Массив значений
} PyDictObject;
```

### Общее

`PyDictObject` — это внутренняя структура CPython, которая описывает объект **словаря (`dict`)**.  
Каждый словарь в Python — это хеш-таблица, оптимизированная для быстрого доступа по ключу (`dict[key]`).  
Эта структура управляет ключами, значениями и вспомогательными данными (например, версией для отслеживания изменений при
итерации).

### Описание

- `Py_ssize_t ma_used` — количество **реальных пар ключ–значение**, которые сейчас находятся в словаре.  
  Это не количество выделенных слотов в таблице, а именно количество активных записей, то есть то, что возвращает
  `len(dict)`.

- `uint64_t ma_version_tag` — номер версии словаря.  
  Он увеличивается при каждом изменении (добавление, удаление, обновление).  
  Используется итераторами и кэшем атрибутов, чтобы понимать, что структура изменилась.

- `PyDictKeysObject *ma_keys` — указатель на структуру, содержащую **ключи и хеш-таблицу**.  
  Эта часть может быть общей для нескольких словарей (например, у экземпляров класса с одинаковыми атрибутами).
  PyDictKeysObject содержит массив записей (PyDictKeyEntry), где хранятся:
    - сам ключ (PyObject *key)
    - хэш этого ключа (целое число, рассчитанное при вставке)
    - индекс (или прямое значение), указывающий на место в ma_values.
    - хэш-индекс — таблицу разных размеров, по которой Python быстрo находит нужный элемент по хэшу. Это и есть
      внутренняя хэш-таблица, которая делает словари такими быстрыми при поиске.

- `PyObject **ma_values` — массив указателей на **значения**, соответствующие индексам ключей из `ma_keys`.  
  Если поле равно `NULL`, значит, это «старый» словарь, где и ключи, и значения лежат в одном месте (используется для
  обычных dict).

### Уточнения

- Современная реализация (с Python 3.6) гарантирует **сохранение порядка вставки** благодаря оптимизированной
  хеш-таблице.
- Разделение `ma_keys` и `ma_values` позволяет экономить память при использовании общих ключей (например, в `__dict__` у
  одинаковых объектов одного класса).
- `ma_used` и `ma_version_tag` применяются для корректной работы итераторов: изменение словаря во время обхода вызывает
  ошибку `RuntimeError`.
- Доступ к элементам (`dict[key]`) осуществляется через хеш функции, хранящиеся в `PyDictKeysObject`, а значения
  подтягиваются из `ma_values` по индексу.

[Содержание](/CONTENTS.md#содержание)

---

# Создание dict (PyDictObject)

```c
PyObject *
PyDict_New(void)
{
    /* We don't incref Py_EMPTY_KEYS here because it is immortal. */
    return new_dict(Py_EMPTY_KEYS, NULL, 0, 0);
}

static PyObject *
new_dict(PyDictKeysObject *keys, PyDictValues *values,
         Py_ssize_t used, int free_values_on_failure)
{
    assert(keys != NULL);
    PyDictObject *mp = _Py_FREELIST_POP(PyDictObject, dicts);
    if (mp == NULL) {
        mp = PyObject_GC_New(PyDictObject, &PyDict_Type);
        if (mp == NULL) {
            dictkeys_decref(keys, false);
            if (free_values_on_failure) {
                free_values(values, false);
            }
            return NULL;
        }
    }
    assert(Py_IS_TYPE(mp, &PyDict_Type));
    mp->ma_keys = keys;
    mp->ma_values = values;
    mp->ma_used = used;
    mp->_ma_watcher_tag = 0;
    ASSERT_CONSISTENT(mp);
    _PyObject_GC_TRACK(mp);
    return (PyObject *)mp;
}
```

### Общее

Создание пустого словаря `dict{}` в CPython начинается с вызова `PyDict_New()`. Эта функция создаёт объект
`PyDictObject` с минимальными начальными настройками, используя предсозданную "пустую" структуру ключей `Py_EMPTY_KEYS`.
Словарь получается компактным и готовым к быстрому заполнению без немедленного выделения дополнительной памяти под
записи.

### Описание

`PyDict_New()` вызывает вспомогательную функцию `new_dict()`, передавая ей бессмертный объект `Py_EMPTY_KEYS` (с
`dk_refcnt = PY_SSIZE_T_MIN`, размер 8 слотов, все индексы `DKIX_EMPTY`). Функция `new_dict()` берёт объект из пула
свободных словарей (`_Py_FREELIST_POP`) или создаёт новый через `PyObject_GC_New()`. Инициализируются поля: `ma_keys`
указывает на пустые ключи, `ma_values = NULL` (комбинированная таблица), `ma_used = 0`. Объект помечается для сборщика
мусора и возвращается.

### Уточнения

- `Py_EMPTY_KEYS` — глобальный бессмертный объект с 8 слотами (минимальный размер `PyDict_MINSIZE=8`), все индексы
  заполнены `DKIX_EMPTY(-1)` для быстрого поиска.
- Комбинированная таблица (`ma_values=NULL`) хранит ключи/значения в `dk_entries` структуры `PyDictKeysObject`.
- При первом добавлении элемента словарь вырастет до нужного размера через `dictresize()` с коэффициентом
  `USABLE_FRACTION=2/3`.
- Пул freelists ускоряет создание малых словарей, снижая нагрузку на `malloc()`.

[Содержание](/CONTENTS.md#содержание)

---

## PyDictKeysObject

```c
struct _dictkeysobject {
    Py_ssize_t dk_size;        // Размер хеш-таблицы
    enum dict_keys_kind dk_kind; // DICT_KEYS_UNICODE и т.д.
    union {
        PyDictUnicodeEntry *dk_entries;  // Полная таблица key+value
        PyDictKeyEntry *dk_indices;      // Только индексы
    } dk;
    uint64_t dk_version_tag;
};
```

### Общее

`PyDictKeysObject` (или `_dictkeysobject`) — это внутренняя структура CPython, которая хранит **ключи и хеш-таблицу**
для словарей.  
Она отделена от `PyDictObject` и может быть **общей** для нескольких словарей (экономия памяти).  
Это "сердце" быстрого поиска в `dict[key]`.

### Описание

- `Py_ssize_t dk_size` — размер **хеш-таблицы** (количество слотов, обычно больше `ma_used`).  
  Python резервирует больше места, чтобы избежать частых перехеширований.

- `enum dict_keys_kind dk_kind` — тип структуры ключей:
    - `DICT_KEYS_UNICODE` — оптимизация для строковых ключей (самый частый случай).
    - `DICT_KEYS_GENERAL` — для произвольных хешируемых объектов.

- `union { ... } dk` — **союз** (union) с разными вариантами хранения:
    - `PyDictUnicodeEntry *dk_entries` — **полная таблица** ключ+значение (для "старых" словарей).
    - `PyDictKeyEntry *dk_indices` — **только индексы** для быстрого поиска (современный split table).

- `uint64_t dk_version_tag` — версия ключей (синхронизируется с `ma_version_tag` словаря).

### Уточнения

- **Union** позволяет экономить память: разные словари используют разные форматы в зависимости от содержимого.
- В Python 3.9+ **преобладает split table** (`dk_indices` + отдельные `ma_values`), `dk_entries` используется редко.
- `dk_size` определяет, сколько элементов в `dk_indices[]` для поиска по хэшу (open addressing с probing).
- Общие `ma_keys` используются в `__dict__` экземпляров одного класса — все объекты делят одну таблицу ключей.

[Содержание](/CONTENTS.md#содержание)

---

## list (PyListObject)

```c
typedef struct {
    PyObject_VAR_HEAD         // PyObject + ob_size (длина)
    PyObject **ob_item;       // Указатели на элементы
    Py_ssize_t allocated;     // Выделенная ёмкость (> ob_size)
} PyListObject;
```

### Общее

`PyListObject` — это внутренняя структура CPython, которая описывает объект **списка (`list`)**.  
Когда вы создаёте список вроде `[1, 2, 3]`, в памяти Python создаёт именно такую структуру.  
Она хранит указатели на элементы, информацию о длине и о том, сколько памяти зарезервировано под будущие добавления.

### Описание

- `PyObject_VAR_HEAD` — стандартный заголовок переменных объектов, который добавляет поля:
    - `ob_refcnt` — счётчик ссылок (управление памятью).
    - `ob_type` — ссылка на тип (`list`).
    - `ob_size` — текущее количество элементов в списке (то, что возвращает `len(lst)`).

- `PyObject **ob_item;` — указатель на **массив ссылок** (указателей) на реальные объекты Python.  
  То есть сам список хранит не данные, а только ссылки — например, `[1, "a"]` содержит ссылки на `PyLongObject(1)` и
  `PyUnicodeObject("a")`.

- `Py_ssize_t allocated;` — количество ячеек, реально выделенных под элементы.  
  Оно может быть больше, чем `ob_size`, чтобы ускорить `append()` и не выделять память при каждом добавлении.

### Уточнения

- Разница между `ob_size` и `allocated` помогает Python делать **динамическое расширение** списка (амортизация вставок).
- Список — это не массив примитивов, а **массив указателей на объекты**, поэтому изменение одного списка не трогает
  содержимое других.
- Элементы списка всегда находятся в куче (heap), и список просто хранит ссылки на них.
- При удалении элементов память под массив может быть перераспределена — это управляется внутренними функциями
  `list_resize()`.

[Содержание](/CONTENTS.md#содержание)

---

## Слоты list'а (PyList_Type)

```c
PyTypeObject PyList_Type = {
    PyVarObject_HEAD_INIT(&PyType_Type, 0)  // Наследуем от type
    "list",                    // tp_name
    sizeof(PyListObject),      // tp_basicsize
    0,                         // tp_itemsize (переменная часть в ob_item)
    
    (destructor)list_dealloc,  // tp_dealloc
    0,                         // tp_print
    0,                         // tp_getattr
    0,                         // tp_setattr
    0,                         // tp_reserved
    list_repr,                 // tp_repr -> str(list)
    
    0,                         // tp_as_number
    &list_as_sequence,         // tp_as_sequence <- ВАЖНО!
    0,                         // tp_as_mapping
    (hashfunc)PyObject_HashNotImplemented,  // tp_hash (списки не хешируемы)
};
```

### Общее

`PyList_Type` — это **определение типа `list` на уровне C** внутри интерпретатора CPython.  
Эта структура описывает, как Python должен работать со списками: как их создавать, удалять, представлять в виде строки и
какие операции они поддерживают.  
По сути, это “паспорт” встроенного типа `list`, с которым работает любая функция `type()` и весь механизм ООП Python.

### Описание

- `PyVarObject_HEAD_INIT(&PyType_Type, 0)` — инициализирует заголовок типа.  
  Указывает, что `list` — это **объект типа `type`** (в Python всё — объект, включая классы).  
  Второй параметр `0` — это базовый размер для `_var`-части (переменная длина задаётся отдельно).

- `"list"` — имя типа, которое возвращает `type([]).__name__`.

- `sizeof(PyListObject)` — базовый размер структуры в памяти (заголовок + указатели, без элементов).

- `tp_itemsize = 0` — размер дополнительного элемента в переменных частях структуры не используется, потому что список
  хранит данные через `ob_item` (указатель на отдельный массив).

- `(destructor)list_dealloc` — функция, которая освобождает память списка при удалении (`tp_dealloc`).

- `list_repr` — функция, которая формирует строковое представление (`repr(list)` или `str(list)`).

- `&list_as_sequence` — ссылка на таблицу функций, описывающих поведение списка как **последовательности** (sequence
  protocol).  
  Именно отсюда берутся операции `len(lst)`, `lst[i]`, `lst.append()` и `lst[i:j]`.

- `(hashfunc)PyObject_HashNotImplemented` — указывает, что список **нельзя хешировать** (поэтому `hash([])` вызывает
  ошибку).

### Уточнения

- `PyList_Type` хранится в памяти как **глобальная структура**, доступная интерпретатору для всех операций со списками.
- В ней много других полей, которые здесь не показаны (например, для инициализации, копирования, итераций).
- Протокол `tp_as_sequence` делает список совместимым с универсальными функциями Python, которые работают со всеми
  последовательностями (`tuple`, `str` и др.).
- Поскольку список изменяемый, Python специально запрещает его хеширование, чтобы не нарушать поведение словарей и
  множеств.

[Содержание](/CONTENTS.md#содержание)

---

## Доступ к элементу у list

```c
static PyObject *list_item(PyListObject *self, Py_ssize_t i) {
    if (i < 0 || i >= Py_SIZE(self)) {
        PyErr_SetString(PyExc_IndexError, "list index out of range");
        return NULL;
    }
    Py_INCREF(self->ob_item[i]);   // Увеличиваем refcnt
    return self->ob_item[i];       // Возвращаем элемент
}
```

### Общее

`list_item` — это внутренняя C‑функция, которая реализует поведение **доступа к элементу списка по индексу** — то есть
выражение `lst[i]` в Python.  
Она безопасно проверяет границы индекса и возвращает нужный элемент, корректно управляя счётчиком ссылок (чтобы объект
не был удалён преждевременно).

### Описание

- `if (i < 0 || i >= Py_SIZE(self))` — проверка выхода за границы.  
  Функция `Py_SIZE(self)` возвращает текущее количество элементов (`ob_size` из `PyListObject`).  
  Если индекс меньше нуля или больше последнего элемента, выбрасывается исключение `IndexError`.

- `PyErr_SetString(PyExc_IndexError, "list index out of range");` — создаёт и поднимает стандартное исключение Python,
  полностью аналогичное тому, что видит пользователь.

- `Py_INCREF(self->ob_item[i]);` — увеличивает счётчик ссылок на возвращаемый объект, чтобы гарантировать, что он не
  удалится из памяти, пока Python-код им пользуется.

- `return self->ob_item[i];` — возвращает указатель на объект, находящийся по данному индексу (элемент списка).

### Уточнения

- Проверка диапазона обязательна, потому что в C нет защиты от обращения к памяти за пределами массива — это
  предотвращает падения интерпретатора.
- В Python отрицательные индексы (`lst[-1]`) обрабатываются **на более высоком уровне** — сюда уже приходит вычисленный
  положительный индекс.
- Увеличение `refcount` обеспечивает **безопасность памяти**: даже если элемент будет удалён из списка, ссылка,
  возвращённая из этой функции, останется действительной в пользовательском коде.
- Эта функция является частью реализации **sequence protocol**, который отвечает за операции индексирования и срезов.

[Содержание](/CONTENTS.md#содержание)

---

# tuple (PyTupleObject)

```c
// Файл: Objects/tupleobject.c (CPython 3.9+)
// PyTupleObject - структура неизменяемого tuple
typedef struct {
    PyObject_VAR_HEAD
    PyObject *ob_item[1];  // гибкий массив указателей на элементы
} PyTupleObject;
```

**Общее**

- Эта структура описывает внутреннее устройство объекта `tuple` (кортежа) в CPython.
- Кортеж — это неизменяемая (immutable) последовательность объектов, где каждый элемент хранится по указателю.
- Структура оптимизирована для компактного хранения и быстрого доступа к элементам по индексу.

**Описание**

- `PyObject_VAR_HEAD` — стандартный макрос для переменных по длине Python-объектов. Добавляет поля:
    - счётчик ссылок,
    - указатель на тип (`PyTypeObject *ob_type`),
    - длину последовательности (`Py_ssize_t ob_size`).
- `PyObject *ob_item[1];` — гибкий массив указателей на элементы кортежа. На самом деле размер этого массива равен длине
  кортежа (`ob_size`), просто объявление использует «1» как технический шаблон.
- Каждый элемент `ob_item[i]` — это указатель на объект Python, который содержится в кортеже.

**Уточнения**

- Кортежи создаются один раз, и их размер не меняется — после инициализации количество элементов фиксировано.
- Реальный размер памяти под `ob_item` выделяется динамически при создании кортежа, исходя из значения `ob_size`.
- Поскольку кортежы неизменяемы, добавление, удаление или замена элементов невозможны — при таких операциях создаётся
  новый объект.
- Гибкое объявление массива делает структуру эффективной: она хранит все элементы подряд в памяти, что ускоряет доступ
  по индексу (`O(1)`).

[Содержание](/CONTENTS.md#содержание)

---

# Создание tuple (PyTupleObject)

```c
// Файл: Objects/tupleobject.c (CPython 3.9+)
// tuple([iterable]) - основной конструктор
PyObject *
PyTuple_New(Py_ssize_t size)
{
    PyTupleObject *op;
    
    if (size < 0) {
        PyErr_SetString(PyExc_ValueError, "negative tuple size");
        return NULL;
    }
    
    /* Выделяем заголовок + size указателей PyObject* */
    op = PyObject_MALLOC(size * sizeof(PyObject *) + sizeof(PyTupleObject) - sizeof(PyObject *));
    if (unlikely(op == NULL))
        return PyErr_NoMemory();
    
    /* Инициализируем: refcnt=1, PyTuple_Type, obsize=size */
    PyObject_INIT_VAR(op, &PyTuple_Type, size);
    
    /* Все слоты ob_item[] = NULL */
    memset(op->ob_item, 0, size * sizeof(PyObject *));
    
    return (PyObject *)op;
}

// Заполнение (крадёт ссылку на item)
int
PyTuple_SetItem(PyTupleObject *tuple, Py_ssize_t index, PyObject *item)
{
    if (index < 0 || index >= PyTuple_GET_SIZE(tuple)) {
        Py_XDECREF(item);
        PyErr_SetString(PyExc_IndexError, "tuple assignment index out of range");
        return -1;
    }
    PyObject **p = tuple->ob_item + index;
    Py_XINCREF(item);
    Py_XDECREF(*p);
    *p = item;
    return 0;
}
```

### Общее

Создание `tuple` — выделение компактного блока памяти под заголовок плюс массив указателей на элементы. CPython
использует формулу `sizeof(PyTupleObject)-sizeof(PyObject*)+size*sizeof(PyObject*)` для точного размера без перерасхода.

### Описание

`PyTuple_New(size)` проверяет size ≥ 0, выделяет память одной порцией `PyObject_MALLOC`. `PyObject_INIT_VAR`
устанавливает refcnt=1, тип `PyTuple_Type`, obsize=size. `memset` обнуляет `ob_item[]`. `PyTuple_SetItem()` только при
создании заменяет NULL на элемент, крадя ссылку (Py_XINCREF + Py_XDECREF).

### Уточнения

- `PyTupleObject_SIZE(n) = sizeof(PyTupleObject)-sizeof(PyObject*)+n*sizeof(PyObject*)`.
- Не GC: чистый refcnt, tuple не содержит ссылок на GC-объекты.
- `PyTuple_GET_ITEM(t, i)` — прямой доступ без проверок: `t->ob_item[i]`.
- Python 3.9+: после создания `sq_ass_item=NULL`, `t[0]=1` → TypeError.
- Пустой `tuple()`: 24 байта (только PyVarObject_HEAD).

[Содержание](/CONTENTS.md#содержание)

---

# bytes (PyBytesObject)

```c
// Файл: Objects/bytesobject.c (CPython 3.9+)
// PyBytesObject - структура bytes
typedef struct {
    PyObject_VAR_HEAD
    char ob_sval[1];  // гибкий массив байтов (об_sval[length])
} PyBytesObject;
```

### Общее

Этот блок кода определяет структуру `PyBytesObject` — внутреннее представление объекта `bytes` в CPython. Она наследует
`PyObject_VAR_HEAD` для базовых полей (refcnt, type, размер) и добавляет гибкий массив `ob_sval[]` для хранения самих
байтов данных сразу после заголовка.

### Описание

`PyObject_VAR_HEAD` расширяется в `PyObject ob_base` (refcnt + obtype) плюс `Py_ssize_t ob_size` (длина в байтах).
`char ob_sval[1]` — это C-трюк с flexible array member: компилятор видит минимум 1 байт, но при выделении памяти (
`PyObject_MALLOC`) добавляется столько байтов, сколько указано в `ob_size`. Байты хранятся компактно: `&obj->ob_sval[0]`
сразу после заголовка.

### Уточнения

- Размер в памяти: 24 байта заголовок (64-бит) + `ob_size` байтов данных.
- Неизменяемый: после создания `ob_sval` не меняется, только refcnt управляет жизнью.
- Python 3.9+: `PyBytes_Type.tp_basicsize = sizeof(PyBytesObject)-1`, чтобы формулы вроде `PyBytesObject_SIZE + length`
  работали правильно.
- Доступ к байтам: `((PyBytesObject*)obj)->ob_sval[i]` или `PyBytes_AS_STRING(obj)`.
- Null-терминирован: `ob_sval[ob_size] = '\0'` для C-строк совместимости.

[Содержание](/CONTENTS.md#содержание)

---

# Создание bytes (PyBytesObject)

```c
// Основной конструктор bytes(string, encoding) или bytes(length)
PyObject *
PyBytes_FromStringAndSize(const char *str, Py_ssize_t size)
{
    PyBytesObject *op;
    
    if (size < 0) {
        PyErr_SetString(PyExc_ValueError, "negative size");
        return NULL;
    }
    
    /* Выделяем память: заголовок + size байтов данных */
    op = (PyBytesObject *)PyObject_MALLOC(size + sizeof(PyBytesObject) - 1);
    if (unlikely(op == NULL))
        return PyErr_NoMemory();
    
    /* Инициализируем: refcnt=1, type=PyBytes_Type, obsize=size */
    PyObject_INIT_VAR(op, &PyBytes_Type, size);
    
    if (str != NULL)
        memcpy(op->ob_sval, str, (size_t)size);
    
    /* Null-терминатор для C-строк */
    op->ob_sval[size] = '\0';
    
    return (PyObject *)op;
}

// bytes() - пустой объект
PyObject *
PyBytes_FromString(const char *str)
{
    return PyBytes_FromStringAndSize(str, strlen(str));
}
```

### Общее

Создание `PyBytesObject` — это выделение памяти под заголовок `PyVarObject` плюс нужное количество байтов данных.
CPython хранит байты компактно сразу после заголовка в поле `ob_sval[]`, делая объект быстрым и экономным по памяти.

### Описание

`PyBytes_FromStringAndSize()` сначала проверяет size ≥ 0, потом выделяет память формулой
`sizeof(PyBytesObject)-1 + size` (минус 1, потому что `ob_sval[1]` уже в структуре). `PyObject_INIT_VAR` устанавливает
refcnt=1, тип `PyBytes_Type` и длину `obsize=size`. Байты копируются в `ob_sval[]`, добавляется `'\0'` для
C-совместимости.

### Уточнения

- `PyObject_VAR_HEAD` расшифровывается в refcnt + obtype + obsize (24 байта на 64-бит).
- `ob_sval[1]` — трюк C: компилятор требует минимум 1 байт, но `MALLOC` даёт столько, сколько нужно.
- Память под bytes не попадает в GC, только refcnt — поэтому `PyObject_MALLOC` вместо `PyObject_GC_New`.
- `PyBytes_Type.tp_basicsize = sizeof(PyBytesObject)-1`, чтобы `PyBytesObject_SIZE` считал правильно.
- Python 3.9+: bytes неизменяемы, как str, но для 0-255 байтов (не unicode).

[Содержание](/CONTENTS.md#содержание)

---

# bytearray (...)

```c
// Файл: Objects/bytearrayobject.c (CPython 3.9+)
// PyByteArrayObject - структура bytearray (изменяемый bytes)
typedef struct {
    PyObject_VAR_HEAD
    Py_ssize_t ob_alloc;  // выделенная ёмкость (capacity)
    char *ob_bytes;       // указатель на изменяемый буфер байтов
} PyByteArrayObject;
```

### **Общее**

- Эта структура описывает внутреннее устройство объекта `bytearray` в CPython (конкретно — его реализацию на языке C в
  исходном коде интерпретатора).
- `bytearray` — это изменяемая последовательность байтов, аналогично `bytes`, но с возможностью изменения содержимого
  без пересоздания объекта.
- Структура используется внутри интерпретатора Python для управления памятью и хранением байтовых данных.

### **Описание**

- `typedef struct { ... } PyByteArrayObject;` — объявление типа структуры в C для представления Python-объекта
  `bytearray`.
- `PyObject_VAR_HEAD` — макрос, который добавляет стандартные служебные поля, общие для всех переменных объектов
  Python (в том числе счётчик ссылок и указатель на тип). Он также хранит текущее количество элементов (`ob_size`).
- `Py_ssize_t ob_alloc;` — размер выделенной под байты памяти (в байтах). Он может быть больше фактической длины
  `bytearray`, чтобы избежать частого перевыделения памяти при росте.
- `char *ob_bytes;` — указатель на область памяти, где физически хранятся байты. Это изменяемый буфер, на который
  ссылается объект `bytearray`.

### **Уточнения**

- `Py_ssize_t` — это знаковый тип, используемый в CPython для хранения размеров и индексов (безопасно заменяет обычный
  `int` при работе с большими объектами).
- `PyObject_VAR_HEAD` обеспечивает совместимость `bytearray` с механизмами Python-объектов — например, с подсчётом
  ссылок и системой типов.
- В отличие от `bytes`, где данные хранятся в неизменяемом массиве, `bytearray` использует `ob_bytes` как изменяемый
  буфер, что позволяет функции вроде `bytearray.append()` работать без пересоздания объекта.

[Содержание](/CONTENTS.md#содержание)

---

# Создание bytearray (PyByteArrayObject)

```c
// Файл: Objects/bytearrayobject.c (CPython 3.9+)
// Основной конструктор bytearray(string) или bytearray(length)
PyObject *
PyByteArray_FromStringAndSize(const char *string, Py_ssize_t len)
{
    PyByteArrayObject *result;
    
    if (len < 0) {
        PyErr_SetString(PyExc_ValueError, "negative count");
        return NULL;
    }
    
    /* Выделяем структуру PyByteArrayObject */
    result = (PyByteArrayObject *)PyObject_MALLOC(sizeof(PyByteArrayObject));
    if (unlikely(result == NULL))
        return PyErr_NoMemory();
    
    /* Инициализируем: refcnt=1, type=PyByteArray_Type, obsize=len */
    PyObject_INIT_VAR(result, &PyByteArray_Type, len);
    
    /* Выделяем отдельный буфер для изменяемых байтов */
    result->ob_alloc = len;
    result->ob_bytes = (char *)PyObject_MALLOC(len ? len : 1);
    if (unlikely(result->ob_bytes == NULL)) {
        PyObject_DEL(result);
        return PyErr_NoMemory();
    }
    
    if (string != NULL)
        memcpy(result->ob_bytes, string, (size_t)len);
    
    return (PyObject *)result;
}

// bytearray() - пустой bytearray
PyObject *
PyByteArray_Dup(PyByteArrayObject *ba)
{
    return PyByteArray_FromStringAndSize(ba->ob_bytes, Py_SIZE(ba));
}
```

### Общее

Создание `PyByteArrayObject` — это выделение структуры плюс отдельного изменяемого буфера байтов. В отличие от `bytes`,
где данные встраиваются в объект, `bytearray` держит `ob_bytes` как указатель на внешнюю память, что позволяет легко
менять размер и содержимое.

### Описание

`PyByteArray_FromStringAndSize()` сначала проверяет len ≥ 0. Выделяет фиксированную структуру `PyByteArrayObject` (24+
байта), инициализирует через `PyObject_INIT_VAR`. Отдельно выделяет буфер `ob_bytes` (минимум 1 байт даже для пустого).
`ob_alloc = len` фиксирует ёмкость, байты копируются, возвращается PyObject*.

### Уточнения

- Два отдельных `PyObject_MALLOC`: структура + данные (независимая жизнь).
- `ob_alloc` отслеживает выделенную ёмкость, `ob_size` — текущую длину (может расти).
- Пустой `bytearray()`: `ob_bytes` указывает на 1 байт, `ob_size=0`, `ob_alloc=0`.
- Python 3.9+: при `append()` проверяется `ob_size < ob_alloc`, иначе realloc большего буфера.
- Очистка: `bytearrayobject_dealloc()` освобождает оба `PyObject_FREE`.

[Содержание](/CONTENTS.md#содержание)

---

# set (PySetObject)

```c
// Файл: Objects/setobject.c (CPython 3.9+)
// PySetObject - структура set (хэш-таблица с уникальными элементами)
typedef struct {
    PyObject_HEAD
    Py_hash_t used;         // количество элементов
    Py_ssize_t fill;        // заполненные слоты
    Py_ssize_t mask;        // размер хэш-таблицы - 1 (2^n - 1)
    PyDictKeysObject *dk;   // общие ключи с dict (compact/ split)
} PySetObject;
```

**Общее**

- Эта структура описывает внутреннее устройство объекта `set` (множества) в CPython.
- `set` хранит уникальные элементы и построен на основе хэш-таблицы, аналогично `dict`, но без значений — у множества
  есть только ключи.
- Структура определяет, как Python управляет памятью, хранит элементы и рассчитывает хэши для быстрой проверки
  принадлежности (`in`) или вставки.

**Описание**

- `typedef struct { ... } PySetObject;` — объявление структуры в языке C, которая представляет объект типа `set` внутри
  интерпретатора.
- `PyObject_HEAD` — стандартный макрос, добавляющий служебную информацию о Python-объекте (счётчик ссылок и указатель на
  тип).
- `Py_hash_t used` — количество активных (реальных) элементов в множестве, то есть тех, что сейчас существуют.
- `Py_ssize_t fill` — общее количество занятых ячеек в хэш-таблице (включает удалённые и активные элементы);
  используется для управления переразмериванием таблицы.
- `Py_ssize_t mask` — маска размера таблицы (`size - 1`), где размер всегда является степенью двойки. Применяется для
  быстрого вычисления позиции элемента через побитовые операции.
- `PyDictKeysObject *dk` — указатель на структуру, общую с реализацией словаря (`dict`), в которой фактически хранится
  хэш-таблица и связанные с ней ключи.

**Уточнения**

- `PyDictKeysObject` используется и для `dict`, и для `set`, что позволяет переиспользовать оптимизированную реализацию
  хранения хэшей и ключей.
- Поскольку `set` хранит только ключи (без значений), часть структуры `PyDictKeysObject`, отвечающая за значения,
  остаётся неиспользованной.
- Поле `mask` определяет диапазон индексов в хэш-таблице: индекс вычисляется как `hash(key) & mask`.
- `used` и `fill` помогают интерпретатору определить, когда требуется увеличить или уплотнить таблицу, чтобы
  поддерживать эффективность операций.

[Содержание](/CONTENTS.md#содержание)

---

# Создание set (PySetObject)

```c
// Файл: Objects/setobject.c (CPython 3.9+)
// set([iterable]) - основной конструктор
PyObject *
PySet_New(PyObject *iterable)
{
    PySetObject *so;
    Py_ssize_t estimate;
    
    /* Оцениваем начальный размер */
    if (iterable == NULL)
        estimate = 0;
    else {
        estimate = PyObject_Length(iterable);
        if (estimate < 0)
            return NULL;
    }
    
    /* Выделяем с GC (set участвует в сборке мусора) */
    so = PyObject_GC_New(PySetObject, &PySet_Type);
    if (so == NULL)
        return NULL;
    
    so->used = 0;        // элементов пока нет
    so->fill = 0;        // слотов заполнено
    so->mask = 0;        // хэш-таблица пустая
    so->dk = NULL;       // ключи пока не созданы
    
    if (estimate > 0) {
        /* Создаём хэш-таблицу подходящего размера */
        if (set_table_resize(so, estimate) < 0) {
            Py_DECREF(so);
            return NULL;
        }
        /* Добавляем все элементы из iterable */
        if (_PySet_Update(so, iterable) < 0) {
            Py_DECREF(so);
            return NULL;
        }
    }
    
    PyObject_GC_Track(so);
    return (PyObject *)so;
}
```

### Общее

Создание `PySetObject` — это построение хэш-таблицы с оценкой начального размера и автоматическим добавлением элементов.
CPython использует GC-объект, чтобы set мог участвовать в цикличной сборке мусора, и общую с `dict` структуру ключей для
экономии памяти.

### Описание

`PySet_New()` сначала узнаёт примерный размер через `PyObject_Length()`. Выделяет `PySetObject` с GC-поддержкой,
обнуляет счётчики. `set_table_resize()` создаёт хэш-таблицу степени двойки большего размера (load factor ~2/3).
`_PySet_Update()` проходит по iterable, хэширует каждый элемент и вставляет с обработкой коллизий. В конце объект
отслеживается GC.

### Уточнения

- `estimate` помогает выбрать размер хэш-таблицы без лишних ресайзов.
- `PyObject_GC_New` вместо `PyObject_MALLOC` — для `tp_traverse` и циклических ссылок.
- Пустой `set()`: все поля 0, `dk=NULL` (compact mode до 5 элементов).
- Python 3.9+: `used` считает только уникальные хэшируемые элементы, дубли пропускаются.
- Не-хэшируемые элементы (list, dict) дают `TypeError: unhashable type`.

[Содержание](/CONTENTS.md#содержание)

---

# frozenset (PySetObject)

```c
// Файл: Objects/setobject.c (CPython 3.9+)
// PySetObject используется для обоих set и frozenset
typedef struct {
    PyObject_HEAD
    Py_hash_t used;         // количество элементов
    Py_ssize_t fill;        // заполненные слоты  
    Py_ssize_t mask;        // размер хэш-таблицы - 1 (2^n - 1)
    PyDictKeysObject *dk;   // общие ключи с dict
} PySetObject;
```

**Общее**

- Эта структура описывает внутреннее устройство объектов `set` и `frozenset` в CPython — оба типа реализованы на одной
  базе.
- Оба представляют собой хэш-таблицу, где хранятся только уникальные элементы, но `set` изменяем, а `frozenset` — нет.
- Цель этой структуры — управлять размещением элементов, их хэшами и оптимизацией доступа (например, при проверке
  «элемент есть в множестве»).

**Описание**

- `PyObject_HEAD` — стандартная часть каждой структуры объекта Python, содержит служебные поля: указатель на тип и
  счётчик ссылок.
- `Py_hash_t used` — текущее количество элементов в множестве (активных записей).
- `Py_ssize_t fill` — количество занятых ячеек в хэш-таблице, включая удалённые и активные элементы; помогает определить
  степень заполнения.
- `Py_ssize_t mask` — используется для вычисления индекса в таблице (`hash & mask`), где `mask = size - 1`, а размер
  всегда равен степени двойки.
- `PyDictKeysObject *dk` — ссылка на общую структуру ключей, используемую и в `dict`; фактически хранит хэш-таблицу, в
  которой лежат ключи (элементы множества).

**Уточнения**

- Хотя `PySetObject` используется и для `set`, и для `frozenset`, разница в поведении достигается флагами и логикой на
  уровне Python API, а не структурой данных.
- `used` и `fill` позволяют определять, когда нужно увеличить или реструктурировать таблицу для поддержания
  производительности.
- Переиспользование `PyDictKeysObject` обеспечивает эффективность и уменьшает дублирование кода между реализациями
  `dict` и `set`.
- Для операций вроде поиска элемента Python использует значение хэша и поле `mask`, что делает доступ к элементам
  множества очень быстрым (амортизированное O(1)).

[Содержание](/CONTENTS.md#содержание)

---

# Создание frozenset (PySetObject)

```c
// Файл: Objects/setobject.c (CPython 3.9+)
// frozenset([iterable]) - конструктор неизменяемого множества
PyObject *
PyFrozenSet_New(PyObject *iterable)
{
    PySetObject *result;
    Py_ssize_t estimate;
    
    /* Оцениваем начальный размер хэш-таблицы */
    if (iterable == NULL)
        estimate = 0;
    else {
        estimate = PyObject_Length(iterable);
        if (estimate < 0)
            return NULL;
    }
    
    /* Выделяем GC-объект с типом frozenset */
    result = PyObject_GC_New(PySetObject, &PyFrozenSet_Type);
    if (result == NULL)
        return NULL;
    
    result->used = 0;    // элементов пока нет
    result->dk = NULL;   // ключи не созданы
    
    if (estimate > 0) {
        /* Создаём хэш-таблицу подходящего размера */
        if (set_table_resize(result, estimate) < 0) {
            Py_DECREF(result);
            return NULL;
        }
        /* Заполняем уникальными элементами */
        if (_PySet_Update(result, iterable) < 0) {
            Py_DECREF(result);
            return NULL;
        }
    }
    
    PyObject_GC_Track(result);
    return (PyObject *)result;
}
```

### Общее

Создание `frozenset` — это построение неизменяемой хэш-таблицы с теми же механизмами, что у `set`, но с типом
`PyFrozenSet_Type`. CPython использует одну структуру `PySetObject` для экономии, блокируя мутации на уровне
`tp_as_sequence` и `tp_as_mapping`.

### Описание

`PyFrozenSet_New()` оценивает размер через `PyObject_Length()`. Выделяет `PySetObject` с GC и типом `PyFrozenSet_Type`.
`set_table_resize()` создаёт хэш-таблицу степени двойки. `_PySet_Update()` добавляет элементы из iterable, автоматически
удаляя дубликаты и проверяя хэшируемость. Объект помечается для GC-отслеживания.

### Уточнения

- `PyFrozenSet_Type` имеет `sq_ass_item = NULL`, `mp_ass_subscript = NULL` — мутации запрещены.
- Хэш вычисляется при создании и кэшируется в `PyDictKeysObject` для использования как dict-ключ.
- Python 3.9+: compact mode для малых frozenset (до 5 элементов без отдельной таблицы).
- Пустой `frozenset()`: все счётчики 0, идеален для дефолтных значений в dict.
- `TypeError` на не-хэшируемых элементах (list, set, dict).

[Содержание](/CONTENTS.md#содержание)

---

# bool (PyBoolObject)

```c
// Файл: Objects/boolobject.c (CPython 3.9+)
// PyBoolObject - True/False (подтипы int)
typedef struct {
    PyLongObject longobj;  // наследует PyLongObject (int)
} PyBoolObject;
```

**Общее**

- Эта структура описывает внутреннее устройство объекта `bool` (`True` и `False`) в CPython.
- В Python тип `bool` является подтипом `int`, поэтому `True` и `False` — это не отдельные базовые типы, а
  специализированные экземпляры целых чисел (`1` и `0` соответственно).
- Структура показывает, что логические значения реализованы на основе уже готового механизма `PyLongObject`.

**Описание**

- `typedef struct { ... } PyBoolObject;` — объявление структуры для объекта `bool`.
- `PyLongObject longobj;` — это встраивание структуры `PyLongObject` (которая описывает `int` в Python). Благодаря этому
  `bool` наследует всю функциональность `int`, включая хранение значений и операции с ними.
- Таким образом, `PyBoolObject` не добавляет новых полей, а лишь определяет новый тип на базе существующего числового
  объекта.

**Уточнения**

- В интерпретаторе Python существуют только два экземпляра `PyBoolObject`: `Py_True` и `Py_False`, они создаются один
  раз при запуске (singleton-объекты).
- Значение поля `longobj` внутри `True` — это `1`, а у `False` — `0`.
- Такое решение экономит память и ускоряет операции, так как логические значения могут использовать ту же арифметическую
  инфраструктуру, что и целые числа.
- При проверке типов `bool` ведёт себя как отдельный тип, но при арифметических операциях — как `int`.

[Содержание](/CONTENTS.md#содержание)

---

# Создание bool (PyBoolObject)

```c
// Файл: Objects/boolobject.c (CPython 3.9+)
// bool(x) - возвращает один из двух глобальных синглтонов
PyObject *
_PyBool_FromLong(long v)
{
    PyObject *result;
    
    if (v == 0) {
        result = Py_NewReference(Py_False);  // увеличивает refcnt False
    }
    else {
        result = Py_NewReference(Py_True);   // увеличивает refcnt True
    }
    
    return result;
}

// Универсальный bool() для Python объектов
PyObject *
PyObject_IsTrue(PyObject *v)
{
    if (v == Py_None)
        Py_RETURN_FALSE;
    if (v == Py_True)
        Py_RETURN_TRUE;
    if (v == Py_False)
        Py_RETURN_FALSE;
    
    return _PyBool_FromLong(PyObject_IsTruthy(v));
}

// Инициализация синглтонов (Python/pythonrun.c)
void _PyBool_Init(void)
{
    Py_True = (PyObject *)_PyLong_New(1L);   // int 1 -> True
    Py_False = (PyObject *)_PyLong_New(0L);  // int 0 -> False
    Py_SET_TYPE(Py_True, &PyBool_Type);
    Py_SET_TYPE(Py_False, &PyBool_Type);
}
```

### Общее

Создание `bool(...)` не создаёт новые объекты — всегда возвращает один из двух глобальных синглтонов `Py_True`/
`Py_False`. Это супероптимизация: экономия памяти, постоянные `id(True)`, мгновенные сравнения `if x is True`.

### Описание

`_PyBool_FromLong()` преобразует C `long` в синглтон: 0 → `Py_False`, иначе → `Py_True` через `Py_NewReference()` (
только +1 к refcnt). `PyObject_IsTrue()` для Python-объектов проверяет специальные случаи (None, True/False) и вызывает
`PyObject_IsTruthy()` (длина>0 для контейнеров). Синглтоны создаются при запуске интерпретатора из int 0/1.

### Уточнения

- `Py_NewReference()` не выделяет память, только `Py_INCREF()`.
- `PyBool_Type` наследует от `PyLong_Type`, поэтому `bool(1) is True`.
- Python 3.9+: `tp_richcompare` оптимизирован для `is True/False`.
- `bool([]) == False` через `PyObject_Length([]) == 0`.
- Синглтоны бессмертны: refcnt никогда не 0, `tp_dealloc` не вызывается.

[Содержание](/CONTENTS.md#содержание)

---

# NoneType (PyNoneObject)

```c
// Файл: Objects/noneobject.c (CPython 3.9+)
// PyNoneObject - единственный объект None
typedef struct {
    PyObject_HEAD
} PyNoneObject;
```

**Общее**

- Эта структура описывает объект `None` в CPython — единственный экземпляр специального значения, обозначающего «ничего»
  или «отсутствие значения».
- `None` существует в единственном экземпляре в программе (singleton), и любая ссылка на `None` указывает на один и тот
  же объект.
- Сам объект не хранит дополнительных данных — он используется только как специальный маркер.

**Описание**

- `typedef struct { ... } PyNoneObject;` — объявление структуры для объекта `None`.
- `PyObject_HEAD` — стандартная часть любой Python-структуры, содержащая служебную информацию: указатель на тип (
  `PyTypeObject *ob_type`) и счётчик ссылок (`Py_ssize_t ob_refcnt`).
- Внутри `PyNoneObject` нет других полей, потому что `None` не содержит данных и не нуждается в дополнительном
  состоянии.

**Уточнения**

- В CPython есть только одна глобальная переменная `Py_None`, представляющая этот объект. Все операции с `None`
  используют именно её.
- `None` имеет собственный тип `NoneType`, который создан один раз и не может порождать новые экземпляры.
- Такое устройство делает проверки вроде `x is None` максимально быстрыми, так как сравниваются указатели, а не
  содержимое.
- Поскольку `None` не хранит значение, а только сам факт «пустоты», структура остаётся минимальной и содержит только
  базовые поля объекта Python.

[Содержание](/CONTENTS.md#содержание)

---

## Создание NoneType (PyNoneObject)

```c
// Файл: Objects/noneobject.c + Python/pythonrun.c (CPython 3.9+)
// Создание единственного None - происходит ОДИН раз при запуске
void
_PyNone_Init(void)
{
    PyNoneObject *none;
    
    none = (PyNoneObject *)PyObject_MALLOC(sizeof(PyNoneObject));
    if (none == NULL)
        Py_FatalError("can't initialize Py_None");
    
    /* Инициализация: refcnt=1, type=PyNone_Type */
    PyObject_INIT(none, &PyNone_Type);
    
    /* Глобальная переменная для всех None */
    Py_None = (PyObject *)none;
}

// Получение ссылки на None (НЕ создание!)
PyObject *
Py_NewReference(Py_None)
{
    Py_INCREF(Py_None);
    return Py_None;
}

// PyNone_Type (Objects/noneobject.c)
PyTypeObject PyNone_Type = {
    PyVarObject_HEAD_INIT(&PyType_Type, 0)
    .tp_name = "NoneType",
    .tp_basicsize = sizeof(PyNoneObject),  // всего PyObject_HEAD
    .tp_flags = Py_TPFLAGS_DEFAULT | Py_TPFLAGS_IMMUTABLETYPE,
};
```

### Общее

`NoneType` не создаётся конструктором — существует ровно один глобальный объект `Py_None`, выделенный при старте
интерпретатора. Все `None` в Python — это ссылка на него через `Py_INCREF(Py_None)`.

### Описание

`_PyNone_Init()` вызывается при инициализации CPython: выделяется минимальная `PyNoneObject` через `PyObject_MALLOC`,
инициализируется `PyObject_INIT` (refcnt=1, type=PyNone_Type), присваивается глобальной `Py_None`. Доступ через
`Py_NewReference(Py_None)` только увеличивает refcnt без выделения памяти.

### Уточнения

- Создание происходит ОДИН раз в `_Py_InitializeEx()`.
- `sizeof(PyNoneObject) == sizeof(PyObject)` — только refcnt + type.
- Python 3.9+: `Py_TPFLAGS_IMMUTABLETYPE` блокирует подклассы None.
- `Py_RETURN_NONE` макрос: `Py_INCREF(Py_None); return Py_None;`.
- Бессмертный синглтон: интерпретатор держит refcnt > 0 вечно.

[Содержание](/CONTENTS.md#содержание)

---

## GC-интеграция для контейнеров

```c
// list_traverse - обход ссылок для GC
static int list_traverse(PyListObject *o, visitproc visit, void *arg) {
    Py_ssize_t i = 0;
    Py_ssize_t len = Py_SIZE(o);
    for (; i < len; i++) {
        Py_VISIT(o->ob_item[i]);  // Отмечаем каждый элемент как живой
    }
    return 0;
}

// list_clear - разрыв циклических ссылок
static int list_clear(PyListObject *o) {
    Py_ssize_t i = 0;
    Py_ssize_t len = Py_SIZE(o);
    for (; i < len; i++) {
        Py_XDECREF(o->ob_item[i]);  // Уменьшаем refcnt элементов
        o->ob_item[i] = NULL;       // NULL'им ссылки
    }
    return 0;
}
```

### Общее

`list_traverse` и `list_clear` — внутренние функции CPython для **работы с сборщиком мусора (GC)**.  
Они помогают Python находить циклические ссылки и правильно освобождать память при удалении списка.  
Вызываются автоматически при сборке мусора или уничтожении объекта.

### Описание

- **`list_traverse(PyListObject *o, visitproc visit, void *arg)`** — обход всех элементов списка:
    - `Py_SIZE(o)` — получает длину списка (`ob_size`).
    - `Py_VISIT(o->ob_item[i])` — вызывает callback `visit()` для каждого элемента, помечая его как "живой" (доступный).
    - Возвращает `0` — успех обхода.

- **`list_clear(PyListObject *o)`** — очищает список от ссылок:
    - `Py_XDECREF(o->ob_item[i])` — уменьшает счётчик ссылок элемента (если он был 1, элемент удаляется).
    - `o->ob_item[i] = NULL` — обнуляет ссылку в списке, разрывая циклические связи.
    - Возвращает `0` — успех очистки.

### Уточнения

- `Py_VISIT()` и `Py_XDECREF()` — **стандартные макросы CPython** для безопасного управления ссылками в GC.
- `list_traverse` используется **generational GC** для предотвращения утечек памяти при циклах (список ссылается на
  себя).
- `list_clear` вызывается в `list_dealloc()` перед освобождением памяти, чтобы элементы могли быть собраны GC.
- Обе функции работают за **O(n)** — линейное время пропорционально длине списка.

[Содержание](/CONTENTS.md#содержание)

---

## Байткод-интеграция: LIST_APPEND

```c
case LIST_APPEND: {
    PyObject *v = TOP();              // Берём значение с вершины стека
    PyObject *list = PEEK(oparg + 1); // Берём список из фиксированной позиции
    Py_ssize_t index = oparg;         // Индекс списка в localsplus
    
    // Вызываем list.append(v)
    int err = PyList_Append(list, v);
    Py_DECREF(v);                     // Освобождаем значение
    
    if (err == 0) {
        STACKADJ(-1);                 // Убираем значение со стека
    } else {
        // Ошибка - прерываем выполнение
        break;
    }
    DISPATCH_SAME_OPARG(1);           // Следующая инструкция
}
```

### Общее

Этот код — **часть байткод-интерпретатора CPython**, обрабатывающая инструкцию `LIST_APPEND`.  
Она реализует операцию `list.append(v)` на уровне виртуальной машины Python.  
Выполняется при вызове `lst.append(value)` в Python-коде.
Инструкция `LIST_APPEND` в байт-коде Python оптимизирована для работы с генераторами списков (list comprehensions),
например `[x for x in lst]`.

### Описание

- `PyObject *v = TOP();` — берёт **значение** с вершины стека виртуальной машины (то, что добавляем в список).
- `PyObject *list = PEEK(oparg + 1);` — берёт **список** из стека по смещению `oparg + 1` (список уже лежит ниже).
- `Py_ssize_t index = oparg;` — `oparg` содержит индекс списка в локальных переменных (`localsplus`).
- `int err = PyList_Append(list, v);` — вызывает стандартную функцию добавления в конец списка.
- `Py_DECREF(v);` — уменьшает счётчик ссылок значения (оно больше не нужно на стеке).
- `STACKADJ(-1);` — убирает значение со стека (при успехе остаётся только список).
- `DISPATCH_SAME_OPARG(1);` — переходит к следующей инструкции байткода.

### Уточнения

- `LIST_APPEND` — **оптимизированная инструкция** Python 3.9+, ускоряет `append()` за счёт прямого вызова C-функции.
- `TOP()` и `PEEK()` — макросы для доступа к стековой машине (frame->stack).
- `PyList_Append()` автоматически увеличивает `allocated`, если список заполнен (over-allocation).
- При ошибке (`err != 0`) выполнение прерывается, выбрасывая исключение (например, `TypeError`).
- `DISPATCH_SAME_OPARG(1)` — "быстрый dispatch" без изменения аргумента операции.

[Содержание](/CONTENTS.md#содержание)