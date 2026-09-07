# Java Day 5 — Jagged Arrays, Enhanced For Loop & Arrays of Objects

## 1. Jagged Array

A jagged array is a **2D array where each row can have a different length**.

```java
int[][] nums = new int[3][];

nums[0] = new int[2];
nums[1] = new int[4];
nums[2] = new int[3];
```

Conceptually:

```text
Row 0 → [0, 0]
Row 1 → [0, 0, 0, 0]
Row 2 → [0, 0, 0]
```

Unlike a rectangular 2D array:

```java
int[][] nums = new int[3][4];
```

where every row has 4 elements.

---

## 2. Enhanced `for` Loop

The enhanced `for` loop is useful when you want to iterate through every element without managing the index manually.

### One-dimensional array

```java
int[] nums = {10, 20, 30};

for (int num : nums) {
    System.out.println(num);
}
```

Equivalent idea:

```java
for (int i = 0; i < nums.length; i++) {
    System.out.println(nums[i]);
}
```

---

## 3. Enhanced `for` with 2D Arrays

A 2D array is effectively an **array of arrays**.

```java
int[][] nums = {
    {1, 2, 3},
    {4, 5, 6}
};
```

Enhanced loop:

```java
for (int[] row : nums) {
    for (int num : row) {
        System.out.println(num);
    }
}
```

Think:

```text
nums
 ↓
row 1 → [1, 2, 3]
row 2 → [4, 5, 6]
```

The outer loop gets each row.

The inner loop gets each element from that row.

---

# 4. Arrays Are Objects

In Java, an array is an **object**.

```java
int[] nums = new int[5];
```

The array object is allocated on the heap.

The variable `nums` contains a reference to that array.

```text
Stack                  Heap

nums ───────────────→ [0, 0, 0, 0, 0]
```

---

# 5. `new` and Object Creation

`new` is used to create objects/arrays.

```java
Student s1 = new Student();
```

```java
int[] nums = new int[5];
```

Both involve heap allocation.

But remember:

> `new` does **not always mean "calling a class."**

For example:

```java
new Student()
```

creates a `Student` object and invokes its constructor.

But:

```java
new int[5]
```

creates an array object. An array is an object, but `int[]` isn't a normal user-defined class that you're explicitly calling.

---

# 6. Default Values in Arrays

When an array is created, its elements receive default values.

For an `int` array:

```java
int[] nums = new int[5];
```

The values are:

```text
[0, 0, 0, 0, 0]
```

Common defaults:

```text
int      → 0
double   → 0.0
boolean  → false
char     → '\u0000'
reference types → null
```

---

# 7. Arrays of Objects

An array can store references to objects.

Example:

```java
Student[] students = new Student[3];
```

This creates an array capable of holding **3 Student references**.

It does **not** create 3 Student objects.

Initially:

```text
students
   ↓
[ null | null | null ]
```

You must create the Student objects separately:

```java
students[0] = new Student();
students[1] = new Student();
students[2] = new Student();
```

Then:

```text
students
   ↓
[ ref | ref | ref ]
   ↓    ↓    ↓
  S1   S2   S3
```

---

# 8. Enhanced `for` with Objects

```java
for (Student stud : students) {
    System.out.println(stud.name);
}
```

Here:

```text
Student stud
```

is a reference variable that refers to the current Student object.

Each iteration:

```text
stud → students[0]

stud → students[1]

stud → students[2]
```

So:

```java
stud.name
```

accesses the `name` of the current Student object.

---

# 9. Disadvantages of Arrays

Important limitations:

- Fixed size after creation
- Can store only a specific type
- Insertion/deletion can require shifting elements
- No built-in dynamic resizing
- Managing collections of objects manually can become inconvenient

Java provides the **Collections Framework** for more flexible data structures, such as `ArrayList`, `LinkedList`, `HashSet`, and `HashMap`.
