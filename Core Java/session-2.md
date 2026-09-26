# Java Day 2 — Type Casting, Promotion & Increment Operators

## 1. Type Conversion vs Type Casting

### Type Conversion

Converting a value from one data type to another.

Java performs some conversions **automatically** when the conversion is safe.

```java
int num = 10;
long value = num;     // implicit conversion
```

This is generally a **widening conversion**:

```text
byte → short → int → long → float → double
```

Smaller type → larger compatible type.

---

### Type Casting

Explicitly telling Java to convert a value to another type.

Syntax:

```java
targetType variable = (targetType) value;
```

Example:

```java
double price = 10.5;
int value = (int) price;
```

Result:

```text
10.5 → 10
```

The fractional part is discarded.

### Remember

```text
Implicit conversion → Java does it automatically

Explicit casting → We tell Java:
                         (targetType)
```

---

## 2. Widening vs Narrowing

### Widening

Smaller range → larger range.

Usually happens automatically.

```java
byte b = 10;
int i = b;
```

```text
byte → int
```

No explicit cast is required.

### Narrowing

Larger type → smaller type.

Requires explicit casting.

```java
int i = 100;
byte b = (byte) i;
```

```text
int → byte
```

Potential information loss can occur.

---

## 3. Why Can't `int` Be Directly Assigned to `byte`?

```java
int num = 10;
byte b = num;       // ❌ compilation error
```

Even though `10` fits inside a byte, the variable `num` is an `int`.

Java does not automatically perform narrowing conversions because the `int` value might not fit inside a `byte`.

Instead:

```java
byte b = (byte) num;   // ✅
```

---

## 4. Casting `int` to `byte`

A `byte` is **8 bits**.

When an `int` is explicitly converted to a `byte`, Java keeps the lower 8 bits of the value.

Therefore, if the value is outside the byte range:

```text
-128 to 127
```

the result may be unexpected.

Example:

```java
int num = 130;
byte b = (byte) num;
```

The result is:

```text
-126
```

### Important

Don't memorize this as:

> "Java divides the value by something."

Instead remember:

> **Narrowing conversion can discard higher-order bits, causing overflow/wrapping.**

---

# 5. Type Promotion

Java automatically promotes smaller integer types during certain arithmetic operations.

For example:

```java
byte a = 10;
byte b = 20;

int result = a * b;
```

Even though both `a` and `b` are `byte`, the arithmetic result is promoted to `int`.

Therefore:

```java
byte result = a * b;   // ❌
int result = a * b;    // ✅
```

### Key rule

For arithmetic operations, `byte`, `short`, and `char` are generally promoted to `int`.

```text
byte + byte → int
byte * byte → int
short + short → int
char + char → int
```

This is called **numeric type promotion**.

---

# 6. Pre-Increment vs Post-Increment

Both increase the value by `1`.

### Pre-increment

```java
++num
```

Increment first → then use the value.

```java
int num = 5;
int result = ++num;
```

Result:

```text
num    = 6
result = 6
```

### Post-increment

```java
num++
```

Use the current value first → then increment.

```java
int num = 5;
int result = num++;
```

Result:

```text
num    = 6
result = 5
```

### Memory trick

```text
++num → increment → use

num++ → use → increment
```

The difference matters **when the expression's value is being used**.

If you're simply doing:

```java
num++;
```

or:

```java
++num;
```

both ultimately increase `num` by `1`.

---

# 7. Short-Circuit Logical Operators

You already know logical operators, so only remember the Java-specific terminology:

```java
&&   // short-circuit AND
||   // short-circuit OR
```

### `&&`

If the first condition is `false`, Java doesn't evaluate the remaining conditions.

```java
false && something
```

`something` is not evaluated.

### `||`

If the first condition is `true`, Java doesn't evaluate the remaining conditions.

```java
true || something
```

`something` is not evaluated.

### Why "short-circuit"?

Because Java **stops evaluating as soon as the final result is already known**.

---

# 8. Java Formatting

Java does not require indentation for the compiler to understand the program.

This is syntactically possible:

```java
if (true) {
System.out.println("Hello");
}
```

But proper indentation is strongly recommended because it makes code **readable and maintainable**.

```java
if (true) {
    System.out.println("Hello");
}
```

> **Indentation is for humans; syntax is for the compiler.**

---

# Quick Recall

- **Widening** → smaller type → larger type → generally implicit
- **Narrowing** → larger type → smaller type → explicit cast required
- `(byte) num` → explicit casting
- `byte + byte` → result is generally `int`
- `++num` → increment first, then use
- `num++` → use first, then increment
- `&&` / `||` → short-circuit operators
- Indentation → not required by compiler, but important for readability
