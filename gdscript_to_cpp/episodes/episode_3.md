# GDScript to C++ Episode 3: Primitives and Pointers

"Primitive", i.e. built-in, data types.

## Char

Integral values that represent visual characters. E.g. `0x68 == 'h'`

How do you know `0x68` should represent `'h'`? It's all an interpretation via "character encodings" (ASCII, UTF-8, etc.).

Syntax:

```cpp
// single quotes, one letter only
char h = 'h';    // 0x68
char zero = '0'; // 0x30
char null = '\0' // 0x00, a.k.a. "null terminator"
```

> See more about how to [configure your compiler's character encoding](https://stackoverflow.com/questions/9739070/char-encoding).

## C-Strings

A sequence of chars that ends with a "null terminator".

```cpp
"hello"
'h',  'e',  'l',  'l',  'o'  ('\0')
0x68, 0x65, 0x6C, 0x6C, 0x6F, 0x00 // ASCII codes

// In this example...
// size = 5 bytes
// capacity = 6 or more bytes
```

> Note: C-strings are *not* objects with methods. They are *primitives* like an int, float, or bool.
> 
> GDScript/Python/C#/etc. implicitly convert c-string literals to String objects. "hello" -> String("hello")

Can also represent numbers, visually, with character encodings.

```cpp
// 1000
'1',  '0',  '0',  '0'  ('\0')
0x31, 0x30, 0x30, 0x30, 0x00

// vs. int
short x = 1000;

// 0x3130303000, 5 bytes, c-string, text
// 0x03E8,       2 bytes, short,    binary
```

This is why binary is smaller than text even before compression algorithms.

## Float/Double

Used for large, small, or partial values.

Store number with scientific notation.

```cpp
// value     sign  significand  exponent
  -3.14   == -1   *   314    *   10^-2
```

Actually uses base 2, not base 10.

Bit Distrubution:

|Type    |Sign|Exponent|Significand|Total
|--------|----|--------|-----------|----|
|`float` |   1|8|23|32|
|`double`|   1|11|52|64|

> [See more about floating-point precision format.](https://floating-point-gui.de/formats/fp/) Too complex for detailing here.

Note the *massive* range difference compared to integers:

|Type      |Bits|Range|
|----------|----|-----|
|`int32_t` |  32|3.3\*10^-4 - 3.3\*10^4 (32,700)
|`float`   |  32|1.2\*10^-38 - 3.4\*10^38 (a lot)
|`int64_t` |  64|2.1\*10^-9 - 2.1\*10^9 (2 billion)
|`double`  |  64|2.2\*10^-308 - 1.8\*10^308 (a lot, a lot)

Exponent flags for special numberline values:

> Exponent: 0x00... == +0 or -0
> 
> Exponent: 0xFF... == +INF, -INF, NaN

The cost? Loss of bit precision.

Can directly compare integers reliably.
`1 + 2 = 3 == 3`

Float/Double are inherently imprecise.
`1.0 + 2.0 = 3.00000000000000004 != 3.0`

Arithmetic -> truncation/overflow/underflow. [Compare 2 values against a threshold value (the "epsilon")](https://godotengine.org/qa/16522/problem-comparing-floats), e.g. `abs(x)-abs(y) < 0.0001`

## References

References are a compile-time search-and-replace mechanism.

Use `&` on a data type to declare it.

```cpp
            // address value
2;          // ------, 0x00000002: literal
int a = 2;  // 450130, 0x00000002: init literal
int b = a;  // 450134, 0x00000002: copy value
int &c = a; // ------, ----------: a compile-time reference
int d = c;  // 450138, 0x00000002: copy value @ `a`
```

The value *must* exist ahead of time. A reference-typed symbol must know how to replace itself with some *concrete addressable value*.

```cpp
int &x = 100; // Error! 100 is not an addressable expression.
int &y; // Error! Declaration does not reference anything.
```

References cannot be mutated themselves.

```cpp
int a = 2;
int b = 5;
int &c = a; // Define `c` referencing `a`.

// Set `a` to value of `b`: 5
// Does NOT mutate `c` to reference `b` instead.
c = b;
```

Similar to an OS "hard link". Two files linked to same data.

No equivalent in GDScript.

> "pass by value" and "pass by reference" = different, but related, concept.

## Pointers

Pointer = integral values storing a memory address to a specific data type.

Pointers are *technically* `uint32_t` or `uint64_y` depending on the architecture.

### **Pointer Syntax**

```cpp
int x = 2;   // 450130, value = 2
int *y = &x; // 450134, value = 450130
```

`*` = pointer (incomplete type).

`int*` = pointer to an int (complete type).

`&` on *expression* (`&(expr)`), not *declaration* (`type&`) = `address_of` operator.

Cannot cast between *primitive* pointer types.

```cpp
int x = 2;
float *y = &x; // Error! Cannot convert `int*` to `float*`
int *xp = &x; // Works!
float *fp = xp; // Error! Cannot convert `int*` to `float*`
```

> There *are* ways to manually copy memory from one data type to another, but this results in [undefined behavior](https://stackoverflow.com/questions/14730896/where-does-the-c-standard-describe-the-casting-of-pointers-to-primitives), so don't do it.

Can nest pointers:

```cpp
int x = 2;
int *y = &x;  // can only point to an `int`
int **z = &y; // can only point to an `int*`
```

Placement of asterisk is driven by convention.

```cpp
int *y = &x;
int* y = &x; // ~equivalent
```

Multi-variable declarations in C++ exist.

```cpp
int x, y, z; // declare 3 uninitialized variables
```

Placement of asterisk affects data type in multi-variable declarations.

```cpp
int* x, y;  // x = int*, y = int
int *x, *y; // x = int*, y = int*
int* x,* y; // same, but poor syntax
```

### **Pointer vs. Reference**

Unlike references, they are variables with memory and a distinct value. Therefore you can...

```cpp
int x;
int a;
int *z; // Declare with no initialization
z = &x; // Initialize later
z = &a; // Mutate value (change what it points to)
```

Access value:

Reference = use directly.

Pointer = *dereference* pointer with `*` operator applied to expression.

```cpp
int x = 2;   // 450130, value = 2
int &y = x;  // ------, ---------
int z = y;   // 450134, value = 2, value copied from `x`
int *a = &x; // 450138, value = 450130
int b = *a;  // 450146, value = 2, load from 450130 (no idea what `x` is)
```

### **Pointer Arithmetic**

Pointers are *integral* values, so you can perform arithmetic with them.

```cpp
char *s = "hello";
char h = *(s+sizeof(char)*0);
char e = *(s+sizeof(char)*1);
char l = *(s+sizeof(char)*2);
char l = *(s+sizeof(char)*3);
char o = *(s+sizeof(char)*4);
```

Pointers have a keyword expression for the 0 address:

```cpp
int *x = nullptr; // value = 0x0000000000000000 (64-bit)
int *y = 0; // signed int implicit cast to int*, zero-filled.
```

> Equivalent to GDScript's `null`.

Because pointer values are mutable via arithmetic/assignment, they are potentially unsafe (can access incorrect memory, get runtime exception). Therefore...

Reference values are **always safe**.

Pointer values are **always potentially unsafe**.

- pointer is mutated to be null.

    ```cpp
    int *xp = nullptr;
    ```

- pointer is mutated to point to other non-null, but invalid location.

    ```cpp
    int *xp = 2;
    ```

- the memory pointed to has been freed (learn about this later).
    
    ```cpp
    int *xp;
    {
        int x = 2;
        xp = &x;
    }
    int y = *xp; // `x` has been unloaded
    ```

Dereferencing a pointer in any of these scenarios creates a runtime exception that crashes the program.

### **Pointer Operators: `.` / `->` / `*`**

Different usage with classes.

```cpp
// Define class
struct SomeClass {
    int x = 2;
};

// Create local instance of class called `sc`.
SomeClass sc;

// `.` operator to get property/method of instance.
int y = sc.x; // value = 2

SomeClass *ptr = &sc;

// Order of operations! *(expr) and &(expr) have low priority!
int c = *ptr.x // Error! Cannot apply `.` to pointer. Use `->` instead.
int c = *ptr->x // Error! Cannot apply `*` to non-pointer type (int).
int c = (*ptr).x // Works!
int c = ptr->x; // Same!

// How does the compiler know the stored address is legit? Could be `nullptr`.
SomeClass &sc_ref = ptr;   // Error! Can't convert SomeClass* to SomeClass&.
SomeClass &sc_ref = *ptr;  // Safe!
SomeClass* &scp_ref = ptr; // Safe! But for pointer.

// Dereferencing `nullptr` is bad!
ptr = nullptr;
int d = *ptr;     // Error! Segmentation fault. Cannot access.
int d = (*ptr).x; // Error! Segmentation fault. Cannot access.
int d = ptr->x;   // Error! Segmentation fault. Cannot access.
int d = sc_ref.x  // 100% safe
```

### Bool

In GDScript, many non-zero things will implicitly cast to a `false` bool, including...

- An empty Data Structure:
    - `[]`
    - `{}`
    - `PoolByteArray()`
- A zero-filled numeric structure...
    - `Vector2()`
    - `Vector3()`
    - `Color(0)`
        - `Color()` is actually `Color(0, 0, 0, 1)`, so `true`.
- An empty string: `""`

These are called `falsy` values. Not truly `false`, but conceptually "empty" in practice. Common in dynamically-typed languages (JS, PHP, GDScript).

```gdscript
# Implicitly casts to bool with `false` value
var arr = []
if not arr:
    print("arr is empty")
```

However, their type information is retained if compared to other types.

```gdscript
var arr = []
var dict = {}

if not arr and arr == dict:
    # This does NOT work.
    # An empty Array is not equal to an empty Dictionary
    print("Both are empty")

if not arr and bool(arr) == bool(dict):
    # This DOES work
    # The boolean representations of these two are equal
    print("Both are empty")
```

In C++, `false` is `0x00`, i.e. all zero bits. `true` is any non-zero number, including negatives.

And in C++, *everything* is a number, so each data type has its own `0` value.

|Type|Value|
|----|-----|
|`bool`|`false`
|`int`|`0`
|`char`|`'\0'`
|`float`/`double`|`0.0`
|`pointer (*)`|`nullptr` or `NULL` before C++11
|`reference (&)`| N/A

C++ Classes, by default, have little-to-no integration with primitives.

```cpp
// Create local instances of objects.
Array arr;
Dictionary dict;
PoolByteArray pbarr;
Vector2 v2;
Vector3 v3;
Color c;

// Errors everywhere!
// None of these know how to implicitly cast to bool.
// Need to explicitly teach each type how to do it.
bool anyNonEmpty = arr || dict || pbarr || v2 || v3 || c;

// No errors.
// Classes have methods to return size.
// Structs are taught how to compare field-by-field.
bool anyNonEmpty = arr.size() || dict.size() || pbarr.size() || 
                   v2 == Vector2(0, 0) ||
                   v3 == Vector3(0, 0, 0) ||
                   C == Color(0);

// This works because if sizes are 0, then they are == `false`, 0x00.
```

`0x00` strictness means C++ behavior can be unexpected compared to dynamic languages.

```cpp
int x = 0;
int *xptr = &x;

(bool)*xptr; // zero value, therefore `false`
(bool)xptr;  // non-zero memory address, therefore `true`

char *s = ""; // an empty c-string

(bool)*s; // deref and cast. Value @ s == '\0'. 0x00. false
(bool)s;  // s == non-zero memory address of literal. true

class A {};
A a;
A *ap = &a;
(bool)a; // Error! `A` does not know how to cast to bool.
(bool)ap; // non-zero memory address. therefore `true`
```
