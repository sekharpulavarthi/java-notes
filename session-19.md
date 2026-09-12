# 1. Parallel Stream

Your statement:

> Parallel stream if you want to work with multiple threads.

That's basically correct.

```java
nums.parallelStream()
```

creates a **parallel stream**, where Java can divide the stream processing across multiple threads.

For example:

```java
List<Integer> nums = Arrays.asList(4, 5, 7, 3, 2, 6);

Stream<Integer> sortedValues = nums.parallelStream()
        .filter(n -> n % 2 == 0)
        .sorted();

sortedValues.forEach(n -> System.out.println(n));
```

The important thing is:

```text
Collection
    ↓
parallelStream()
    ↓
multiple threads may process elements
    ↓
filter
    ↓
sorted
    ↓
forEach
```

### One important point about `sorted()`

Even though you're using a parallel stream, `sorted()` needs to coordinate the elements because they must ultimately be ordered.

So parallel doesn't mean:

> Every operation will independently finish in random order.

The stream framework handles the coordination.

Also, this:

```java
parallelStream().forEach(...)
```

does **not guarantee encounter order**.

If you specifically want ordered output, you can use:

```java
forEachOrdered(...)
```

For learning Core Java, remember:

```text
stream()          → sequential processing
parallelStream()  → parallel processing may happen
```

And don't automatically use parallel streams just because multiple threads are available. For small collections, the overhead can make them unnecessary.

---

# 2. Optional — what problem does it solve?

This is the important part.

Suppose you have:

```java
List<Integer> nums = Arrays.asList(4, 5, 7, 3, 2, 6);
```

You want to find an element:

```java
Optional<Integer> result = nums.stream()
        .filter(n -> n > 10)
        .findFirst();
```

There is **no number greater than 10**.

So what does `findFirst()` return?

It returns:

```text
Optional.empty()
```

If there is a matching element:

```java
Optional<Integer> result = nums.stream()
        .filter(n -> n > 5)
        .findFirst();
```

It could return:

```text
Optional[7]
```

So Optional represents:

```text
Value may exist
       OR
Value may not exist
```

This is particularly useful for methods where returning `null` would otherwise be common.

---

### Why not just return `null`?

You could do:

```java
Integer result = findStudent();
```

and then:

```java
if (result != null) {
    System.out.println(result);
}
```

But forgetting the null check can cause:

```text
NullPointerException
```

Optional makes the possibility of absence explicit:

```java
Optional<Integer> result = ...
```

Now you know:

> This operation may or may not produce a value.

---

### Basic Optional example

```java
Optional<Integer> result = nums.stream()
        .filter(n -> n > 10)
        .findFirst();

if (result.isPresent()) {
    System.out.println(result.get());
} else {
    System.out.println("No value found");
}
```

But modern Java usually prefers methods such as:

```java
result.ifPresent(System.out::println);
```

or:

```java
int value = result.orElse(0);
```

Meaning:

```text
If value exists → use it
If not → use 0
```

For example:

```java
Optional<Integer> result = nums.stream()
        .filter(n -> n > 10)
        .findFirst();

int value = result.orElse(0);

System.out.println(value);
```

Output:

```text
0
```

because no matching number exists.

### One warning

Don't blindly do:

```java
result.get();
```

without knowing whether a value exists.

If the Optional is empty:

```java
result.get();
```

throws:

```text
NoSuchElementException
```

So `Optional` is not simply "a safer get." Its purpose is to explicitly model **presence or absence**.

---

# 3. Method Reference

Your understanding is **very close**, but one sentence needs correction.

You said:

> Instead of map, we can call toUpperCase directly.

Not exactly.

We **still use `map()`**.

What we remove is the **lambda expression** inside `map()`.

For example, normally:

```java
List<String> names = Arrays.asList("navin", "rahul", "amit");

List<String> result = names.stream()
        .map(name -> name.toUpperCase())
        .toList();
```

Here:

```java
name -> name.toUpperCase()
```

is a lambda.

But the lambda is simply saying:

> Take a String called `name` and call `toUpperCase()` on it.

Java allows us to shorten that using a method reference:

```java
List<String> result = names.stream()
        .map(String::toUpperCase)
        .toList();
```

### What does `String::toUpperCase` mean?

It basically means:

> For each String provided by `map`, call its `toUpperCase()` method.

So:

```java
name -> name.toUpperCase()
```

becomes:

```java
String::toUpperCase
```

The `String` is the class containing the method.

`::` is the **method reference operator**.

---

## Why does `map()` accept it?

Remember what `map()` expects.

Conceptually:

```java
map(Function<T, R>)
```

And `Function` has:

```java
R apply(T value);
```

For:

```java
.map(name -> name.toUpperCase())
```

the lambda is implementing:

```java
Function<String, String>
```

The method reference:

```java
String::toUpperCase
```

also provides a compatible function:

```text
String → String
```

So:

```java
.map(name -> name.toUpperCase())
```

and:

```java
.map(String::toUpperCase)
```

are equivalent in this case.

---

# 4. "We can pass a method name inside a method"

Your basic idea is right, but let's make it technically accurate.

A **method reference is a shorthand for a lambda expression when an existing method already matches the required functional-interface method.**

Example:

```java
name -> name.toUpperCase()
```

can become:

```java
String::toUpperCase
```

Another example:

```java
n -> System.out.println(n)
```

becomes:

```java
System.out::println
```

So:

```java
nums.forEach(n -> System.out.println(n));
```

can become:

```java
nums.forEach(System.out::println);
```

Notice here:

```text
String::toUpperCase
System.out::println
```

These are both method references.

There are different forms of method references, including:

```text
ClassName::staticMethod
ClassName::instanceMethod
object::instanceMethod
ClassName::new
```

You don't need to memorize all of them immediately, but understand the basic idea.

---

# 5. Constructor Reference

This is where you had one small syntax misunderstanding.

You said:

> class colon colon and then the new keyword

The syntax is actually:

```java
ClassName::new
```

**You don't write the `new` keyword separately in a constructor reference.**

For example:

```java
class Student {
    String name;

    Student(String name) {
        this.name = name;
    }
}
```

Normally:

```java
Student s = new Student("Navin");
```

Now suppose we have:

```java
List<String> names = Arrays.asList("Navin", "Rahul", "Amit");
```

We want:

```text
String → Student
```

Using a lambda:

```java
List<Student> students = names.stream()
        .map(name -> new Student(name))
        .toList();
```

We are saying:

```text
For every name:
    create a new Student(name)
```

Because:

```java
name -> new Student(name)
```

is simply calling the constructor, we can use a constructor reference:

```java
List<Student> students = names.stream()
        .map(Student::new)
        .toList();
```

That's the constructor reference.

---

## What exactly does `Student::new` mean?

It means:

> Use the appropriate `Student` constructor to create an object.

So:

```java
name -> new Student(name)
```

becomes:

```java
Student::new
```

The compiler knows which constructor to use based on the target functional interface.

For example:

```java
Function<String, Student> creator = Student::new;
```

Here `Function<String, Student>` tells Java:

```text
Input  → String
Output → Student
```

So Java looks for:

```java
Student(String name)
```

and uses it.

---

# 6. One important connection

You're now seeing how several Stream API concepts fit together:

```java
names.stream()
     .map(name -> name.toUpperCase())
```

Lambda version.

Then:

```java
names.stream()
     .map(String::toUpperCase)
```

Method-reference version.

And:

```java
names.stream()
     .map(name -> new Student(name))
```

Lambda + constructor.

Then:

```java
names.stream()
     .map(Student::new)
```

Constructor-reference version.

So the progression is:

```text
Lambda
   ↓
Method Reference
```

when an existing method already matches what the lambda needs.

And:

```text
name -> new Student(name)
          ↓
Student::new
```

when the lambda simply creates an object using a constructor.

---

# README.md Notes

Here is the **separate notes section**, ready to copy directly into your README.

````markdown
# Session: Java Stream API — Parallel Stream, Optional, Method Reference & Constructor Reference

## 1. Parallel Stream

A parallel stream allows stream operations to be processed using multiple threads.

```java
List<Integer> nums = Arrays.asList(4, 5, 7, 3, 2, 6);

Stream<Integer> sortedValues = nums.parallelStream()
        .filter(n -> n % 2 == 0)
        .sorted();

sortedValues.forEach(n -> System.out.println(n));
```
````

### Sequential vs Parallel

```java
nums.stream()
```

→ Sequential stream.

```java
nums.parallelStream()
```

→ Parallel stream; Java may divide the work among multiple threads.

### Important

Parallel stream does NOT mean every operation independently runs in a random order.

Some operations, such as `sorted()`, require coordination.

Also:

```java
parallelStream().forEach(...)
```

does not guarantee encounter order.

If ordered processing is required:

```java
parallelStream().forEachOrdered(...)
```

Use parallel streams when parallel processing can actually provide a benefit. For small/simple collections, parallelism can introduce unnecessary overhead.

---

# 2. Optional

`Optional<T>` is a container that represents:

```text
Value is present
        OR
Value is absent
```

It is commonly used when an operation may not return a value.

Example:

```java
List<Integer> nums = Arrays.asList(4, 5, 7, 3, 2, 6);

Optional<Integer> result = nums.stream()
        .filter(n -> n > 10)
        .findFirst();
```

There is no number greater than 10, so:

```java
result
```

contains:

```text
Optional.empty()
```

If we search for a number greater than 5:

```java
Optional<Integer> result = nums.stream()
        .filter(n -> n > 5)
        .findFirst();
```

The result can be:

```text
Optional[7]
```

## Why Optional?

Without Optional, a method might return:

```java
null
```

and the caller might forget to check for null, causing:

```text
NullPointerException
```

Optional makes the possibility of absence explicit.

## Checking the value

```java
if (result.isPresent()) {
    System.out.println(result.get());
}
```

But avoid blindly using:

```java
result.get();
```

If the Optional is empty, `get()` throws:

```text
NoSuchElementException
```

## ifPresent()

```java
result.ifPresent(value -> System.out.println(value));
```

Can also use a method reference:

```java
result.ifPresent(System.out::println);
```

## orElse()

Provide a default value if the Optional is empty:

```java
int value = result.orElse(0);
```

Meaning:

```text
Value exists → use the value
Value absent → use 0
```

### Common Stream operations returning Optional

Some terminal operations can return Optional because a result may not exist:

```java
findFirst()
findAny()
max()
min()
```

Example:

```java
Optional<Integer> result = nums.stream()
        .filter(n -> n > 10)
        .findFirst();
```

---

# 3. Method Reference

A method reference is a shorthand for a lambda expression when an existing method already matches the required functional-interface method.

Syntax:

```java
ClassName::methodName
```

or:

```java
object::methodName
```

## Example without Method Reference

```java
List<String> names = Arrays.asList("navin", "rahul", "amit");

List<String> result = names.stream()
        .map(name -> name.toUpperCase())
        .toList();
```

The lambda:

```java
name -> name.toUpperCase()
```

takes a String and calls `toUpperCase()`.

## Using Method Reference

```java
List<String> result = names.stream()
        .map(String::toUpperCase)
        .toList();
```

These are equivalent:

```java
name -> name.toUpperCase()
```

```java
String::toUpperCase
```

### Important

Method reference does NOT replace `map()`.

This:

```java
.map(name -> name.toUpperCase())
```

becomes:

```java
.map(String::toUpperCase)
```

`map()` is still present.

Only the lambda expression is replaced by the method reference.

## Why does String::toUpperCase work?

`map()` expects a `Function`.

Conceptually:

```java
Function<T, R>
```

For Strings:

```text
String → String
```

The lambda:

```java
name -> name.toUpperCase()
```

matches that function.

Therefore it can be shortened to:

```java
String::toUpperCase
```

Meaning:

```text
For each String:
    call its toUpperCase() method
```

---

## Another Method Reference Example

Without method reference:

```java
nums.forEach(n -> System.out.println(n));
```

With method reference:

```java
nums.forEach(System.out::println);
```

Here:

```java
System.out::println
```

means:

```text
Call println() on System.out
```

---

# 4. Constructor Reference

A constructor reference is a shorthand for a lambda expression that creates an object.

Syntax:

```java
ClassName::new
```

Important:

We do NOT write:

```java
ClassName::new keyword
```

The correct syntax is simply:

```java
ClassName::new
```

## Example

Class:

```java
class Student {
    String name;

    Student(String name) {
        this.name = name;
    }
}
```

Normally:

```java
Student s = new Student("Navin");
```

Using Stream + lambda:

```java
List<String> names = Arrays.asList("Navin", "Rahul", "Amit");

List<Student> students = names.stream()
        .map(name -> new Student(name))
        .toList();
```

The lambda:

```java
name -> new Student(name)
```

can be shortened to:

```java
Student::new
```

So:

```java
List<Student> students = names.stream()
        .map(Student::new)
        .toList();
```

## Meaning of Student::new

```java
Student::new
```

means:

```text
Use the appropriate Student constructor
to create a new Student object.
```

It is equivalent to:

```java
name -> new Student(name)
```

when the target type matches the constructor.

---

# 5. Function + Constructor Reference

A constructor reference can be assigned to a functional interface.

```java
Function<String, Student> creator = Student::new;
```

Meaning:

```text
Input  → String
Output → Student
```

So Java uses:

```java
Student(String name)
```

Then:

```java
Student s = creator.apply("Navin");
```

creates a new Student object.

---

# 6. Lambda → Method Reference

When a lambda only calls an existing method, it can often be replaced with a method reference.

```java
name -> name.toUpperCase()
```

↓

```java
String::toUpperCase
```

Another example:

```java
n -> System.out.println(n)
```

↓

```java
System.out::println
```

---

# 7. Lambda → Constructor Reference

When a lambda only creates an object using a constructor:

```java
name -> new Student(name)
```

↓

```java
Student::new
```

---

# Quick Recall

### Parallel Stream

```java
nums.parallelStream()
```

→ Allows parallel processing using multiple threads.

### Optional

```java
Optional<Integer>
```

→ Represents a value that may or may not exist.

```java
findFirst()
findAny()
max()
min()
```

can return Optional.

### Method Reference

```java
String::toUpperCase
```

→ Shorthand for:

```java
name -> name.toUpperCase()
```

### Object Method Reference

```java
System.out::println
```

→ Shorthand for:

```java
n -> System.out.println(n)
```

### Constructor Reference

```java
Student::new
```

→ Shorthand for:

```java
name -> new Student(name)
```

### Core Idea

```text
Lambda
  ↓
If it simply calls an existing method
  ↓
Method Reference

Lambda
  ↓
If it simply creates an object using a constructor
  ↓
Constructor Reference
```

```

```
