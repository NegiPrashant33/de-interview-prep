## Python Data Structures: List vs Set vs Tuple vs Dictionary

### 1. Quick Overview

| Feature | List | Set | Tuple | Dictionary |
| --- | --- | --- | --- | --- |
| Syntax | `[1, 2, 3]` | `{1, 2, 3}` | `(1, 2, 3)` | `{'a': 1, 'b': 2}` |
| Ordered | Yes (insertion order) | No (unordered) | Yes (insertion order) | Yes (insertion order, guaranteed since 3.7) |
| Mutable | Yes | Yes | No | Yes |
| Duplicates allowed | Yes | No | Yes | Keys: No, Values: Yes |
| Indexable | Yes (`lst[0]`) | No | Yes (`tup[0]`) | No (access via key: `d['a']`) |
| Hashable (can be dict key/set element) | No | No | Yes (if all elements hashable) | No |
| Class | `list` | `set` | `tuple` | `dict` |

---

### 2. Creation

```python
# List
lst = [1, 2, 3]
lst2 = list((1, 2, 3))        # from iterable
lst3 = [x for x in range(5)]  # comprehension

# Set
s = {1, 2, 3}
s2 = set([1, 2, 2, 3])        # {1, 2, 3} - dedups automatically
s3 = {x for x in range(5)}    # comprehension
empty_set = set()             # {} creates a dict, NOT a set!

# Tuple
t = (1, 2, 3)
t2 = tuple([1, 2, 3])
single = (1,)                 # comma needed, (1) is just int 1

# Dictionary
d = {'a': 1, 'b': 2}
d2 = dict(a=1, b=2)
d3 = dict([('a', 1), ('b', 2)])
d4 = {k: v for k, v in [('a', 1)]}  # comprehension
```

**Interview gotcha:** `set()` must be used for empty set — `{}` creates an empty **dict**.

---

### 3. Mutability Details

| Structure | Can add/remove elements | Can modify in place | Notes |
| --- | --- | --- | --- |
| List | Yes | Yes | `append`, `remove`, `sort`, item assignment |
| Set | Yes | N/A (elements aren't indexed) | `add`, `remove`, `discard` |
| Tuple | No | No | Immutable — but if it holds a mutable object (e.g., a list), *that inner object* can still be changed |
| Dict | Yes | Yes | `d[key] = value`, `pop`, `update` |

```python
t = (1, [2, 3])
t[1].append(4)   # Works! t is now (1, [2, 3, 4])
t[0] = 99        # TypeError: 'tuple' object does not support item assignment
```

---

### 4. Common Methods

#### List

| Method | Description | Return Type |
| --- | --- | --- |
| `append(x)` | Add item to end | `None` |
| `extend(iterable)` | Add all items from iterable | `None` |
| `insert(i, x)` | Insert at index | `None` |
| `remove(x)` | Remove first matching value | `None` (raises `ValueError` if not found) |
| `pop(i=-1)` | Remove & return item at index | element |
| `clear()` | Remove all items | `None` |
| `index(x)` | Return index of first match | `int` (raises `ValueError`) |
| `count(x)` | Count occurrences | `int` |
| `sort(key=, reverse=)` | Sort in place | `None` |
| `sorted(lst)` (builtin) | Return new sorted list | `list` |
| `reverse()` | Reverse in place | `None` |
| `copy()` | Shallow copy | `list` |

#### Set

| Method | Description | Return Type |
| --- | --- | --- |
| `add(x)` | Add element | `None` |
| `remove(x)` | Remove element | `None` (raises `KeyError` if missing) |
| `discard(x)` | Remove element | `None` (no error if missing) |
| `pop()` | Remove & return arbitrary element | element |
| `clear()` | Empty the set | `None` |
| `union(s2)` / ` | ` | Combine sets | `set` |
| `intersection(s2)` / `&` | Common elements | `set` |
| `difference(s2)` / `-` | Elements only in first | `set` |
| `symmetric_difference(s2)` / `^` | Elements in either but not both | `set` |
| `issubset(s2)` / `<=` | Check subset | `bool` |
| `issuperset(s2)` / `>=` | Check superset | `bool` |
| `isdisjoint(s2)` | No overlap check | `bool` |
| `update(s2)` | In-place union | `None` |

#### Tuple (very few - immutability means no mutating methods)

| Method | Description | Return Type |
| --- | --- | --- |
| `count(x)` | Count occurrences | `int` |
| `index(x)` | Return index of first match | `int` |

#### Dictionary

| Method | Description | Return Type |
| --- | --- | --- |
| `get(key, default=None)` | Safe access, no `KeyError` | value or default |
| `keys()` | View of keys | `dict_keys` |
| `values()` | View of values | `dict_values` |
| `items()` | View of key-value pairs | `dict_items` |
| `pop(key, default)` | Remove & return value | value |
| `popitem()` | Remove last inserted (key,value) | `tuple` |
| `update(d2)` | Merge another dict in | `None` |
| `setdefault(key, default)` | Get value, or set+return default if missing | value |
| `clear()` | Empty the dict | `None` |
| `copy()` | Shallow copy | `dict` |

**Interview note:** `keys()`, `values()`, `items()` return **view objects**, not lists — they're dynamic and reflect live changes to the dict.

```python
d = {'a': 1}
keys = d.keys()
d['b'] = 2
print(keys)  # dict_keys(['a', 'b']) — updated automatically
```

---

### 5. Operations & Behavior

| Operation | List | Set | Tuple | Dict |
| --- | --- | --- | --- | --- |
| Indexing `x[0]` | Yes | No (`TypeError`) | Yes | No (use `x['key']`) |
| Slicing `x[1:3]` | Yes | No | Yes | No |
| Concatenation `+` | Yes → new list | No (use ` | `) | Yes → new tuple | No (use ` | ` in 3.9+ or `update`) |
| Repetition `*` | Yes (`[1,2]*2`) | No | Yes | No |
| Membership `in` | O(n) | O(1) avg | O(n) | O(1) avg (checks keys) |
| Iteration order | Insertion order | Arbitrary/hash-based | Insertion order | Insertion order |
| Nesting | Yes | Only immutable/hashable elements | Yes | Values: yes; Keys: must be hashable |

**Interview favorite:** membership testing (`in`) is **O(1) average** for sets/dicts (hash table) vs **O(n)** for lists/tuples (linear scan). This is why sets are preferred for deduplication and lookups.

---

### 6. Hashability Rules

- **Sets and dict keys** require elements to be **hashable** (must implement `__hash__`).
- Hashable = immutable in practice: `int`, `float`, `str`, `tuple` (if its contents are hashable), `frozenset`.
- **Not hashable**: `list`, `dict`, `set` (mutable → can't be hashed, since hash must stay constant).

```python
{[1,2]: 'x'}       # TypeError: unhashable type: 'list'
{(1,2): 'x'}       # Works fine — tuple is hashable
s = {[1,2,3]}      # TypeError: unhashable type: 'list'
s = {(1,2,3)}      # Works
```

This is why **tuples are often used as dict keys** (e.g., coordinates `(x, y)`), while lists cannot be.

---

### 7. Memory & Performance

| Structure | Memory overhead | Access speed | When to use |
| --- | --- | --- | --- |
| List | Moderate | Index: O(1); search: O(n) | Ordered, mutable sequence of items, allows duplicates |
| Set | Higher (hash table overhead) | Add/lookup/delete: O(1) avg | Fast membership tests, uniqueness, set algebra |
| Tuple | Lowest (fixed size, immutable) | Index: O(1) | Fixed collections, dict keys, function returns, data integrity |
| Dict | Higher (hash table) | Lookup/insert/delete: O(1) avg | Key-value mapping, fast lookups by key |

Tuples are generally **faster to create and more memory-efficient** than lists because of immutability (Python can optimize storage).

---

### 8. Type Flexibility

All four can hold **mixed data types** simultaneously:

```python
lst = [1, "two", 3.0, [4, 5]]
t = (1, "two", 3.0)
s = {1, "two", 3.0}          # elements must be hashable
d = {"a": 1, 2: "b", (1,2): [1,2]}  # keys hashable, values anything
```

---

### 9. Common Conversions

```python
list_to_set = set([1, 2, 2, 3])       # {1, 2, 3} — removes duplicates
set_to_list = list({1, 2, 3})         # [1, 2, 3] — order not guaranteed pre-3.7 semantics don't apply (sets never ordered)
list_to_tuple = tuple([1, 2, 3])      # (1, 2, 3)
tuple_to_list = list((1, 2, 3))       # [1, 2, 3]
dict_keys_to_list = list(d.keys())
list_of_pairs_to_dict = dict([('a', 1), ('b', 2)])
zip_to_dict = dict(zip(['a','b'], [1,2]))  # {'a': 1, 'b': 2}
```

**Classic interview trick — dedup while preserving order:**

```python
lst = [3, 1, 2, 3, 1]
unique_ordered = list(dict.fromkeys(lst))  # [3, 1, 2]
```

---

### 10. Key Interview Q&A Points

**Q: Why use a tuple instead of a list?**
Immutability → safer (no accidental modification), hashable (usable as dict key/set member), slightly faster and more memory efficient, signals "fixed data" intent.

**Q: How does a dict maintain insertion order?**
Since Python 3.7 (CPython 3.6 as implementation detail), dicts maintain insertion order as a language guarantee, implemented via a combination of a hash table and a compact array tracking insertion sequence.

**Q: Why can't lists be dict keys?**
Because they're mutable, so their hash could change over time — hash tables require keys with a stable hash value.

**Q: What's the difference between `remove()` and `discard()` on a set?**`remove()` raises `KeyError` if the element doesn't exist; `discard()` silently does nothing.

**Q: What's a `frozenset`?**
An immutable version of `set` — hashable, so it can be used as a dict key or set element, unlike a regular set.

**Q: Shallow vs deep copy?**`copy()` (or `list(x)`, `dict(x)`) creates a shallow copy — nested mutable objects are still shared references. Use `copy.deepcopy()` for full independence.

```python
import copy
original = [[1, 2], [3, 4]]
shallow = original.copy()
deep = copy.deepcopy(original)
shallow[0].append(99)  # affects original too
deep[0].append(99)     # does NOT affect original
```

**Q: Time complexity cheat sheet (average case)**

| Operation | List | Set | Dict |
| --- | --- | --- | --- |
| Access by index/key | O(1) | N/A | O(1) |
| Search (`in`) | O(n) | O(1) | O(1) |
| Insert (end) | O(1) | O(1) | O(1) |
| Insert (middle/arbitrary) | O(n) | O(1) | O(1) |
| Delete | O(n) | O(1) | O(1) |
