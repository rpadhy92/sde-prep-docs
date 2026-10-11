# 01 — Coding Round Preparation

## ⭐ Fundamentals First (start here)
**Time:** 1-month plan **17 h** · 2-month plan **24 h** for this file — plus **3 h / 6 h** for Java 8+ Streams in [13_JAVA8_STREAMS_CODING.md](13_JAVA8_STREAMS_CODING.md) (total coding budget 20 h / 30 h; see [README](README.md)). Spread as 45–60 min *every day*.

**Learn in this order**
1. The approach: **C-E-A-C-T-T** framework (§2) and the **4-pass practice method** (§1.2) — 30 min, once.
2. Java API fluency: `String/StringBuilder`, `ArrayList`, `HashMap` (`getOrDefault`, `merge`, `computeIfAbsent`), `HashSet`, `ArrayDeque`, `PriorityQueue`, `Comparator`, `Arrays.sort`.
3. The 6 core patterns (§3): hashing, two pointers, sliding window, fast/slow pointers, size-K heap, stack.

**Starter set — solve these 12 first, aloud, with a timer**
Two Sum · valid anagram · first non-repeating char · palindrome · reverse linked list · merge two sorted lists · cycle detection · valid parentheses · Kth largest (heap) · top-K frequent · group anagrams · LRU cache.

**Done when:** a 🟢 takes ≤10 min, a 🟡 ≤25 min, and you state complexity + 2 edge cases every time.
**Skip if short on time:** all 🔴 items, trapping rain water, custom HashMap, histogram, DI container.

> Goal: solve **basic-to-intermediate** problems cleanly, in Java, while talking. For service/mid-product companies, **clarity, correctness, edge cases and idiomatic Java** beat clever algorithms.
> Legend: 🟢 ≤10 min · 🟡 ≤25 min · 🔴 ≤40 min

**Contents**
1. [How to Improve Your Coding Skills (Study Method)](#1-how-to-improve-your-coding-skills)
2. [In-Interview Approach (Framework)](#2-in-interview-approach-framework)
3. [Pattern Cheat-Sheet](#3-pattern-cheat-sheet)
4. [Strings & Arrays](#4-strings--arrays)
5. [LinkedList (approach + templates + questions)](#5-linkedlist)
6. [HashMap / HashSet (approach + templates + questions)](#6-hashmap--hashset)
7. [PriorityQueue / Heap (approach + templates + questions)](#7-priorityqueue--heap)
8. [Stack / Queue / Deque](#8-stack--queue--deque)
9. [Java 8+ Streams → separate file `13_JAVA8_STREAMS_CODING.md`](13_JAVA8_STREAMS_CODING.md) (grouping, 2-level grouping, `joining`, sorting, date/time)
10. [OOP-flavoured Coding](#10-oop-flavoured-coding)
11. [Concurrency Coding](#11-concurrency-coding)
12. [Machine-Coding / Practical Rounds](#12-machine-coding--practical-rounds)
13. [Coding Plan & Self-Check](#13-coding-plan--self-check)

---

## 1. How to Improve Your Coding Skills

### 1.1 The core idea
Interview coding is a **pattern-recognition + communication + discipline** skill, not raw intelligence. ~60–80 well-chosen problems, solved *deliberately*, beat 400 solved carelessly.

### 1.2 The 4-Pass Method (use for every problem)
| Pass | Action | Time |
|---|---|---|
| 1. Attempt | Set a timer (Easy 15 / Medium 30). Solve **without** hints, no autocomplete for half of them. Talk aloud. | 15–30 min |
| 2. Struggle | If stuck, write the brute force. A working brute force is better than nothing. Then think "what repeated work can I remove?" | 10 min |
| 3. Learn | Read the best solution. Identify **the pattern name** and **the one insight** (e.g., "prefix-sum + HashMap"). Write it in your notes. | 10 min |
| 4. Redo | Re-solve from a blank file **after 3 days**, then **after 10 days**. If you can't, it goes into the mistakes log. | 10 min each |

### 1.3 Build skill in this order
1. **Fluency in Java basics**: `String/StringBuilder`, `char` arithmetic, `int[]`, `List/Map/Set/Deque` APIs, `Collections.sort`, `Comparator`, `Arrays.fill/sort/copyOf`. If you google an API during practice, add it to a cheat-sheet.
2. **Patterns** (Section 3): learn each pattern with 3–5 problems before moving on.
3. **Speed + accuracy**: timed sets (2 problems/45 min), then 60-min mixed mock.
4. **Communication**: record yourself solving; check you (a) clarified, (b) explained approach before coding, (c) tested with examples, (d) stated complexity.
5. **Real-world flavour** (senior): thread-safety, clean class design, null-safety, readable naming, small methods, tests.

### 1.4 Weekly practice schedule (parallel to theory)
| Day | Practice |
|---|---|
| Mon–Fri | 2 problems/day: 1 new, 1 redo from the mistakes log |
| Sat | 60–90 min "mock": 2 problems back-to-back, timed, whiteboard/plain editor |
| Sun | Review solutions, update pattern cheat-sheet, light revision |

### 1.5 Problem-count targets
| Bucket | # problems |
|---|---|
| Strings/Arrays | 15 |
| LinkedList | 8 |
| HashMap/HashSet | 12 |
| PriorityQueue/Heap | 8 |
| Stack/Queue | 6 |
| Streams (see [file 13](13_JAVA8_STREAMS_CODING.md): Top-10 / Top-25) | 10–25 |
| Concurrency + OOP | 8–10 |
| Machine-coding | 3–4 full builds |
| **Total** | **~70–80** |

### 1.6 Habits that separate senior candidates
- Write **meaningful names** (`frequencyByChar`, not `m`).
- **Validate inputs** (null/empty) before the algorithm.
- **Extract helper methods** instead of one 60-line method.
- Mention **complexity** and **trade-offs** unprompted.
- **Dry-run** your code on a small example before saying "done".
- Know the **Java standard library** (use `Deque` not `Stack`, `computeIfAbsent`, `Map.merge`, `Collectors.groupingBy`).
- **Don't silently switch** approaches; say "my first idea is X, but it's O(n²); let me try Y".

### 1.7 Common mistakes (check yourself)
| Mistake | Fix |
|---|---|
| Jumping into code in 20 seconds | Spend first 3–5 min on clarifying + approach |
| Ignoring edge cases | Checklist: null, empty, single element, duplicates, negatives, overflow, very large input |
| `Integer` compared with `==` | Use `.equals()` / unbox |
| `a - b` comparator overflow | `Integer.compare(a, b)` |
| Mutating a collection while iterating | Use `Iterator.remove()` / `removeIf` / copy |
| Off-by-one in loops/binary search | Dry-run with 1- and 2-element inputs |
| Using `Stack`/`Vector` | Use `ArrayDeque` |
| Forgetting to state complexity | Always say it |

### 1.8 Tools
- IDE-free practice: plain text editor / online compiler once a week (interviews often use CoderPad/HackerRank).
- Keep a **template file** (Section 5–7 templates) — retype it weekly until automatic.

---

## 2. In-Interview Approach (Framework)

Use **C-E-A-C-T-T**:

1. **C**larify — input size, null/empty, duplicates, sorted?, negative values, memory limits, expected return on invalid input.
2. **E**xamples — draw 1 normal + 1 edge example, confirm expected output with the interviewer.
3. **A**pproach — state brute force + complexity; then the optimized idea & the pattern name; get a nod.
4. **C**ode — clean, named, helper methods; narrate key decisions.
5. **T**est — dry-run with your example + 1–2 edge cases; fix bugs calmly.
6. **T**alk complexity & follow-ups — time/space; what if the data doesn't fit in memory / multiple threads / streaming input?

**If stuck:** (a) restate the problem, (b) solve a smaller case by hand, (c) ask "can I sort / hash / use two pointers / use a heap?", (d) offer the brute force, (e) ask for a hint *after* showing effort.

---

## 3. Pattern Cheat-Sheet

| Signal in the problem | Pattern | Typical structure |
|---|---|---|
| "Find pair/complement", "seen before?", "count occurrences" | **HashMap / HashSet** | one pass, store what you've seen |
| "Sorted array", "pair with sum", "remove duplicates in place" | **Two pointers** | `l`, `r` from ends or `slow`, `fast` |
| "Longest/shortest substring/subarray with condition" | **Sliding window** | expand `r`, shrink `l` while invalid |
| "Subarray sum = K" with negatives | **Prefix sum + HashMap** | map(prefixSum → count) |
| "Top K", "Kth largest", "merge K sorted", "stream median" | **Heap / PriorityQueue** | size-K heap |
| "Cycle", "middle", "Nth from end" in list | **Fast/slow pointers** | `slow=slow.next; fast=fast.next.next` |
| "Reverse / merge / partition list" | **Dummy node + pointer rewiring** | `dummy.next = head` |
| "Matching brackets", "next greater", "undo" | **Stack** | push/pop; monotonic stack |
| "Level by level", "shortest path (unweighted)" | **Queue / BFS** | `ArrayDeque` |
| "All combinations/permutations" | **Backtracking** | choose → recurse → un-choose |
| "Sorted + find", "min feasible value" | **Binary search** | `lo + (hi-lo)/2` |
| "Overlapping ranges" | **Sort + merge** | sort by start |
| "Optimal count/ways with overlapping subproblems" | **DP (basic: fib, climbing stairs, coin change)** | memo/table |

---

## 4. Strings & Arrays

### Strings
- 🟢 Reverse a string / reverse words in a sentence (without `StringBuilder.reverse()`)
- 🟢 Palindrome (ignore case & non-alphanumerics)
- 🟢 Two strings are anagrams (int[26] or HashMap)
- 🟢 Character frequency; first non-repeating character
- 🟢 Remove duplicate characters preserving order (`LinkedHashSet`)
- 🟡 Longest substring without repeating characters (**sliding window**)
- 🟡 Valid parentheses `()[]{}` (**Stack**)
- 🟡 String compression (`aaabb` → `a3b2`)
- 🟡 Implement `atoi` with validation
- 🟡 Longest common prefix
- 🔴 Group anagrams (**HashMap** with sorted key)

### Arrays & Numbers
- 🟢 Largest / second-largest; reverse in place; rotate by K
- 🟢 Missing number in 1..N (sum or XOR); find duplicate
- 🟢 Fibonacci (iterative/recursive/memo), factorial, prime check, GCD, FizzBuzz
- 🟡 Two Sum (HashMap), Three Sum (sort + two pointers)
- 🟡 Merge two sorted arrays; remove duplicates from sorted array
- 🟡 Move zeroes; Kadane's max subarray
- 🟡 Binary search (and first/last occurrence)
- 🟡 Intersection/union of two arrays
- 🔴 Merge overlapping intervals
- 🔴 Trapping rain water *(optional; product companies)*

#### Template — Sliding window (longest substring without repeats)
```java
public int lengthOfLongestSubstring(String s) {
    if (s == null || s.isEmpty()) return 0;
    Map<Character, Integer> lastSeen = new HashMap<>();
    int best = 0, left = 0;
    for (int right = 0; right < s.length(); right++) {
        char c = s.charAt(right);
        if (lastSeen.containsKey(c) && lastSeen.get(c) >= left) {
            left = lastSeen.get(c) + 1;           // shrink past previous occurrence
        }
        lastSeen.put(c, right);
        best = Math.max(best, right - left + 1);
    }
    return best;                                   // O(n) time, O(min(n, charset)) space
}
```

#### Template — Two pointers (sorted array, pair with target sum)
```java
int l = 0, r = a.length - 1;
while (l < r) {
    int sum = a[l] + a[r];
    if (sum == target) return new int[]{l, r};
    if (sum < target) l++; else r--;
}
return new int[]{-1, -1};
```

---

## 5. LinkedList

### 5.1 How to approach linked-list problems
1. **Draw it.** Boxes and arrows, label `prev/curr/next`. Never code a pointer problem in your head.
2. **Know the 5 tools**:
   - **Dummy (sentinel) node** — removes head special cases (merge, delete, partition, add numbers).
   - **Two pointers (fast/slow)** — middle, cycle, Nth from end (gap of N).
   - **In-place reversal** — save `next` *before* rewiring.
   - **Recursion** — elegant but O(n) stack; mention it.
   - **HashMap** — when pointers aren't enough (copy random list, LRU).
3. **Always check**: `head == null`, single node, two nodes, `k` larger than length, even vs odd length.
4. **Order of pointer operations matters**: save → rewire → advance.
5. **Complexities:** typical O(n) time, O(1) space for pointer solutions.

### 5.2 Node definition
```java
class ListNode {
    int val;
    ListNode next;
    ListNode(int val) { this.val = val; }
}
```

### 5.3 Templates
**Reverse (iterative)**
```java
public ListNode reverse(ListNode head) {
    ListNode prev = null, curr = head;
    while (curr != null) {
        ListNode next = curr.next;   // 1. save
        curr.next = prev;            // 2. rewire
        prev = curr;                 // 3. advance
        curr = next;
    }
    return prev;
}
```

**Middle / cycle detection (fast–slow)**
```java
public ListNode middle(ListNode head) {            // for even length returns 2nd middle
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) { slow = slow.next; fast = fast.next.next; }
    return slow;
}

public boolean hasCycle(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next; fast = fast.next.next;
        if (slow == fast) return true;
    }
    return false;
}
```

**Cycle start (Floyd phase 2)** — after `slow == fast`, reset one pointer to `head`, move both one step at a time; they meet at the cycle start.

**Merge two sorted lists (dummy node)**
```java
public ListNode merge(ListNode a, ListNode b) {
    ListNode dummy = new ListNode(0), tail = dummy;
    while (a != null && b != null) {
        if (a.val <= b.val) { tail.next = a; a = a.next; }
        else                { tail.next = b; b = b.next; }
        tail = tail.next;
    }
    tail.next = (a != null) ? a : b;
    return dummy.next;
}
```

**Remove Nth node from end (gap of N, dummy)**
```java
public ListNode removeNthFromEnd(ListNode head, int n) {
    ListNode dummy = new ListNode(0); dummy.next = head;
    ListNode fast = dummy, slow = dummy;
    for (int i = 0; i < n; i++) fast = fast.next;     // create gap n
    while (fast.next != null) { fast = fast.next; slow = slow.next; }
    slow.next = slow.next.next;
    return dummy.next;
}
```

**Palindrome list** — find middle → reverse second half → compare → (optionally restore).

### 5.4 Questions
- 🟢 Reverse a linked list (iterative **and** recursive)
- 🟢 Find middle node
- 🟢 Merge two sorted lists
- 🟢 Delete a node given only that node (copy next, skip)
- 🟢 Remove duplicates from sorted list
- 🟡 Detect cycle; find cycle start; find cycle length
- 🟡 Remove Nth node from end
- 🟡 Palindrome linked list
- 🟡 Intersection of two linked lists (length difference or two-pointer swap trick)
- 🟡 Add two numbers represented as lists
- 🟡 Reverse nodes in a sub-range (m to n)
- 🟡 Rotate list by K
- 🟡 Reorder list (L0→Ln→L1→Ln-1…): middle + reverse + interleave
- 🔴 Reverse nodes in K-group
- 🔴 Copy list with random pointer (HashMap or interleaving)
- 🔴 Merge K sorted lists (→ **PriorityQueue**, Section 7)
- 🔴 **LRU Cache** with HashMap + doubly linked list (see below)
- 🔴 Sort a linked list (merge sort, O(n log n), O(1) extra space for bottom-up)

### 5.5 Java `LinkedList` API trivia (often asked verbally)
- `LinkedList` implements `List` **and** `Deque`; doubly linked.
- `get(i)` is **O(n)**; add/remove at ends O(1); add/remove in middle O(1) *only after* you hold the node (iterator) — finding it is O(n).
- In practice `ArrayDeque` beats `LinkedList` for stack/queue; `ArrayList` beats it for most lists (cache locality).

### 5.6 LRU Cache — two implementations
**A. Quick (LinkedHashMap, access-order)**
```java
class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private final int capacity;
    LRUCache(int capacity) { super(16, 0.75f, true); this.capacity = capacity; }
    @Override protected boolean removeEldestEntry(Map.Entry<K, V> eldest) { return size() > capacity; }
}
```
**B. Manual (HashMap + doubly linked list)** — expected in senior rounds
```java
class LRUCache {
    private static class Node { int key, val; Node prev, next; Node(int k, int v){key=k;val=v;} }
    private final int capacity;
    private final Map<Integer, Node> map = new HashMap<>();
    private final Node head = new Node(0, 0), tail = new Node(0, 0);   // sentinels

    LRUCache(int capacity) { this.capacity = capacity; head.next = tail; tail.prev = head; }

    public int get(int key) {
        Node n = map.get(key);
        if (n == null) return -1;
        remove(n); addFirst(n);                 // mark most-recently-used
        return n.val;
    }
    public void put(int key, int value) {
        Node n = map.get(key);
        if (n != null) { n.val = value; remove(n); addFirst(n); return; }
        if (map.size() == capacity) {           // evict LRU = node before tail
            Node lru = tail.prev; remove(lru); map.remove(lru.key);
        }
        n = new Node(key, value); map.put(key, n); addFirst(n);
    }
    private void remove(Node n)   { n.prev.next = n.next; n.next.prev = n.prev; }
    private void addFirst(Node n) { n.next = head.next; n.prev = head; head.next.prev = n; head.next = n; }
}
```
Follow-ups: thread-safety (`synchronized`/`ReentrantLock`/segmenting), TTL expiry, LFU, distributed cache.

---

## 6. HashMap / HashSet

### 6.1 How to approach
**Ask yourself:** *"Am I re-scanning data to find/count/compare something I could have stored?"* If yes → hash it, trading O(n) space for O(1) lookup.

Recognition signals:
| Signal | Structure |
|---|---|
| "Does X exist / seen before / duplicate?" | `HashSet` |
| "Count occurrences / frequency / top-K by count" | `HashMap<T,Integer>` |
| "Find complement / pair / index of …" | `HashMap<value,index>` |
| "Group by …" (anagrams, by key) | `HashMap<key,List<…>>` |
| "Subarray with sum K" | prefix-sum → `HashMap<sum,count>` |
| "Longest consecutive / sequence without sorting" | `HashSet` + start-of-sequence check |
| "Cache / memoization / LRU" | `HashMap` (+ linked list) |

**Checklist:** what's the key? what's the value? what to store *before* vs *after* checking (order matters in Two Sum)? are keys mutable (never!)? do I need insertion order (`LinkedHashMap`) or sorted order (`TreeMap`)?

**Java API fluency (use these, interviewers notice):**
```java
map.merge(key, 1, Integer::sum);                       // count
map.getOrDefault(key, 0);
map.computeIfAbsent(key, k -> new ArrayList<>()).add(x);   // group-by
map.putIfAbsent(key, v);
for (Map.Entry<K,V> e : map.entrySet()) { ... }
set.add(x)  // returns false if already present → dedupe/detect duplicate in one call
```

### 6.2 Templates
**Two Sum (store after check)**
```java
public int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> idx = new HashMap<>();
    for (int i = 0; i < nums.length; i++) {
        Integer j = idx.get(target - nums[i]);
        if (j != null) return new int[]{j, i};
        idx.put(nums[i], i);
    }
    return new int[]{-1, -1};
}
```

**Subarray sum equals K (prefix sum)**
```java
public int subarraySum(int[] nums, int k) {
    Map<Integer, Integer> freq = new HashMap<>();
    freq.put(0, 1);                       // empty prefix
    int sum = 0, count = 0;
    for (int x : nums) {
        sum += x;
        count += freq.getOrDefault(sum - k, 0);
        freq.merge(sum, 1, Integer::sum);
    }
    return count;
}
```

**Group anagrams**
```java
public List<List<String>> groupAnagrams(String[] strs) {
    Map<String, List<String>> groups = new HashMap<>();
    for (String s : strs) {
        char[] c = s.toCharArray();
        Arrays.sort(c);
        groups.computeIfAbsent(new String(c), k -> new ArrayList<>()).add(s);
    }
    return new ArrayList<>(groups.values());
}
```

**Longest consecutive sequence (O(n))**
```java
public int longestConsecutive(int[] nums) {
    Set<Integer> set = new HashSet<>();
    for (int n : nums) set.add(n);
    int best = 0;
    for (int n : set) {
        if (!set.contains(n - 1)) {                // only start at sequence beginnings
            int cur = n, len = 1;
            while (set.contains(cur + 1)) { cur++; len++; }
            best = Math.max(best, len);
        }
    }
    return best;
}
```

### 6.3 Questions
**HashSet**
- 🟢 Contains duplicate; first duplicate in array
- 🟢 Intersection / union / difference of two arrays
- 🟢 Remove duplicates preserving order (`LinkedHashSet`)
- 🟢 Happy number (cycle detection with set)
- 🟡 Longest consecutive sequence
- 🟡 Check if two strings are isomorphic; word pattern
- 🟡 Valid Sudoku (sets per row/col/box)

**HashMap**
- 🟢 Two Sum; character frequency; first non-repeating char
- 🟢 Majority element (Boyer–Moore or map)
- 🟢 Ransom note / valid anagram
- 🟡 Group anagrams
- 🟡 Top-K frequent elements (map + heap/bucket)
- 🟡 Subarray sum equals K; contiguous array (0/1 equal counts)
- 🟡 Four Sum II (map of pair sums)
- 🟡 Sort a map by value; find the key with max value
- 🟡 Find all duplicates; find missing numbers
- 🔴 LRU / LFU cache
- 🔴 Implement your own `HashMap` (array of buckets, chaining, resize at 0.75, `hashCode` → index, null-key)
- 🔴 Design a data structure with insert/delete/getRandom in O(1) (HashMap + ArrayList, swap-with-last on delete)
- 🔴 Time-based key-value store (map of key → sorted list, binary search)

### 6.4 Verbal HashMap/HashSet traps
- `HashSet` is backed by a `HashMap` (value = dummy `PRESENT`).
- Mutable keys break lookups (hash changes after insertion).
- `equals`/`hashCode` contract; `hashCode` collisions degrade to O(log n) via tree bins (Java 8, bin ≥ 8 and table ≥ 64) or O(n) for non-comparable keys in older versions.
- Iteration order of `HashMap` is unspecified; use `LinkedHashMap`/`TreeMap` when order matters.
- `HashMap` is not thread-safe (infinite loops in old resizes, lost updates) → `ConcurrentHashMap`.
- Pre-size: `new HashMap<>((int)(n / 0.75f) + 1)` to avoid resizes.

---

## 7. PriorityQueue / Heap

### 7.1 How to approach
- A `PriorityQueue` is a **binary heap**: `offer/poll` O(log n), `peek` O(1), `remove(Object)` O(n), build from collection O(n). Java default = **min-heap**.
- **Recognition signals:** "K largest/smallest", "Kth …", "merge K sorted …", "running median", "schedule by earliest/ smallest", "always process the best next".
- **Golden trick — size-K heap:**
  - **K largest** → keep a **min-heap of size K** (evict the smallest). Answer is the heap's contents; `peek()` = Kth largest.
  - **K smallest** → keep a **max-heap of size K**.
  - Time O(n log K), space O(K) — better than sorting O(n log n) when K ≪ n.
- **Comparator care:** use `Integer.compare(a, b)` (never `a - b`; overflow). Max-heap: `Collections.reverseOrder()` or `(a, b) -> Integer.compare(b, a)`.
- Store `int[]{value, index}` or a small record when you need extra data.
- Iterating a `PriorityQueue` is **not sorted** — only `poll()` gives order.

### 7.2 Templates
**Kth largest element**
```java
public int findKthLargest(int[] nums, int k) {
    PriorityQueue<Integer> minHeap = new PriorityQueue<>();
    for (int n : nums) {
        minHeap.offer(n);
        if (minHeap.size() > k) minHeap.poll();   // drop smallest
    }
    return minHeap.peek();
}
```

**Top-K frequent elements**
```java
public int[] topKFrequent(int[] nums, int k) {
    Map<Integer, Integer> freq = new HashMap<>();
    for (int n : nums) freq.merge(n, 1, Integer::sum);

    PriorityQueue<Map.Entry<Integer, Integer>> heap =
        new PriorityQueue<>(Comparator.comparingInt(Map.Entry::getValue));   // min-heap by count
    for (Map.Entry<Integer, Integer> e : freq.entrySet()) {
        heap.offer(e);
        if (heap.size() > k) heap.poll();
    }
    int[] res = new int[k];
    for (int i = k - 1; i >= 0; i--) res[i] = heap.poll().getKey();
    return res;
}
```

**Merge K sorted lists**
```java
public ListNode mergeKLists(ListNode[] lists) {
    PriorityQueue<ListNode> pq = new PriorityQueue<>(Comparator.comparingInt(n -> n.val));
    for (ListNode l : lists) if (l != null) pq.offer(l);
    ListNode dummy = new ListNode(0), tail = dummy;
    while (!pq.isEmpty()) {
        ListNode n = pq.poll();
        tail.next = n; tail = n;
        if (n.next != null) pq.offer(n.next);
    }
    return dummy.next;                           // O(N log K)
}
```

**Running median (two heaps)**
```java
class MedianFinder {
    private final PriorityQueue<Integer> low  = new PriorityQueue<>(Collections.reverseOrder()); // max-heap (lower half)
    private final PriorityQueue<Integer> high = new PriorityQueue<>();                           // min-heap (upper half)

    public void addNum(int num) {
        low.offer(num);
        high.offer(low.poll());                      // balance order
        if (high.size() > low.size()) low.offer(high.poll());   // keep low >= high in size
    }
    public double findMedian() {
        return low.size() > high.size() ? low.peek() : (low.peek() + high.peek()) / 2.0;
    }
}
```

**Meeting rooms II (min rooms)** — sort by start; min-heap of end times; if `heap.peek() <= start` reuse (poll), always offer new end; answer = max heap size.

### 7.3 Questions
- 🟢 Kth largest / smallest element in array
- 🟢 Last stone weight (max-heap simulation)
- 🟡 Top-K frequent elements / words
- 🟡 K closest points to origin
- 🟡 Sort characters by frequency
- 🟡 Meeting rooms II
- 🟡 Task scheduler / reorganize string (max-heap by count)
- 🟡 Connect ropes to minimize cost (Huffman-style)
- 🟡 Kth largest element in a stream (class with `add`)
- 🟡 Find K pairs with smallest sums
- 🔴 Merge K sorted lists / arrays
- 🔴 Find median from data stream
- 🔴 Smallest range covering elements from K lists
- 🔴 Implement your own min-heap (`siftUp`/`siftDown`, array-backed)
- 🔴 Top-K trending with sliding window / expiry (heap + lazy deletion)

### 7.4 Verbal PriorityQueue questions
- Internal structure? (array-backed binary heap; parent `(i-1)/2`, children `2i+1`, `2i+2`)
- Why is `PriorityQueue.remove(obj)` O(n)? Is it thread-safe? (No → `PriorityBlockingQueue`)
- Null elements? (not allowed) · Ordering of equal elements? (not stable)
- Why `Comparator` with subtraction is dangerous? (overflow)

---

## 8. Stack / Queue / Deque
- 🟢 Valid parentheses; implement queue using two stacks; stack using two queues
- 🟡 Min Stack (O(1) `getMin`: store min alongside, or push `(val, currentMin)`)
- 🟡 Next greater element (monotonic stack); daily temperatures
- 🟡 Evaluate reverse Polish notation; decode string
- 🟡 Sliding window maximum (monotonic `Deque`)
- 🔴 Largest rectangle in histogram *(optional)*

```java
// Monotonic stack: next greater element
int[] res = new int[n]; Arrays.fill(res, -1);
Deque<Integer> st = new ArrayDeque<>();           // stores indices
for (int i = 0; i < n; i++) {
    while (!st.isEmpty() && a[st.peek()] < a[i]) res[st.pop()] = a[i];
    st.push(i);
}
```

---

## 9. Java 8+ Streams → moved to its own file

**Go to [13_JAVA8_STREAMS_CODING.md](13_JAVA8_STREAMS_CODING.md)** — sorting, single-level and **two-level grouping**, `Collectors.joining`, strings/characters, `flatMap`, maps, **date/time with streams**, advanced collectors, pitfalls, verbal questions and a Top-10 / Top-25 practice plan.

**Top-10 to start with (≈ 3 h):** count per dept · avg salary per dept · highest paid per dept · 2nd highest salary · multi-field sort · dept → joined names · **dept → city → names (2-level grouping)** · char frequency + first non-repeating char · word frequency + top 3 · group by joining year.

---

## 10. OOP-Flavoured Coding
- 🟢 Immutable class (final class/fields, no setters, defensive copies)
- 🟢 `equals`/`hashCode`/`toString` correctly
- 🟡 Thread-safe Singleton (enum / Bill Pugh / double-checked locking with `volatile`)
- 🟡 Builder for a class with 6+ fields
- 🟡 Custom checked & unchecked exception in a service
- 🟡 `Comparable` vs `Comparator` for a custom class
- 🔴 Mini DI container (annotations + reflection)
- 🔴 Custom `ArrayList<T>` with resize + `Iterator`

---

## 11. Concurrency Coding
- 🟡 Print odd/even with two threads (`wait/notify`, then `Semaphore`)
- 🟡 Print `1 A 2 B 3 C…` alternately
- 🟡 Create and then fix a deadlock (lock ordering)
- 🟡 `ExecutorService` + `Future` / `CompletableFuture` for N tasks
- 🔴 **Producer–Consumer** with `BlockingQueue` and with `wait/notify`
- 🔴 Bounded blocking queue (lock + 2 conditions)
- 🔴 Thread-safe rate limiter (token bucket / sliding window)
- 🔴 Simple custom thread pool
- 🔴 `CompletableFuture`: call 3 services in parallel, combine, timeout + fallback

```java
// Producer-Consumer with BlockingQueue
BlockingQueue<Integer> q = new ArrayBlockingQueue<>(5);
Thread producer = new Thread(() -> {
    try { for (int i = 0; i < 20; i++) q.put(i); q.put(-1); }          // -1 = poison pill
    catch (InterruptedException e) { Thread.currentThread().interrupt(); }
});
Thread consumer = new Thread(() -> {
    try { for (int v; (v = q.take()) != -1; ) System.out.println("consumed " + v); }
    catch (InterruptedException e) { Thread.currentThread().interrupt(); }
});
producer.start(); consumer.start();
```

---

## 12. Machine-Coding / Practical Rounds (60–90 min)
- 🟡 Parse CSV/log file → aggregates (streams, I/O, exceptions)
- 🟡 REST CRUD API (Spring Boot) with validation + exception handling + tests
- 🟡 Retry with exponential backoff + jitter
- 🔴 In-memory key-value store with TTL
- 🔴 In-memory Pub-Sub / event bus
- 🔴 URL shortener (service + storage + collision handling)
- 🔴 Rate limiter, parking lot, splitwise (see [10_LLD_DESIGN_PATTERNS.md](10_LLD_DESIGN_PATTERNS.md))

**Approach for machine-coding:** (1) clarify scope & write requirements as bullets, (2) model entities/interfaces, (3) get the **happy path working first**, (4) then validation/exceptions, (5) then thread-safety, (6) add 3–5 focused unit tests, (7) leave 5 min to discuss extensions. Prefer readability & working code over patterns for patterns' sake.

---

## 13. Coding Plan & Self-Check

Coding hours match the [README](README.md) budget: **20 h (1-month)** / **30 h (2-month)** = this file (**17 h / 24 h**) + Streams in [13_JAVA8_STREAMS_CODING.md](13_JAVA8_STREAMS_CODING.md) (**3 h / 6 h**). Always start with the ⭐ starter set above, then follow the plan for your timeline.

**1-Month plan (20 h ≈ 35–40 problems)**
| Week | Hours | Focus | Problems |
|---|---|---|---|
| 1 | 6 | Strings, Arrays, LinkedList basics, HashMap basics | ~10 |
| 2 | 5 | HashMap/HashSet, PriorityQueue, Stack/Queue | ~10 |
| 3 | 5 | **Streams Top-10 ([file 13](13_JAVA8_STREAMS_CODING.md) §15, ≈3 h)**, then concurrency basics + 1 OOP-flavoured item (≈2 h) | ~12 (10 stream + 2) |
| 4 | 4 | Mixed timed sets (2 problems / 45 min) + redo mistakes-log; re-code LRU, producer-consumer, reverse list, top-K, group anagrams from memory | ~8 |

**2-Month plan (30 h ≈ 60–70 problems)**
| Week | Hours | Focus |
|---|---|---|
| 1 | 4 | Strings + Arrays (≈10 problems) |
| 2 | 4 | LinkedList (≈8) incl. LRU |
| 3 | 4 | HashMap/HashSet (≈10) |
| 4 | 4 | PriorityQueue (≈8) + Stack/Queue (≈5) |
| 5 | 4 | **Streams Top-10 + variants (≈3 h, [file 13](13_JAVA8_STREAMS_CODING.md) §15)** + OOP-flavoured (≈4 items, ≈1 h) |
| 6 | 4 | Concurrency (≈6) |
| 7 | 3 | Machine-coding builds (2) + timed sets incl. **Streams Top-25 items 11–18** |
| 8 | 3 | Mixed timed sets incl. **Streams Top-25 items 19–25** + redo mistakes-log + final from-memory rewrite of the starter set |

**Self-check — you're ready when you can:**
- [ ] Solve a 🟢 in ≤10 min and a 🟡 in ≤25 min, talking aloud, in a plain editor
- [ ] Write reverse-list, merge-lists, cycle detection, LRU, top-K (heap), Two Sum, group-anagrams, sliding window **from memory**
- [ ] State time/space complexity & 2 edge cases for every solution
- [ ] Explain why you chose HashMap vs TreeMap vs PriorityQueue for a problem
