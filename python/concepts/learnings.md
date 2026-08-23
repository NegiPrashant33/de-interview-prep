# Python Concepts - Categorized Notes

## 1. Sorting

### `sorted()` vs `.sort()`

**Explanation:** `sorted()` returns a *new* sorted list and works on any iterable. `.sort()` sorts a list *in place* and returns `None`.
**Example:**

```python
nums = [3, 1, 2]
sorted(nums)   # [1, 2, 3] -> nums unchanged
nums.sort()    # nums is now [1, 2, 3], returns None
```

**Reference:** [docs.python.org/3/howto/sorting.html](https://docs.python.org/3/howto/sorting.html)

---

## 2. Dictionaries & Dict-like Structures

### `defaultdict` vs normal `{}`

**Explanation:** `defaultdict(type)` auto-creates a default value (e.g., `0`, `[]`) when you access a missing key, instead of raising `KeyError` like a normal dict.
**Example:**

```python
from collections import defaultdict
d = defaultdict(int)
d['a'] += 1   # works, no KeyError -> {'a': 1}
```

**Reference:** [docs.python.org/3/library/collections.html#collections.defaultdict](https://docs.python.org/3/library/collections.html#collections.defaultdict)

### `.get()` and `dict[]` on missing keys

**Explanation:** `dict[key]` raises `KeyError` if the key is absent. `dict.get(key, default)` returns `None` (or your default) instead of erroring.
**Example:**

```python
d = {'a': 1}
d.get('b', 0)   # 0
d['b']          # raises KeyError
```

**Reference:** [docs.python.org/3/library/stdtypes.html#dict.get](https://docs.python.org/3/library/stdtypes.html#dict.get)

### Creating a default value for a non-existing key

**Explanation:** Besides `defaultdict`, you can use `.setdefault()` to insert a default only if the key is missing.
**Example:**

```python
d = {}
d.setdefault('a', []).append(1)   # {'a': [1]}
```

**Reference:** [docs.python.org/3/library/stdtypes.html#dict.setdefault](https://docs.python.org/3/library/stdtypes.html#dict.setdefault)

### `Counter`

**Explanation:** A dict subclass built specifically for counting hashable items; auto-initializes counts at 0.
**Example:**

```python
from collections import Counter
Counter("banana")   # Counter({'a': 3, 'n': 2, 'b': 1})
```

**Reference:** [docs.python.org/3/library/collections.html#collections.Counter](https://docs.python.org/3/library/collections.html#collections.Counter)

### Creating a frequency map, sorting it by key or value

**Explanation:** Build counts with `Counter` or `defaultdict(int)`, then sort using `sorted()` with a `key=` function (`key=lambda x: x[0]` for keys, `x[1]` for values).
**Example:**

```python
freq = Counter("banana")
sorted(freq.items(), key=lambda x: x[1], reverse=True)
# [('a', 3), ('n', 2), ('b', 1)]
```

**Reference:** [docs.python.org/3/howto/sorting.html#sort-stability-and-complex-sorts](https://docs.python.org/3/howto/sorting.html#sort-stability-and-complex-sorts)

---

## 3. Iteration & Comprehensions

### Comprehensions

**Explanation:** Concise syntax to build a list, dict, or set from an iterable in one line, optionally with a filter condition.
**Example:**

```python
squares = [x**2 for x in range(5) if x % 2 == 0]   # [0, 4, 16]
```

**Reference:** [docs.python.org/3/tutorial/datastructures.html#list-comprehensions](https://docs.python.org/3/tutorial/datastructures.html#list-comprehensions)

### Loops (`enumerate`, `.items()/.values()/.keys()`, `zip`)

**Explanation:** `enumerate()` gives index+value pairs; `.items()/.keys()/.values()` iterate dict contents; `zip()` pairs up multiple iterables element-wise.
**Example:**

```python
for i, v in enumerate(['a', 'b']):     # (0,'a'), (1,'b')
    ...
for k, v in {'x':1}.items():           # ('x', 1)
    ...
for a, b in zip([1,2], [3,4]):         # (1,3), (2,4)
    ...
```

**Reference:** [docs.python.org/3/library/functions.html#enumerate](https://docs.python.org/3/library/functions.html#enumerate)

### `extend`

**Explanation:** Adds all elements of an iterable to a list in place (unlike `.append()`, which adds the iterable as a single item).
**Example:**

```python
a = [1, 2]
a.extend([3, 4])   # [1, 2, 3, 4]
```

**Reference:** [docs.python.org/3/tutorial/datastructures.html#more-on-lists](https://docs.python.org/3/tutorial/datastructures.html#more-on-lists)

---

## 4. Built-in Functions / Introspection

### `help()`

**Explanation:** Prints the docstring/documentation for any object, function, or module — useful for quick reference without leaving the interpreter.
**Example:**

```python
help(str.split)
```

**Reference:** [docs.python.org/3/library/functions.html#help](https://docs.python.org/3/library/functions.html#help)

### `dir()`

**Explanation:** Lists all attributes and methods available on an object — useful for exploring what you can do with it.
**Example:**

```python
dir([])   # ['append', 'clear', 'copy', ...]
```

**Reference:** [docs.python.org/3/library/functions.html#dir](https://docs.python.org/3/library/functions.html#dir)

### `ord()`

**Explanation:** Returns the Unicode code point (integer) of a single character. The inverse is `chr()`.
**Example:**

```python
ord('a')   # 97
chr(97)    # 'a'
```

**Reference:** [docs.python.org/3/library/functions.html#ord](https://docs.python.org/3/library/functions.html#ord)

---

## 5. Code Design Patterns

### Dispatcher pattern

**Explanation:** Instead of a long `if/elif` chain, map keys (e.g., strings) to functions in a dict, then call the matching function directly — cleaner and more extensible. This works because functions are **first-class objects** in Python — they can be stored as dict values just like any other object, and then invoked through the reference stored in the dict.
**Example:**

```python
def add(a, b): return a + b
def sub(a, b): return a - b

ops = {'add': add, 'sub': sub}
ops['add'](2, 3)   # 5
```

**Reference:** [Python docs — "Dictionaries as switch statements"](https://docs.python.org/3/faq/design.html#why-isn-t-there-a-switch-or-case-statement-in-python)

### `Callable` (typing)

**Explanation:** `Callable` from the `typing` module is used to type-hint something that can be *called* like a function — e.g., a variable, dict value, or parameter that holds a function reference. It's the natural type annotation for dispatcher-style dicts, since each value in the dict is itself a function.
**Example:**

```python
from typing import Callable

ops: dict[str, Callable[[int, int], int]] = {
    'add': lambda a, b: a + b,
    'sub': lambda a, b: a - b,
}
ops['add'](2, 3)   # 5
```

**Reference:** [docs.python.org/3/library/typing.html#typing.Callable](https://docs.python.org/3/library/typing.html#typing.Callable)

---

## 6. System, CLI & OS Interaction

### `subprocess`

**Explanation:** Lets Python run external shell commands/programs and capture their output.
**Example:**

```python
import subprocess
result = subprocess.run(['ls', '-l'], capture_output=True, text=True)
print(result.stdout)
```

**Reference:** [docs.python.org/3/library/subprocess.html](https://docs.python.org/3/library/subprocess.html)

### `os`

**Explanation:** Provides functions to interact with the operating system — file paths, environment variables, directory operations.
**Example:**

```python
import os
os.getcwd()          # current directory
os.listdir('.')      # list files
```

**Reference:** [docs.python.org/3/library/os.html](https://docs.python.org/3/library/os.html)

### `argparse`

**Explanation:** Standard library for building command-line interfaces — parses arguments/flags passed to a script.
**Example:**

```python
import argparse
parser = argparse.ArgumentParser()
parser.add_argument('--name')
args = parser.parse_args()   # run: script.py --name Bob
```

**Reference:** [docs.python.org/3/library/argparse.html](https://docs.python.org/3/library/argparse.html)

### `logging`

**Explanation:** Standard library for recording runtime events/messages at different severity levels (DEBUG, INFO, WARNING, ERROR) — preferred over `print()` for production code.
**Example:**

```python
import logging
logging.basicConfig(level=logging.INFO)
logging.info("Process started")
```

**Reference:** [docs.python.org/3/library/logging.html](https://docs.python.org/3/library/logging.html)
