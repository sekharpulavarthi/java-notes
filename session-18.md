## 1. HashMap — start with the mental model

A `Map` stores **key-value pairs**:

```java
Map<String, Integer> students = new HashMap<>();

students.put("Navin", 56);
students.put("Harsh", 23);
students.put("Sushil", 67);
students.put("Kiran", 92);
```

Conceptually:

```text
Key       Value
----------------
Navin       56
Harsh       23
Sushil      67
Kiran       92
```

The key is used to find the value.

```java
students.get("Harsh");
```

gives:

```text
23
```

---

## 2. What happens when you put the same key?

You wrote:

```java
students.put("Harsh", 23);
students.put("Harsh", 45);
```

The second `put()` **replaces the value associated with that key**.

```text
Before:
Harsh → 23

After:
Harsh → 45
```

The key remains unique.

So:

```text
Map
 ↓
unique keys
 ↓
each key maps to a value
```

Values can be duplicated.

```java
students.put("A", 50);
students.put("B", 50);
```

is perfectly valid.

---

# 3. Is HashMap a combination of Set + List?

Your statement:

> "HashMap is a combination of Set and List."

is a useful intuition, but **technically don't write this in your notes**.

A `Map` is a separate collection hierarchy.

```text
Collection
├── List
├── Set
└── Queue

Map
├── HashMap
├── LinkedHashMap
└── TreeMap
```

A better mental model is:

> **Map = key → value association**

Internally, `HashMap` is hash-table based, and its keys are unique.

---

# 4. HashMap example

```java
Map<String, Integer> students = new HashMap<>();

students.put("Navin", 56);
students.put("Harsh", 23);
students.put("Sushil", 67);
students.put("Kiran", 92);
students.put("Harsh", 45);

System.out.println(students);
```

You can get the keys:

```java
students.keySet();
```

Then:

```java
for (String key : students.keySet()) {
    System.out.println(key + ":" + students.get(key));
}
```

Mental model:

```text
keySet()
   ↓
gives all keys

get(key)
   ↓
finds value associated with that key
```

---

# 5. What is a Hash Table?

A **hash table** is a data-structure technique that uses a **hash function** to determine where data should be stored.

Simplified example:

Suppose we have:

```text
Hash table size = 10
```

and a key:

```text
"Harsh"
```

A hash function produces some hash value:

```text
"Harsh"
   ↓
hash function
   ↓
some hash value
   ↓
index/bucket
```

Conceptually:

```text
Bucket
  0
  1
  2
  3
  4  ← "Harsh" → 45
  5
  6
  7
  8
  9
```

Java's `HashMap` uses this **hashing-based approach** internally.

You don't manually calculate the bucket.

---

# 6. HashMap vs Hashtable

This is an important interview comparison.

| `HashMap`                                 | `Hashtable`                  |
| ----------------------------------------- | ---------------------------- |
| Modern general-purpose Map implementation | Legacy class                 |
| Not synchronized by default               | Synchronized methods         |
| Allows one `null` key                     | Does not allow `null` keys   |
| Allows `null` values                      | Does not allow `null` values |
| Generally preferred                       | Usually avoid in new code    |

So your code:

```java
Map<String, Integer> students = new Hashtable<>();
```

works, but if you don't specifically need `Hashtable`, modern code generally uses:

```java
Map<String, Integer> students = new HashMap<>();
```

### Important

Don't confuse:

```text
HashMap
```

with:

```text
Hashtable
```

just because their names sound similar.

---

# 7. Now the confusing part — `toString()`

You have:

```java
class Student {
    int age;
    String name;

    public Student(int age, String name) {
        this.age = age;
        this.name = name;
    }

    public String toString() {
        return "Student [age=" + age + ", name=" + name + "]";
    }
}
```

Then:

```java
studs.add(new Student(21, "Navin"));
```

You're asking:

> "I never called `toString()`. How am I getting the output?"

Because when you do:

```java
System.out.println(s);
```

Java needs to convert the object into a String representation.

`println(Object)` effectively uses the object's string representation, which comes from `toString()`.

So:

```java
System.out.println(s);
```

conceptually becomes:

```java
System.out.println(s.toString());
```

Therefore:

```java
System.out.println(s);
```

can produce:

```text
Student [age=21, name=Navin]
```

because your `Student.toString()` overrides the method inherited from `Object`.

### Important distinction

This:

```java
studs.add(new Student(21, "Navin"));
```

does **not** call `toString()`.

This:

```java
System.out.println(s);
```

is what causes the string representation to be obtained.

---

# 8. Now `Comparable` — what is it?

This is the biggest concept.

Suppose you have:

```java
List<Student> studs = new ArrayList<>();
```

and:

```text
Student
21 Navin
12 John
18 Parul
20 Kiran
```

You tell Java:

> "Sort these Students."

But Java has a problem:

**Sort according to what?**

Age?

Name?

Salary?

Marks?

You need to define the ordering.

That's where `Comparable` comes in.

---

# 9. Comparable = object's natural ordering

You can make `Student` implement `Comparable<Student>`:

```java
class Student implements Comparable<Student> {

    int age;
    String name;

    public Student(int age, String name) {
        this.age = age;
        this.name = name;
    }

    @Override
    public int compareTo(Student that) {
        return this.age - that.age;
    }
}
```

Now `Student` itself says:

> "My natural ordering is based on age."

Then:

```java
Collections.sort(studs);
```

knows how to compare two Students because `Student` provides:

```java
compareTo()
```

---

# 10. What does `compareTo()` return?

This is extremely important.

```java
this.compareTo(that)
```

returns:

```text
negative → this comes before that

0        → they are considered equal in ordering

positive → this comes after that
```

For example:

```java
return this.age - that.age;
```

Suppose:

```text
this.age  = 12
that.age  = 20
```

Then:

```text
12 - 20 = -8
```

Negative → 12 comes before 20.

If:

```text
this.age = 20
that.age = 12
```

then:

```text
20 - 12 = 8
```

Positive → 20 comes after 12.

---

# 11. Your original `compareTo()` logic

You had:

```java
if (this.age > that.age)
    return 1;
else
    return -1;
```

There is a problem here.

If:

```text
this.age == that.age
```

you still return `-1`.

But equal values should normally return:

```text
0
```

Better:

```java
@Override
public int compareTo(Student that) {
    return Integer.compare(this.age, that.age);
}
```

This is safer than:

```java
this.age - that.age
```

because subtraction can overflow for extreme integer values.

---

# 12. So why do we need `Comparator`?

Now suppose you don't want Student's natural ordering to always be age.

Maybe sometimes you want:

```text
sort by age
```

and sometimes:

```text
sort by name
```

and sometimes:

```text
sort by salary
```

You don't want to keep changing `Student.compareTo()`.

That's where `Comparator` comes in.

A `Comparator` is an **external comparison strategy**.

Example:

```java
Comparator<Student> com =
    (i, j) -> Integer.compare(i.age, j.age);
```

Then:

```java
Collections.sort(studs, com);
```

You're telling Java:

> "For this particular sorting operation, use this comparison logic."

---

# 13. Comparable vs Comparator

This is the most important thing to remember.

### Comparable

The class itself defines its **natural ordering**.

```java
class Student implements Comparable<Student> {

    @Override
    public int compareTo(Student that) {
        return Integer.compare(this.age, that.age);
    }
}
```

Then:

```java
Collections.sort(studs);
```

### Comparator

The sorting logic is provided **externally**.

```java
Comparator<Student> byAge =
    (a, b) -> Integer.compare(a.age, b.age);

Collections.sort(studs, byAge);
```

Mental model:

```text
Comparable
    ↓
Student decides how Students naturally compare


Comparator
    ↓
Outside object decides how Students should compare
```

---

# 14. When should you use which?

Imagine:

```java
class Student
```

Maybe your application's natural definition of a Student is:

```text
age
```

Then:

```java
Comparable<Student>
```

makes sense.

But you might also need:

```text
sort by name
sort by age
sort by marks
sort by salary
```

You can create multiple Comparators:

```java
Comparator<Student> byAge =
    (a, b) -> Integer.compare(a.age, b.age);

Comparator<Student> byName =
    (a, b) -> a.name.compareTo(b.name);
```

Then:

```java
Collections.sort(studs, byAge);
```

or:

```java
Collections.sort(studs, byName);
```

That's the real advantage of `Comparator`.

---

# 15. Why does Comparator work with lambda?

`Comparator<T>` is a **functional interface**.

It has one abstract method:

```java
int compare(T o1, T o2);
```

Therefore:

```java
Comparator<Student> com =
    (i, j) -> Integer.compare(i.age, j.age);
```

is a lambda implementation of that method.

Similarly:

```java
Comparator<Integer> com =
    (i, j) -> Integer.compare(i, j);
```

---

# 16. `compareTo()` vs `compare()`

Don't mix these up.

### Comparable

```java
int compareTo(Student other)
```

One object compares **itself** with another object.

```text
this  vs  other
```

### Comparator

```java
int compare(Student a, Student b)
```

The comparator compares **two supplied objects**.

```text
a  vs  b
```

So:

```text
Comparable → compareTo()

Comparator → compare()
```

---

# 17. Your code has one important mistake

You wrote:

```java
Comparator<Student> com=(i,j) -> i.age > j.age?1:-1;
```

That's a comparator.

But then you wrote:

```java
Collections.sort(studs);
```

That does **not use `com`**.

If you want to use your comparator:

```java
Collections.sort(studs, com);
```

Alternatively:

```java
studs.sort(com);
```

If you write:

```java
Collections.sort(studs);
```

then `Student` needs to implement `Comparable<Student>`.

So these are two different versions:

```java
// Comparable
Collections.sort(studs);
```

versus:

```java
// Comparator
Collections.sort(studs, com);
```

This is probably one of the reasons the session felt confusing.

---

# 18. `forEach()` and Consumer

You also mentioned:

> "forEach takes Consumer object."

Correct.

`Consumer<T>` is a **functional interface** with:

```java
void accept(T t);
```

For example:

```java
Consumer<Integer> c = n -> System.out.println(n);
```

Then:

```java
c.accept(10);
```

prints:

```text
10
```

A collection has:

```java
nums.forEach(...)
```

and you can provide a Consumer using a lambda:

```java
nums.forEach(n -> System.out.println(n));
```

Conceptually:

```text
forEach()
   ↓
takes Consumer
   ↓
Consumer has accept()
   ↓
lambda provides implementation
```

---

# 19. Stream API

Your understanding is mostly correct.

A Stream is a way to process a sequence of elements through operations such as:

```text
filter
map
reduce
forEach
sorted
```

Example:

```java
List<Integer> nums =
    Arrays.asList(4, 5, 7, 3, 2, 6);
```

Your pipeline:

```java
int result = nums.stream()
                 .filter(n -> n % 2 == 0)
                 .map(n -> n * 2)
                 .reduce(0, (c, e) -> c + e);
```

Let's execute it mentally.

Original:

```text
4  5  7  3  2  6
```

### `filter`

Keep even numbers:

```text
4  2  6
```

### `map`

Multiply each by 2:

```text
8  4  12
```

### `reduce`

Add everything:

```text
0 + 8 + 4 + 12
= 24
```

So:

```text
result = 24
```

---

# 20. Does Stream modify the original collection?

Normally, these stream operations don't modify the source collection themselves.

```java
nums.stream()
    .filter(...)
    .map(...)
```

creates a processing pipeline.

The original:

```text
4 5 7 3 2 6
```

remains unchanged unless your operations explicitly mutate something.

Think:

```text
Original Collection
       │
       ↓
     stream()
       │
       ↓
    filter()
       │
       ↓
      map()
       │
       ↓
    reduce()
       │
       ↓
     result
```

---

# 21. A Stream is one-time use

This part of your notes is correct.

Once a terminal operation has consumed a stream:

```java
Stream<Integer> s = nums.stream();

s.filter(n -> n % 2 == 0)
 .forEach(System.out::println);
```

you cannot reuse `s`:

```java
s.forEach(System.out::println);  // IllegalStateException
```

Instead, create another stream:

```java
nums.stream()
    .filter(...);

nums.stream()
    .map(...);
```

### Important terminology

Operations such as:

```text
filter()
map()
```

are **intermediate operations**.

They produce another Stream.

Operations such as:

```text
forEach()
reduce()
```

are **terminal operations**.

They consume/terminate the stream.

Mental model:

```text
stream()
   ↓
filter()  ← intermediate
   ↓
map()     ← intermediate
   ↓
reduce()  ← terminal
```

---

# Quick Recall

```text
HashMap
→ key-value pairs
→ unique keys
→ values can duplicate
→ hashing-based
→ no guaranteed iteration order

Hashtable
→ legacy synchronized Map
→ no null key/value
→ generally not preferred for new code
```

```text
Comparable
→ class defines natural ordering
→ implements Comparable<T>
→ compareTo()

Comparator
→ external/custom ordering
→ Comparator<T>
→ compare()
→ can create multiple sorting strategies
```

```text
compareTo() / compare()

negative → first comes before second
0        → equal in ordering
positive → first comes after second
```

```text
toString()
→ inherited from Object
→ overridden for meaningful object representation
→ println(object) obtains the object's string representation
```

```text
Consumer<T>
→ functional interface
→ accept(T)
→ commonly used with forEach()
```

```text
Stream
→ processes data through a pipeline
→ doesn't inherently modify source collection
→ intermediate operations return streams
→ terminal operation consumes the stream
→ a stream cannot be reused after terminal operation
```

The **big three concepts I'd make sure you can explain without notes** after this session are:

**1. `HashMap` vs `Hashtable`**
**2. `Comparable` vs `Comparator`**
**3. `compareTo()` vs `compare()`**

Those are much more important than memorizing the syntax.
