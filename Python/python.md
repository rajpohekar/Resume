# ⚡ PYTHON — INTERVIEW REVISION CHEAT SHEET
> **Goal:** 30-minute pre-interview revision. Backend handbook + Internals cheat sheet + Last-minute guide.

---

# 1. MOST ASKED PYTHON INTERVIEW TOPICS

### 🔴 Very Frequently Asked
1. Mutable vs Immutable + side effects
2. GIL — what, why, impact
3. `is` vs `==`
4. Mutable default argument trap
5. List vs Tuple vs Set vs Dict (time complexity)
6. Shallow vs Deep copy
7. Generators & `yield`
8. Decorators
9. Pass by object reference
10. List comprehensions
11. `*args` / `**kwargs`
12. Exception handling patterns
13. OOP — `super()`, `classmethod`, `staticmethod`
14. Multithreading vs Multiprocessing

### 🟡 Frequently Asked
- Closures and late binding
- `__init__` vs `__new__`
- `async`/`await` basics
- `map`, `filter`, `lambda`, `reduce`
- Memory management / reference counting
- Duck typing
- Monkey patching
- `__slots__`

### 🟢 Rarely Asked
- Metaclasses
- Descriptors
- Context managers (`__enter__`/`__exit__`)
- `functools.lru_cache`

---

# 2. PYTHON MEMORY MODEL

## Key Concepts

- **Everything is an object** — even integers, booleans, functions
- **Variables are references** (labels), not containers
- **Reference counting** — CPython tracks how many references point to each object
- **Garbage collector** — handles cyclic references (generational GC)

```python
x = [1, 2, 3]
y = x          # y and x reference the SAME list object
y.append(4)
print(x)       # [1, 2, 3, 4] — same object!
```

## Memory Visualization

```
Stack (references):       Heap (objects):
┌────────┐               ┌──────────────┐
│ x ─────────────────→   │  [1,2,3,4]   │
│ y ─────────────────→   │  (list obj)  │
└────────┘               └──────────────┘
     Both point to same object
```

## Reference Counting

```python
import sys
x = [1, 2, 3]
sys.getrefcount(x)   # at least 2 (x + function arg)

del x   # refcount drops; if 0, object freed
```

## Object Identity

```python
x = [1, 2, 3]
y = [1, 2, 3]
x == y   # True  — same value
x is y   # False — different objects

# id() gives memory address:
id(x) == id(y)  # False
```

## Interned Objects ⚠️ Interview Trap

CPython interns small integers (-5 to 256) and some strings:

```python
a = 256; b = 256
a is b  # True  — interned

a = 257; b = 257
a is b  # False — not interned (implementation detail)

s1 = "hello"; s2 = "hello"
s1 is s2  # True  — interned (compile-time string)

s1 = "hello world"; s2 = "hello world"
s1 is s2  # May be False — longer strings may not be interned
```

**Rule:** Never use `is` to compare values — use `==`. `is` checks identity.

---

# 3. MUTABLE VS IMMUTABLE

| Type | Mutable? | Examples |
|---|---|---|
| `int`, `float`, `bool` | ❌ No | `x = 5` |
| `str` | ❌ No | `s = "hello"` |
| `tuple` | ❌ No | `t = (1, 2)` |
| `frozenset` | ❌ No | `fs = frozenset({1,2})` |
| `list` | ✅ Yes | `l = [1, 2]` |
| `dict` | ✅ Yes | `d = {}` |
| `set` | ✅ Yes | `s = {1, 2}` |

## Why interviewers ask this

- Tests understanding of **reference semantics**
- Exposes knowledge of **side effects in functions**
- Tests **default argument trap** knowledge

## Memory behavior

```python
# Immutable — new object created
x = "hello"
x += " world"   # x now points to NEW string object

# Mutable — same object modified
lst = [1, 2]
lst.append(3)   # SAME list object, just changed
```

## Tricky examples

```python
# Tuple with mutable inside — partially mutable!
t = (1, [2, 3])
t[1].append(4)   # ✅ works! List inside is mutable
t[1] = [9]       # ❌ TypeError — can't reassign tuple element

# String "modification"
s = "hello"
s[0] = "H"       # ❌ TypeError — strings immutable
s = "H" + s[1:]  # ✅ creates new string
```

## Common bugs

```python
# Bug: thinking += creates new object for mutables
lst = [1, 2]
def f(l):
    l += [3]    # modifies original list!
f(lst)
print(lst)      # [1, 2, 3] — surprised?

# vs
def g(l):
    l = l + [3]  # creates NEW list, original unchanged
g(lst)
print(lst)       # [1, 2, 3] — unchanged from g
```

---

# 4. LIST vs TUPLE vs SET vs DICTIONARY

| Feature | List | Tuple | Set | Dict |
|---|---|---|---|---|
| Ordered | ✅ (3.7+) | ✅ | ❌ | ✅ (3.7+) |
| Mutable | ✅ | ❌ | ✅ | ✅ |
| Duplicates | ✅ | ✅ | ❌ | Keys: ❌ |
| Indexable | ✅ | ✅ | ❌ | By key |
| Hashable | ❌ | ✅ (if elements hashable) | ❌ | ❌ |
| Memory | High | Low | Medium | High |

## Time Complexity

| Operation | List | Set | Dict | Tuple |
|---|---|---|---|---|
| Access by index | O(1) | N/A | O(1) by key | O(1) |
| Search (`in`) | O(n) | O(1) avg | O(1) avg | O(n) |
| Insert/Append | O(1) amort | O(1) avg | O(1) avg | N/A |
| Delete | O(n) | O(1) avg | O(1) avg | N/A |
| Pop (end) | O(1) | O(1) | O(1) | N/A |

**Key interview point:** Use `set` or `dict` for O(1) lookup; `list` for sequential access.

## Real usage examples

```python
# Tuple: fixed record, dict key, namedtuple
point = (10, 20)
my_dict = {(1, 2): "value"}  # tuple as key ✅

# Set: deduplication, membership test
unique = list(set([1, 2, 2, 3]))

# Dict: frequency count
from collections import Counter
freq = Counter("abracadabra")  # {'a': 5, 'b': 2, ...}

# List: ordered mutable sequence, stack (append/pop)
stack = []
stack.append(1); stack.pop()
```

---

# 5. PYTHON FUNCTION CONCEPTS

## ⚠️ Mutable Default Argument Trap (VERY IMPORTANT)

```python
def f(lst=[]):     # ❌ lst is created ONCE at function definition
    lst.append(1)
    return lst

f()  # [1]
f()  # [1, 1]  ← SURPRISED? Same list object reused!
f()  # [1, 1, 1]

# Fix:
def f(lst=None):   # ✅
    if lst is None:
        lst = []
    lst.append(1)
    return lst
```

**Why:** Default arguments are evaluated once when the function is defined, not each call.

## *args and **kwargs

```python
def f(*args, **kwargs):
    print(args)    # tuple of positional args
    print(kwargs)  # dict of keyword args

f(1, 2, 3, name="Alice", age=30)
# (1, 2, 3)
# {'name': 'Alice', 'age': 30}

# Unpacking:
nums = [1, 2, 3]
f(*nums)          # unpacks list as positional args

info = {"x": 1, "y": 2}
f(**info)         # unpacks dict as keyword args
```

## Lambda

```python
square = lambda x: x**2
add = lambda x, y: x + y

# Common with sorted, map, filter:
lst = [(1, 'b'), (3, 'a'), (2, 'c')]
sorted(lst, key=lambda x: x[0])  # sort by first element
```

## Closures

```python
def outer(x):
    def inner(y):
        return x + y    # x is captured from outer scope
    return inner

add5 = outer(5)
add5(3)   # 8
```

**Late binding closure trap:**

```python
funcs = [lambda: i for i in range(3)]
[f() for f in funcs]   # [2, 2, 2] ← NOT [0, 1, 2]!
# i is looked up at call time, not definition time

# Fix: capture i as default argument
funcs = [lambda i=i: i for i in range(3)]
[f() for f in funcs]   # [0, 1, 2] ✅
```

## First-class functions

```python
def apply(func, value):
    return func(value)

apply(str.upper, "hello")  # "HELLO"
apply(lambda x: x*2, 5)   # 10
```

---

# 6. ITERATORS & GENERATORS

## Iterator Protocol

```python
# Any object with __iter__() and __next__() is an iterator
lst = [1, 2, 3]
it = iter(lst)      # __iter__()
next(it)            # 1 — __next__()
next(it)            # 2
next(it)            # 3
next(it)            # StopIteration raised
```

## Generators — `yield`

```python
def count_up(n):
    for i in range(n):
        yield i          # pauses here, resumes on next()

gen = count_up(3)
next(gen)   # 0
next(gen)   # 1
next(gen)   # 2
next(gen)   # StopIteration
```

**Generator expression (lazy list comp):**

```python
squares_list = [x**2 for x in range(1000000)]   # 8MB in memory NOW
squares_gen  = (x**2 for x in range(1000000))   # almost 0 memory
```

## Generator vs List

| Property | List | Generator |
|---|---|---|
| Memory | Stores all values | Computes on demand |
| Reusable | ✅ Yes | ❌ Exhausted after use |
| Length | `len()` works | `len()` doesn't work |
| Speed | Slower to build | Faster to start |
| Use case | Random access | Large/infinite streams |

**Why generators are asked:** Shows you understand lazy evaluation and memory efficiency — backend-critical.

```python
# Infinite generator
def naturals():
    n = 1
    while True:
        yield n
        n += 1

gen = naturals()
next(gen)  # 1
next(gen)  # 2  # never exhausted
```

---

# 7. DECORATORS

## How decorators work

```python
def my_decorator(func):
    def wrapper(*args, **kwargs):
        print("Before")
        result = func(*args, **kwargs)
        print("After")
        return result
    return wrapper

@my_decorator         # syntactic sugar for: greet = my_decorator(greet)
def greet(name):
    print(f"Hello, {name}")

greet("Alice")
# Before
# Hello, Alice
# After
```

## Preserving function metadata

```python
from functools import wraps

def my_decorator(func):
    @wraps(func)           # preserves __name__, __doc__
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper
```

## Practical backend examples

```python
# Authentication decorator
def require_auth(func):
    @wraps(func)
    def wrapper(request, *args, **kwargs):
        if not request.user.is_authenticated:
            raise PermissionError("Login required")
        return func(request, *args, **kwargs)
    return wrapper

# Logging decorator
def log(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}")
        result = func(*args, **kwargs)
        print(f"Done {func.__name__}")
        return result
    return wrapper

# Retry decorator
def retry(times=3):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for i in range(times):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if i == times - 1:
                        raise
        return wrapper
    return decorator

@retry(times=3)
def fetch_data(): ...
```

---

# 8. OOP IN PYTHON

## `classmethod` vs `staticmethod` vs regular method

```python
class MyClass:
    count = 0

    def instance_method(self):     # receives instance
        return self

    @classmethod
    def class_method(cls):         # receives class — can access class state
        cls.count += 1
        return cls()               # used as alternative constructors

    @staticmethod
    def static_method(x, y):       # receives nothing — pure utility
        return x + y

# Usage:
MyClass.class_method()    # ✅ works without instance
MyClass.static_method(1, 2)  # ✅ works without instance
```

## `super()`

```python
class Animal:
    def __init__(self, name):
        self.name = name

class Dog(Animal):
    def __init__(self, name, breed):
        super().__init__(name)   # calls Animal.__init__
        self.breed = breed
```

## `__init__` vs `__new__`

```python
# __new__ creates the instance (rarely overridden)
# __init__ initializes the instance (commonly overridden)

class Singleton:
    _instance = None
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance
```

## Method Resolution Order (MRO)

```python
class A: pass
class B(A): pass
class C(A): pass
class D(B, C): pass

D.__mro__  # (D, B, C, A, object) — C3 linearization
```

**Interview trap:** In multiple inheritance, Python uses MRO — `super()` doesn't always call the parent you think.

## Dunder Methods

```python
class Vector:
    def __init__(self, x, y): self.x, self.y = x, y
    def __repr__(self): return f"Vector({self.x}, {self.y})"
    def __add__(self, other): return Vector(self.x+other.x, self.y+other.y)
    def __len__(self): return 2
    def __eq__(self, other): return self.x==other.x and self.y==other.y
```

## Encapsulation (Python style)

```python
class Person:
    def __init__(self, name):
        self._name = name       # convention: "protected" (not enforced)
        self.__age = 0          # name-mangled → _Person__age

    @property
    def age(self):
        return self.__age

    @age.setter
    def age(self, value):
        if value < 0: raise ValueError
        self.__age = value
```

---

# 9. SHALLOW COPY vs DEEP COPY

```
Original:    a = [[1, 2], [3, 4]]

Shallow:     b = a.copy()
             b → [[1,2],[3,4]]   ← new outer list
             b[0] is a[0]        ← SAME inner list!

Deep:        c = copy.deepcopy(a)
             c → [[1,2],[3,4]]   ← new outer list
             c[0] is not a[0]    ← NEW inner list
```

## Memory diagram

```
Original:  a ──→ [ ref0, ref1 ]
                    ↓       ↓
                 [1,2]   [3,4]

Shallow:   b ──→ [ ref0, ref1 ]   ← new list, same refs
                    ↓       ↓
                 [1,2]   [3,4]   ← SHARED objects

Deep:      c ──→ [ ref2, ref3 ]   ← new list, new refs
                    ↓       ↓
                 [1,2]'  [3,4]'  ← COPIED objects
```

## Code examples

```python
import copy

a = [[1, 2], [3, 4]]

b = a.copy()         # or list(a) or a[:]
b[0].append(99)
print(a)   # [[1, 2, 99], [3, 4]] ← original changed!

c = copy.deepcopy(a)
c[0].append(99)
print(a)   # [[1, 2], [3, 4]] ← original safe
```

**Tricky interview example:**
```python
a = [1, 2, 3]
b = a.copy()
b.append(4)
print(a)   # [1, 2, 3] — flat list, copy is sufficient

a = [[1, 2]]
b = a.copy()
b[0].append(99)
print(a)   # [[1, 2, 99]] ← shallow copy trap with nested
```

---

# 10. EXCEPTION HANDLING

```python
try:
    result = 10 / 0
except ZeroDivisionError as e:
    print(f"Error: {e}")
except (TypeError, ValueError):    # catch multiple
    pass
else:
    print("No exception occurred")  # runs if no exception
finally:
    print("Always runs")            # cleanup (close files, DB connections)
```

## Custom exceptions

```python
class InsufficientFundsError(Exception):
    def __init__(self, amount, balance):
        super().__init__(f"Need {amount}, have {balance}")
        self.amount = amount
        self.balance = balance

raise InsufficientFundsError(100, 50)
```

## Re-raising exceptions

```python
try:
    risky_operation()
except Exception as e:
    logger.error(f"Failed: {e}")
    raise    # re-raise the same exception
```

## Production patterns

```python
# Context manager for resources
with open("file.txt") as f:   # auto-closes even on exception
    data = f.read()

# Exception chaining
try:
    fetch_data()
except HTTPError as e:
    raise DatabaseError("Could not fetch") from e
```

---

# 11. MULTITHREADING vs MULTIPROCESSING

| Property | Threading | Multiprocessing |
|---|---|---|
| Memory | Shared | Separate |
| GIL impact | Limited by GIL | No GIL (separate process) |
| Best for | I/O-bound tasks | CPU-bound tasks |
| Overhead | Low | High (process creation) |
| Communication | Shared vars (careful!) | Queues, Pipes |
| Crash isolation | No | Yes |

```python
# Threading — I/O bound (file reads, network)
from threading import Thread

def fetch(url): ...

threads = [Thread(target=fetch, args=(url,)) for url in urls]
for t in threads: t.start()
for t in threads: t.join()

# Multiprocessing — CPU bound (computation, image processing)
from multiprocessing import Pool

with Pool(4) as p:
    results = p.map(heavy_computation, data)
```

**Interview line:** *"For I/O-bound tasks, use threading — threads wait on I/O, GIL is released. For CPU-bound tasks, use multiprocessing — each process has its own GIL."*

---

# 12. GIL — GLOBAL INTERPRETER LOCK

## What is GIL?

> A mutex in CPython that **allows only one thread to execute Python bytecode at a time**.

## Why does it exist?

- CPython uses reference counting for memory management
- Without GIL, concurrent threads could corrupt reference counts
- GIL simplifies memory management at the cost of parallelism

## Impact on multithreading

```python
# Even with threads, Python runs one at a time (for CPU work):
import threading

counter = 0
def increment():
    global counter
    for _ in range(1000000):
        counter += 1   # NOT thread-safe despite GIL!

# GIL is released during I/O, so threading DOES help there
```

## The full interview answer

> "The GIL is a lock in CPython that prevents multiple threads from executing Python bytecode simultaneously. It exists because CPython's memory management (reference counting) isn't thread-safe. For I/O-bound code, the GIL is released during I/O operations, so threading is effective. For CPU-bound code, threads don't get true parallelism — use `multiprocessing` or `concurrent.futures` instead. Jython and PyPy have different GIL implementations."

## GIL released for:
- I/O operations
- Time-consuming C extensions (NumPy, etc.)
- `time.sleep()`

---

# 13. ASYNC PROGRAMMING BASICS

```python
import asyncio

async def fetch_data(url):
    await asyncio.sleep(1)   # simulates I/O — doesn't block event loop
    return f"data from {url}"

async def main():
    results = await asyncio.gather(
        fetch_data("url1"),
        fetch_data("url2"),    # runs concurrently!
        fetch_data("url3"),
    )
    print(results)

asyncio.run(main())
```

## async vs threading vs multiprocessing

| | Async | Threading | Multiprocessing |
|---|---|---|---|
| Concurrency model | Cooperative | Preemptive | True parallelism |
| GIL affected | Yes (single thread) | Yes | No |
| Best for | Many I/O tasks | I/O bound | CPU bound |
| Overhead | Very low | Low | High |

**Key concept:** `async` is single-threaded. The event loop switches between coroutines at `await` points.

**Interview line:** *"async/await is cooperative multitasking — only one coroutine runs at a time, but while one awaits I/O, others can run. No GIL issues since it's single-threaded."*

---

# 14. PYTHON INTERNALS & COMMON TRAPS

## `is` vs `==`

```python
a = [1, 2, 3]
b = [1, 2, 3]
a == b   # True  — value equality
a is b   # False — different objects in memory

# Correct patterns:
x is None    # ✅ always use is for None
x is True    # ✅ can use for singleton booleans
x == "hello" # ✅ always use == for values
```

## List multiplication trap

```python
# 1D — fine
lst = [0] * 5   # [0, 0, 0, 0, 0]

# 2D — TRAP!
matrix = [[0] * 3] * 3   # ❌ all rows are the SAME list!
matrix[0][0] = 1
print(matrix)   # [[1, 0, 0], [1, 0, 0], [1, 0, 0]] ← all rows changed!

# Fix:
matrix = [[0] * 3 for _ in range(3)]   # ✅ each row is a new list
```

## Variable scope — LEGB Rule

```
L → Local (inside function)
E → Enclosing (outer function, for closures)
G → Global (module level)
B → Built-in (print, len, etc.)
```

```python
x = 10   # Global

def f():
    print(x)   # reads global x ✅

def g():
    x = 20     # local x, shadows global
    print(x)   # 20

def h():
    global x
    x = 99     # modifies global x
```

## Pass by object reference

```python
def modify(lst):
    lst.append(4)     # modifies original (mutable)

def rebind(lst):
    lst = [9, 9, 9]   # rebinds local variable, original unchanged

a = [1, 2, 3]
modify(a)
print(a)   # [1, 2, 3, 4] ← changed

rebind(a)
print(a)   # [1, 2, 3, 4] ← unchanged
```

**Interview line:** *"Python passes object references by value. You can mutate the object, but you can't rebind the caller's variable."*

---

# 15. TIME COMPLEXITY CHEAT SHEET

## List

| Operation | Complexity | Notes |
|---|---|---|
| `append` | O(1) amortized | |
| `pop()` (end) | O(1) | |
| `pop(i)` | O(n) | shifts elements |
| `insert(i, x)` | O(n) | shifts elements |
| `x in lst` | O(n) | linear scan |
| `lst[i]` | O(1) | random access |
| `len(lst)` | O(1) | cached |
| `sorted(lst)` | O(n log n) | Timsort |

## Set & Dict

| Operation | Avg | Worst |
|---|---|---|
| `x in s` | O(1) | O(n) hash collision |
| `s.add(x)` | O(1) | O(n) |
| `s.remove(x)` | O(1) | O(n) |
| `d[key]` | O(1) | O(n) |
| `d[key] = val` | O(1) | O(n) |
| `key in d` | O(1) | O(n) |

## String

| Operation | Complexity |
|---|---|
| Concatenation `+` | O(n) — creates new string each time |
| `"".join(lst)` | O(n) — preferred for building strings |
| `s[i]` | O(1) |
| `s in t` | O(n*m) |

**String building trap:**
```python
# ❌ O(n²) — new string created each iteration
result = ""
for word in words:
    result += word

# ✅ O(n)
result = "".join(words)
```

---

# 16. TOP OUTPUT-BASED QUESTIONS

**Q1 — Mutable default argument**
```python
def f(x=[]):
    x.append(1)
    return x

print(f())   # [1]
print(f())   # [1, 1]
print(f())   # [1, 1, 1]
```
**Trap:** Default list is created once at definition. Same object reused.

---

**Q2 — List reference**
```python
a = [1, 2, 3]
b = a
b.append(4)
print(a)
```
**Output:** `[1, 2, 3, 4]`
**Trap:** `b = a` is not a copy — both reference same list.

---

**Q3 — Late binding closure**
```python
funcs = [lambda: i for i in range(3)]
print([f() for f in funcs])
```
**Output:** `[2, 2, 2]`
**Trap:** `i` is looked up at call time — it's 2 (final loop value) for all.

---

**Q4 — is vs ==**
```python
a = 1000
b = 1000
print(a == b)   # True
print(a is b)   # False (> 256, not interned)

a = 5
b = 5
print(a is b)   # True (small int, interned)
```

---

**Q5 — Tuple immutability trap**
```python
t = (1, [2, 3], 4)
t[1].append(99)
print(t)
```
**Output:** `(1, [2, 3, 99], 4)`
**Trap:** Tuple is immutable, but the list inside is mutable.

---

**Q6 — Shallow copy**
```python
import copy
a = [[1, 2], [3, 4]]
b = a.copy()
b[0].append(99)
print(a)
```
**Output:** `[[1, 2, 99], [3, 4]]`
**Trap:** Shallow copy — inner lists are shared.

---

**Q7 — Static method / class method**
```python
class A:
    x = 10
    def __init__(self): self.x = 20

    @classmethod
    def get_x(cls): return cls.x

print(A.get_x())   # 10 (class variable)
print(A().get_x()) # 10 (still class variable)
```

---

**Q8 — Generator exhaustion**
```python
gen = (x for x in range(3))
print(list(gen))   # [0, 1, 2]
print(list(gen))   # [] ← exhausted!
```
**Trap:** Generators are single-use.

---

**Q9 — `*args` unpacking**
```python
def f(a, b, c): return a + b + c
lst = [1, 2, 3]
print(f(*lst))    # 6
```

---

**Q10 — List multiplication**
```python
matrix = [[0]*3]*3
matrix[0][0] = 9
print(matrix)
```
**Output:** `[[9, 0, 0], [9, 0, 0], [9, 0, 0]]`
**Trap:** All rows are the same list object.

---

**Q11 — Exception `else`**
```python
try:
    x = 10
except ZeroDivisionError:
    print("error")
else:
    print("success")
finally:
    print("done")
```
**Output:** `success` then `done`

---

**Q12 — Walrus operator**
```python
nums = [1, 2, 3, 4, 5]
if (n := len(nums)) > 3:
    print(f"Long list: {n}")
```
**Output:** `Long list: 5`

---

**Q13 — Class variable vs instance variable**
```python
class A:
    x = []

a = A()
b = A()
a.x.append(1)
print(b.x)   # [1] — shared class variable!

a.x = [99]   # now a.x is instance variable
print(b.x)   # [1] — b.x still class variable
```

---

**Q14 — `del` and reference counting**
```python
a = [1, 2, 3]
b = a
del a
print(b)   # [1, 2, 3] — object still exists (b references it)
```
**Trap:** `del a` deletes the name, not the object. Object freed when refcount = 0.

---

# 17. TOP INTERVIEW QUESTIONS WITH SHORT ANSWERS

**GIL?**
Global Interpreter Lock — a mutex in CPython allowing only one thread to run Python bytecode at a time. Exists because reference counting isn't thread-safe. Threads still help for I/O-bound tasks (GIL released during I/O). For CPU-bound, use multiprocessing.

**Deep copy vs shallow copy?**
Shallow copy creates a new outer object but inner objects are shared references. Deep copy creates fully independent nested copies. Use `copy.copy()` vs `copy.deepcopy()`. Flat structures (lists of ints) — shallow is fine. Nested mutable structures — need deep copy.

**List vs Tuple?**
Lists are mutable, more memory, can't be dict keys. Tuples are immutable, less memory, hashable (can be dict keys). Use tuples for fixed data records and as dict keys. Tuple access is marginally faster.

**is vs ==?**
`==` checks value equality (`__eq__`). `is` checks object identity (same memory address, same `id()`). Always use `==` for value comparison. Use `is` only for singletons: `None`, `True`, `False`.

**Generator advantages?**
Lazy evaluation — values computed on demand, not all at once. Memory-efficient for large or infinite sequences. Use when processing streams or large files. Downside: single-use and no `len()`.

**Decorator usage?**
Decorators wrap functions to add behavior without modifying the original. Common for authentication, logging, caching (`@lru_cache`), retry logic, timing. Use `@wraps` to preserve the original function's metadata.

**Multithreading vs Multiprocessing?**
Threading: shared memory, GIL-limited, good for I/O-bound (network, file). Multiprocessing: separate processes/GIL, true parallelism, good for CPU-bound (computation). Use `concurrent.futures` for cleaner API to both.

**Mutable vs Immutable?**
Immutable objects (int, str, tuple) can't be changed in place — operations create new objects. Mutable (list, dict, set) can be changed in place. Matters for function side effects, dict keys, and default arguments.

**Python memory management?**
CPython uses reference counting — objects freed when refcount hits 0. Cyclic references handled by generational garbage collector. Small integers (-5–256) and some strings are interned to save memory.

**Why is Python slow?**
Interpreted (not compiled), dynamic typing requires runtime type checks, GIL limits parallelism, everything is an object with overhead. Use NumPy, Cython, PyPy, or C extensions for performance-critical paths.

**Monkey patching?**
Replacing or extending existing code at runtime. E.g., replacing a method on a class/module without modifying source. Useful in testing (mock), dangerous in production (hard to debug).

**Duck typing?**
"If it walks like a duck and quacks like a duck, it's a duck." Python doesn't check types — checks for the presence of methods/attributes. Enables polymorphism without inheritance. `isinstance()` checks available but often not needed.

---

# 18. COMMON INTERVIEW TRAPS

**Trap 1 — Modifying list during iteration**
```python
lst = [1, 2, 3, 4, 5]
for x in lst:
    if x % 2 == 0:
        lst.remove(x)   # ❌ skips elements!
print(lst)   # [1, 3, 5] — looks right but isn't always!

# Fix:
lst = [x for x in lst if x % 2 != 0]   # ✅
```

---

**Trap 2 — Mutable default argument**
```python
def append_to(elem, to=[]):   # ❌
    to.append(elem)
    return to

append_to(1)   # [1]
append_to(2)   # [1, 2] ← not [2]!
```
Fix: use `None` as default, create list inside.

---

**Trap 3 — Late binding in lambdas**
```python
multipliers = [lambda x: x * i for i in range(4)]
multipliers[0](5)   # 15 (not 0!) — i is 3 at call time
```

---

**Trap 4 — Using `is` for value comparison**
```python
x = input("Enter: ")   # "256"
x = int(x)
x is 256   # may be True or False — don't rely on interning!
```
Always use `==` for values.

---

**Trap 5 — Circular imports**
```python
# a.py: from b import func
# b.py: from a import other_func  ← circular import error
```
Fix: restructure, use local imports, or use `importlib`.

---

**Trap 6 — Recursion limit**
```python
import sys
sys.getrecursionlimit()   # 1000 default

# Deep recursion → RecursionError
# Fix: sys.setrecursionlimit(10000) or convert to iteration
```

---

**Trap 7 — `+=` on immutable inside function**
```python
def f(s):
    s += " world"    # creates new string, doesn't modify original

name = "hello"
f(name)
print(name)   # "hello" — unchanged
```

---

**Trap 8 — Class variable mutation**
```python
class Counter:
    count = 0

a = Counter()
b = Counter()
a.count += 1     # creates INSTANCE variable for a!
print(b.count)   # 0 — class variable unchanged
Counter.count += 1  # modifies class variable
print(b.count)   # 1
```

---

# 19. TOP CODING QUESTIONS

**Reverse string:**
```python
s[::-1]              # O(n)
"".join(reversed(s)) # O(n)
```

**Palindrome:**
```python
def is_palindrome(s):
    s = s.lower().replace(" ", "")
    return s == s[::-1]   # O(n)
```

**Anagram:**
```python
from collections import Counter
def is_anagram(a, b):
    return Counter(a) == Counter(b)   # O(n)
```

**Frequency count:**
```python
from collections import Counter
freq = Counter("banana")   # {'a': 3, 'n': 2, 'b': 1}
freq.most_common(2)        # [('a', 3), ('n', 2)]
```

**Two Sum:**
```python
def two_sum(nums, target):
    seen = {}
    for i, n in enumerate(nums):
        complement = target - n
        if complement in seen:
            return [seen[complement], i]
        seen[n] = i
# O(n) time, O(n) space
```

**Remove duplicates (preserve order):**
```python
def dedup(lst):
    return list(dict.fromkeys(lst))   # O(n), preserves order
# or: seen = set(); [x for x in lst if not (x in seen or seen.add(x))]
```

**Merge intervals:**
```python
def merge(intervals):
    intervals.sort(key=lambda x: x[0])
    merged = [intervals[0]]
    for start, end in intervals[1:]:
        if start <= merged[-1][1]:
            merged[-1][1] = max(merged[-1][1], end)
        else:
            merged.append([start, end])
    return merged
# O(n log n)
```

**Generator — infinite Fibonacci:**
```python
def fib():
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b

gen = fib()
[next(gen) for _ in range(8)]  # [0, 1, 1, 2, 3, 5, 8, 13]
```

**Dict-based grouping:**
```python
from collections import defaultdict

words = ["cat", "car", "bar", "bat", "cab"]
groups = defaultdict(list)
for w in words:
    groups[tuple(sorted(w))].append(w)
# Groups anagrams: {('a','c','t'): ['cat'], ...}
```

**Flatten nested list:**
```python
def flatten(lst):
    for item in lst:
        if isinstance(item, list):
            yield from flatten(item)
        else:
            yield item

list(flatten([1, [2, [3, 4]], 5]))  # [1, 2, 3, 4, 5]
```

---

# 20. 5-MINUTE FINAL REVISION SHEET

### ⚡ Mutability Rules
- Immutable: `int`, `str`, `tuple`, `frozenset` — operations create NEW objects
- Mutable: `list`, `dict`, `set` — in-place modification
- Immutable ≠ safe inside tuple (list inside tuple can be mutated)
- `+=` on list mutates; `+=` on string/int creates new object

### ⚡ GIL Recap
- CPython only: one thread runs Python bytecode at a time
- I/O-bound → threading is fine (GIL released on I/O)
- CPU-bound → use `multiprocessing` (each process has own GIL)
- GIL does NOT make code thread-safe (e.g., `count += 1` still needs a lock)

### ⚡ Complexity Recap
- `list` search → O(n) | `set`/`dict` search → O(1) avg
- `list.append` → O(1) amort | `list.insert(0, x)` → O(n)
- `"".join(lst)` → O(n) | `s += piece` in loop → O(n²)

### ⚡ Generator Syntax
```python
# Function generator
def gen(): yield value

# Expression generator
g = (x**2 for x in range(10))
```

### ⚡ Decorator Syntax
```python
@decorator
def func(): ...
# Equivalent to: func = decorator(func)

# With args: func = decorator(arg)(func)
```

### ⚡ Common Traps — One-Liners
- `def f(x=[])` → same list reused each call → use `x=None`
- `b = a` for list → same object, not a copy → use `b = a.copy()`
- `lambda: i` in loop → late binding → `lambda i=i: i`
- `[[0]*3]*3` → all rows same object → use list comp
- `is` for integers > 256 → unreliable → use `==`
- Modifying list while iterating → skip elements → use list comp
- Generator exhausted → second `list(gen)` returns `[]`
- `sizeof` equivalent: `sys.getsizeof(obj)`

### ⚡ Interview One-Liners
| Question | Answer |
|---|---|
| `is` vs `==`? | Identity vs value equality |
| GIL impact? | No CPU parallelism in threads |
| Generator vs List? | Lazy vs eager; memory efficient |
| shallow vs deep? | Shared refs vs full copy |
| Best for CPU tasks? | `multiprocessing` |
| Best for I/O tasks? | `threading` or `async` |
| Tuple vs List? | Immutable, hashable, less memory |
| Default mutable arg fix? | Use `None`, create inside |
| `del x` frees object? | Only if refcount hits 0 |
| Python is slow because? | Interpreted, dynamic, GIL |

---
*🎯 Good luck. You've got this.*
