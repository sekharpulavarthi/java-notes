# Java Day 4 — Stack, Heap & Arrays

## 1. Stack and Heap

Java runtime memory is commonly explained using two important areas:

```text
Stack → method execution / local variables / references
Heap  → objects and arrays
```

### Stack

When a method is invoked, a **stack frame** is created for that method invocation.

It contains information needed for that particular method execution, such as:

- local variables
- method parameters
- references to objects
- execution-related information

Example:

```java
void calculate() {
    int x = 10;
}
```

`x` is a local variable associated with the `calculate()` method's stack frame.

### Stack behavior

The stack follows:

**LIFO — Last In, First Out**

Example:

```text
main()
  ↓
methodA()
  ↓
methodB()
```

`methodB()` finishes first, then `methodA()`, then `main()`.

---

## 2. Heap

The heap is the area of memory where objects and arrays are allocated.

Example:

```java
Calculator calculator = new Calculator();
```

Conceptually:

```text
Stack                         Heap

calculator ────────────────→ Calculator object
```

The variable `calculator` holds a **reference** to the object.

---

## 3. Object and Reference

Consider:

```java
Calculator calculator = new Calculator();
```

There are two different things here:

```text
calculator
    ↓
reference variable

new Calculator()
    ↓
object
```

The object is created in the heap, while the reference variable is associated with the current method's stack frame.

---

## 4. Instance Variables

Instance variables belong to an object.

```java
class Calculator {
    int value;
}
```

When an object is created:

```java
Calculator calculator = new Calculator();
```

the object has its own `value`.

```text
Heap

Calculator object
└── value
```

Different objects can have different values:

```java
Calculator c1 = new Calculator();
Calculator c2 = new Calculator();

c1.value = 10;
c2.value = 20;
```

Conceptually:

```text
c1 → Calculator object → value = 10

c2 → Calculator object → value = 20
```

---

## 5. Local Variables vs Instance Variables

### Local variable

Declared inside a method/block.

```java
void calculate() {
    int result = 100;
}
```

`result` belongs to that method invocation.

### Instance variable

Declared inside a class but outside methods.

```java
class Calculator {
    int value;
}
```

`value` belongs to each object.

### Remember

```text
Local variable
→ associated with method execution

Instance variable
→ associated with object
```

---

## 6. Method Calls and Stack Frames

Suppose:

```java
public static void main(String[] args) {
    calculate();
}

static void calculate() {
    int x = 10;
}
```

Conceptually:

```text
main() starts
    ↓
main stack frame
    ↓
calculate() called
    ↓
calculate stack frame created
    ↓
calculate() finishes
    ↓
calculate stack frame removed
    ↓
main() continues
```

Each **method invocation** gets its own stack frame.

---

# 7. Arrays

An array stores multiple values of the same type.

Example:

```java
int[] numbers = new int[5];
```

This creates an array capable of storing **5 integers**.

Indexes start from `0`:

```text
numbers[0]
numbers[1]
numbers[2]
numbers[3]
numbers[4]
```

---

## 8. Array Initialization

### Fixed size

```java
int[] numbers = new int[5];
```

The size is determined when the array is created.

### Direct initialization

```java
int[] numbers = {10, 20, 30, 40};
```

The array size is automatically determined from the number of elements.

---

## 9. Array Access

```java
int[] numbers = {10, 20, 30};

System.out.println(numbers[0]);  // 10
System.out.println(numbers[1]);  // 20
System.out.println(numbers[2]);  // 30
```

Changing a value:

```java
numbers[1] = 50;
```

Now:

```text
{10, 50, 30}
```

---

## 10. Array Length

Use the `length` property:

```java
int[] numbers = {10, 20, 30};

System.out.println(numbers.length);  // 3
```

> Arrays use `length`, not `length()`.

---

## 11. Multidimensional Arrays

A two-dimensional array can be created using:

```java
int[][] matrix = new int[2][3];
```

Conceptually:

```text
2 rows × 3 columns

[ 0  0  0 ]
[ 0  0  0 ]
```

Access elements using two indexes:

```java
matrix[0][1] = 10;
```

Java also supports multidimensional arrays with direct initialization:

```java
int[][] matrix = {
    {1, 2, 3},
    {4, 5, 6}
};
```

---

# Quick Recall

- **Stack** → method invocation stack frames
- **Heap** → objects and arrays
- **LIFO** → stack behavior
- Each **method invocation** gets a stack frame.
- Object created using `new` → heap allocation
- Reference variable → refers to an object
- Instance variable → belongs to an object
- Local variable → belongs to method execution
- Array → fixed-size collection of same-type elements
- Array indexes start at `0`
- Array size → `length`
- 2D array → `type[][]`
