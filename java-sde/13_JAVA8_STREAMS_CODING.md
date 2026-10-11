# 13 — Java 8+ Streams & Functional Coding (High-Probability Round)

> Companion to [01_CODING.md](01_CODING.md). Streams questions come up in **almost every** Java 8+ interview — usually on an `Employee`/`Order`/`String`/`List<Integer>` dataset. Learn the patterns once and most questions become routine.
> Legend: 🟢 basic · 🟡 intermediate · 🔴 senior-depth

## ⭐ Fundamentals First (start here)
**Time:** 1-month plan **3 h** · 2-month plan **6 h** (inside the coding budget — see [README](README.md)).

**Learn in this order**
1. **Approach + collectors toolbox** (§1) — 20 min. Always state the **result type first** (e.g., `Map<String, Map<String, List<String>>>`); it tells you how many `groupingBy` levels you need.
2. **Dataset** (§2), then **grouping** (§5) → **two-level grouping** (§6) → **`joining`** (§7) → **sorting** (§4).
3. **Date/time with streams** (§11): `LocalDate` filters, group by year/`YearMonth`, `ChronoUnit` tenure.
4. **Pitfalls** (§13): `toMap` duplicate keys, null keys, stream reuse, `HashMap` ordering.

**Do the Top-10 first** (§15). 1-month plan = Top-10; 2-month plan = full Top-25.
**Done when:** you write a 2-level `groupingBy`, `mapping`+`joining` per group, `Optional`-safe `maxBy` per group, a `LinkedHashMap`-sorted value map and a `YearMonth` report — each in ≤10 min without autocomplete.
**Skip if short on time:** §12 advanced collectors (`teeing`, `reducing`), custom `Collector`, parallel-stream details.

---



**Sections:** [1. Approach & Collectors Toolbox](#1-approach--collectors-toolbox) · [2. Sample Dataset (used in all examples)](#2-sample-dataset-used-in-all-examples) · [3. Basics & Numbers](#3-basics--numbers) · [4. Sorting](#4-sorting) · [5. Grouping (Single Level)](#5-grouping-single-level) · [6. Two-Level / Nested Grouping (`groupingBy` inside `groupingBy`)](#6-two-level--nested-grouping-groupingby-inside-groupingby) · [7. `Collectors.joining`](#7-collectorsjoining) · [8. Strings & Characters](#8-strings--characters) · [9. `flatMap` — Collections of Collections](#9-flatmap--collections-of-collections) · [10. Working with Maps](#10-working-with-maps) · [11. Date & Time (`java.time`) with Streams](#11-date--time-javatime-with-streams) · [12. Advanced Collectors](#12-advanced-collectors) · [13. Pitfalls & Gotchas Interviewers Love](#13-pitfalls--gotchas-interviewers-love) · [14. Verbal Questions](#14-verbal-questions) · [15. Practice Plan](#15-practice-plan)

## 1. Approach & Collectors Toolbox
**4-step thinking:** (1) What is the **source**? (list / array / string / map entries) → (2) Need to **filter** or **transform** first? → (3) Is the result a **single value** (reduce/max/count), a **list**, or a **Map** (group/partition/toMap)? → (4) Need **ordering** (sort before collect, or `LinkedHashMap`/`TreeMap` as the target)?

| I need… | Use |
|---|---|
| a count / sum / average | `count()`, `mapToInt/Double(..).sum()/average()`, `summaryStatistics()` |
| max / min by a field | `max(Comparator.comparing…)` → returns `Optional` |
| group by key | `groupingBy(classifier)` (+ downstream collector) |
| count / sum / avg per group | `groupingBy(k, counting() / summingX() / averagingX())` |
| a different field per group | `groupingBy(k, mapping(field, toList()/joining()/toSet()))` |
| best element per group | `groupingBy(k, maxBy(cmp))` or `collectingAndThen(maxBy(cmp), Optional::get)` |
| group inside a group | `groupingBy(k1, groupingBy(k2, …))` |
| sorted/ordered result map | `groupingBy(k, TreeMap::new, …)` / `toMap(…, LinkedHashMap::new)` |
| true/false split | `partitioningBy(predicate[, downstream])` |
| list → map | `toMap(keyFn, valueFn[, mergeFn[, mapSupplier]])` |
| one string from many | `Collectors.joining(delim[, prefix, suffix])` |
| flatten | `flatMap(x -> x.stream())` |
| stats in one pass | `summarizingInt/Double/Long` |
| two results in one pass | `Collectors.teeing` (Java 12) |

```java
// Typical imports for the examples below
import java.time.*; import java.time.format.DateTimeFormatter; import java.time.temporal.*;
import java.util.*; import java.util.function.Function; import java.util.stream.*;
import static java.util.stream.Collectors.*;
import static java.util.Comparator.*;
```

## 2. Sample Dataset (used in all examples)
```java
record Employee(int id, String name, String dept, String city, String gender,
                double salary, int age, LocalDate joinDate, List<String> skills) {}
record Order(long id, String customer, double amount, String status, LocalDateTime createdAt) {}

List<Employee> emps = List.of(
  new Employee(1, "Asha",  "IT",      "Pune",    "F", 90000, 34, LocalDate.of(2019, 3, 15), List.of("Java", "Spring")),
  new Employee(2, "Ravi",  "IT",      "Mumbai",  "M", 75000, 29, LocalDate.of(2021, 7, 1),  List.of("Java", "Kafka")),
  new Employee(3, "Meena", "HR",      "Pune",    "F", 55000, 41, LocalDate.of(2016, 11, 20),List.of("Recruiting")),
  new Employee(4, "John",  "Finance", "Mumbai",  "M", 80000, 38, LocalDate.of(2018, 1, 10), List.of("Excel", "SQL")),
  new Employee(5, "Sara",  "IT",      "Pune",    "F", 120000,45, LocalDate.of(2015, 6, 5),  List.of("Java", "AWS", "Kafka")),
  new Employee(6, "Imran", "Finance", "Delhi",   "M", 62000, 27, LocalDate.of(2023, 2, 28), List.of("SQL")));
```
> **Java 8 note:** records are Java 16+. In Java 8, write a POJO with getters and use `Employee::getDept`, `Employee::getSalary`, etc. `List.of`/`Map.entry`/`Stream.toList()` are also newer — use `Arrays.asList`, `new AbstractMap.SimpleEntry<>`, `collect(toList())` when told "Java 8 only".

## 3. Basics & Numbers
- 🟢 Sum of squares of even numbers · average · max/min · count · summary statistics
- 🟢 Numbers starting with digit `1` · distinct / limit / skip · check `allMatch`/`anyMatch`/`noneMatch`
- 🟢 Sum of digits · factorial · first N Fibonacci numbers · prime check / primes up to N
- 🟡 Second highest / Nth highest number · find duplicates · common elements of two lists · union (distinct+sorted)
- 🟡 Even / odd partition · sort array descending · array ↔ list · missing number in 1..N
- 🟡 Convert `List<String>` → `Map<String,Integer>` (word → length)

```java
List<Integer> nums = List.of(5, 2, 8, 2, 9, 1, 8, 10, 12);

int sumSqEven = nums.stream().filter(n -> n % 2 == 0).mapToInt(n -> n * n).sum();
double avg    = nums.stream().mapToInt(Integer::intValue).average().orElse(0);
IntSummaryStatistics st = nums.stream().mapToInt(Integer::intValue).summaryStatistics(); // min,max,avg,sum,count
Optional<Integer> max = nums.stream().max(Integer::compare);
List<Integer> startsWith1 = nums.stream().map(String::valueOf).filter(s -> s.startsWith("1"))
                               .map(Integer::valueOf).collect(toList());

// Nth highest distinct (N = 2) ----------------------------------------------
Optional<Integer> second = nums.stream().distinct().sorted(reverseOrder()).skip(1).findFirst();

// Duplicates -----------------------------------------------------------------
Set<Integer> seen = new HashSet<>();
Set<Integer> dups = nums.stream().filter(n -> !seen.add(n)).collect(toSet());   // stateful but fine sequentially
Set<Integer> dups2 = nums.stream().collect(groupingBy(Function.identity(), counting()))
        .entrySet().stream().filter(e -> e.getValue() > 1).map(Map.Entry::getKey).collect(toSet()); // cleaner/safer

// Common elements / union -----------------------------------------------------
List<Integer> a = List.of(1, 2, 3, 4), b = List.of(3, 4, 5);
Set<Integer> bSet = new HashSet<>(b);
List<Integer> common = a.stream().filter(bSet::contains).collect(toList());
List<Integer> union  = Stream.concat(a.stream(), b.stream()).distinct().sorted().collect(toList());

// Partition even/odd ---------------------------------------------------------
Map<Boolean, List<Integer>> evenOdd = nums.stream().collect(partitioningBy(n -> n % 2 == 0));

// Primes up to N / factorial / Fibonacci / digit sum ---------------------------
int num = 29;
boolean isPrime = num > 1 && IntStream.rangeClosed(2, (int) Math.sqrt(num)).noneMatch(i -> num % i == 0);
List<Integer> primes = IntStream.rangeClosed(2, 50).filter(x -> IntStream.rangeClosed(2, (int) Math.sqrt(x)).noneMatch(i -> x % i == 0)).boxed().collect(toList());
long fact = LongStream.rangeClosed(1, 10).reduce(1, (x, y) -> x * y);
List<Long> fib = Stream.iterate(new long[]{0, 1}, f -> new long[]{f[1], f[0] + f[1]}).limit(10).map(f -> f[0]).collect(toList());
int digitSum = String.valueOf(12345).chars().map(c -> c - '0').sum();

// Arrays <-> lists -----------------------------------------------------------
int[] arr = {5, 3, 9, 1};
int[] desc = Arrays.stream(arr).boxed().sorted(reverseOrder()).mapToInt(Integer::intValue).toArray();
List<Integer> asList = Arrays.stream(arr).boxed().collect(toList());

// Missing number in 1..n ------------------------------------------------------
Set<Integer> present = Arrays.stream(arr).boxed().collect(toSet());
List<Integer> missing = IntStream.rangeClosed(1, 10).filter(i -> !present.contains(i)).boxed().collect(toList());

// word -> length (merge fn guards duplicates) ----------------------------------
Map<String, Integer> lengths = words.stream().collect(toMap(Function.identity(), String::length, (x, y) -> x));
```

## 4. Sorting
- 🟢 Sort numbers asc/desc · sort strings alphabetically / by length / case-insensitively
- 🟢 Sort employees by salary (asc, desc) · by name
- 🟡 Multi-field: **dept asc → salary desc → name asc** · nulls first/last
- 🟡 Top 3 highest-paid · Nth highest salary (with employee) · lowest paid per dept
- 🟡 Sort a `Map` by value (desc) / by key · sort `Map<String, List<…>>` by list size
- 🔴 Sort by custom order (e.g., priority `HIGH, MEDIUM, LOW`) · stable multi-key sort with a `Map` index

```java
List<Employee> bySalaryAsc  = emps.stream().sorted(comparingDouble(Employee::salary)).collect(toList());
List<Employee> bySalaryDesc = emps.stream().sorted(comparingDouble(Employee::salary).reversed()).collect(toList());

// dept asc, salary desc, name asc
List<Employee> multi = emps.stream().sorted(
        comparing(Employee::dept)
            .thenComparing(Employee::salary, reverseOrder())
            .thenComparing(Employee::name)).collect(toList());

// nulls last
Comparator<Employee> byCityNullsLast = comparing(Employee::city, nullsLast(naturalOrder()));

// Top 3 earners / 2nd highest salary employee
List<Employee> top3 = emps.stream().sorted(comparingDouble(Employee::salary).reversed()).limit(3).collect(toList());
Optional<Employee> secondHighest = emps.stream().sorted(comparingDouble(Employee::salary).reversed()).skip(1).findFirst();
// 2nd highest *salary value* (distinct):
Optional<Double> secondSalary = emps.stream().map(Employee::salary).distinct().sorted(reverseOrder()).skip(1).findFirst();

// Strings: by length then alphabetical; case-insensitive
words.stream().sorted(comparingInt(String::length).thenComparing(naturalOrder())).collect(toList());
words.stream().sorted(String.CASE_INSENSITIVE_ORDER).collect(toList());

// Sort Map by value desc (keep order → LinkedHashMap)
Map<String, Long> countByDept = emps.stream().collect(groupingBy(Employee::dept, counting()));
Map<String, Long> sortedByValue = countByDept.entrySet().stream()
        .sorted(Map.Entry.<String, Long>comparingByValue().reversed())
        .collect(toMap(Map.Entry::getKey, Map.Entry::getValue, (x, y) -> x, LinkedHashMap::new));

// Sort by key
countByDept.entrySet().stream().sorted(Map.Entry.comparingByKey()).forEach(System.out::println);

// Custom order (priority) via index
List<String> order = List.of("HIGH", "MEDIUM", "LOW");
tasks.stream().sorted(comparingInt(t -> order.indexOf(t.priority()))).collect(toList());
```
**Remember:** `comparing(lambda).reversed()` may fail type inference → use method reference or `Comparator.comparing((Employee e) -> e.salary()).reversed()`. For doubles prefer `comparingDouble` (no boxing).

## 5. Grouping (Single Level)
- 🟢 Group employees by dept · count per dept · total / average salary per dept
- 🟡 Names per dept (`mapping`) · highest-paid employee per dept · youngest per dept · min/max/avg in one go (`summarizingDouble`)
- 🟡 Partition by salary > X (list / count) · group by gender · group by **age bucket** · group by first letter
- 🟡 Dept with the **highest average salary** · dept with **most employees** · depts having > 1 employee
- 🟡 `toMap`: id → name; dept → total salary (merge) · handle duplicate keys
- 🔴 **Top 2 earners per dept** · group + sort groups by size · group with `TreeMap`/`LinkedHashMap` for ordering

```java
Map<String, List<Employee>> byDept       = emps.stream().collect(groupingBy(Employee::dept));
Map<String, Long>           countByDept  = emps.stream().collect(groupingBy(Employee::dept, counting()));
Map<String, Double>         avgSalary    = emps.stream().collect(groupingBy(Employee::dept, averagingDouble(Employee::salary)));
Map<String, Double>         totalSalary  = emps.stream().collect(groupingBy(Employee::dept, summingDouble(Employee::salary)));
Map<String, List<String>>   namesByDept  = emps.stream().collect(groupingBy(Employee::dept, mapping(Employee::name, toList())));

// Highest paid per dept (Optional → unwrap)
Map<String, Employee> topPerDept = emps.stream().collect(
        groupingBy(Employee::dept, collectingAndThen(maxBy(comparingDouble(Employee::salary)), Optional::get)));

// Min/Max/Avg/Count/Sum in one pass per dept
Map<String, DoubleSummaryStatistics> stats = emps.stream().collect(groupingBy(Employee::dept, summarizingDouble(Employee::salary)));

// Ordered result: TreeMap (sorted keys) or LinkedHashMap (encounter order)
Map<String, Long> sortedKeys = emps.stream().collect(groupingBy(Employee::dept, TreeMap::new, counting()));

// Partition
Map<Boolean, List<Employee>> highLow = emps.stream().collect(partitioningBy(e -> e.salary() > 80000));
Map<Boolean, Long> highLowCount      = emps.stream().collect(partitioningBy(e -> e.salary() > 80000, counting()));

// Group by age bucket
Map<String, List<String>> byAge = emps.stream().collect(groupingBy(
        e -> e.age() < 30 ? "<30" : e.age() <= 40 ? "30-40" : ">40", TreeMap::new, mapping(Employee::name, toList())));

// Dept with highest average salary
Optional<Map.Entry<String, Double>> bestDept = avgSalary.entrySet().stream().max(Map.Entry.comparingByValue());
// Dept with most employees
String biggest = countByDept.entrySet().stream().max(Map.Entry.comparingByValue()).map(Map.Entry::getKey).orElse(null);
// Depts with more than one employee
List<String> multi = countByDept.entrySet().stream().filter(e -> e.getValue() > 1).map(Map.Entry::getKey).collect(toList());

// toMap: id -> name ; dept -> total salary (merge duplicates) ; keep order
Map<Integer, String> idToName = emps.stream().collect(toMap(Employee::id, Employee::name));
Map<String, Double> deptTotal = emps.stream().collect(toMap(Employee::dept, Employee::salary, Double::sum));
Map<String, Employee> byName  = emps.stream().collect(toMap(Employee::name, Function.identity(), (x, y) -> x, LinkedHashMap::new));

// Top 2 earners per dept
Map<String, List<Employee>> top2 = emps.stream().collect(groupingBy(Employee::dept,
        collectingAndThen(toList(), l -> l.stream().sorted(comparingDouble(Employee::salary).reversed()).limit(2).collect(toList()))));

// Java 9+: filtering / flatMapping inside groups
Map<String, Long> seniorCount = emps.stream().collect(groupingBy(Employee::dept, filtering(e -> e.age() > 35, counting())));
```

## 6. Two-Level / Nested Grouping (`groupingBy` inside `groupingBy`)
**Pattern:** `groupingBy(outerKey, groupingBy(innerKey, downstream))`. The result type nests: `Map<Outer, Map<Inner, Downstream>>`. Three levels = another `groupingBy`.

- 🟡 Dept → City → list of employees
- 🟡 Dept → Gender → count · Dept → Gender → average salary
- 🟡 Dept → City → **names** (`mapping`) · Dept → City → **joined names** (`mapping` + `joining`)
- 🟡 Dept → City → highest-paid employee
- 🔴 Dept → City → Gender → count (3 levels) · both levels **sorted** (`TreeMap`)
- 🔴 **Print** the nested map neatly · **flatten** the nested map to rows · group by a **composite key** (record)
- 🔴 Dept → Joining year → count · City → Dept → total salary (then pick the max city)

```java
// Dept -> City -> List<Employee>
Map<String, Map<String, List<Employee>>> deptCity = emps.stream()
        .collect(groupingBy(Employee::dept, groupingBy(Employee::city)));

// Dept -> Gender -> count
Map<String, Map<String, Long>> deptGender = emps.stream()
        .collect(groupingBy(Employee::dept, groupingBy(Employee::gender, counting())));

// Dept -> Gender -> average salary
Map<String, Map<String, Double>> avgByDeptGender = emps.stream()
        .collect(groupingBy(Employee::dept, groupingBy(Employee::gender, averagingDouble(Employee::salary))));

// Dept -> City -> names list
Map<String, Map<String, List<String>>> namesDeptCity = emps.stream()
        .collect(groupingBy(Employee::dept, groupingBy(Employee::city, mapping(Employee::name, toList()))));

// Dept -> City -> "Asha, Sara"   (two-level grouping + joining)
Map<String, Map<String, String>> joinedNames = emps.stream()
        .collect(groupingBy(Employee::dept, groupingBy(Employee::city, mapping(Employee::name, joining(", ")))));

// Dept -> City -> highest paid employee
Map<String, Map<String, Employee>> topDeptCity = emps.stream().collect(groupingBy(Employee::dept,
        groupingBy(Employee::city, collectingAndThen(maxBy(comparingDouble(Employee::salary)), Optional::get))));

// Three levels: Dept -> City -> Gender -> count, with sorted keys at every level
Map<String, Map<String, Map<String, Long>>> three = emps.stream().collect(
        groupingBy(Employee::dept, TreeMap::new,
            groupingBy(Employee::city, TreeMap::new,
                groupingBy(Employee::gender, TreeMap::new, counting()))));

// Dept -> joining year -> count
Map<String, Map<Integer, Long>> deptYear = emps.stream().collect(
        groupingBy(Employee::dept, groupingBy(e -> e.joinDate().getYear(), TreeMap::new, counting())));

// Composite key alternative (single level, easy to read/sort/stream later)
record DeptCity(String dept, String city) {}
Map<DeptCity, Long> byComposite = emps.stream().collect(groupingBy(e -> new DeptCity(e.dept(), e.city()), counting()));

// Print nested map
deptCity.forEach((dept, cityMap) -> {
    System.out.println(dept);
    cityMap.forEach((city, list) ->
        System.out.println("  " + city + " -> " + list.stream().map(Employee::name).collect(joining(", "))));
});

// Flatten nested map into rows "Dept | City | Count"
List<String> rows = deptCity.entrySet().stream()
        .flatMap(d -> d.getValue().entrySet().stream()
                .map(c -> d.getKey() + " | " + c.getKey() + " | " + c.getValue().size()))
        .collect(toList());

// City with the highest total salary: group by city -> sum, then take the max entry
Map<String, Double> cityTotals = emps.stream().collect(groupingBy(Employee::city, summingDouble(Employee::salary)));
Optional<Map.Entry<String, Double>> richestCity = cityTotals.entrySet().stream().max(Map.Entry.comparingByValue());
```
**How to answer fast:** write the *innermost* collector first (what do I want at the leaf: list / count / names / best?), then wrap it with `groupingBy` once per level, **from innermost to outermost**.

## 7. `Collectors.joining`
- 🟢 Names as comma-separated string · with prefix/suffix `[a, b, c]` · join numbers with `-`
- 🟡 **Dept → "name1, name2"** (`groupingBy` + `mapping` + `joining`) · unique sorted skills as `A|B|C`
- 🟡 First letters of each word · capitalize each word · reverse each word · join in reverse order
- 🟡 Print `Dept: Names` lines · build a CSV from objects · join only non-empty/non-null values
- 🔴 Dept → City → joined names (see §6) · join top-3 names by salary per dept

```java
String names   = emps.stream().map(Employee::name).collect(joining(", "));                  // Asha, Ravi, ...
String wrapped = emps.stream().map(Employee::name).collect(joining(", ", "[", "]"));        // [Asha, Ravi, ...]
String numbers = nums.stream().map(String::valueOf).collect(joining("-"));                  // 5-2-8...

// Dept -> joined names
Map<String, String> namesByDept = emps.stream().collect(groupingBy(Employee::dept, TreeMap::new, mapping(Employee::name, joining(", "))));
namesByDept.forEach((d, n) -> System.out.println(d + ": " + n));

// Unique sorted skills
String skills = emps.stream().flatMap(e -> e.skills().stream()).distinct().sorted().collect(joining("|"));

// Initials / capitalize / reverse each word / reverse word order
String sentence = "java streams are fun";
String initials = Arrays.stream(sentence.split("\\s+")).map(w -> w.substring(0, 1).toUpperCase()).collect(joining());          // JSAF
String capital  = Arrays.stream(sentence.split("\\s+")).map(w -> Character.toUpperCase(w.charAt(0)) + w.substring(1)).collect(joining(" "));
String revEach  = Arrays.stream(sentence.split("\\s+")).map(w -> new StringBuilder(w).reverse()).collect(joining(" "));          // StringBuilder is a CharSequence
List<String> parts = Arrays.asList(sentence.split("\\s+")); Collections.reverse(parts); String revOrder = String.join(" ", parts);

// CSV (skip nulls/blank)
String csv = Stream.of("a", null, "", "b").filter(s -> s != null && !s.isBlank()).collect(joining(","));   // a,b   (isBlank = Java 11)

// Top 3 names per dept, joined
Map<String, String> top3Names = emps.stream().collect(groupingBy(Employee::dept, collectingAndThen(toList(),
        l -> l.stream().sorted(comparingDouble(Employee::salary).reversed()).limit(3).map(Employee::name).collect(joining(", ")))));
```
**Note:** `Collectors.joining` needs a `Stream<CharSequence>` — `map(String::valueOf)` for non-strings. `String.join(delim, iterable)` is simpler when you already have a collection of strings.

## 8. Strings & Characters
- 🟢 Char frequency · first non-repeating · first repeating · duplicate chars · vowel/consonant count
- 🟢 Uppercase / lowercase all · filter strings starting with a letter · longest / shortest word · words by length (`groupingBy(length)`)
- 🟡 Palindrome · anagram check · reverse string · remove duplicate chars (preserve order) · sort chars
- 🟡 **Word frequency** · **top-3 frequent words** · most frequent char · count of each word length
- 🔴 Group anagrams · longest word per first letter · case-insensitive word count with punctuation removed

```java
String s = "programming";
// Char frequency in insertion order
Map<Character, Long> freq = s.chars().mapToObj(c -> (char) c)
        .collect(groupingBy(Function.identity(), LinkedHashMap::new, counting()));
Character firstNonRepeating = freq.entrySet().stream().filter(e -> e.getValue() == 1).map(Map.Entry::getKey).findFirst().orElse(null);
Character firstRepeating    = freq.entrySet().stream().filter(e -> e.getValue() > 1).map(Map.Entry::getKey).findFirst().orElse(null);
List<Character> duplicates  = freq.entrySet().stream().filter(e -> e.getValue() > 1).map(Map.Entry::getKey).collect(toList());
Character mostFrequent      = freq.entrySet().stream().max(Map.Entry.comparingByValue()).map(Map.Entry::getKey).orElse(null);

long vowels = s.toLowerCase().chars().filter(c -> "aeiou".indexOf(c) >= 0).count();
String noDupes = s.chars().distinct().mapToObj(c -> String.valueOf((char) c)).collect(joining());
String sortedChars = s.chars().sorted().mapToObj(c -> String.valueOf((char) c)).collect(joining());
String reversed = new StringBuilder(s).reverse().toString();                           // simplest; stream version below
String reversedStream = IntStream.range(0, s.length()).mapToObj(i -> String.valueOf(s.charAt(s.length() - 1 - i))).collect(joining());

boolean palindrome = IntStream.range(0, s.length() / 2).allMatch(i -> s.charAt(i) == s.charAt(s.length() - 1 - i));
boolean anagram = Arrays.equals(s1.chars().sorted().toArray(), s2.chars().sorted().toArray());

// Word frequency + top 3 (ties broken alphabetically)
String text = "the quick brown fox jumps over the lazy dog the fox";
Map<String, Long> wf = Arrays.stream(text.toLowerCase().split("\\W+")).filter(w -> !w.isEmpty())
        .collect(groupingBy(Function.identity(), counting()));
List<String> top3Words = wf.entrySet().stream()
        .sorted(Map.Entry.<String, Long>comparingByValue().reversed().thenComparing(Map.Entry.comparingByKey()))
        .limit(3).map(e -> e.getKey() + "=" + e.getValue()).collect(toList());

// Longest word / words grouped by length / longest per first letter
String longest = Arrays.stream(text.split(" ")).max(comparingInt(String::length)).orElse("");
Map<Integer, List<String>> byLen = Arrays.stream(text.split(" ")).distinct().collect(groupingBy(String::length));
Map<Character, String> longestByLetter = Arrays.stream(text.split(" ")).collect(
        groupingBy(w -> w.charAt(0), collectingAndThen(maxBy(comparingInt(String::length)), o -> o.get())));

// Group anagrams
Map<String, List<String>> anagrams = Arrays.stream(new String[]{"eat", "tea", "tan", "ate", "nat", "bat"})
        .collect(groupingBy(w -> { char[] c = w.toCharArray(); Arrays.sort(c); return new String(c); }));
```

## 9. `flatMap` — Collections of Collections
- 🟡 Flatten `List<List<Integer>>` · words from sentences · distinct sorted skills across employees
- 🟡 **Skill → list of employee names** (inverse of a one-to-many) · Dept → distinct skills (`flatMapping` or Java 8 workaround)
- 🔴 Pairs / cartesian product · `Optional.flatMap`

```java
List<Integer> flat = List.of(List.of(1, 2), List.of(3), List.of(4, 5)).stream().flatMap(List::stream).collect(toList());
List<String> words = sentences.stream().flatMap(x -> Arrays.stream(x.split("\\s+"))).collect(toList());
List<String> allSkills = emps.stream().flatMap(e -> e.skills().stream()).distinct().sorted().collect(toList());

// Skill -> employee names   (emit (skill, name) pairs, then group)
Map<String, List<String>> bySkill = emps.stream()
        .flatMap(e -> e.skills().stream().map(sk -> new AbstractMap.SimpleEntry<>(sk, e.name())))       // Java 8-friendly
        .collect(groupingBy(Map.Entry::getKey, TreeMap::new, mapping(Map.Entry::getValue, toList())));

// Dept -> distinct skills   (Java 9+)
Map<String, Set<String>> deptSkills = emps.stream().collect(groupingBy(Employee::dept, flatMapping(e -> e.skills().stream(), toCollection(TreeSet::new))));
// Java 8 alternative
Map<String, Set<String>> deptSkills8 = emps.stream().collect(groupingBy(Employee::dept,
        collectingAndThen(toList(), l -> l.stream().flatMap(e -> e.skills().stream()).collect(toCollection(TreeSet::new)))));

// Cartesian product
List<String> pairs = IntStream.rangeClosed(1, 2).boxed().flatMap(i -> Stream.of("a", "b").map(c -> i + c)).collect(toList()); // 1a,1b,2a,2b
```

## 10. Working with Maps
- 🟡 Filter map by value · find key with max value · invert map (value → keys) · merge two maps (sum on clash)
- 🟡 Sort by value/key (see §4) · `Map<String,List<X>>` → count per key · entries to `"k=v"` string

```java
Map<String, Integer> m = Map.of("a", 3, "b", 7, "c", 5);
Map<String, Integer> filtered = m.entrySet().stream().filter(e -> e.getValue() > 4).collect(toMap(Map.Entry::getKey, Map.Entry::getValue));
String maxKey = m.entrySet().stream().max(Map.Entry.comparingByValue()).get().getKey();

// Invert: value -> list of keys
Map<Integer, List<String>> inverted = m.entrySet().stream().collect(groupingBy(Map.Entry::getValue, mapping(Map.Entry::getKey, toList())));

// Merge two maps, summing on clash
Map<String, Integer> merged = Stream.of(m1, m2).flatMap(x -> x.entrySet().stream())
        .collect(toMap(Map.Entry::getKey, Map.Entry::getValue, Integer::sum));

String asString = m.entrySet().stream().map(e -> e.getKey() + "=" + e.getValue()).collect(joining(", ", "{", "}"));
```

## 11. Date & Time (`java.time`) with Streams
**API quick reference (be able to explain):**
| Class | Use |
|---|---|
| `LocalDate` / `LocalTime` / `LocalDateTime` | date / time / both — **no time zone** |
| `Instant` | machine timestamp (UTC epoch) |
| `ZonedDateTime` / `OffsetDateTime` | date-time with zone / offset (use for cross-region) |
| `Period` | **date-based** amount (years, months, days) — `Period.between(d1, d2)` |
| `Duration` | **time-based** amount (hours, minutes, seconds, nanos) — `Duration.between(t1, t2)` |
| `ChronoUnit` | `ChronoUnit.DAYS.between(a, b)` — total units between two points |
| `DateTimeFormatter` | parse/format (`ofPattern("dd-MMM-yyyy")`, thread-safe) |
| `TemporalAdjusters` | `firstDayOfMonth()`, `lastDayOfMonth()`, `next(DayOfWeek.MONDAY)` |
| `YearMonth`, `Year`, `DayOfWeek`, `Month` | grouping keys & helpers |
| `Clock` | inject for **testable** "now" (`LocalDate.now(clock)`) |

Why `java.time` over `Date`/`Calendar`/`SimpleDateFormat`: **immutable, thread-safe**, clearer API, proper time-zone support; `SimpleDateFormat` is **not thread-safe**.

**Questions**
- 🟢 Employees who joined in a given year / after a date / between two dates
- 🟢 Sort employees by joining date (newest first) · oldest / newest employee
- 🟡 **Group employees by joining year / month / YearMonth** · count per year (sorted)
- 🟡 Tenure in years/days per employee (`ChronoUnit`, `Period`) · employees with > 5 years · **average tenure per dept**
- 🟡 Work anniversaries / birthdays **this month** · age from DOB
- 🟡 Orders in the **last 7 days** · **group orders by day** (sum amount) · **monthly revenue** (`YearMonth`)
- 🟡 Orders on weekends · orders per **day of week** (EnumMap) · orders per **hour**
- 🟡 Parse a list of date strings → `LocalDate` → sort → format
- 🔴 **Latest order per customer** · gap between consecutive orders · peak hour per day
- 🔴 List all dates between two dates (Java 8 and Java 9 `datesUntil`) · count **business days** (skip weekends)
- 🔴 Convert across **time zones** · legacy `Date` ⇄ `LocalDate`/`LocalDateTime` · leap years in a range · first/last day of month

```java
LocalDate today = LocalDate.now();

// --- Filtering ---------------------------------------------------------------
List<Employee> joined2021Plus = emps.stream().filter(e -> e.joinDate().getYear() >= 2021).collect(toList());
LocalDate from = LocalDate.of(2018, 1, 1), to = LocalDate.of(2021, 12, 31);
List<Employee> between = emps.stream().filter(e -> !e.joinDate().isBefore(from) && !e.joinDate().isAfter(to)).collect(toList());

// --- Sorting -----------------------------------------------------------------
Employee newest = emps.stream().max(comparing(Employee::joinDate)).get();
Employee oldest = emps.stream().min(comparing(Employee::joinDate)).get();
List<Employee> newestFirst = emps.stream().sorted(comparing(Employee::joinDate).reversed()).collect(toList());

// --- Grouping by date parts --------------------------------------------------
Map<Integer, Long> perYear   = emps.stream().collect(groupingBy(e -> e.joinDate().getYear(), TreeMap::new, counting()));
Map<Month, Long>   perMonth  = emps.stream().collect(groupingBy(e -> e.joinDate().getMonth(), () -> new EnumMap<>(Month.class), counting()));
Map<YearMonth, List<String>> byYearMonth = emps.stream().collect(groupingBy(e -> YearMonth.from(e.joinDate()), TreeMap::new, mapping(Employee::name, toList())));

// --- Tenure -------------------------------------------------------------------
Map<String, Long> tenureYears = emps.stream().collect(toMap(Employee::name, e -> ChronoUnit.YEARS.between(e.joinDate(), today)));
Map<String, Double> avgTenureDaysByDept = emps.stream().collect(groupingBy(Employee::dept,
        averagingLong(e -> ChronoUnit.DAYS.between(e.joinDate(), today))));
List<String> over5y = emps.stream().filter(e -> Period.between(e.joinDate(), today).getYears() >= 5).map(Employee::name).collect(toList());

// --- Anniversaries / birthdays this month -------------------------------------
List<String> anniversaryThisMonth = emps.stream().filter(e -> e.joinDate().getMonth() == today.getMonth()).map(Employee::name).collect(toList());
int age = Period.between(LocalDate.of(1990, 5, 20), today).getYears();

// --- Orders -------------------------------------------------------------------
LocalDateTime now = LocalDateTime.now();
List<Order> last7 = orders.stream().filter(o -> o.createdAt().isAfter(now.minusDays(7))).collect(toList());
Map<LocalDate, Double> revenuePerDay = orders.stream().collect(groupingBy(o -> o.createdAt().toLocalDate(), TreeMap::new, summingDouble(Order::amount)));
Map<YearMonth, Double> monthlyRevenue = orders.stream().collect(groupingBy(o -> YearMonth.from(o.createdAt()), TreeMap::new, summingDouble(Order::amount)));
Map<DayOfWeek, Long> perWeekday = orders.stream().collect(groupingBy(o -> o.createdAt().getDayOfWeek(), () -> new EnumMap<>(DayOfWeek.class), counting()));
Map<Integer, Long> perHour = orders.stream().collect(groupingBy(o -> o.createdAt().getHour(), TreeMap::new, counting()));
List<Order> weekend = orders.stream().filter(o -> { DayOfWeek d = o.createdAt().getDayOfWeek(); return d == DayOfWeek.SATURDAY || d == DayOfWeek.SUNDAY; }).collect(toList());

// Latest order per customer
Map<String, Order> latestPerCustomer = orders.stream().collect(
        groupingBy(Order::customer, collectingAndThen(maxBy(comparing(Order::createdAt)), Optional::get)));

// Gap between consecutive orders (sorted)
List<Order> sorted = orders.stream().sorted(comparing(Order::createdAt)).collect(toList());
List<Duration> gaps = IntStream.range(1, sorted.size()).mapToObj(i -> Duration.between(sorted.get(i - 1).createdAt(), sorted.get(i).createdAt())).collect(toList());
Duration total = gaps.stream().reduce(Duration.ZERO, Duration::plus);

// --- Parse / sort / format date strings ---------------------------------------
DateTimeFormatter in  = DateTimeFormatter.ofPattern("dd-MM-yyyy");
DateTimeFormatter out = DateTimeFormatter.ofPattern("dd MMM yyyy");
List<String> formatted = Stream.of("15-03-2019", "01-07-2021", "10-01-2018").map(d -> LocalDate.parse(d, in)).sorted().map(out::format).collect(toList());

// --- Dates between two dates --------------------------------------------------
LocalDate start = LocalDate.of(2024, 1, 29), end = LocalDate.of(2024, 2, 3);
List<LocalDate> days8 = Stream.iterate(start, d -> d.plusDays(1)).limit(ChronoUnit.DAYS.between(start, end) + 1).collect(toList()); // Java 8
List<LocalDate> days9 = start.datesUntil(end.plusDays(1)).collect(toList());                                                       // Java 9+
long businessDays = Stream.iterate(start, d -> d.plusDays(1)).limit(ChronoUnit.DAYS.between(start, end) + 1)
        .filter(d -> d.getDayOfWeek() != DayOfWeek.SATURDAY && d.getDayOfWeek() != DayOfWeek.SUNDAY).count();

// --- Adjusters, leap years, time zones, legacy Date ---------------------------
LocalDate firstOfMonth = today.with(TemporalAdjusters.firstDayOfMonth());
LocalDate lastOfMonth  = today.with(TemporalAdjusters.lastDayOfMonth());
LocalDate nextMonday   = today.with(TemporalAdjusters.next(DayOfWeek.MONDAY));
List<Integer> leapYears = IntStream.rangeClosed(2000, 2030).filter(y -> Year.isLeap(y)).boxed().collect(toList());

ZonedDateTime ist = ZonedDateTime.of(2024, 6, 1, 9, 0, 0, 0, ZoneId.of("Asia/Kolkata"));
ZonedDateTime ny  = ist.withZoneSameInstant(ZoneId.of("America/New_York"));                      // same instant, different zone

Date legacy = Date.from(LocalDateTime.now().atZone(ZoneId.systemDefault()).toInstant());          // LocalDateTime -> Date
LocalDate fromLegacy = legacy.toInstant().atZone(ZoneId.systemDefault()).toLocalDate();           // Date -> LocalDate

// --- Testability: inject Clock rather than calling now() everywhere ------------
Clock fixed = Clock.fixed(Instant.parse("2024-06-01T00:00:00Z"), ZoneOffset.UTC);
LocalDate testToday = LocalDate.now(fixed);
```
**Date/time gotchas:** `Period` vs `Duration` (date vs time based) · `ChronoUnit.between` gives *total* units, `Period.getDays()` gives only the *day component* · `LocalDateTime` has no zone (don't store as "instant") · `DateTimeFormatter` is thread-safe, `SimpleDateFormat` is not · `MM` = month, `mm` = minute, `yyyy` vs `YYYY` (week-year bug) · DST gaps/overlaps with `ZonedDateTime` · store UTC, convert at the edge.

## 12. Advanced Collectors
```java
// teeing (Java 12): two collectors, one pass -> combine
record MinMax(double min, double max) {}
MinMax mm = emps.stream().collect(teeing(
        minBy(comparingDouble(Employee::salary)), maxBy(comparingDouble(Employee::salary)),
        (mn, mx) -> new MinMax(mn.get().salary(), mx.get().salary())));

// summarizing: count/sum/min/avg/max in one pass
DoubleSummaryStatistics ss = emps.stream().collect(summarizingDouble(Employee::salary));

// reducing (per group): total salary per dept using reducing
Map<String, Double> tot = emps.stream().collect(groupingBy(Employee::dept, reducing(0.0, Employee::salary, Double::sum)));

// toCollection: pick exact collection type
TreeSet<String> cities = emps.stream().map(Employee::city).collect(toCollection(TreeSet::new));

// Unmodifiable results
List<String> ro1 = emps.stream().map(Employee::name).collect(collectingAndThen(toList(), Collections::unmodifiableList));
List<String> ro2 = emps.stream().map(Employee::name).collect(toUnmodifiableList());  // Java 10
List<String> ro3 = emps.stream().map(Employee::name).toList();                       // Java 16

// reduce: sum, max, string concat (identity + accumulator [+ combiner for parallel])
int sum = nums.stream().reduce(0, Integer::sum);
Optional<Integer> mx = nums.stream().reduce(Integer::max);

// takeWhile / dropWhile (Java 9) — on ordered streams
List<Integer> head = Stream.of(1, 2, 3, 10, 4).takeWhile(n -> n < 5).collect(toList());   // [1,2,3]

// Generate infinite then limit
List<Integer> evens = Stream.iterate(0, n -> n + 2).limit(5).collect(toList());           // [0,2,4,6,8]
```

## 13. Pitfalls & Gotchas Interviewers Love
| Gotcha | What happens / fix |
|---|---|
| **Reusing a stream** | `IllegalStateException: stream has already been operated upon` → create a new stream (or use a `Supplier<Stream<T>>`) |
| `toMap` with **duplicate keys** | `IllegalStateException: Duplicate key` → supply a **merge function** `(a,b)->a` or `Integer::sum` |
| `toMap` / `groupingBy` with **null** value / key | `NullPointerException` → filter nulls, use `Optional`, or `groupingBy(e -> Objects.toString(e.city(), "UNKNOWN"))` |
| `groupingBy`/`toMap` give a `HashMap` | **Order is undefined** → pass `LinkedHashMap::new` / `TreeMap::new` |
| Sorting a `HashMap` result | Collect back into `LinkedHashMap` (see §4) |
| `Optional.get()` on empty | `NoSuchElementException` → `orElse`, `orElseGet`, `orElseThrow`, `ifPresent` |
| `sorted()` on non-`Comparable` | `ClassCastException` → pass a `Comparator` |
| **Side effects / shared mutable state** in lambdas | Wrong results in parallel; avoid `list.add` inside `forEach` — use `collect` |
| `peek()` for logic | Meant for debugging only; may be skipped (e.g., `count()` short-circuits on sized streams in Java 9+) |
| Infinite stream without `limit` | Never terminates |
| **Boxing overhead** | Prefer `mapToInt/Double/Long`, `IntStream`; `boxed()` only when needed |
| `distinct()` on custom type | Needs correct `equals`/`hashCode` |
| `collect(toList())` mutability | Not guaranteed; `Stream.toList()` is unmodifiable |
| `parallelStream()` | Don't use for small data, blocking I/O, ordering-sensitive, or stateful lambdas; shares the common `ForkJoinPool`; use `forEachOrdered` for order |
| Checked exceptions in lambdas | Wrap in `try/catch` or a helper that rethrows unchecked |
| `Collectors.counting()` returns `Long` | `averaging*` returns `Double`; `summingInt` returns `Integer` — mind the types |
| `Comparator.comparing(lambda).reversed()` | Type-inference error → use method refs or explicit lambda type |

## 14. Verbal Questions
1. 🟢 What is a stream? How is it different from a collection? (no storage, lazy, single-use, doesn't modify the source)
2. 🟢 Intermediate vs terminal operations; what is **lazy evaluation** and short-circuiting?
3. 🟢 `map` vs `flatMap`; `filter` vs `takeWhile`; `findFirst` vs `findAny`
4. 🟡 `Collectors.toMap` vs `groupingBy` — differences and failure modes (duplicate keys)
5. 🟡 `groupingBy` vs `partitioningBy` — when use which? (partition always has `true`/`false` keys)
6. 🟡 What does `collectingAndThen` do? Give an example (`maxBy` → `Optional::get`).
7. 🟡 `reduce` vs `collect` — mutable reduction, when each is appropriate
8. 🟡 Sequential vs parallel streams; when **not** to use parallel
9. 🟡 `Optional` best practices (`orElse` vs `orElseGet`; avoid `get()`)
10. 🟡 Primitive streams (`IntStream`) — why; `boxed()`, `mapToObj`
11. 🟡 `Period` vs `Duration`; `LocalDateTime` vs `ZonedDateTime` vs `Instant`; why `java.time` replaced `Date`
12. 🔴 How does a stream pipeline execute? (one element flows through the whole pipeline; no intermediate collections; sorted/distinct are stateful barriers)
13. 🔴 How would you debug a stream pipeline? (`peek`, break into steps/variables, IDE stream debugger)
14. 🔴 Write a custom `Collector` (`Collector.of(supplier, accumulator, combiner, finisher)`)
15. 🔴 Why is `Stream.iterate`/`generate` dangerous without `limit`? How does `Stream.iterate(seed, hasNext, next)` (Java 9) help?

## 15. Practice Plan
**Time inside the coding budget (see [01_CODING.md §13](01_CODING.md)):** **1-month plan ≈ 3 h → do the Top-10** (the first 10 items of the Top-25 list below, ~15 min each + review). **2-month plan ≈ 6 h → do the full Top-25** (Top-10 + variants in Week 5, the remaining 15 inside the timed sets of Weeks 7–8).

**Do in this order:**
1. **Round 1 — Top-25 (aloud, plain editor, ~15–20 min each):** **Top-10 (1-month plan):** (1) count per dept · (2) avg salary per dept · (3) highest paid per dept · (4) 2nd highest salary · (5) sort by dept asc → salary desc → name · (6) dept → joined names (`joining`) · (7) **dept → city → names (2-level grouping)** · (8) char frequency + first non-repeating char · (9) word frequency + top 3 · (10) group employees by joining year (`TreeMap`).
**Items 11–25 (2-month plan adds):** (11) **dept → gender → count (2-level)** · (12) top-3 salaried · (13) names joined by comma with prefix/suffix · (14) partition by salary · (15) group anagrams · (16) flatten list of lists · (17) distinct sorted skills (`flatMap`) · (18) sort map by value desc (`LinkedHashMap`) · (19) joined in year X / between two dates · (20) tenure in years (`ChronoUnit`) · (21) orders in the last 7 days · (22) monthly revenue by `YearMonth` · (23) latest order per customer · (24) dates between two dates (Java 8 + 9) · (25) average tenure per dept.
2. **Round 2 — Variants:** change the grouping key, add a `TreeMap`, add a third level, swap `counting()` for `averagingDouble`/`summarizingDouble`, add `joining`.
3. **Round 3 — Without looking:** re-type the 10 you struggled with after 3 and 10 days (mistakes-log).
4. **Always state:** the **result type** (`Map<String, Map<String, List<String>>>`) *before* writing the collector — it tells you how many `groupingBy` levels you need.

**Self-check:** you can write 2-level grouping, `joining` after `mapping`, `Optional`-safe `maxBy` per group, a `LinkedHashMap`-sorted value map, and a `YearMonth` revenue report — each in ≤10 min, without autocomplete.
