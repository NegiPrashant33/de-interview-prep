# DSA Patterns and Focus Areas for Data Engineering Interviews

Reference: [DSA Patterns You Need to Know — by Anubhav](https://leetcode.com/discuss/post/5886397/dsa-patterns-you-need-to-know-by-anubhav-x7og/)

## Top Priority

- **Hash Table / Hash Maps** — The single most important pattern. Hash maps/dictionaries appear constantly since data engineers work with key-value lookups, deduplication, and grouping operations every day.
- **Two Pointers & Sliding Window** — Common for string/array parsing tasks (log parsing, batching, dedup windows).
- **Prefix Sum** — Useful for aggregation-style problems, which mirror real ETL logic.
- **Top 'K' Elements** — Heaps show up in streaming/aggregation-flavored questions (e.g., top-K frequent items, running stats).
- **Sorting-based patterns (Overlapping Intervals, Cyclic Sort)** — Intervals especially, since scheduling/merging time ranges resembles real pipeline work (dedup, SCD, watermarking).
- **Matrix Manipulation** — Less common, but occasionally shows up for tabular data manipulation questions.

## Medium Priority

- **BFS/DFS** — Occasionally used for dependency graphs (DAG traversal is very relevant to DE, since Airflow/dbt DAGs are literally graphs), but usually asked conceptually rather than as a hard LeetCode graph puzzle.
- **Topological Sort** — Actually more relevant to DE than most SWE patterns, since it maps directly to DAG scheduling (Airflow task ordering, dbt model dependencies). Worth understanding well even if rarely coded live.
- **Modified Binary Search** — Occasionally asked, not a priority.

## Low Priority

- Backtracking, Bitwise XOR, K-way Merge, Two Heaps, Monotonic Stack, most of Trees beyond basics, Dynamic Programming, and advanced Graph Algorithms (Dijkstra, Floyd-Warshall, Bellman-Ford).
  > For data engineers, don't spend much time on trees, linked lists, graphs, or hard LeetCode challenges unless you have specific evidence the company uses them.
- Reversal of Linked List, Fast & Slow Pointer, Design Data Structure (e.g., LRU/LFU cache) — Asked occasionally at large tech companies but not DE-specific; low ROI unless you know your target company does classic SWE-style rounds.

## 4 Focus Areas

1. **SQL fluency** — Window functions, CTEs, self-joins, deduplication, and slowly changing dimensions come up constantly.
2. **Python data manipulation** — Pandas, dictionary operations, file parsing, and API interactions, plus general Python scripting on messy real-world data rather than abstract algorithms.
3. **Data modeling** — Designing schemas, dimensional models, star schemas.
4. **Pipeline/system design** — Batch vs. streaming architecture, tool tradeoffs (Spark, Kafka, Airflow), scalability reasoning.

---

## Python Data Structures to Master for DE Interviews

| Data Structure | Python Tool | Use Case |
|---|---|---|
| Hash Map / Dict | `dict`, `collections.defaultdict`, `collections.Counter` | Lookups, grouping, frequency counts, deduplication |
| Set | `set` | Fast membership checks, uniqueness |
| Array/List | `list` | Sliding window, two pointers, prefix sums |
| String | `str`, slicing | Parsing, pattern matching |
| Heap / Priority Queue | `heapq` | Top-K problems, streaming aggregation |
| Deque | `collections.deque` | Sliding window max/min, queue-based BFS |
| Ordered Dict | `collections.OrderedDict` | LRU-style problems (rare but occasionally asked) |
| Tuple | `tuple` | Immutable keys in dicts/sets, multi-value returns |
| Stack | `list` (append/pop) | Occasionally for parsing, rarely core to DE rounds |

> Get comfortable with `Counter`, `defaultdict(list)` / `defaultdict(int)`, and `heapq.nlargest` / `heapq.nsmallest` specifically — they show up constantly.

---

## 55 Non-Premium LeetCode Questions

### 1. Hash Table / Dictionary (10)

1. [Two Sum](https://leetcode.com/problems/two-sum/)
2. [Contains Duplicate](https://leetcode.com/problems/contains-duplicate/)
3. [Valid Anagram](https://leetcode.com/problems/valid-anagram/)
4. [Group Anagrams](https://leetcode.com/problems/group-anagrams/)
5. [Top K Frequent Elements](https://leetcode.com/problems/top-k-frequent-elements/)
6. [Longest Consecutive Sequence](https://leetcode.com/problems/longest-consecutive-sequence/)
7. [Subarray Sum Equals K](https://leetcode.com/problems/subarray-sum-equals-k/)
8. [Ransom Note](https://leetcode.com/problems/ransom-note/)
9. [Isomorphic Strings](https://leetcode.com/problems/isomorphic-strings/)
10. [Word Pattern](https://leetcode.com/problems/word-pattern/)

### 2. Two Pointers (7)

1. [Two Sum II - Input Array Is Sorted](https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/)
2. [Valid Palindrome](https://leetcode.com/problems/valid-palindrome/)
3. [3Sum](https://leetcode.com/problems/3sum/)
4. [Container With Most Water](https://leetcode.com/problems/container-with-most-water/)
5. [Sort Colors](https://leetcode.com/problems/sort-colors/)
6. [Remove Duplicates from Sorted Array](https://leetcode.com/problems/remove-duplicates-from-sorted-array/)
7. [Trapping Rain Water](https://leetcode.com/problems/trapping-rain-water/)

### 3. Sliding Window (8)

1. [Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters/)
2. [Minimum Size Subarray Sum](https://leetcode.com/problems/minimum-size-subarray-sum/)
3. [Permutation in String](https://leetcode.com/problems/permutation-in-string/)
4. [Fruit Into Baskets](https://leetcode.com/problems/fruit-into-baskets/)
5. [Max Consecutive Ones III](https://leetcode.com/problems/max-consecutive-ones-iii/)
6. [Longest Repeating Character Replacement](https://leetcode.com/problems/longest-repeating-character-replacement/)
7. [Find All Anagrams in a String](https://leetcode.com/problems/find-all-anagrams-in-a-string/)
8. [Subarray Product Less Than K](https://leetcode.com/problems/subarray-product-less-than-k/)

### 4. Prefix Sum (6)

1. [Running Sum of 1d Array](https://leetcode.com/problems/running-sum-of-1d-array/)
2. [Product of Array Except Self](https://leetcode.com/problems/product-of-array-except-self/)
3. [Find the Middle Index in Array](https://leetcode.com/problems/find-the-middle-index-in-array/)
4. [Contiguous Array](https://leetcode.com/problems/contiguous-array/)
5. [Range Sum Query - Immutable](https://leetcode.com/problems/range-sum-query-immutable/)
6. [Continuous Subarray Sum](https://leetcode.com/problems/continuous-subarray-sum/)

### 5. Top 'K' Elements / Heaps (6)

1. [Kth Largest Element in an Array](https://leetcode.com/problems/kth-largest-element-in-an-array/)
2. [K Closest Points to Origin](https://leetcode.com/problems/k-closest-points-to-origin/)
3. [Last Stone Weight](https://leetcode.com/problems/last-stone-weight/)
4. [Find Median from Data Stream](https://leetcode.com/problems/find-median-from-data-stream/)
5. [Task Scheduler](https://leetcode.com/problems/task-scheduler/)
6. [Reorganize String](https://leetcode.com/problems/reorganize-string/)

### 6. Intervals / Sorting (6)

1. [Merge Intervals](https://leetcode.com/problems/merge-intervals/)
2. [Insert Interval](https://leetcode.com/problems/insert-interval/)
3. [Non-overlapping Intervals](https://leetcode.com/problems/non-overlapping-intervals/)
4. [Minimum Number of Arrows to Burst Balloons](https://leetcode.com/problems/minimum-number-of-arrows-to-burst-balloons/)
5. [My Calendar I](https://leetcode.com/problems/my-calendar-i/)
6. [Car Pooling](https://leetcode.com/problems/car-pooling/)

### 7. Cyclic Sort / Array-Index Tricks (5)

1. [Missing Number](https://leetcode.com/problems/missing-number/)
2. [Find All Numbers Disappeared in an Array](https://leetcode.com/problems/find-all-numbers-disappeared-in-an-array/)
3. [Find the Duplicate Number](https://leetcode.com/problems/find-the-duplicate-number/)
4. [First Missing Positive](https://leetcode.com/problems/first-missing-positive/)
5. [Set Mismatch](https://leetcode.com/problems/set-mismatch/)

### 8. Matrix Manipulation (5)

1. [Rotate Image](https://leetcode.com/problems/rotate-image/)
2. [Spiral Matrix](https://leetcode.com/problems/spiral-matrix/)
3. [Set Matrix Zeroes](https://leetcode.com/problems/set-matrix-zeroes/)
4. [Search a 2D Matrix](https://leetcode.com/problems/search-a-2d-matrix/)
5. [Game of Life](https://leetcode.com/problems/game-of-life/)

### 9. DAG / Topological Sort (2)

*DE-relevant, since it maps to Airflow/dbt scheduling*

1. [Course Schedule](https://leetcode.com/problems/course-schedule/)
2. [Course Schedule II](https://leetcode.com/problems/course-schedule-ii/)