# Python Interview Questions & Answers

## Core Python Concepts

### 1. Data Types & Structures

**Q: What are the built-in data types in Python?**
- **Numeric**: int, float, complex, bool
- **Sequence**: list, tuple, range, str
- **Mapping**: dict
- **Set**: set, frozenset
- **None**: NoneType

**Q: Difference between list and tuple?**
- **List**: Mutable, slower, uses more memory, `[1, 2, 3]`
- **Tuple**: Immutable, faster, less memory, `(1, 2, 3)`
- **Use cases**: List for dynamic data, tuple for fixed collections, dictionary keys

**Q: What is the difference between `==` and `is`?**
- `==`: Equality comparison (values)
- `is`: Identity comparison (memory addresses)
- Example: `a = [1, 2]; b = [1, 2]; a == b` is `True`, but `a is b` is `False`

**Q: Explain shallow vs deep copy**
```python
import copy

# Create a nested list to demonstrate copying behavior
original = [[1, 2], [3, 4]]
shallow = copy.copy(original)      # Copies outer list only
deep = copy.deepcopy(original)     # Copies all nested objects

# Modify the original to see the difference
original[0][0] = 99
# shallow[0][0] becomes 99 (shared reference)
# deep[0][0] remains 1 (independent copy)
```

### 2. Functions & Scope

**Q: What are Python's variable scopes?**
- **LEGB Rule**: Local → Enclosing → Global → Built-in
- **Local**: Inside function
- **Enclosing**: In nested functions
- **Global**: Module level
- **Built-in**: Python built-ins

**Q: What are closures?**
```python
# Closure: Inner function remembers outer function's variables
def outer_func(x):
    def inner_func(y):
        return x + y  # x is captured from outer scope
    return inner_func

# Create a closure that adds 5 to any number
add_five = outer_func(5)
result = add_five(3)  # Returns 8
```

**Q: Explain `*args` and `**kwargs`**
```python
# Function that accepts variable number of arguments
def func(*args, **kwargs):
    print(args)      # Tuple of positional arguments
    print(kwargs)    # Dictionary of keyword arguments

# Call with both positional and keyword arguments
func(1, 2, 3, name="Alice", age=25)
# Output: (1, 2, 3)
#         {'name': 'Alice', 'age': 25}
```

### 3. Object-Oriented Programming

**Q: What are dunder methods?**
```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age
    
    def __str__(self):
        # String representation for users
        return f"{self.name} ({self.age})"
    
    def __eq__(self, other):
        # Equality comparison between Person objects
        return self.name == other.name and self.age == other.age
    
    def __add__(self, years):
        # Add years to person's age
        return Person(self.name, self.age + years)
```

**Q: What is the difference between `@staticmethod` and `@classmethod`?**
```python
class MyClass:
    @staticmethod
    def static_method():
        # No access to instance or class
        return "No access to instance or class"
    
    @classmethod
    def class_method(cls):
        # Access to class but not instance
        return f"Access to class: {cls}"
    
    def instance_method(self):
        # Access to instance and class
        return f"Access to instance: {self}"
```

**Q: Explain multiple inheritance and MRO**
```python
# Multiple Inheritance and Method Resolution Order (MRO)
class A:
    def method(self):
        print("A")

class B(A):
    def method(self):
        print("B")

class C(A):
    def method(self):
        print("C")

class D(B, C):  # Multiple inheritance
    pass

d = D()
d.method()  # Prints "B" (MRO: D -> B -> C -> A -> object)
```

## Advanced Python Concepts

### 4. Decorators & Metaclasses

**Q: How do decorators work?**
```python
# Decorator that measures function execution time
def timing_decorator(func):
    import time
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(f"{func.__name__} took {end - start:.2f} seconds")
        return result
    return wrapper

@timing_decorator
def slow_function():
    time.sleep(1)

# Property decorator for computed attributes
class Circle:
    def __init__(self, radius):
        self.radius = radius
    
    @property
    def area(self):
        # Computed property: calculates area on access
        return 3.14 * self.radius ** 2
```

**Q: What are metaclasses?**
```python
# Metaclass for implementing Singleton pattern
class SingletonMeta(type):
    _instances = {}
    
    def __call__(cls, *args, **kwargs):
        # Create only one instance per class
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class Singleton(metaclass=SingletonMeta):
    def __init__(self):
        self.value = 0

# All instances will be the same object
s1 = Singleton()
s2 = Singleton()
print(s1 is s2)  # True
```

### 5. Generators & Iterators

**Q: Explain generators and yield**
```python
# Generator function that yields Fibonacci numbers
def fibonacci_generator(n):
    a, b = 0, 1
    for _ in range(n):
        yield a  # Pause execution and return value
        a, b = b, a + b

# Generator expression (memory efficient)
squares = (x**2 for x in range(10))

# Context manager implemented with generator
from contextlib import contextmanager

@contextmanager
def file_manager(filename, mode):
    f = open(filename, mode)
    try:
        yield f  # Pass control to with block
    finally:
        f.close()  # Always executed
```

**Q: What is the iterator protocol?**
```python
# Custom iterator that counts down
class Countdown:
    def __init__(self, start):
        self.start = start
    
    def __iter__(self):
        # Return iterator object (self in this case)
        return self
    
    def __next__(self):
        # Return next value or raise StopIteration
        if self.start <= 0:
            raise StopIteration
        self.start -= 1
        return self.start + 1
```

### 6. Concurrency & Parallelism

**Q: Difference between threading and multiprocessing?**
- **Threading**: Shared memory, GIL limitations, good for I/O-bound
- **Multiprocessing**: Separate memory, bypasses GIL, good for CPU-bound

```python
import threading
import multiprocessing
import time

def cpu_bound_task(n):
    count = 0
    for i in range(n):
        count += i
    return count

# Threading (not effective for CPU-bound due to GIL)
threads = []
start = time.time()
for _ in range(4):
    t = threading.Thread(target=cpu_bound_task, args=(10**7,))
    threads.append(t)
    t.start()
for t in threads:
    t.join()
print(f"Threading: {time.time() - start:.2f}s")

# Multiprocessing (effective for CPU-bound)
processes = []
start = time.time()
for _ in range(4):
    p = multiprocessing.Process(target=cpu_bound_task, args=(10**7,))
    processes.append(p)
    p.start()
for p in processes:
    p.join()
print(f"Multiprocessing: {time.time() - start:.2f}s")
```

**Q: What is asyncio?**
```python
import asyncio

async def fetch_data(url):
    await asyncio.sleep(1)  # Simulate I/O operation
    return f"Data from {url}"

async def main():
    tasks = [fetch_data(f"url{i}") for i in range(5)]
    results = await asyncio.gather(*tasks)
    return results

# Run async code
asyncio.run(main())
```

### 7. Memory Management & Performance

**Q: How does Python's garbage collection work?**
- **Reference counting**: Automatic cleanup when reference count = 0
- **Generational GC**: Handles circular references
- **gc module**: Manual control over garbage collection

```python
import gc
import weakref

class MyClass:
    def __del__(self):
        print("Object deleted")

# Weak references don't increase reference count
obj = MyClass()
weak_ref = weakref.ref(obj)

print(weak_ref())  # Returns object if still alive
del obj
print(weak_ref())  # Returns None if object was deleted
```

**Q: What are Python's optimization techniques?**
```python
# List comprehension vs map/filter
squares = [x**2 for x in range(1000)]  # Faster and more readable

# Generator for memory efficiency
def process_large_file(filename):
    with open(filename) as f:
        for line in f:  # Processes one line at a time
            yield process_line(line)

# __slots__ for memory optimization
class Point:
    __slots__ = ['x', 'y']  # Prevents __dict__ creation
    def __init__(self, x, y):
        self.x = x
        self.y = y
```

## Practical Coding Questions

### 8. Algorithm Implementations

**Q: Implement a binary search**
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

**Q: Implement a decorator that caches function results**
```python
def memoize(func):
    cache = {}
    
    def wrapper(*args):
        if args not in cache:
            cache[args] = func(*args)
        return cache[args]
    
    return wrapper

@memoize
def fibonacci(n):
    if n <= 1:
        return n
    return fibonacci(n-1) + fibonacci(n-2)
```

**Q: Implement a context manager for database connections**
```python
class DatabaseConnection:
    def __init__(self, db_config):
        self.db_config = db_config
        self.connection = None
    
    def __enter__(self):
        # Simulate database connection
        self.connection = f"Connected to {self.db_config['database']}"
        print("Database connection opened")
        return self.connection
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        # Close connection
        self.connection = None
        print("Database connection closed")
        return False  # Don't suppress exceptions

# Usage
with DatabaseConnection({'database': 'mydb'}) as conn:
    print(f"Using connection: {conn}")
```

### 9. Design Patterns

**Q: Implement Singleton pattern**
```python
class Singleton:
    _instance = None
    _initialized = False
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance
    
    def __init__(self):
        if not self._initialized:
            self.value = 0
            Singleton._initialized = True
```

**Q: Implement Observer pattern**
```python
class Subject:
    def __init__(self):
        self._observers = []
    
    def attach(self, observer):
        self._observers.append(observer)
    
    def detach(self, observer):
        self._observers.remove(observer)
    
    def notify(self, message):
        for observer in self._observers:
            observer.update(message)

class Observer:
    def update(self, message):
        raise NotImplementedError
```

## Python Standard Library

### 10. Common Modules

**Q: What are your most used standard library modules?**
- **collections**: Counter, defaultdict, OrderedDict, deque
- **itertools**: chain, combinations, permutations, product
- **functools**: reduce, lru_cache, partial
- **datetime**: date, time, datetime, timedelta
- **json**: JSON serialization/deserialization
- **re**: Regular expressions
- **os**: Operating system interface
- **sys**: System-specific parameters

```python
from collections import Counter, defaultdict, deque
from itertools import combinations, chain
from functools import lru_cache
import json

# Counter for counting elements
word_count = Counter("hello world hello".split())

# defaultdict for missing keys
dd = defaultdict(list)
dd['key'].append('value')

# deque for efficient appends/pops
dq = deque([1, 2, 3])
dq.appendleft(0)
dq.pop()

# itertools combinations
pairs = list(combinations([1, 2, 3, 4], 2))

# lru_cache decorator
@lru_cache(maxsize=128)
def expensive_function(x):
    return x ** 2
```

## Testing & Debugging

### 11. Testing Best Practices

**Q: How do you test Python code?**
```python
import unittest
from unittest.mock import patch, MagicMock

class TestMathFunctions(unittest.TestCase):
    def test_addition(self):
        self.assertEqual(add(2, 3), 5)
    
    def test_division_by_zero(self):
        with self.assertRaises(ZeroDivisionError):
            divide(10, 0)
    
    @patch('module.external_api_call')
    def test_with_mock(self, mock_api):
        mock_api.return_value = {"status": "success"}
        result = process_data()
        self.assertTrue(result['success'])

# Using pytest
def test_example():
    assert add(2, 3) == 5

@pytest.fixture
def sample_data():
    return {"key": "value"}

def test_with_fixture(sample_data):
    assert sample_data["key"] == "value"
```

**Q: What debugging techniques do you use?**
- **print() statements**: Quick and simple
- **pdb module**: Interactive debugging
- **logging**: Structured debugging information
- **IDE debuggers**: Visual debugging

```python
import pdb
import logging

# pdb debugging
def problematic_function(x):
    pdb.set_trace()  # Breakpoint
    result = x / 0
    return result

# Logging setup
logging.basicConfig(level=logging.DEBUG)
logger = logging.getLogger(__name__)

def logged_function(x):
    logger.debug(f"Starting with x={x}")
    try:
        result = x * 2
        logger.info(f"Result: {result}")
        return result
    except Exception as e:
        logger.error(f"Error: {e}")
        raise
```

## Python 3.x Features

### 12. Modern Python Features

**Q: What are type hints and how do you use them?**
```python
from typing import List, Dict, Optional, Union, Callable
from dataclasses import dataclass

def process_items(items: List[str]) -> Dict[str, int]:
    return {item: len(item) for item in items}

def optional_param(name: str, age: Optional[int] = None) -> str:
    return f"{name}: {age or 'Unknown'}"

def union_type(value: Union[int, float, str]) -> str:
    return str(value)

@dataclass
class Person:
    name: str
    age: int
    email: Optional[str] = None
```

**Q: What are f-strings and format specifications?**
```python
name = "Alice"
age = 25

# Basic f-string
message = f"{name} is {age} years old"

# Format specifications
price = 19.99
formatted = f"Price: ${price:.2f}"

# Expression evaluation
result = f"2 + 2 = {2 + 2}"

# Dictionary access
person = {"name": "Bob", "age": 30}
info = f"{person['name']} is {person['age']}"

# Method calls
text = "hello world"
uppercase = f"Uppercase: {text.upper()}"
```

**Q: What are async/await and context managers?**
```python
import aiofiles
import asyncio

async def async_file_reader(filename: str):
    async with aiofiles.open(filename) as f:
        content = await f.read()
        return content

# Async context manager
class AsyncTimer:
    async def __aenter__(self):
        self.start = asyncio.get_event_loop().time()
        return self
    
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        self.end = asyncio.get_event_loop().time()
        print(f"Elapsed: {self.end - self.start:.2f}s")
```

## Framework & Ecosystem

### 13. Web Development

**Q: Django vs Flask?**
- **Django**: Batteries-included, ORM, admin panel, large applications
- **Flask**: Minimal, flexible, small applications, microservices

**Q: What are decorators in Flask/Django?**
```python
# Flask route decorator
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route('/api/users', methods=['GET', 'POST'])
def users():
    if request.method == 'POST':
        return create_user(request.json)
    return get_users()

# Django view decorator
from django.http import JsonResponse
from django.views.decorators.http import require_http_methods
from django.views.decorators.csrf import csrf_exempt

@require_http_methods(["GET", "POST"])
@csrf_exempt
def api_view(request):
    if request.method == 'POST':
        return JsonResponse({"status": "created"})
    return JsonResponse({"status": "ok"})
```

### 14. Data Science & Machine Learning

**Q: What Python libraries do you use for data science?**
- **NumPy**: Numerical computing, arrays
- **Pandas**: Data manipulation, DataFrames
- **Matplotlib/Seaborn**: Data visualization
- **Scikit-learn**: Machine learning algorithms
- **TensorFlow/PyTorch**: Deep learning

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from sklearn.linear_model import LinearRegression

# NumPy operations
arr = np.array([1, 2, 3, 4, 5])
squared = arr ** 2

# Pandas DataFrame
df = pd.DataFrame({
    'name': ['Alice', 'Bob', 'Charlie'],
    'age': [25, 30, 35],
    'salary': [50000, 60000, 70000]
})

# Simple linear regression
X = df[['age']].values
y = df['salary'].values
model = LinearRegression()
model.fit(X, y)
```

## Best Practices & Code Quality

### 15. PEP 8 & Code Style

**Q: What are the key PEP 8 guidelines?**
- **Naming**: snake_case for variables/functions, PascalCase for classes
- **Line length**: Maximum 79 characters
- **Imports**: At top, grouped by standard library, third-party, local
- **Spacing**: Around operators, after commas
- **Documentation**: Docstrings for all public modules/functions/classes

```python
# Good example
import os
import sys
from typing import List, Optional

def calculate_average(numbers: List[float]) -> Optional[float]:
    """
    Calculate the average of a list of numbers.
    
    Args:
        numbers: List of numerical values
        
    Returns:
        Average value or None if list is empty
    """
    if not numbers:
        return None
    return sum(numbers) / len(numbers)


class DataProcessor:
    """Class for processing numerical data."""
    
    def __init__(self, data: List[float]):
        self.data = data
        self.processed = False
    
    def process(self) -> bool:
        """Process the data and return success status."""
        try:
            # Processing logic here
            self.processed = True
            return True
        except Exception as e:
            print(f"Processing failed: {e}")
            return False
```

### 16. Error Handling

**Q: What are Python's exception handling best practices?**
```python
# Specific exception handling
try:
    result = divide_numbers(a, b)
except ZeroDivisionError:
    logger.error("Division by zero attempted")
    result = None
except TypeError as e:
    logger.error(f"Invalid types: {e}")
    raise
else:
    logger.info(f"Division successful: {result}")
finally:
    cleanup_resources()

# Custom exceptions
class InvalidInputError(Exception):
    """Raised when input validation fails."""
    pass

class DatabaseConnectionError(Exception):
    """Raised when database connection fails."""
    pass

def validate_input(value):
    if not isinstance(value, (int, float)):
        raise InvalidInputError(f"Expected number, got {type(value)}")
    if value < 0:
        raise InvalidInputError("Value must be non-negative")
    return value
```

## Performance Optimization

### 17. Profiling & Optimization

**Q: How do you profile Python code?**
```python
import cProfile
import timeit
from memory_profiler import profile

# Time-based profiling
def profile_function():
    pr = cProfile.Profile()
    pr.enable()
    
    # Code to profile
    result = expensive_computation()
    
    pr.disable()
    pr.print_stats(sort='cumulative')
    return result

# Timeit for small snippets
execution_time = timeit.timeit(
    'sum(range(1000))',
    number=1000
)

# Memory profiling
@profile
def memory_intensive_function():
    large_list = [i for i in range(100000)]
    return sum(large_list)
```

**Q: What are common performance optimization techniques?**
```python
# Use built-in functions (implemented in C)
fast_sum = sum(range(1000000))  # Fast
slow_sum = 0
for i in range(1000000):  # Slow
    slow_sum += i

# List comprehensions vs loops
squares = [x**2 for x in range(1000)]  # Fast
squares_slow = []
for x in range(1000):  # Slow
    squares_slow.append(x**2)

# Generator expressions for memory efficiency
total = sum(x**2 for x in range(1000000))  # Memory efficient

# Use appropriate data structures
# Set for membership testing (O(1))
valid_items = {1, 2, 3, 4, 5}
if item in valid_items:  # Fast
    pass

# List for membership testing (O(n))
valid_items = [1, 2, 3, 4, 5]
if item in valid_items:  # Slow
    pass
```

## Interview Tips

### 18. Common Pitfalls

**Q: What are common Python interview mistakes?**
- Not understanding the GIL (Global Interpreter Lock)
- Confusing `==` with `is`
- Not knowing the difference between mutable/immutable types
- Poor understanding of scope and closures
- Not knowing when to use different data structures
- Ignoring exception handling best practices

### 19. Preparation Strategy

**Q: How should I prepare for a Python interview?**
1. **Master fundamentals**: Data types, control flow, functions, OOP
2. **Practice coding**: Implement common algorithms and data structures
3. **Understand internals**: GIL, memory management, garbage collection
4. **Know the standard library**: collections, itertools, functools
5. **Study frameworks**: Django, Flask, FastAPI for web development
6. **Practice system design**: Design patterns, architecture decisions
7. **Review your projects**: Be ready to discuss your code and decisions

### 20. Sample Questions to Practice

1. Implement a decorator that measures execution time
2. Create a class that behaves like a dictionary
3. Write a function to find duplicate elements in a list
4. Implement a binary tree with traversal methods
5. Create a simple web server using Flask or FastAPI
6. Write a context manager for file operations
7. Implement a rate limiter using decorators
8. Create a generator that yields prime numbers
9. Write a function to merge two sorted lists
10. Implement a simple caching mechanism

## Resources for Further Learning

- **Official Documentation**: docs.python.org
- **Effective Python**: Brett Slatkin
- **Fluent Python**: Luciano Ramalho
- **Python Cookbook**: David Beazley & Brian K. Jones
- **Real Python**: realpython.com
- **Python Weekly Newsletter**: pythonweekly.com
- **Exercism**: exercism.org (Python track)
- **LeetCode**: Practice algorithmic problems
- **HackerRank**: Python coding challenges
