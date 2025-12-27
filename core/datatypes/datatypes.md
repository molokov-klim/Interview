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

## int

## **1. Создание int**

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

## **2. Арифметические операции**

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

## **3. Битовые операции**

```python
a, b = 0b1100, 0b1010  # 12 и 10

print(bin(a & b))  # 0b1000 (8) - AND
print(bin(a | b))  # 0b1110 (14) - OR
print(bin(a ^ b))  # 0b0110 (6) - XOR
print(bin(~a))  # -0b1101 (-13) - NOT
print(bin(a << 2))  # 0b110000 (48) - сдвиг влево
print(bin(a >> 2))  # 0b11 (3) - сдвиг вправо
```

## **4. Методы класса int**

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

## **5. Сравнение**

```python
a, b = 10, 20

print(a == b)  # False
print(a != b)  # True
print(a < b)  # True
print(a <= b)  # True
print(a > b)  # False
print(a >= b)  # False
```

## **6. Преобразование в другие системы счисления**

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

## **7. Математические функции**

```python
import math

x = -10
print(math.fabs(x))  # 10.0 (модуль, возвращает float)
print(math.factorial(5))  # 120 (5!)
print(math.gcd(48, 18))  # 6 (НОД)
print(math.lcm(12, 18))  # 36 (НОК, Python 3.9+)
```

## **8. Операции присваивания**

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

## **9. Проверка типа и преобразования**

```python
x = 42

print(type(x))  # <class 'int'>
print(isinstance(x, int))  # True
print(float(x))  # 42.0
print(complex(x))  # (42+0j)
print(str(x))  # "42"
print(bool(x))  # True (кроме 0)
```

## **10. Особенности int в Python 3**

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

## **1. Создание float**

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

## **2. Арифметические операции**

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

## **3. Методы класса float**

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

## **4. Сравнение чисел с плавающей точкой**

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

## **5. Проверка специальных значений**

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

## **6. Округление**

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

## **7. Математические функции из модуля math**

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

## **8. Преобразование типов**

```python
x = 3.14

print(int(x))  # 3 (отбрасывает дробную часть)
print(str(x))  # "3.14"
print(repr(x))  # "3.14"
print(bool(x))  # True (кроме 0.0)
print(complex(x))  # (3.14+0j)
```

## **9. Операции присваивания**

```python
x = 5.5
x += 2.2  # 7.7
x -= 1.1  # 6.6
x *= 2  # 13.2
x /= 4  # 3.3
x //= 2  # 1.0
x **= 3  # 1.0
```

## **10. Форматирование вывода**

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

## **11. Полезные константы**

```python
import math

print(math.pi)  # 3.141592653589793
print(math.e)  # 2.718281828459045
print(math.tau)  # 6.283185307179586 (2*pi)
print(math.inf)  # inf
print(-math.inf)  # -inf
print(math.nan)  # nan
```

## **12. Особенности float**

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

## **13. Десятичные дроби для точных вычислений**

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

## **14. Полезные функции**

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

## **15. Сортировка и сравнение**

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

## **1. Импорт и создание Decimal**

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

## **2. Настройка контекста (точности и округления)**

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

## **3. Арифметические операции**

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

## **4. Методы Decimal**

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

## **5. Математические функции**

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

## **6. Специальные значения и проверки**

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

## **7. Сравнение с другими типами**

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

## **8. Работа с денежными значениями**

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

## **9. Форматирование вывода**

```python
d = Decimal('1234.5678')

# Форматирование с помощью format()
print(f"{d:,.2f}")  # 1,234.57
print(f"{d:.0f}")  # 1235
print(f"{d:.4e}")  # 1.2346e+03

# Использование to_eng_string()
print(d.to_eng_string())  # 1234.5678
```

## **10. Полезные константы**

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

## **11. Обработка исключений**

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

## **12. Примеры использования**

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

## **1. Создание комплексных чисел**

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

## **2. Атрибуты комплексного числа (только для чтения)**

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

## **3. Методы типа complex**

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

## **4. Таблица методов и атрибутов complex**

| Атрибут/Метод | Тип/Сигнатура            | Описание                         | Пример (результат)                                                                              |
|:--------------|:-------------------------|:---------------------------------|:------------------------------------------------------------------------------------------------|
| **Атрибуты**  | `z.real`                 | `float`                          | Действительная часть числа (только чтение).                                                     | `(3+4j).real` → `3.0` |
|               | `z.imag`                 | `float`                          | Мнимая часть числа (только чтение).                                                             | `(3-4j).imag` → `-4.0` |
| **Методы**    | `z.conjugate()`          | `complex`                        | Возвращает комплексно-сопряжённое число.                                                        | `(1+2j).conjugate()` → `(1-2j)` |
| **Создание**  | `complex(x[, y])`        | Конструктор                      | Создаёт число из действительной (`x`) и мнимой (`y`) части. Если `y` не указан, по умолчанию 0. | `complex(2, 3)` → `(2+3j)` |
|               | `complex.from_number(x)` | Статический метод (Python 3.14+) | Безопасное преобразование объектов в `complex`.                                                 | `complex.from_number(5)` → `(5+0j)` |

## **5. Поддерживаемые и неподдерживаемые операции**

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

## **6. Операции сравнения**

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

## **7. Преобразование в другие типы и форматирование**

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

## **8. Отличия от модуля cmath**

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

## **1. Импорт и создание Fraction**

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

## **2. Арифметические операции**

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

## **3. Атрибуты и методы экземпляра Fraction**

### **3.1. Базовые атрибуты (только для чтения)**

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

### **3.2. as_integer_ratio() - представление в виде пары целых чисел**

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

### **3.3. is_integer() - проверка, является ли число целым**

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

### **3.4. limit_denominator() - рациональное приближение**

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

## **4. Методы округления и преобразования**

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

## **5. Форматирование вывода (Python 3.12+)**

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

## **6. Проверки и сравнения**

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

## **7. Примеры практического использования**

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

## **8. Важные особенности и ограничения**

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

## **1. Создание булевых значений**

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

## **2. Арифметические операции**

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

## **3. Методы типа bool**

### **3.1. as_integer_ratio() - представление в виде дроби**

```python
print(True.as_integer_ratio())  # (1, 1)
print(False.as_integer_ratio())  # (0, 1)

# Полезно для некоторых математических операций
ratio = True.as_integer_ratio()
print(f"True как дробь: {ratio[0]}/{ratio[1]}")
```

### **3.2. bit_length() - минимальное количество бит для представления**

```python
print(True.bit_length())  # 1 (для 1 нужен 1 бит)
print(False.bit_length())  # 0 (0 представляется пустой битовой строкой)

# Сравнение с int
print((1).bit_length())  # 1
print((0).bit_length())  # 0
```

### **3.3. conjugate() - комплексное сопряжение**

```python
# Для совместимости с int (возвращает self)
print(True.conjugate())  # 1
print(False.conjugate())  # 0

# Фактически то же самое, что и:
print(True.real)  # 1
print(True.imag)  # 0
```

### **3.4. Магические методы для операторов сравнения**

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

### **3.5. Магические методы для логических операторов**

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

## **4. Специальные методы bool**

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

## **5. Преобразование в другие типы**

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

## **6. Операции сравнения**

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

## **7. Проверка типа и идентичности**

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

## **8. Важные особенности**

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

## **9. Примеры использования**

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

## **1. Создание объекта None**

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

## **2. Атрибуты типа NoneType**

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

## **3. Методы типа NoneType**

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

## **5. Поддерживаемые и неподдерживаемые операции**

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

## **6. Операции сравнения**

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

## **7. Преобразование в другие типы и форматирование**

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

## **8. Отличия от других "пустых" значений**

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

[Содержание](/CONTENTS.md#содержание)

---

## list

[Содержание](/CONTENTS.md#содержание)

---

## tuple

[Содержание](/CONTENTS.md#содержание)

---

## dict

[Содержание](/CONTENTS.md#содержание)

---

## set

[Содержание](/CONTENTS.md#содержание)

---

## frozenset

[Содержание](/CONTENTS.md#содержание)

---

## bytes

[Содержание](/CONTENTS.md#содержание)

---

## bytearray

[Содержание](/CONTENTS.md#содержание)

---

## namedtuple

[Содержание](/CONTENTS.md#содержание)

---

## deque

[Содержание](/CONTENTS.md#содержание)

---

## Counter

[Содержание](/CONTENTS.md#содержание)

---

## defaultdict

[Содержание](/CONTENTS.md#содержание)

---

## OrderedDict

[Содержание](/CONTENTS.md#содержание)

---

## NamedTuple

[Содержание](/CONTENTS.md#содержание)

---

## array

[Содержание](/CONTENTS.md#содержание)

---

## SimpleNamespace

[Содержание](/CONTENTS.md#содержание)

---

## Path

[Содержание](/CONTENTS.md#содержание)

---

## UUID

[Содержание](/CONTENTS.md#содержание)

---

## datetime

[Содержание](/CONTENTS.md#содержание)

---

## Enum

[Содержание](/CONTENTS.md#содержание)

---

## dataclass

[Содержание](/CONTENTS.md#содержание)

---

## re.Pattern & re.Match

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