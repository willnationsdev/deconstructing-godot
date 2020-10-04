# GDScript to C++ Episode 3: Primitives and Pointers

"Primitive" data types:

- Built-in.
- Usually small enough to fit in one CPU register. No more than 4 or 8 bytes (32bit vs 64bit).

`int`, `unsigned int`, etc. are primitives + many more...

## Char

Used for low integral values. Also used for character encodings (ASCII, UTF8, etc.).

```cpp
char c; // 1 byte, range: -128-127
unsigned char c; // 1 byte, range: 0-256 (can b/c integral)
wchar_t wc; // 2 bytes, "wide char"
```

Also supports "character" values. Interpreted via "character encoding", e.g. ASCII, UTF-8, etc.

```cpp
char c = 0x68;
char h = 'h'; // single quotes, one letter only
c == h; // true
```

> [See more about how to configure the used character encoding](https://stackoverflow.com/questions/9739070/char-encoding).

Can be used for "C-strings":

A sequence of characters, represented with numbers, which ends when the value `0x00` is encountered. The `0x00` is expressed with `'\0'`, the "null terminator".

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
> GDScript/Python/C#/etc. = opposite. Implicitly convert literals to String objects. "hello" == String("hello")

Can also represent numbers, visually, with character encodings.

```cpp
// 1000
'1',  '0',  '0',  '0'  ('\0')
0x31, 0x30, 0x30, 0x30, 0x00

// vs. int
short x = 1000; // 0x03E8

// 0x3130303000, 5 bytes, c-string
// 0x03E8,       2 bytes, short
```

### Float/Double

Used for targeting very large or small numbers with only a few bits.

Idea:

Only store the number's scientific notation, e.g. `-3.14 * 10^2 == -314`.

```cpp
// value     sign  sinificand  exponent
   -314   == -1   *   3.14   *   10^2
```

Actually uses base 2, not base 10.

Float: sign=1, exp=8, sig=23: 32 bits ~= 1.2\*10^-38 - 3.4\*10^38

Double: sign=1, exp=11, sig=52: 64 bits ~= 2.2\*10^-308 - 1.8\*10^308

> [See more about floating-point precision format.](https://floating-point-gui.de/formats/fp/) Too complex for detailing here.

> Exponent: 0x00... == +0 or -0
> 
> Exponent: 0xFF... == +INF, -INF, NaN

### References

References are like a compile-time search-and-replace *exclusively* for variables.

They are their own data type which can be applied to any other data type using an `&` to mark it.

```cpp
2;          // ------, 0x00000002: literal
int a = 2;  // 450130, 0x00000002: init literal
int b = a;  // 450134, 0x00000002: copy value
int &c = a; // ------, ----------: a compile-time reference
int d = c;  // 450138, 0x00000002: copy value @ `a`
```

The value *must* exist ahead of time.

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

### Pointers

Pointer = integral, memory address to a specific data type.

```cpp
int x = 2;   // 450130, value = 2
int *y = &x; // 450134, value = 450130
```

`*` = pointer (incomplete type).

`int*` = pointer to an int (complete type).

`&` on *expression*, not *declaration*, = `address_of` operator.

Cannot mix *primitive* pointer types (cover non-primitives later).

```cpp
int x = 2;
float *y = &x; // Error! Cannot convert `int*` to `float*`
```

You can successfully *cast* a pointer of one type to another, but in most cases, this results in [undefined behavior](https://stackoverflow.com/questions/14730896/where-does-the-c-standard-describe-the-casting-of-pointers-to-primitives), so be careful.

```cpp
int i = 2;
int *ip = &i;
float *fp = ip; // Implicit cast
float f = *fp; // Undefined behavior! float bits -> int bits

// Non-pointer casting attempts to convert value safely.
float f2 = i; // 2.0
```

Can nest pointers (will cover later in "arrays" lesson).

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
```

Unlike references, they are variables with memory and a distinct value. Therefore...

...you can initialize them later

```cpp
int x;
int &y; // Error! Must initialize a reference
int *z; // Works
z = &x; // Works
```

...and re-assign them to new values.

```cpp
int x;
int y;
int *z = &x;
z = &y; // Works
```

Pointers have a keyword expression for the 0 address:

```cpp
int *x = nullptr; // value = 0x0000000000000000 (64-bit)
int *y = 0; // signed int implicit cast to int*, zero-filled.
```

> Equivalent to GDScript's `null`.

Reference = alt symbol for addressable value. Therefore, can use variable directly.

```cpp
int x = 2;  // 450130, value = 2
int &y = x; // ------, value = 2
int z = y;  // 450134, value = 2, value copied from `x`
```

Pointers store memory addresses though! Must *dereference* address (i.e. load address / go to the address).

```cpp
int x = 2;   // 450130, value = 2
int *y = &x; // 450134, value = 450130
int mem = y; // 450142, value = 450130
int z = *y;  // 450146, value = 2
```

`*` on *expression*, not *declaration*, = "dereference" operator.

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

// WRONG order of operations:
// Use `.` operator on `ptr`,
// get `x` symbol from it,
// and THEN try to dereference it.
int c = *ptr.x // Error! Cannot apply `.` to pointer.

// CORRECT order of operations:
// 1. Dereference
// 2. Parenthesize to forcibly reduce expression
// 3. THEN do `.` operator
int a = (*ptr).x;

// OR arrow syntax
int b = ptr->x;

// Remember, references require *actual* value.
// Error! Can't convert SomeClass* to SomeClass&.
SomeClass &sc_ref = ptr; 

SomeClass &sc_ref = *ptr; // safe!

// References use `.`, not `->` operator.
// `.` always safe.
// `*` and `->` potentially unsafe.
int c = sc_ref.x;

// Beware dereferencing nullptr.
ptr = nullptr;
int d = *ptr; // Error! Segmentation fault. Cannot access.
```

Pointers are *integral* values, so you can perform math operations with them.

```cpp
char *cstring = "hello";
char h = *(c+0);
char e = *(c+1);
char l = *(c+2);
char l = *(c+3);
char o = *(c+4);
```

`*` and `->` have low precedence. To use operators, often must wrap in `()` or call operator explicitly.

```cpp
class BigInteger { ... };
BigInteger bi;
BigInteger *ptr = &bi;

*ptr++; // Wrong! Mutated memaddress, THEN dereferenced `ptr`.
ptr->++; // Syntax error!
(*ptr)++; // Works! Use () to deref first, then increment. 
ptr->operator++(); // Works! Called op function directly.

// Cannot call explicit operators from primitives.
int x = 2;
int *y = &x;
(*y)++; // Works!
y->operator++(); // Error! Request operator++ on non-class.
```

### Bool

In GDScript, many non-zero things will implicitly cast to a `false` bool, including...

- An empty Array: `[]`
- An empty Dictionary: `{}`
- An empty Pool Array: `PoolByteArray()`
- A zero-filled numeric structure...
    - `Vector2()`
    - `Vector3()`
    - `Color(0)`
        - `Color()` is actually `Color(0, 0, 0, 1)`, so `true`.
- An empty string: `""`

These are called `falsy` values. Common in dynamically typed languages.

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

if arr == dict:
    # This does NOT work.
    # An empty Array is not equal to an empty Dictionary
    print("Both are empty")

if bool(arr) == bool(dict):
    # This DOES work
    # The boolean representations of these two are equal
    print("Both are empty")
```

In C++, `false` is `0x00`, i.e. all zero bits. Else `true`.

Each data type has its own potential representation of 0.

|Type|Value|
|----|-----|
|`bool`|`false`
|`int`|`0`
|`char`|`'\0'`
|`float`|`0.0`
|`pointer (*)`|`nullptr` or `NULL` before C++11

Classes, by default, have little-to-no integration with primitives.

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
// Need implement operator method to support it.
arr || dict || pbarr || v2 || v3 || c;
```

`0x00` strictness means the behavior in C++ is different than expected.

```cpp
int x = 2;
int *y = &x;

(bool)*y; // non-zero value, therefore `true`
(bool)y;  // non-zero memory address, therefore `true`

char *s = ""; // an empty c-string

(bool)*s; // deref and cast. Value @ s == '\0'. 0x00. false
(bool)s;  // s == non-zero memory address of literal. true

class A {};
A a;
A *ap = &a;
(bool)a; // Error! `A` does not know how to cast to bool.
(bool)ap; // non-zero memory address. therefore `true`
```

## Union

How does GDScript's dynamic typing work?

C++:

```cpp
// Does not change type of `x` to `bool`.
// Performs `bool` to `int` cast.
int x = 2;
x = true; // 1, still an `int`
x = false; // 0, still an `int`
```

GDScript:

```gdscript
var x = 2
x = true # `x` is now a bool
```

In C++, this behavior is accomplished using a `union`.

```cpp
struct BoolAndYup {
    bool as_bool; // 1 byte
    char y; // 1 byte
    char u; // 1 byte
    char p; // 1 byte
};

// Combines each field into same byte region.
union MyUnion {
    int as_int;              // 4 bytes
    BoolAndYup as_bay;       // 4 bytes
    float as_float;          // 4 bytes
};

// 4 bytes
MyUnion mu;
mu.as_int;   // interpret bytes as int
mu.as_float; // interpret bytes as float
mu.as_bay;   // interpret bytes as BoolAndYup
```

Can't tell what type `mu` is at runtime...hmmm.

Use an enum!

```cpp
enum MyUnionType {
    Int,
    Float,
    Bay
}

// Make the following change to MyUnion
struct MyUnionData {
    int as_int;              // 4 bytes
    BoolAndYup as_bay;       // 4 bytes
    float as_float;          // 4 bytes
};

union MyUnion {
    MyUnionData data;        // 4 bytes
    MyUnionType type;        // 4 bytes
};

// use as int
mu.data.as_int = 2;
mu.type = Int;

mu.type == Int; // true
```

Now we can check what "type" it is based on the enum, and access the correct field.