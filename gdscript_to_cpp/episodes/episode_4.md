# GDScript to C++ Episode 4: Scopes, Control Flow, and Functions

How to manipulate *code flow*?

## Scopes

`{ ... }` == new "scope".

"scope" == layer of symbols.

GDScript:

Create with...

- conditionals
- loops
- functions
- inner classes

> Anything that requires a colon (`:`).

```gdscript
var x = true
if true:
    var y = true

    # both exist

# only x exists
```

C++:

Only requires `{}`. No conditional (or whatever) necessary.

```cpp
bool x = true;
{
    bool y = true;

    // both exist
}
// only x exists
```

> Some edge cases don't require `{}`, but bad practice not to use them.

## Shadowing

Redeclare a variable in a new scope. For scope duration, it "shadows" or hides the original symbol.

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

    // Also would've worked. Data type doesn't matter.
    bool x = false; 
}
```

Still not supported *within same scope*.

```cpp
bool x = true;
float x = 2.0; // Error! Conflicting declaration `x`.
bool x = false; // Error! Redeclaration of `bool x`!
```

> Some languages, like F# or Rust, support this.

```fsharp
// F#
let x = 10 // x == 10
let x = x + 10 // x == 20
let x = x + x // x == 40
```

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

C++:

The `expression` must be an *integral* value.

```cpp
int value = <expression>;
// Supported: bool, char, short, int
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

## Functions (Basic Syntax)

> A `function` is an input/output mechanism that abstracts a set of instructions.
> 
> A `method` is a function that belongs to a `class`.

GDScript methods have optional static typing.

```gdscript
# dynamic
func add(left, right):
    return left + right

# static
func add(left: int, right: int) -> int:
    return left + right

# no return value
func do_nothing():
    pass

func _ready():
    var x = add(3, 4) # 7
    var y = do_nothing() # null
```

In C++, statically typed. Not optional addendum at end of info. *Start* with the data type.

```cpp
int add(int left, int right) {
    return left + right;
}

// no return value, void 'type'
void do_nothing() {
}

// usage:
int x = add(3, 4);     // 7
int x = do_nothing();  // Error! Cannot assign `void`
void x = do_nothing(); // Error! `void` not a type.
do_nothing();          // Correct
```
