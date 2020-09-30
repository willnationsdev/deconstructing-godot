# GDScript to C++ Episode 2: Manipulating Memory

Programs write data to memory addresses.

```cpp
int x; // arbitrary memory address, e.g. 450130
```

How to reference?

Map symbol to location via "symbol table".

Symbol tables record variable locations for later reference.

In GDScript, this would be like a Dictionary of strings mapped to integers of array indices.

```gdscript
var symbols = {}
var values = PoolByteArray()

# int x = 2; // C++
symbols['x'] = 450130
values[450130] = 0
values[450131] = 0
values[450132] = 0
values[450133] = 2
```

But, *how* do we write data?

**Data Type** determines read/write algorithm.

## Data Type: Byte Size & Arrangement

*How many bytes ("byte size")?*

Get size with `sizeof(type)` operator.

```cpp
// `int` is 4 bytes.
sizeof(int) == 4; // true
```

Program views memory as sequentially ordered:

```cpp
int x; // 450130-450133
int y; // 450134-450137 (+ 4)
```

Ways to express integer value:

```cpp
// decimal
// Syntax: None
// Values: 0-9
// 1 byte: N/A
int x = 1;

// binary
// Syntax: `b` suffix
// Values: 0-1
// 1 byte: 8 digits
int x = 10110110111100001111111101010101b;

// octal
// Syntax: `0` prefix
// Values: 0-7
// 1 byte: 4 digits
int x = 00123456701234567;

// hexadecimal
// Syntax: `0x` prefix
// Values: 0-9, A-F (0-15)
// 1 byte: 2 digits
int x = 0x02468ACE;
```

*How are the bits arranged?*

```cpp
// `int` bit arrangement? "Two's Complement"
// 1. Leading bit == sign.
// 2. Remaining bits == value.
int sx = 0;  // 0x00000000
int sy = 1;  // 0x00000001
int sz = -1; // 0xFFFFFFFF

// Negated value == flip bits + 1

// pos -> neg
// -0x00000001 + 1
//  0xFFFFFFFE + 1
//  0xFFFFFFFF

// neg -> pos
// -0xFFFFFFFF + 1
//  0x00000000 + 1
//  0x00000001
```

## Data Types: Interpreting Bits

Data type controls data *interpretation*.

"casting" == converting type A to type B.

In GDScript:

```gdscript
var x = "1"
x = int(x) # cast value to int
x = str(x) # cast value back to string
x = str(int("1")) # nested casts
```

In C++, same, but the data type controls interpretation of *bits*.

```cpp
unsigned int ux = 0;  // 0x00000000
unsigned int uy = 1;  // 0x00000001
unsigned int uz = -1; // 0xFFFFFFFF, becomes 4294967295
```

> Note: `int` is `signed int` by default. Literals default to `int`.

```cpp
// unsigned int <- signed int
unsigned int uz = -1;
```

Static Cast: *compile-time* conversion. The code *won't compile* if incompatible.

Implicit static cast:

```cpp
// unsigned int <- signed int
unsigned int uz = -1;
```

Explicit static cast:

```cpp
// unsigned int <- (unsigned int <- signed int)
unsigned int uz = (unsigned int) -1;
```

Notice:

- GDScript: Calls global function and passes expression as parameter.
- C++: Wrap the *type* in parentheses. Order of operations applies type to expression.

Wrap in parentheses to nest casts.

```cpp
int nested = (int) ((unsigned int) -1);
int nested = (int) 4294967295;
int nested = -1;
```

Other ways to manipulate data type?

## Data Types: Byte Size Variation

Changing byte size...

```cpp
char c; // 1-byte signed integer (>= 8 bits)
short s; // 2-byte signed integer (>= 16 bits)
int x; // 4-byte signed integer (~32 bits, >= 16 bits)
long x2; // 4-byte signed integer (>= 32 bits)
long long y; // 8-byte signed integer (>= 64 bits)
```

Exact size depends on compiler implementation.

> [See more about int sizes per platform](https://stackoverflow.com/a/589685).
> 
> [See more about all types' size variance](https://en.cppreference.com/w/cpp/language/types).

Using sizes in code: always use `sizeof(type)`, never a literal.

GDScript == only 64-bit signed integers. Why?

![Don't need to calculate integer size if you always use the biggest one](img/calculate_integer_size_biggest.png)

So, how does byte size affect casting?

```cpp
// size: small -> big
char c = 1; // 0x01
int x = c;  // 0x00000001, zero-filled

// size: big -> small
int x = 256; // 0x00000100
char c = x;   // 0x00, truncated: 0x000001|00
```

How does the byte size reality affect data? Underflow and overflow:

```cpp
// Underflow
unsigned int x = 0; // 0x00000000
x = x - 1; // 0xFFFFFFFF

// Overflow
int y = INT_MAX; // ~ 32k or ~ 2b
y = y + 1;       // ~ -32k or ~ -2b
```

> Note: alternate, explicit names for byte sizes and signage of integers exist.
> 
> ```cpp
> int8_t  // signed
> int16_t
> int32_t
> int64_t
> uint8_t // unsigned
> uint16_t
> uint32_t
> uint64_t
>
> size_t  // unsigned int on native platform
> ```

## Data Types: Primitives

"Primitive" data types:

- Built-in.
- Small enough to fit in one CPU register (no more than 8 bytes)

`int`, `unsigned int`, etc. are primitives + many more...

### Char

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

A sequence of characters, represented with numbers, which ends when the value `0x00` is encountered.

```cpp
"hello"
'h',  'e',  'l',  'l',  'o'  ('\0')
0x68, 0x65, 0x6C, 0x6C, 0x6F, 0x00 // ASCII for example

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

// 5 bytes vs. 2 bytes!
```

### Float/Double

Used for targeting very large or small numbers with only a few bits.

Idea:

```cpp
-314 == -1 * 3.14 * 10^2
// 1 bit for "sign".
// some bits for moving decimal point, "exponent".
// remaining bits for value, "significand" or "mantissa".
// Use base 2, not base 10.
```

Float: sign=1, exp=8, sig=23: 32 bits

Double: sign=1, exp=11, sig=52: 64 bits

```cpp
float f = -3.14;
// sign|    exp|                    sig|
//  -1|       2|                    314|
//  -1| 128-127|                      3|
//   1|10000000|10010001111010111000011|
//   1100|0000|0100|1000|1111|0101|1100|0011
// 0xC    0    4    8    F    5    C    3
// 0xC048F5C3

```

> [See more about the algorithm](https://matthew-brett.github.io/teaching/floating_point.html). Too complex for detailing here.

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
int &c = a; // 450134, ----------: a compile-time reference
int d = c;  // 450138, 0x00000002: copy value @ `a`
```

The value *must* exist ahead of time.

```cpp
int &x = 100; // Error! 100 is not an addressable value.
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

`&` on *expression*, not *declaration*, = "get address" operator.

Cannot mix *primitive* pointer types (cover non-primitives later).

```cpp
int x = 2;
float *y = &x; // Error! Cannot convert `int*` to `float*`
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
SomeClass &sc2 = ptr; 

SomeClass &sc2 = *ptr; // safe!

// Beware dereferencing nullptr.
ptr = nullptr;
int c = *ptr; // Error! Segmentation fault. Cannot access.
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

## GDScript, Variant, and Union

Required: C++ knows location/size at compile time.

How does GDScript work then?
