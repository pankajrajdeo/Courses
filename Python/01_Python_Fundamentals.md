# 📘 Lecture Notes – Python Basics (Day 1)

## 1. Class Plan & Overview

* **Goal of the class:** Learn Python basics up to the `if-else` statement.

* **Session flow:**

  1. Discuss plan of action.
  2. Learn topics one by one.
  3. Short breaks for Q\&A after each topic.
  4. Recording available immediately after the class.
  5. Next morning → **Task** with \~20 programming problems.

* **Learning approach:**

  * Start from absolute basics.
  * Assume **no prior knowledge** of Python.
  * If you already know, it will serve as **revision**.

---

## 2. Why Python? (Importance & Popularity)

### Evolution

* Python was created in **1989** (earlier than Java).
* Initially less popular, but now rivals Java in usage.

### Reasons for Popularity

1. **Design Philosophy**

   * Very simple & readable.
   * Indentation-based → cleaner code.
   * Example: Python = *friendly like Shahrukh Khan*,
     Java = *strict like Amitabh Bachchan*.

2. **Batteries Included**

   * Rich set of built-in functions & data structures.
   * Example:

     * Reversing a string in C → requires loops.
     * In Python → `my_string[::-1]`.

3. **General-Purpose Language**

   * Supports multiple paradigms: procedural, OOP, functional.
   * Can build:

     * Desktop software (e.g., Google Chrome).
     * Web applications (e.g., Instagram backend).
     * Data Science & ML code.

4. **Community & Ecosystem**

   * Huge, active community.
   * Abundant libraries:

     * Data Science: NumPy, Pandas, SciPy.
     * ML/DL: TensorFlow, Keras, PyTorch.
     * Web: Flask, Django.

### Python in Data Science

* **Why not Java / PHP / R?**

  * Python is **easy to learn** → suits mathematicians/statisticians.
  * Close to mathematical notation (`numpy`, `scipy`).
  * Strong **open-source libraries**.
  * Massive **industry adoption**.

---

## 3. Getting Started with Python

### Hello World Program

```python
print("Hello World")
```

* `print()` is a **function**.
* Case-sensitive: `Print` ≠ `print`.

### Printing Examples

```python
print("Salman Khan is the best actor")   # String
print(7)                                # Integer
print(3.14)                             # Float
print(True)                             # Boolean
```

### Multiple Values

```python
print("Hello", 14, 5)
# Default separator = space
```

Change separator using `sep`:

```python
print("Hello", "World", sep="-")
# Output: Hello-World
```

Change line ending using `end`:

```python
print("Hello", end=" ")
print("World")
# Output: Hello World
```

---

## 4. Comments in Python

* Used to explain code; ignored by interpreter.
* Single-line comments:

```python
# This is a comment
x = 5  # Variable storing value 5
```

* No true multi-line comments, but can simulate using `""" ... """`.

---

## 5. Data Types in Python

### Primitive Types

1. **Integers** – whole numbers

   ```python
   x = 10
   ```

2. **Floats** – decimal numbers

   ```python
   y = 3.14
   ```

3. **Booleans** – True/False

   ```python
   flag = True
   ```

4. **Strings** – text

   ```python
   name = "Python"
   ```

5. **Complex Numbers**

   ```python
   z = 5 + 6j
   ```

### Collections

1. **List** – ordered, mutable

   ```python
   my_list = [1, 2, 3, "Hello"]
   ```

2. **Tuple** – ordered, immutable

   ```python
   my_tuple = (1, 2, 3)
   ```

3. **Set** – unordered, unique elements

   ```python
   my_set = {1, 2, 3}
   ```

4. **Dictionary** – key-value pairs

   ```python
   my_dict = {"name": "Nitish", "age": 27}
   ```

---

## 6. Variables

* Containers to store values.

```python
a = 5
b = 6
print(a + b)  # 11
```

### Dynamic Typing

* No need to declare type.
* Python decides type at runtime.

```python
x = 5       # int
x = "Hello" # str
```

### Multiple Assignment

```python
a, b, c = 1, 2, 3
x = y = z = 100
```

---

## 7. Keywords & Identifiers

* **Keywords** → Reserved words (cannot be used as variable names).

  * Examples: `if`, `else`, `for`, `while`, `class`, `def`, `True`, `False`, `None`.

* **Identifiers** → Names given to variables, functions, classes.

  * Rules:

    * Cannot start with a number.
    * Can use letters, digits, `_`.
    * Case-sensitive.
    * Cannot be a keyword.

✅ Valid:

```python
first_name = "Rahul"
_age = 25
```

❌ Invalid:

```python
2name = "Ravi"
if = 10
```

---

## 8. Taking Input

* Use `input()` function.

```python
name = input("Enter your name: ")
print("Hello", name)
```

⚠️ All input is stored as **string**.

Example:

```python
a = input("Enter first number: ")
b = input("Enter second number: ")
print(a + b)  # concatenation, not addition
```

---

## 9. Type Conversion (Casting)

### Why Needed?

* Inputs are strings → convert to int/float before arithmetic.

### Functions

* `int()`, `float()`, `str()`, `bool()`

```python
a = int(input("Enter first number: "))
b = int(input("Enter second number: "))
result = a + b
print("Result:", result)
```

---

## 10. Literals

* **Literals** = actual fixed values in code.

Examples:

```python
10            # Integer literal
3.14          # Float literal
"Hello"       # String literal
True          # Boolean literal
None          # Special literal (no value)
```

### Special Representations

* **Binary:** `0b1010` → 10
* **Octal:** `0o12` → 10
* **Hexadecimal:** `0xA` → 10
* **Scientific Notation:** `1.5e2` → 150.0

---

## 11. Summary of Today’s Topics

1. Why Python is popular.
2. Printing output with `print()`.
3. Comments in Python.
4. Data types (int, float, str, bool, list, tuple, set, dict, complex).
5. Variables & dynamic typing.
6. Keywords & identifiers.
7. Taking input with `input()`.
8. Type conversion (`int()`, `float()`, etc.).
9. Literals (different representations).

---

✅ Next class: **Operators, If-Else, and Loops**
