# GDScript to C++ Episode 4: Scopes, Control Flow, and Functions

What's a scope? Shadowing?

- `{}`

How to conditionally execute code?

- `if`, `else if`, `else`
- `switch`, `case`, `default`

How to re-execute existing code? 

- `while` loop
- `do-while` loop
- `for` loop

How to re-execute existing code *with slight changes*?

- functions

## Scopes

`{ ... }` == new "scope".

"scope" == layer of symbols.

GDScript:

Must use conditional/loop/function/classs to create scope.

```gdscript
var x = true
if true:
    var y = true

    # both exist

# only x exists
```

C++:

Only requires `{}`.

```cpp
bool x = true;
{
    bool y = true;

    // both exist
}
// only x exists
```

## Shadowing

Redeclare a variable in a new scope. For scope duration, it "shadows" or hides the original.

Not supported in GDScript:

```gdscript
var x = true
if true:
    var x = 2 # Error! Cannot redeclare `x`
```

Supported in C++:

```cpp
bool x = true;
{
    int x = 2; // Works! `bool x` doesn't exist now!
}
```

Still not supported *within same scope*.

```cpp
bool x = true;
float x = 2.0; // Error! Conflicting declaration `x`.
bool x = false; // Error! Redeclaration of `bool x`!
```

> Some languages, like F# or Rust, support this.

## If Statements

In GDScript:

```gdscript
var x = true
if x:
    # do A
elif x == 2: # compares bool(true) against int(2)
    # do B
else:
    # do C
```

C++:

```cpp
bool x = true;
if (x) {
    // do A
} else if (x == 2) { // 2 implicitly cast to bool
    // do B
} else {
    // do C
}
```

Alternative syntax (placement of curly braces):

```cpp
bool x = true;
if (x)
{
    // do A statements
}
else if (x == 2)
{
    // do B statements
}
else
{
    // do C statements
}
```

Yet another shorthand syntax (no fluff):

```cpp
bool x = true;
if (x)
    // do A;
else if (x == 2)
    // do B;
else
    // do C;
```

1 style per project convention, for consistency/readability.

Main differences:

- C++: `else if`

    GDScript: `elif`

- C++: `()` optional

    GDScript: `()` optional/unconventional.

- C++: `{}` required for body.

    GDScript: indented body.

## Switch

GDScript:

```gdscript
const SomeType = preload("some_type.gd")
var obj = SomeType.new()
var value = obj.some_method()

# Who knows what the heck this is? We don't know.
match value:

    # Match multiple things with one block
    [], {}, PoolByteArray(), PoolIntArray():
        print("value is some empty container")

    15:
        print("value is `int(15)`")

    "hello":

        print("value is `str('hello')`")

    # Matches all remaining cases
    _:
        print("value was something else")
```

C++:

```cpp
class SomeClass {};
SomeClass sc;
int value = sc.some_method(); // call function in sc

switch (value) {
    case 0:
        // `value` is `0`

        int local_var = 1;

        // explicitly terminate block
        break;
    
    case 1:
    case 2:
    case 3:
        // "fallthrough" mechanic
        // `value` is `1`, `2`, or `3`

        int local_var = 1; // Error! Redeclaring var!

        break;

    default: {

        int local_var = 1; // safe! new scope
        
    } break;

}
```


```gdscript
var sentence = "Hello World Will"
var words = sentence.split(" ")
var word = words[0]

# If statements
# Distinct evaluation per expression
if word:
    print("Non-empty string!")
elif word == "Hello":
    print("First word is \"Hello\"")
else:
    print("First word is not empty or \"Hello\"")

# match statement
# multiple comparisons to one expression
match word:
    "Hello":
        print("First word = \"Hello\"")
    "World":
        print("First word = \"World\"")
    "Will":
        print("First word = \"Will\"")
    
    # Supports comparing other types.
    # No compile-time error.
    15:
        print("First word was not the number 15")
    
    []:
        print("First word was not an empty array")

# Actually makes for very flexible APIs
var obj = SomeType.new()
var value = obj.some_method()

# Who knows what the heck this is? We don't know.
match value:

    # Match multiple things with one block
    [], {}, PoolByteArray(), PoolIntArray():
        print("value is some empty container")

    # Matches all remaining cases
    _:
        print("value was something else")
```

In C++, you have some similar constructs.

Syntax:

```cpp
if (bool_expression) {
    // do if true
}
```

```cpp
int x = 10;
int y = 20;

if ()
```