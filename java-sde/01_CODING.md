# 01 — Coding Questions for Interview

> **Target:** Service-based to mid-product companies. They test **clarity, correctness, edge cases and Java fluency** — not hard DS-Algo.
> **Total:** ~85 problems across all sections. Pick ~50 for 1-month plan, all ~85 for 2-month.
> **Java 8+ Streams** are in a separate file → [13_JAVA8_STREAMS_CODING.md](13_JAVA8_STREAMS_CODING.md)

**Difficulty:** 🟢 Easy (≤10 min) · 🟡 Medium (≤20 min) · 🔴 Hard (≤35 min — asked for senior differentiation)

---

## How to Practice

1. **Daily:** 1–2 problems, 45 min. Use a timer. Talk aloud (interviews test *explaining*).
2. **Method per problem:** Read → think 3 min → write brute force → optimize → code → test with an example → state time/space complexity.
3. **If stuck:** write the brute force first. Then ask: "what repeated work can I remove? Can I hash / sort / use two pointers?"
4. **Re-do:** every problem you struggled with → redo after 3 days and again after 10 days.
5. **Edge cases (always check):** null, empty, single element, duplicates, negatives, very large input.

---

## How to Approach a Coding Question (Step-by-Step)

Use this **every time** you get a problem — in practice and in the real interview. The steps should become automatic.

### Step 1 — Understand the problem (2–3 min)
- **Read the problem fully.** Don't start coding after the first sentence.
- **Repeat it back** in your own words: "So I need to find ... given ... and return ..."
- **Ask clarifying questions** (interviewers expect this):
  - Can the input be null or empty?
  - Can there be duplicates? Negative numbers?
  - Is the array/list sorted?
  - Should I return indices or values? What if there are multiple answers?
  - What's the expected size of the input? (tells you if O(n²) is acceptable)

### Step 2 — Work through examples (2 min)
- **Trace through the given example** by hand on paper/whiteboard.
- **Create your own example** — a small one (3–5 elements) and an edge case (empty, single element, all same values).
- This often reveals the pattern before you write any code.

### Step 3 — Identify the pattern (1–2 min)
**Read the problem clues and match them to a pattern:**

| When the problem says... | Think of... | Why |
|---|---|---|
| "Find if X exists", "count occurrences", "find duplicates", "find pair with sum" | **HashMap / HashSet** | O(1) lookup trades space for time |
| "Find pair in a **sorted** array", "remove duplicates in place", "container with most water" | **Two pointers** (from both ends or slow/fast) | Sorted order lets you decide which pointer to move |
| "Longest/shortest **substring** or **subarray** with some condition" | **Sliding window** | Expand right, shrink left — avoids checking all subarrays |
| "Top K", "Kth largest", "K most frequent", "merge K sorted" | **Heap / PriorityQueue** | Min-heap of size K finds Kth largest in O(n log K) |
| "Linked list: cycle, middle, Nth from end" | **Fast & slow pointers** | Fast moves 2x, slow 1x — they meet or fast hits end |
| "Linked list: reverse, merge, delete" | **Dummy node + pointer rewiring** | Dummy node eliminates head edge cases |
| "Matching brackets", "next greater element", "undo" | **Stack** | LIFO naturally handles nesting/matching |
| "Find element in **sorted** data", "find boundary / first occurrence" | **Binary search** | O(log n) by halving the search space |
| "Merge intervals", "overlapping ranges" | **Sort by start, then merge** | Sorting makes overlaps adjacent |
| "All combinations / permutations / subsets" | **Backtracking** | Choose → recurse → un-choose |
| "Optimal count, minimum cost, number of ways" | **Dynamic Programming** (basic: fibonacci, climbing stairs) | Overlapping subproblems → memoize |
| "Group items by a key", "frequency count", "aggregate by category" | **Java Streams** `groupingBy` / `toMap` | Declarative, concise for collection transforms |

**If no pattern jumps out:** start with the **brute force**. A working O(n²) beats an unfinished O(n). Then ask: "What repeated work am I doing? Can I remember (hash) previous results?"

### Step 4 — State your approach before coding (1 min)
- **Say it out loud:** "I'll use a HashMap to store each number and its index. For each number, I check if target minus that number is already in the map."
- **State the expected time and space complexity.**
- **Get a nod** from the interviewer before writing code. This saves you from coding the wrong approach.

### Step 5 — Write clean code (10–15 min)
- **Start with the method signature** — name, params, return type.
- **Handle edge cases first** at the top: `if (arr == null || arr.length == 0) return ...`
- **Use meaningful names:** `frequencyMap` not `m`, `leftPointer` not `l`.
- **Extract helper methods** if the code is getting long. Don't write one 50-line method.
- **Talk while you code** — explain each block: "Here I'm iterating through the array and checking..."

### Step 6 — Test with examples (2–3 min)
- **Dry-run** your code on the small example from Step 2. Walk through each line with actual values.
- **Then test an edge case:** empty input, single element, all duplicates, negative numbers.
- **Fix bugs calmly.** Don't panic — it's normal to have a small off-by-one or missed condition. The interviewer watches how you debug.

### Step 7 — Discuss complexity and follow-ups (1 min)
- **State time and space complexity** without being asked.
- **Mention trade-offs:** "This uses O(n) extra space for the HashMap. If space is a concern, I could sort the array and use two pointers for O(n log n) time and O(1) space."
- **Be ready for follow-ups:** "What if the array is too large for memory?" "What if we need this to be thread-safe?" "Can you do it in place?"

### Common Mistakes to Avoid
| Mistake | What to do instead |
|---|---|
| Start coding immediately after reading | Spend 3–5 min on Steps 1–4 first |
| Go silent while thinking | Think aloud — the interviewer can't read your mind and can help if you're on the wrong track |
| Chase the optimal solution when stuck | Write the brute force first, then optimize. A working O(n²) scores points; an unfinished O(n) scores zero |
| Compare `Integer` objects with `==` | Use `.equals()` — `==` only works for -128 to 127 (Integer cache) |
| Use `a - b` in a Comparator | Use `Integer.compare(a, b)` — subtraction can overflow |
| Forget null/empty checks | Add them at the top before the main logic |
| Mutate a collection while iterating | Use `Iterator.remove()`, `removeIf()`, or iterate a copy |
| Say "I'm done" without testing | Always dry-run with one example before declaring done |
| Never mention complexity | State it proactively — this is a senior-level signal |

### Pattern Decision Flowchart

```
START: What does the problem ask for?
│
├── Does it involve a LINKED LIST?
│   ├── Reverse / merge / delete → Dummy node + pointer manipulation
│   ├── Find middle / cycle / Nth from end → Fast-slow pointers
│   └── O(1) access + ordering (LRU cache) → HashMap + doubly linked list
│
├── Does it involve a STRING or ARRAY?
│   ├── Is the data SORTED?
│   │   ├── Find an element → Binary search
│   │   ├── Find a pair with a sum → Two pointers (from both ends)
│   │   └── Remove duplicates in place → Two pointers (slow-fast)
│   │
│   ├── Is it about a SUBSTRING / SUBARRAY with a condition?
│   │   └── Sliding window (expand right, shrink left)
│   │
│   ├── "Find / count / check existence / pair / duplicate"?
│   │   └── HashMap or HashSet
│   │
│   ├── "Subarray sum equals K" (with negatives)?
│   │   └── Prefix sum + HashMap
│   │
│   ├── "Top K / Kth largest / most frequent"?
│   │   └── Heap (PriorityQueue)
│   │
│   └── "Matching / nesting / next greater"?
│       └── Stack (often monotonic)
│
├── Does it involve INTERVALS / RANGES?
│   └── Sort by start → merge or sweep
│
├── Is it about GROUPING / AGGREGATION on objects?
│   └── Java Streams: groupingBy, toMap, joining
│
├── Is it about CONCURRENCY / THREADS?
│   └── wait/notify, Semaphore, BlockingQueue, CompletableFuture
│
└── Nothing fits?
    └── Write the BRUTE FORCE first (nested loops).
        Then optimize: "Can I hash previous results? Can I sort and use pointers?"
```

---

## Plan (~96 problems, 1 per day)

| Week | Sections |
|---|---|
| 1 | §1 Strings + §2 Numbers |
| 2 | §3 Arrays |
| 3 | §4 HashMap/HashSet |
| 4 | §5 LinkedList |
| 5 | §6 Heap + §7 Stack/Queue |
| 6 | §8 Sorting + §9 OOP + §10 Concurrency |
| 7 | §11 Machine-coding (1–2 builds) + Streams Top-10 |
| 8 | Timed mixed practice + redo mistakes + Streams Top-25 |

---

## 1. Strings (12 problems)

| # | Problem | Difficulty | Hint |
|---|---|---|---|
| 1 | **Reverse a string** — Given a string, return it reversed. Do not use `StringBuilder.reverse()`. Example: `"hello"` → `"olleh"` | 🟢 | Convert to `char[]`, swap from both ends with two pointers |
| 2 | **Palindrome check** — Given a string, check if it reads the same forwards and backwards. Ignore case and non-alphanumeric characters. Example: `"A man, a plan, a canal: Panama"` → `true` | 🟢 | Two pointers from both ends, skip non-alphanumeric, compare `toLowerCase` |
| 3 | **Anagram check** — Given two strings, check if one is an anagram of the other (same characters, same frequency, different order). Example: `"listen"`, `"silent"` → `true` | 🟢 | Count character frequency in an `int[26]` array, or sort both and compare |
| 4 | **First non-repeating character** — Given a string, find the first character that appears only once. Example: `"aabbcdd"` → `'c'` | 🟢 | Use `LinkedHashMap` to count frequency (preserves insertion order), then find first entry with count 1 |
| 5 | **Character frequency count** — Given a string, return the count of each character. Example: `"apple"` → `{a:1, p:2, l:1, e:1}` | 🟢 | `HashMap<Character, Integer>` with `merge(c, 1, Integer::sum)` or `int[26]` for lowercase letters |
| 6 | **Remove duplicate characters** — Given a string, remove duplicate characters while preserving the original order. Example: `"programming"` → `"progamin"` | 🟢 | Use `LinkedHashSet<Character>` (maintains insertion order, rejects duplicates) |
| 7 | **Reverse words in a sentence** — Given a sentence, reverse the order of words (not the characters within words). Example: `"hello world java"` → `"java world hello"` | 🟢 | Split by space, reverse the array, join back. Or use two-pointer reversal approach |
| 8 | **String compression** — Given a string, compress consecutive repeated characters with their count. Example: `"aaabbc"` → `"a3b2c1"`. If compressed is not shorter, return the original | 🟡 | Iterate through the string, count consecutive same characters, build result with `StringBuilder` |
| 9 | **Longest substring without repeating characters** — Given a string, find the length of the longest substring that has no repeating characters. Example: `"abcabcbb"` → `3` (substring `"abc"`) | 🟡 | **Sliding window**: expand right pointer, when a duplicate is found move left pointer past the previous occurrence. Use `HashMap<Character, Integer>` to track last index |
| 10 | **Balanced parentheses** — Given a string containing `()`, `[]`, `{}`, check if every opening bracket has a matching closing bracket in the correct order. Example: `"{[()]}"` → `true`, `"{[(])}"` → `false` | 🟡 | **Stack**: push opening brackets, pop on closing bracket and check if it matches |
| 11 | **Longest common prefix** — Given an array of strings, find the longest prefix string that is common to all of them. Example: `["flower", "flow", "flight"]` → `"fl"` | 🟡 | Compare character by character across all strings. Stop at the shortest string's length or first mismatch |
| 12 | **Group anagrams** — Given a list of strings, group the ones that are anagrams of each other. Example: `["eat","tea","tan","ate","nat","bat"]` → `[["eat","tea","ate"], ["tan","nat"], ["bat"]]` | 🟡 | **HashMap** with sorted-word as key: sort each word's characters, use that as the map key, collect words into lists |

---

## 2. Numbers & Math (8 problems)

| # | Problem | Difficulty | Hint |
|---|---|---|---|
| 1 | **FizzBuzz** — Print numbers from 1 to N. For multiples of 3 print "Fizz", multiples of 5 print "Buzz", multiples of both print "FizzBuzz". Example: `15` → `"FizzBuzz"` | 🟢 | Use modulo `%`. Check 15 (both) before 3 or 5 individually |
| 2 | **Prime number check** — Given a number, determine if it is a prime number (divisible only by 1 and itself). Example: `29` → `true`, `15` → `false` | 🟢 | Loop from 2 to √n. If any number divides evenly, it's not prime. Handle edge cases: 0, 1, 2, negatives |
| 3 | **Fibonacci sequence** — Generate the first N Fibonacci numbers, or return the Nth Fibonacci number. Example: N=7 → `[0, 1, 1, 2, 3, 5, 8]`. Implement iteratively, recursively, and with memoization | 🟢 | Iterative: two variables. Recursive: `fib(n) = fib(n-1) + fib(n-2)`. Memoized: cache results in an array |
| 4 | **Factorial** — Calculate the factorial of a number (n! = n × (n-1) × ... × 1). Example: `5! = 120`. Implement both iteratively and recursively | 🟢 | Iterative: loop multiply. Recursive: `n * factorial(n-1)`, base case `0! = 1` |
| 5 | **GCD (Greatest Common Divisor)** — Find the GCD of two numbers using the Euclidean algorithm. Example: `gcd(12, 8)` → `4` | 🟢 | `gcd(a, b) = gcd(b, a % b)` until `b == 0`, then `a` is the GCD |
| 6 | **Swap two numbers without a temp variable** — Given two variables, swap their values without using a third variable. Example: `a=5, b=10` → `a=10, b=5` | 🟢 | XOR: `a ^= b; b ^= a; a ^= b;` or arithmetic: `a = a + b; b = a - b; a = a - b;` |
| 7 | **Palindrome number** — Check if an integer reads the same forwards and backwards, without converting to a string. Example: `121` → `true`, `-121` → `false` | 🟢 | Reverse the digits by extracting last digit (`% 10`) and building reversed number. Compare with original. Negatives are not palindromes |
| 8 | **Sieve of Eratosthenes** — Find all prime numbers up to a given number N. Example: N=20 → `[2, 3, 5, 7, 11, 13, 17, 19]` | 🟡 | Create boolean array `isPrime[0..N]` initialized to `true`. For each prime p starting from 2, mark all multiples of p as `false` |

---

## 3. Arrays (15 problems)

| # | Problem | Difficulty | Hint |
|---|---|---|---|
| 1 | **Largest and second-largest** — Given an array of integers, find the largest and second-largest elements in a single pass. Example: `[5, 2, 8, 1, 9]` → largest=9, second=8 | 🟢 | Track two variables: `first` and `second`. Update both as you iterate |
| 2 | **Reverse an array in place** — Reverse the elements of an array without using a new array. Example: `[1, 2, 3, 4]` → `[4, 3, 2, 1]` | 🟢 | Two pointers from start and end, swap and move inward |
| 3 | **Missing number in 1..N** — Given an array containing N-1 distinct numbers from the range 1 to N, find the missing number. Example: `[1, 2, 4, 5]` → `3` | 🟢 | Calculate expected sum `N*(N+1)/2`, subtract actual sum. Or use XOR of all numbers with 1..N |
| 4 | **Find the duplicate** — Given an array of N+1 numbers where each number is between 1 and N, find the duplicate. Example: `[1, 3, 4, 2, 2]` → `2` | 🟢 | HashSet (`add` returns false for duplicates), or sort and check adjacent, or sum/XOR approach |
| 5 | **Rotate array by K positions** — Rotate an array to the right by K positions. Example: `[1, 2, 3, 4, 5]`, K=2 → `[4, 5, 1, 2, 3]` | 🟢 | Reverse trick: reverse entire array → reverse first K → reverse remaining. Handle K > length with `K %= length` |
| 6 | **Move all zeroes to end** — Move all zeroes in an array to the end while maintaining the relative order of non-zero elements. Example: `[0, 1, 0, 3, 12]` → `[1, 3, 12, 0, 0]` | 🟡 | Two pointers: a "write" pointer for non-zero elements and a "read" pointer scanning forward |
| 7 | **Two Sum** — Given an array of integers and a target sum, find the indices of two numbers that add up to the target. Example: `[2, 7, 11, 15]`, target=9 → `[0, 1]` | 🟡 | **HashMap**: for each number, check if `target - number` is already in the map. Store number → index |
| 8 | **Merge two sorted arrays** — Given two sorted arrays, merge them into a single sorted array. Example: `[1, 3, 5]` + `[2, 4, 6]` → `[1, 2, 3, 4, 5, 6]` | 🟡 | Two pointers, one per array. Compare and pick the smaller, advance that pointer |
| 9 | **Remove duplicates from sorted array (in place)** — Given a sorted array, remove duplicates in place and return the new length. The array should have unique elements at the front. Example: `[1, 1, 2, 2, 3]` → `[1, 2, 3, _, _]`, return 3 | 🟡 | Two pointers: slow (write position) and fast (scanner). Copy when `fast != slow` |
| 10 | **Binary search** — Given a sorted array and a target value, return its index or -1 if not found. Implement iteratively. Example: `[1, 3, 5, 7, 9]`, target=7 → index 3 | 🟡 | `lo=0`, `hi=len-1`, `mid = lo + (hi-lo)/2`. Compare and adjust `lo` or `hi`. Use `(hi-lo)/2` to avoid integer overflow |
| 11 | **First and last occurrence in sorted array** — Given a sorted array with duplicates and a target, find the first and last index of the target. Example: `[1, 2, 2, 2, 3]`, target=2 → `[1, 3]` | 🟡 | Run binary search twice: one modified to find the leftmost match, another for the rightmost match |
| 12 | **Maximum subarray sum (Kadane's algorithm)** — Given an array of integers (can be negative), find the contiguous subarray with the largest sum. Example: `[-2, 1, -3, 4, -1, 2, 1, -5, 4]` → `6` (subarray `[4, -1, 2, 1]`) | 🟡 | Keep a running sum. If it goes negative, reset to 0. Track the maximum seen so far |
| 13 | **Intersection and union of two arrays** — Given two arrays, find (a) common elements and (b) all unique elements across both. Example: `[1, 2, 2, 3]` + `[2, 3, 4]` → intersection: `[2, 3]`, union: `[1, 2, 3, 4]` | 🟡 | Use `HashSet` for each array. Intersection: elements in both sets. Union: add all to one set |
| 14 | **Three Sum** — Given an array of integers, find all unique triplets that sum to zero. Example: `[-1, 0, 1, 2, -1, -4]` → `[[-1, -1, 2], [-1, 0, 1]]` | 🟡 | Sort the array. Fix one element, use two pointers for the remaining pair. Skip duplicates |
| 15 | **Merge overlapping intervals** — Given a list of intervals `[start, end]`, merge all overlapping intervals. Example: `[[1,3],[2,6],[8,10],[15,18]]` → `[[1,6],[8,10],[15,18]]` | 🔴 | Sort intervals by start time. Iterate: if current overlaps with previous, merge (extend the end). Otherwise, add as new |

---

## 4. HashMap / HashSet (13 problems)

| # | Problem | Difficulty | Hint |
|---|---|---|---|
| 1 | **Contains duplicate** — Given an array, return `true` if any value appears at least twice. Example: `[1, 2, 3, 1]` → `true` | 🟢 | `HashSet.add(x)` returns `false` if element already exists |
| 2 | **Intersection of two arrays** — Given two arrays, return their common elements (unique). Example: `[1, 2, 2, 1]` + `[2, 2]` → `[2]` | 🟢 | Put one array in a `HashSet`, iterate the other and check `contains` |
| 3 | **Two Sum (return indices)** — Given an array and a target, return the indices of two numbers that add up to the target. Each input has exactly one solution. Example: `[2, 7, 11, 15]`, target=9 → `[0, 1]` | 🟢 | `HashMap<Integer, Integer>` storing value → index. For each number, check if `target - num` exists in the map |
| 4 | **Valid anagram** — Given two strings, check if the second is an anagram of the first (same characters, same count). Example: `"anagram"`, `"nagaram"` → `true` | 🟢 | Count character frequency using `int[26]`. Increment for string 1, decrement for string 2. All counts should be 0 |
| 5 | **First non-repeating character** — Given a string, find the first character that occurs only once. Example: `"aabbcdd"` → `'c'` | 🟢 | Use `LinkedHashMap` (preserves insertion order) to count frequency. Then find first entry with value 1 |
| 6 | **Majority element** — Given an array, find the element that appears more than N/2 times. It is guaranteed to exist. Example: `[3, 2, 3]` → `3` | 🟡 | HashMap counting, or **Boyer–Moore voting algorithm**: maintain a candidate and count; increment for same, decrement for different; when count hits 0, switch candidate |
| 7 | **Group anagrams** — Given a list of strings, group strings that are anagrams of each other together. Example: `["eat","tea","tan","ate","nat","bat"]` → `[["eat","tea","ate"], ["tan","nat"], ["bat"]]` | 🟡 | Sort each word's characters to create a key. Use `HashMap<String, List<String>>` with `computeIfAbsent` |
| 8 | **Top-K frequent elements** — Given an integer array and a number K, return the K most frequent elements. Example: `[1,1,1,2,2,3]`, K=2 → `[1, 2]` | 🟡 | First count frequency with `HashMap`. Then use a **min-heap (PriorityQueue)** of size K sorted by frequency. Or bucket sort by frequency |
| 9 | **Longest consecutive sequence** — Given an unsorted array, find the length of the longest sequence of consecutive numbers. Example: `[100, 4, 200, 1, 3, 2]` → `4` (sequence: 1, 2, 3, 4) | 🟡 | Put all numbers in a `HashSet`. For each number, only start counting if `num - 1` is NOT in the set (this is the start of a sequence). Count upward with `num + 1`, `num + 2`, etc. |
| 10 | **Subarray sum equals K** — Given an array of integers and a target K, find the total number of contiguous subarrays whose sum equals K. Example: `[1, 1, 1]`, K=2 → `2` | 🟡 | **Prefix sum + HashMap**: maintain running sum. Store `HashMap<sum, count>`. For each position, check if `sum - K` exists in the map (that means there's a subarray ending here with sum K) |
| 11 | **Sort a HashMap by value** — Given a `HashMap<String, Integer>`, return the entries sorted by value in descending order. Example: `{apple:3, banana:1, cherry:5}` → `cherry=5, apple=3, banana=1` | 🟡 | Stream `entrySet()`, sort with `Map.Entry.comparingByValue().reversed()`, collect into `LinkedHashMap` to preserve order |
| 12 | **LRU Cache** — Design a data structure that supports `get(key)` and `put(key, value)` in O(1). When the cache exceeds its capacity, evict the **least recently used** item. Example: capacity=2, put(1,1), put(2,2), get(1)→1, put(3,3) evicts key 2 | 🔴 | `HashMap<Key, Node>` for O(1) lookup + **doubly linked list** for O(1) add/remove (most recent at head, LRU at tail). Or use `LinkedHashMap` with access-order constructor |
| 13 | **Insert, delete, getRandom in O(1)** — Design a data structure that supports `insert(val)`, `remove(val)`, and `getRandom()` each in average O(1) time. `getRandom` should return a random element with equal probability | 🔴 | `HashMap<value, index>` + `ArrayList`. Insert: append to list, store index in map. Remove: swap the element with the last element in the list, update map, remove last. Random: `list.get(random.nextInt(size))` |

**Key Java APIs to know:**
```java
map.getOrDefault(key, 0)
map.merge(key, 1, Integer::sum)              // increment count
map.computeIfAbsent(key, k -> new ArrayList<>()).add(val)   // group-by
map.putIfAbsent(key, val)
set.add(x)   // returns false if already present
```

---

## 5. LinkedList (12 problems)

| # | Problem | Difficulty | Hint |
|---|---|---|---|
| 1 | **Reverse a linked list** — Given the head of a singly linked list, reverse it and return the new head. Example: `1→2→3→4→5` → `5→4→3→2→1` | 🟢 | Three pointers: `prev`, `curr`, `next`. Save next, rewire curr.next to prev, advance both |
| 2 | **Find the middle node** — Given a linked list, return the middle node. If even length, return the second middle. Example: `1→2→3→4→5` → node with value 3 | 🟢 | **Fast/slow pointers**: slow moves 1 step, fast moves 2 steps. When fast reaches the end, slow is at middle |
| 3 | **Merge two sorted linked lists** — Given two sorted linked lists, merge them into one sorted list. Example: `1→3→5` + `2→4→6` → `1→2→3→4→5→6` | 🟢 | **Dummy node** technique: create a dummy head, compare nodes from both lists, append the smaller one |
| 4 | **Remove duplicates from sorted list** — Given a sorted linked list, delete all duplicate values, leaving only distinct values. Example: `1→1→2→3→3` → `1→2→3` | 🟢 | Compare `curr.val` with `curr.next.val`. If equal, skip the next node (`curr.next = curr.next.next`) |
| 5 | **Detect cycle** — Given a linked list, determine if it has a cycle (a node's next points back to a previous node). Example: `1→2→3→4→2` (4 points back to 2) → `true` | 🟡 | **Floyd's cycle detection**: fast pointer moves 2 steps, slow moves 1. If they meet, there is a cycle |
| 6 | **Find the start of the cycle** — If a linked list has a cycle, find the node where the cycle begins. Example: `1→2→3→4→5→3` (5 points back to 3) → node with value 3 | 🟡 | After fast and slow meet (Floyd's), reset one pointer to head. Move both one step at a time — they will meet at the cycle start |
| 7 | **Remove Nth node from end** — Given a linked list and N, remove the Nth node from the end. Example: `1→2→3→4→5`, N=2 → `1→2→3→5` (removed 4) | 🟡 | **Two pointers with gap**: move the first pointer N steps ahead. Then move both until the first reaches the end. The second pointer is just before the node to remove. Use a **dummy node** to handle removing the head |
| 8 | **Palindrome linked list** — Check if a linked list is a palindrome. Example: `1→2→2→1` → `true` | 🟡 | Find middle (fast/slow) → reverse the second half → compare first half with reversed second half |
| 9 | **Add two numbers** — Two numbers are represented as linked lists where each node contains a single digit in reverse order. Add them and return the sum as a linked list. Example: `2→4→3` + `5→6→4` → `7→0→8` (342 + 465 = 807) | 🟡 | Iterate both lists simultaneously, sum digits + carry. Create new nodes for the result. Don't forget the final carry |
| 10 | **Intersection of two linked lists** — Given two linked lists that may merge at some point, find the node where they intersect, or null if they don't. Example: `1→2→3→6→7` and `4→5→6→7` → node with value 6 | 🟡 | Calculate length difference, advance the longer list by the difference, then walk both together until they meet. Or: two-pointer swap trick (when one reaches the end, redirect to the other's head) |
| 11 | **Reorder list** — Rearrange a linked list from `L0→L1→L2→...→Ln` to `L0→Ln→L1→Ln-1→L2→Ln-2→...`. Example: `1→2→3→4→5` → `1→5→2→4→3` | 🟡 | Three steps: (1) find middle with fast/slow, (2) reverse the second half, (3) merge the two halves alternately |
| 12 | **LRU Cache using HashMap + doubly linked list** — Implement an LRU cache with O(1) `get` and `put`. On `get`, move the accessed node to the front (most recently used). On `put`, add to front; if capacity exceeded, remove the node at the tail (least recently used) | 🔴 | Maintain a `HashMap<key, Node>` for O(1) lookup and a doubly linked list with sentinel head/tail nodes for O(1) add-to-front and remove-from-tail |

**Templates to memorize:**
```java
// Reverse
ListNode prev = null, curr = head;
while (curr != null) {
    ListNode next = curr.next;
    curr.next = prev;
    prev = curr;
    curr = next;
}
return prev;

// Fast-slow (middle / cycle)
ListNode slow = head, fast = head;
while (fast != null && fast.next != null) {
    slow = slow.next;
    fast = fast.next.next;
}
// slow is at middle (or meeting point if checking cycle)

// Dummy node (simplifies head edge cases)
ListNode dummy = new ListNode(0);
dummy.next = head;
// ... work with dummy ...
return dummy.next;
```

---

## 6. PriorityQueue / Heap (8 problems)

| # | Problem | Difficulty | Hint |
|---|---|---|---|
| 1 | **Kth largest element** — Given an unsorted array and a number K, find the Kth largest element. Example: `[3, 2, 1, 5, 6, 4]`, K=2 → `5` | 🟢 | Maintain a **min-heap of size K**. Add elements; if heap size exceeds K, remove the smallest. At the end, `peek()` gives the Kth largest |
| 2 | **Last stone weight** — You have a collection of stones with integer weights. Each turn, pick the two heaviest stones and smash them. If they're not equal, the remainder goes back. Return the weight of the last remaining stone (or 0). Example: `[2, 7, 4, 1, 8, 1]` → `1` | 🟢 | Use a **max-heap**. Poll two largest, push their difference (if non-zero) back. Repeat until ≤1 stone remains |
| 3 | **Top-K frequent elements** — Given an array and K, return the K elements that appear most frequently. Example: `[1,1,1,2,2,3]`, K=2 → `[1, 2]` | 🟡 | Count frequency with `HashMap`. Then use a **min-heap of size K** sorted by frequency — this keeps only the K most frequent |
| 4 | **K closest points to origin** — Given a list of 2D points and K, return the K points closest to the origin (0,0). Distance = √(x² + y²), but you can compare x²+y² directly. Example: `[[1,3],[-2,2]]`, K=1 → `[[-2,2]]` | 🟡 | **Max-heap of size K** sorted by distance. Add each point; if heap exceeds K, remove the farthest. Remaining K points are the closest |
| 5 | **Sort characters by frequency** — Given a string, sort its characters in decreasing order of frequency. Example: `"tree"` → `"eert"` or `"eetr"` | 🟡 | Count frequency with `HashMap`, then use a **max-heap** sorted by frequency. Poll characters and append count times. Or use bucket sort |
| 6 | **Merge K sorted lists** — Given K sorted linked lists, merge them into one sorted list. Example: `[1→4→5, 1→3→4, 2→6]` → `1→1→2→3→4→4→5→6` | 🟡 | Put the head of each list into a **min-heap** (sorted by node value). Poll the smallest, add it to the result, and push its `.next` back into the heap. Time: O(N log K) |
| 7 | **Kth largest element in a stream** — Design a class that, given a stream of integers, can return the Kth largest element at any point. Example: K=3, stream `[4,5,8,2]` → after adding 2, 3rd largest = 4 | 🟡 | Maintain a **min-heap of size K**. On each `add()`: if heap size < K, add. Otherwise, if new element > `peek()`, poll and add. `peek()` always gives the Kth largest |
| 8 | **Find median from data stream** — Design a data structure that supports adding numbers from a stream and finding the median at any time. Example: add 1, add 2 → median=1.5; add 3 → median=2 | 🔴 | Use **two heaps**: a max-heap for the lower half and a min-heap for the upper half. Balance so they differ by at most 1 in size. Median = top of the larger heap, or average of both tops |

**Key points:**
- Java `PriorityQueue` = **min-heap** by default.
- Max-heap: `new PriorityQueue<>(Collections.reverseOrder())`
- Use `Integer.compare(a, b)`, never `a - b` (overflow risk).
- `poll()` gives elements in sorted order; iterating the queue does **not**.

```java
// Kth largest — keep min-heap of size K
PriorityQueue<Integer> minHeap = new PriorityQueue<>();
for (int n : nums) {
    minHeap.offer(n);
    if (minHeap.size() > k) minHeap.poll();
}
return minHeap.peek();  // Kth largest
```

---

## 7. Stack / Queue (6 problems)

| # | Problem | Difficulty | Hint |
|---|---|---|---|
| 1 | **Valid parentheses** — Given a string containing `(`, `)`, `{`, `}`, `[`, `]`, determine if the brackets are valid (every open bracket has a matching close bracket in the correct nesting order). Example: `"{[()]}"` → `true`, `"([)]"` → `false` | 🟢 | **Stack**: push opening brackets. On a closing bracket, pop and check if it matches. At the end, the stack should be empty |
| 2 | **Implement queue using two stacks** — Build a queue (FIFO) using only two stacks (LIFO). Support `enqueue`, `dequeue`, and `peek` operations | 🟢 | Push stack and pop stack. Enqueue: push onto stack 1. Dequeue: if stack 2 is empty, transfer all from stack 1 to stack 2 (this reverses order); then pop from stack 2 |
| 3 | **Min Stack** — Design a stack that supports `push`, `pop`, `top`, and `getMin` — all in O(1) time. Example: push 5, push 2, push 7 → `getMin()` = 2, `pop()` removes 7, `getMin()` still = 2 | 🟡 | Store pairs of `(value, currentMin)` on the stack. When pushing, the new min is `Math.min(value, currentTop.min)`. When popping, the min is always in the top element |
| 4 | **Next greater element** — Given an array, for each element find the next element to its right that is greater. If none exists, return -1. Example: `[4, 5, 2, 10]` → `[5, 10, 10, -1]` | 🟡 | **Monotonic stack** (decreasing): iterate through the array. Pop elements from the stack while the current element is greater than the top — for each popped element, the answer is the current element. Push current index |
| 5 | **Daily temperatures** — Given a list of daily temperatures, for each day find how many days you have to wait until a warmer temperature. If no warmer day, return 0. Example: `[73, 74, 75, 71, 69, 72, 76, 73]` → `[1, 1, 4, 2, 1, 1, 0, 0]` | 🟡 | **Monotonic stack** of indices. Same pattern as "next greater element" but store index differences |
| 6 | **Evaluate reverse Polish notation** — Evaluate an arithmetic expression in Reverse Polish Notation (postfix). Tokens are numbers or operators `+`, `-`, `*`, `/`. Example: `["2","1","+","3","*"]` → `9` (i.e., `(2+1)*3`) | 🟡 | **Stack**: push numbers. When you encounter an operator, pop two operands, apply the operator, push the result back. The final element on the stack is the answer |

```java
// Monotonic stack — next greater element
int[] result = new int[n];
Arrays.fill(result, -1);
Deque<Integer> stack = new ArrayDeque<>();   // stores indices
for (int i = 0; i < n; i++) {
    while (!stack.isEmpty() && arr[stack.peek()] < arr[i])
        result[stack.pop()] = arr[i];
    stack.push(i);
}
```
**Note:** Use `ArrayDeque` (not `Stack` class) for stack and queue in Java.

---

## 8. Sorting & Searching (5 problems)

| # | Problem | Difficulty | Hint |
|---|---|---|---|
| 1 | **Sort 0s, 1s and 2s (Dutch National Flag)** — Given an array containing only 0s, 1s, and 2s, sort it in place in a single pass. Example: `[2, 0, 1, 2, 0, 1]` → `[0, 0, 1, 1, 2, 2]` | 🟡 | Three pointers: `low`, `mid`, `high`. Swap 0s to the left (low), 2s to the right (high), 1s stay in the middle |
| 2 | **Sort a list of objects by multiple fields** — Given a list of `Employee` objects, sort by department (ascending), then by salary (descending) within each department. Example: sort employees first by dept name A-Z, then highest salary first within each dept | 🟡 | `Comparator.comparing(Employee::getDept).thenComparing(Employee::getSalary, Comparator.reverseOrder())` |
| 3 | **Find peak element** — A peak element is one that is strictly greater than its neighbors. Given an array, find any peak element's index. The array may have multiple peaks. Example: `[1, 2, 3, 1]` → index 2 (value 3) | 🟡 | **Binary search**: if `mid` element < its right neighbor, a peak exists on the right half; otherwise on the left half |
| 4 | **Search in rotated sorted array** — Given a sorted array that has been rotated at some pivot (e.g., `[4,5,6,7,0,1,2]`), search for a target value. Example: `[4,5,6,7,0,1,2]`, target=0 → index 4 | 🟡 | **Modified binary search**: determine which half is sorted, then check if the target lies in the sorted half |
| 5 | **Integer square root** — Given a non-negative integer N, compute the integer part of √N without using `Math.sqrt`. Example: `sqrt(8)` → `2` (since √8 ≈ 2.828) | 🟡 | **Binary search** on the answer space: search for the largest `x` such that `x * x <= N`. Search range [0, N] |

---

## 9. OOP-Flavoured Coding (6 problems)

| # | Problem | Difficulty | What it tests |
|---|---|---|---|
| 1 | **Create an immutable class** — Design a Java class whose instances cannot be modified after creation. The class should have fields like name, date-of-birth, and a list of hobbies. Ensure no one can change any field or the list's contents after construction | 🟢 | `final` class, `final` fields, no setters, constructor with **defensive copies** of mutable objects (like `List` and `Date`), return copies from getters |
| 2 | **Implement `equals()`, `hashCode()`, `toString()`** — For a `Student` class with fields `id`, `name`, and `email`, implement all three methods correctly following the Java contract. Explain what breaks if `hashCode()` is wrong | 🟢 | `equals`: check type, compare fields. `hashCode`: use same fields as `equals` (contract: equal objects **must** have equal hash codes). `toString`: return meaningful representation |
| 3 | **Thread-safe Singleton (3 ways)** — Implement the Singleton design pattern (only one instance of a class exists) in a thread-safe way. Show three different approaches and explain trade-offs | 🟡 | (1) **Enum singleton** (simplest, serialization-safe). (2) **Bill Pugh** (inner static holder class, lazy). (3) **Double-checked locking** with `volatile` (explain why `volatile` is needed) |
| 4 | **Builder pattern** — Design a `UserProfile` class with 8+ fields (name, email, phone, address, age, etc.) where most fields are optional. Use the Builder pattern so callers can set only the fields they need | 🟡 | Static inner `Builder` class with fluent setter methods that return `this`. The `build()` method validates required fields and returns the immutable object |
| 5 | **Comparable vs Comparator** — Given an `Employee` class, implement natural ordering by `id` using `Comparable`, and create separate `Comparator` implementations for sorting by name, by salary, and by multiple fields | 🟡 | `Comparable`: implement `compareTo()` in the class. `Comparator`: create external comparator objects, chain with `thenComparing()` |
| 6 | **Custom exceptions** — Create a custom checked exception (`InsufficientFundsException`) and a custom unchecked exception (`InvalidAccountException`). Use them in a `BankAccount` service. Explain when to use checked vs unchecked | 🟡 | Checked: extends `Exception` (caller must handle or declare). Unchecked: extends `RuntimeException` (programming errors, validation failures). Include meaningful messages and cause chains |

---

## 10. Concurrency Coding (6 problems)

| # | Problem | Difficulty | Hint |
|---|---|---|---|
| 1 | **Print odd/even using two threads** — Create two threads: one prints odd numbers (1, 3, 5, ...) and the other prints even numbers (2, 4, 6, ...) up to N. They must alternate correctly: 1, 2, 3, 4, 5, ... | 🟡 | Use `synchronized` + `wait()`/`notify()` on a shared lock object. Or use two `Semaphore`s |
| 2 | **Print numbers and letters alternately** — Two threads: Thread-1 prints 1, 2, 3, ... and Thread-2 prints A, B, C, ... They must alternate: `1 A 2 B 3 C ...` | 🟡 | Same `wait/notify` pattern. Shared boolean flag to indicate whose turn it is |
| 3 | **Create a deadlock and then fix it** — Write code that deliberately creates a deadlock between two threads (each waiting for a lock the other holds). Then modify it to fix the deadlock | 🟡 | Deadlock: Thread-1 locks A then B, Thread-2 locks B then A. Fix: both threads lock in the **same order** (A then B), or use `tryLock` with timeout |
| 4 | **Producer-Consumer with BlockingQueue** — Implement the producer-consumer pattern: one thread produces items and adds them to a shared queue, another thread consumes items from the queue. Use `BlockingQueue` for thread-safe communication | 🟡 | `ArrayBlockingQueue` with bounded capacity. Producer calls `put()` (blocks if full). Consumer calls `take()` (blocks if empty). Use a **poison pill** value to signal termination |
| 5 | **Run N tasks in parallel and collect results** — Given a list of tasks (e.g., call 5 different APIs), run them all in parallel using a thread pool, wait for all to complete, and collect all results | 🟡 | `ExecutorService` + `submit()` returns `Future<T>`. Collect all futures, call `get()` on each. Or use `CompletableFuture.supplyAsync()` + `join()` |
| 6 | **Call 3 APIs in parallel, combine results with timeout** — Make 3 independent API calls concurrently. Combine their results into one response. If any call takes more than 2 seconds, use a fallback value | 🔴 | `CompletableFuture.supplyAsync()` for each call. Combine with `thenCombine()` or `allOf()`. Apply timeout with `.orTimeout(2, SECONDS)` and fallback with `.exceptionally(ex -> defaultValue)` |

```java
// Producer-Consumer
BlockingQueue<Integer> q = new ArrayBlockingQueue<>(5);
Thread producer = new Thread(() -> {
    try { for (int i = 0; i < 10; i++) q.put(i); q.put(-1); }
    catch (InterruptedException e) { Thread.currentThread().interrupt(); }
});
Thread consumer = new Thread(() -> {
    try { for (int v; (v = q.take()) != -1; ) System.out.println(v); }
    catch (InterruptedException e) { Thread.currentThread().interrupt(); }
});
producer.start(); consumer.start();
```

---

## 11. Machine-Coding (60–90 min rounds) (5 problems)

These are practical build-and-code rounds. Get the **happy path working first**, then add validation, error handling, and tests.

| # | Problem | Difficulty | What to build |
|---|---|---|---|
| 1 | **Parse a CSV/log file and compute aggregates** — Read a CSV file of sales records (product, region, amount). Compute total sales per region, top product by sales, and average sale amount. Handle malformed rows gracefully | 🟡 | File I/O (`BufferedReader`/`Files.lines`), `Stream` processing, `groupingBy`, `summingDouble`, exception handling for bad rows |
| 2 | **REST CRUD API with validation** — Build a Spring Boot REST API for managing `Employee` records (create, read, update, delete). Add input validation (name required, salary > 0) and return proper error responses (400, 404, 500) | 🟡 | `@RestController`, `@Valid`, `@ExceptionHandler`/`@ControllerAdvice`, proper HTTP status codes, DTO vs entity |
| 3 | **Retry with exponential backoff** — Implement a generic retry mechanism that retries a failed operation up to N times with exponentially increasing delays (1s, 2s, 4s, 8s...). Add jitter (random variation) to prevent thundering herd | 🟡 | Generic method accepting a `Supplier<T>`, loop with `Thread.sleep(delay)`, `delay *= 2`, add random jitter. Distinguish retryable vs non-retryable exceptions |
| 4 | **In-memory key-value store with TTL** — Design a `put(key, value, ttlSeconds)` / `get(key)` store where entries expire after their TTL. `get` should return null for expired entries. Handle cleanup of expired entries | 🔴 | `ConcurrentHashMap<Key, Entry>` where `Entry` holds value + expiry timestamp. Lazy cleanup on `get`, plus a background `ScheduledExecutorService` for periodic cleanup |
| 5 | **URL shortener** — Build a service that takes a long URL and returns a short code (e.g., `abc123`). Given a short code, redirect to the original URL. Handle collisions, track click counts, and support expiry | 🔴 | Counter/hash-based ID → base62 encode, `HashMap<code, URL>` or DB, collision check, atomic counter or `UUID` |

---

## 12. Java 8+ Streams

→ **Separate file: [13_JAVA8_STREAMS_CODING.md](13_JAVA8_STREAMS_CODING.md)**

Covers: sorting, grouping, **two-level grouping**, `Collectors.joining`, strings, `flatMap`, maps, **date/time**, pitfalls.

**Start with the Top-10:** count per dept · avg salary per dept · highest paid per dept · 2nd highest salary · multi-field sort · dept → joined names · dept → city → names (2-level grouping) · char frequency · word frequency top 3 · group by joining year.

---

## Summary — Problem Count by Section

| Section | Topic | Problems |
|---|---|---|
| §1 | Strings | 12 |
| §2 | Numbers & Math | 8 |
| §3 | Arrays | 15 |
| §4 | HashMap / HashSet | 13 |
| §5 | LinkedList | 12 |
| §6 | PriorityQueue / Heap | 8 |
| §7 | Stack / Queue | 6 |
| §8 | Sorting & Searching | 5 |
| §9 | OOP-Flavoured | 6 |
| §10 | Concurrency | 6 |
| §11 | Machine-Coding | 5 |
| §12 | Streams | [separate file](13_JAVA8_STREAMS_CODING.md) |
| | **Total** | **96** |

For the **1-month plan**, pick ~50 (all 🟢 + select 🟡). For the **2-month plan**, do all ~96.

---

## Self-Check

You're ready when you can:
- [ ] Solve a 🟢 in ≤10 min and a 🟡 in ≤20 min, talking aloud
- [ ] Write from memory: reverse list, Two Sum, valid parentheses, Kth largest (heap), LRU cache, producer-consumer
- [ ] State time/space complexity for every solution
- [ ] Name 2 edge cases for every problem before coding
