Отличная идея! Вот подробные примеры для каждой встроенной функции Python:

📊 1. Числовые функции

abs() — абсолютное значение

```python
abs(-5)       # 5
abs(3.14)     # 3.14
abs(0)        # 0
abs(-10.5)    # 10.5
```

divmod() — частное и остаток

```python
divmod(10, 3)     # (3, 1) - 10 ÷ 3 = 3, остаток 1
divmod(20, 7)     # (2, 6)
divmod(15, 5)     # (3, 0)
```

pow() — степень

```python
pow(2, 3)        # 8 (2³)
pow(5, 2)        # 25
pow(2, 3, 3)     # 2 (2³ % 3 = 8 % 3 = 2)
```

round() — округление

```python
round(3.14159, 2)    # 3.14
round(2.675, 2)      # 2.67 (осторожно с float!)
round(123.456)       # 123
round(123.456, -1)   # 120.0
```

🔄 2. Системные и объектные функции

id() — идентификатор объекта в памяти

```python
x = [1, 2, 3]
y = [1, 2, 3]
id(x) == id(y)     # False - разные объекты
z = x
id(x) == id(z)     # True - один объект
```

hash() — хеш-значение

```python
hash("hello")           # -8683395998751147536
hash((1, 2, 3))         # 529344067295497451
# hash([1, 2, 3])       # Ошибка! Список нехешируемый
```

memoryview() — представление памяти

```python
data = bytearray(b"hello")
mv = memoryview(data)
print(mv[0])           # 104 (код 'h')
mv[0] = 106            # Изменяем через memoryview
print(data)            # bytearray(b'jello')
```

object() — базовый объект

```python
obj = object()
type(obj)              # <class 'object'>
# obj.x = 1           # Ошибка! Нельзя добавить атрибуты
```

🔤 3. Строковые и символьные функции

chr() и ord() — символы и коды

```python
chr(65)                # 'A'
ord('A')               # 65
chr(128512)            # '😀'
ord('😀')              # 128512
chr(0x41)              # 'A' (hex)
```

ascii() — ASCII представление

```python
ascii("Привет")        # "'\\u041f\\u0440\\u0438\\u0432\\u0435\\u0444'"
ascii("hello")         # "'hello'"
ascii("café")          # "'caf\\xe9'"
```

repr() — строковое представление для отладки

```python
repr("Hello\nWorld")   # "'Hello\\nWorld'"
repr([1, 2, 3])        # '[1, 2, 3]'
repr(3.14)             # '3.14'
str("Hello\nWorld")    # 'Hello\nWorld' (для сравнения)
```

format() — форматирование

```python
format(123, "05d")     # '00123'
format(3.14159, ".2f") # '3.14'
format(255, "x")       # 'ff' (hex)
format(255, "b")       # '11111111' (binary)
format(1000000, ",")   # '1,000,000'
```

🔢 4. Преобразование типов

bin(), oct(), hex() — системы счисления

```python
bin(10)      # '0b1010'
oct(10)      # '0o12'
hex(255)     # '0xff'
int('0b1010', 2)  # 10 (обратное преобразование)
```

complex() — комплексные числа

```python
complex(3, 4)      # (3+4j)
complex('3+4j')    # (3+4j)
complex(5)         # (5+0j)
```

bytearray() и bytes() — байтовые данные

```python
# bytes (неизменяемые)
b = bytes([65, 66, 67])  # b'ABC'
# b[0] = 68              # Ошибка! bytes неизменяемы

# bytearray (изменяемые)
ba = bytearray([65, 66, 67])  # bytearray(b'ABC')
ba[0] = 68                    # Можно изменить
print(ba)                     # bytearray(b'DBC')
```

frozenset() — неизменяемое множество

```python
fs = frozenset([1, 2, 3, 2])
print(fs)        # frozenset({1, 2, 3})
# fs.add(4)      # Ошибка! Неизменяемый
```

🧠 5. Функции высшего порядка

filter() — фильтрация

```python
numbers = [1, -2, 3, -4, 5]
positive = filter(lambda x: x > 0, numbers)
list(positive)  # [1, 3, 5]

# С использованием None (истинные значения)
list(filter(None, [0, 1, False, True, '', 'a']))  # [1, True, 'a']
```

map() — применение функции

```python
nums = [1, 2, 3, 4]
squares = map(lambda x: x**2, nums)
list(squares)  # [1, 4, 9, 16]

# Несколько итерируемых объектов
list(map(pow, [2, 3, 4], [1, 2, 3]))  # [2, 9, 64]
```

zip() — параллельная итерация

```python
names = ['Alice', 'Bob', 'Charlie']
ages = [25, 30, 35]
list(zip(names, ages))  # [('Alice', 25), ('Bob', 30), ('Charlie', 35)]

# Разной длины
list(zip([1, 2, 3], ['a', 'b']))  # [(1, 'a'), (2, 'b')]

# Распаковка (транспонирование матрицы)
matrix = [[1, 2, 3], [4, 5, 6]]
list(zip(*matrix))  # [(1, 4), (2, 5), (3, 6)]
```

🔄 6. Итераторы и генераторы

iter() и next() — ручное управление итерацией

```python
numbers = [1, 2, 3]
iterator = iter(numbers)
print(next(iterator))  # 1
print(next(iterator))  # 2
print(next(iterator))  # 3
# print(next(iterator))  # StopIteration (ошибка)

# С значением по умолчанию
iterator = iter([1])
print(next(iterator, 'конец'))  # 1
print(next(iterator, 'конец'))  # 'конец'
```

enumerate() — итерация с индексами

```python
fruits = ['apple', 'banana', 'cherry']
for i, fruit in enumerate(fruits):
    print(f"{i}: {fruit}")
# 0: apple
# 1: banana
# 2: cherry

# С начальным индексом
list(enumerate(fruits, start=1))  # [(1, 'apple'), (2, 'banana'), (3, 'cherry')]
```

reversed() — обратный порядок

```python
list(reversed([1, 2, 3]))      # [3, 2, 1]
list(reversed("hello"))        # ['o', 'l', 'l', 'e', 'h']

# Для строк лучше использовать срез
"hello"[::-1]                  # 'olleh'
```

slice() — объект среза

```python
items = [0, 1, 2, 3, 4, 5, 6]
s = slice(1, 5, 2)  # start=1, stop=5, step=2
items[s]            # [1, 3] (эквивалент items[1:5:2])

# Можно использовать многократно
slicer = slice(None, None, -1)  # [::-1]
items[slicer]        # [6, 5, 4, 3, 2, 1, 0]
```

🏗️ 7. Функции для классов и объектов

classmethod() и staticmethod()

```python
class MyClass:
    value = "class value"
    
    def instance_method(self):
        return f"Instance: {self.value}"
    
    @classmethod
    def class_method(cls):
        return f"Class: {cls.value}"
    
    @staticmethod
    def static_method():
        return "Static: no access to class or instance"

obj = MyClass()
obj.instance_method()  # 'Instance: class value'
MyClass.class_method() # 'Class: class value'
MyClass.static_method() # 'Static: no access to class or instance'
```

property() — управляемые атрибуты

```python
class Circle:
    def __init__(self, radius):
        self._radius = radius
    
    @property
    def radius(self):
        print("Getting radius")
        return self._radius
    
    @radius.setter
    def radius(self, value):
        print(f"Setting radius to {value}")
        if value < 0:
            raise ValueError("Radius cannot be negative")
        self._radius = value
    
    @radius.deleter
    def radius(self):
        print("Deleting radius")
        del self._radius

c = Circle(5)
print(c.radius)    # Getting radius → 5
c.radius = 10      # Setting radius to 10
# c.radius = -1    # ValueError
```

super() — доступ к родительскому классу

```python
class Parent:
    def __init__(self):
        print("Parent init")
    
    def method(self):
        return "Parent method"

class Child(Parent):
    def __init__(self):
        super().__init__()  # Вызов конструктора родителя
        print("Child init")
    
    def method(self):
        parent_result = super().method()
        return f"Child method → {parent_result}"

c = Child()
# Parent init
# Child init
print(c.method())  # Child method → Parent method
```

🔍 8. Функции интроспекции

type() — определение типа

```python
type(42)                # <class 'int'>
type("hello")           # <class 'str'>
type([1, 2, 3])         # <class 'list'>
type(lambda x: x*2)     # <class 'function'>

# Создание класса динамически
MyClass = type('MyClass', (), {'x': 42})
obj = MyClass()
obj.x  # 42
```

isinstance() и issubclass()

```python
class Animal: pass
class Dog(Animal): pass
class Cat(Animal): pass

dog = Dog()
isinstance(dog, Dog)      # True
isinstance(dog, Animal)   # True (наследование)
isinstance(dog, Cat)      # False

issubclass(Dog, Animal)   # True
issubclass(Dog, object)   # True (все классы от object)
```

callable() — проверка на вызываемость

```python
callable(print)           # True
callable(len)             # True
callable("hello")         # False
callable(lambda: None)    # True
callable(int)             # True (классы вызываемы)

class CallableClass:
    def __call__(self):
        return "Called!"
    
obj = CallableClass()
callable(obj)            # True
obj()                    # 'Called!'
```

hasattr(), getattr(), setattr(), delattr()

```python
class Person:
    def __init__(self, name):
        self.name = name

p = Person("Alice")

hasattr(p, 'name')       # True
hasattr(p, 'age')        # False

getattr(p, 'name')       # 'Alice'
getattr(p, 'age', 25)    # 25 (значение по умолчанию)

setattr(p, 'age', 30)    # Устанавливаем атрибут
p.age                    # 30

delattr(p, 'age')        # Удаляем атрибут
# p.age                  # AttributeError
```

📁 9. Работа с пространствами имен

globals() и locals()

```python
x = 10
y = 20

def test():
    a = 1
    b = 2
    print("Локальные:", locals())   # {'a': 1, 'b': 2}
    print("Глобальные:", globals().get('x'))  # 10

test()
print("Все глобальные:", list(globals().keys())[:5])  # ['__name__', '__doc__', ...]
```

vars() — атрибуты объекта

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

p = Point(3, 4)
vars(p)                # {'x': 3, 'y': 4}

# Можно изменять
vars(p)['z'] = 5
p.z                    # 5
```

dir() — список атрибутов

```python
import math
dir(math)[:5]          # ['__doc__', '__loader__', '__name__', ...]

class Example:
    def __init__(self):
        self.value = 42
    
    def method(self):
        pass

obj = Example()
dir(obj)[:5]           # ['__class__', '__delattr__', '__dict__', ...]
```

⚡ 10. Выполнение кода

eval() и exec()

```python
# eval - вычисляет выражение
result = eval("2 + 3 * 4")      # 14
result = eval("'hello'.upper()")  # 'HELLO'

# exec - выполняет код
exec("""
x = 10
y = 20
print(x + y)  # 30
""")

# Разница
eval("print('Hello')")    # Ошибка! print - statement
exec("print('Hello')")    # Hello
```

compile() — компиляция кода

```python
# Компиляция выражения
code = compile("x + y", "<string>", "eval")
x, y = 5, 3
eval(code)  # 8

# Компиляция нескольких выражений
code = compile("""
for i in range(3):
    print(i)
""", "<string>", "exec")
exec(code)  # 0 1 2
```

🐞 11. Отладка и справка

breakpoint() — отладчик

```python
def buggy_function():
    x = 10
    y = 0
    breakpoint()  # Запустит отладчик pdb
    result = x / y
    return result

# buggy_function()  # Раскомментируйте для отладки
```

help() — встроенная справка

```python
help(print)        # Показывает документацию print
help(str.upper)    # Показывает документацию метода
help('modules')    # Список всех модулей
help('keywords')   # Список ключевых слов Python
```

🔄 12. Асинхронные функции (Python 3.10+)

aiter() и anext()

```python
import asyncio

async def async_gen():
    for i in range(3):
        await asyncio.sleep(0.1)
        yield i

async def main():
    ag = aiter(async_gen())  # Асинхронный итератор
    print(await anext(ag))   # 0
    print(await anext(ag))   # 1
    print(await anext(ag))   # 2
    # print(await anext(ag)) # StopAsyncIteration

# asyncio.run(main())
```

🎯 13. Практические примеры комбинаций

Сортировка сложных структур

```python
data = [
    {'name': 'Alice', 'age': 25},
    {'name': 'Bob', 'age': 20},
    {'name': 'Charlie', 'age': 25}
]

# Сортировка по нескольким ключам
sorted_data = sorted(data, key=lambda x: (x['age'], x['name']))
```

Валидация данных

```python
def validate_user_data(data):
    # Все обязательные поля присутствуют?
    required = ['name', 'email', 'age']
    if not all(field in data for field in required):
        return False
    
    # Хотя бы один контактный метод?
    contacts = ['phone', 'telegram', 'whatsapp']
    if not any(contact in data for contact in contacts):
        return False
    
    return True

user = {'name': 'John', 'email': 'john@example.com', 'age': 30, 'phone': '123456'}
validate_user_data(user)  # True
```

Генератор конфигурации

```python
config_template = {
    'host': 'localhost',
    'port': 8080,
    'debug': True
}

# Динамическое обновление конфигурации
overrides = {'port': 9000, 'timeout': 30}
config = {**config_template, **overrides}
# {'host': 'localhost', 'port': 9000, 'debug': True, 'timeout': 30}
```

💡 Важные советы:

1. isinstance() предпочтительнее type() для проверки типов
2. Используйте getattr() с default чтобы избежать AttributeError
3. map() и filter() возвращают генераторы — оборачивайте в list() при необходимости
4. eval() опасен — никогда не используйте с пользовательским вводом
5. property() делает код чище, чем геттеры/сеттеры вручную
6. functools.lru_cache полезен для меморизации, но не встроен
