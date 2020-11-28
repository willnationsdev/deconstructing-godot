# GDScript to C++ Episode N: Unions and Variant

## Union

How does GDScript's dynamic typing work?

C++:

```cpp
// Does not change type of `x` to `bool`.
// Performs `bool` to `int` cast.
int x = 2;
x = true;  // 1, still an `int`
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
    bool b; // 1 byte
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