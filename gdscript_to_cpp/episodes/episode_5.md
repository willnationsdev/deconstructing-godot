## Conditionals

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

Yet another shorthand syntax (not recommended):

```cpp
bool x = true;
if (x)
    // do A;
else if (x == 2)
    // do B;
else
    // do C;
```

Above syntax == only 1 statement.

```cpp
if (true)
    bool x = true;
    bool y = false;
// equivalent to...
if (true)
    bool x = true;
bool y = false;
```

> Remember when I said you usually have *either* `{}` OR `;`? 1+ vs 1 statement.
> 
> Truth be told, class declarations actually support this:
> 
> ```cpp
> class A {} a;
> // equal to...
> class A {};
> A a;
> ```
> 
> \^ Reason for both `{}` and `;` with class declaration.

Can also inline statement. Not recommended.

```cpp
if (true) bool x = true;
```

1 style per project convention, for consistency/readability.

Main differences:

- C++: `else if`

    GDScript: `elif`

- C++: `()` required for expression.

    GDScript: `()` optional/unconventional.

- C++: `{}` recommended for block. No `{}` = only 1 statement (`;`).

    GDScript: indent after colon (`:`) required. Block ends with dedent.

Both languages also support ternary operators.

GDScript:

```gdscript
# <true> if <expression> else <false>
"true" if 1 == 1 else "false"
```

C++:

```cpp
// <expression> ? <true> : <false>
1 == 1 ? "true" : "false"
```

## Switch

Compare many values against the same expression.

GDScript:

```gdscript
var value = <expression>
match value:

    # Match multiple things with one block.
    [], {}, PoolByteArray(), PoolIntArray():
        print("value is some empty container")

    15:
        print("value is `int(15)`")

    "hello":

        print("value is `str('hello')`")

    # `_` matches all remaining cases.
    _:
        print("value was something else")
```

In GDScript, you can even explicitly check for different data types, not just different values of data types.

```gdscript
var value = <expression>
match typeof(value):
    TYPE_NIL:
        pass # logic if null
    TYPE_ARRAY:
        pass # logic if [ ... ]
    TYPE_DICTIONARY:
        pass # logic if { ... }
    _:
        pass # etc.
```

C++:

The `expression` must be an *integral* value.

```cpp
int value = <expression>;
// Supported: bool, char, short, int, long, long long
// Not supported: float, X, ""
// Can cast pointers to `size_t` / `uint64_t`

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

## Loops: While / Do-While

Repeatedly execute a block. Evaluate a bool expression to determine (re)entry.

GDScript:

```gdscript
while true:
    print("hi") # infinite loop
```

State management occurs separately.

```gdscript
var i = 0        # set iterator
var size = 10    # set "base case"
while i < 10:    # eval itr. At base case to quit?
    var x = i    # logic body
    i += 1       # advance towards base case
```

Equivalent in C++ with identical caveats of if statements:

- Require `()` around expression.
- Replace `:` indented block with `{}`.
- Can replace `{}` with single statement `;`.

```cpp
while (true) {
    print_line("hi"); // Godot print, infinite loop
}

// State management
int i = 0;          // set iterator
int size = 10;      // set "base case"
while (i < size) {  // eval itr. At base case to quit?
    int x = i;      // logic body
    i++;            // advance towards base case
}
```

Need to change to "do logic, advance, THEN eval itr?"

Do-While Loop:

```cpp
do {
    int x = i;      // logic body
    i++;            // advance towards base case
} while (i < size); // eval itr. At base to quit?
```

`;` after while eval because must conclude statement. Didn't already end with curly braces.

C++ also has `for` loop. Bottles up state management into local/consistent format:

```cpp
// syntax:
// for (initializers; eval expression; advance) {
//     logic body
// }
// initializers: Executed once at start.
//               Symbols locally scoped.
// eval expr   : executed before every body exec.
//             : only enter body if `true`.
// advance     : executed at end of every body exec.

// Example:
for (int i = 0, int size = 10; i < size; i++) {
    int x = i;
}
// `i` and `size` are scoped to for loop. Don't exist.

// More commonly seen like this:
for (int i = 0; i < 10; i++) {

}
```

GDScript *does* have a `for` loop, but no state management. Iterates through data structure (aka "for-each" loop).

```gdscript
var numbers = [0, 1, 2, 3]
for i in numbers:
    print(i)
# equivalent
for i in 4: # implicit array expansion, 4 elements
    print(i)
```

C++ added "range-based for loops". Collection class must support it.

This isn't legit, but something like this:

```cpp
CollectionOfInts coi;
for (int elem : coi) {

}
```
