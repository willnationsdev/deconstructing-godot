# GDScript to C++ Episode 5: Control Flow

## Conditionals

In GDScript:

All variables are implicitly wrapped in a C++ Variant object that understands some type comparisons.

```gdscript
var x = true
if x:        # true (obviously)
    # do A
elif 2:      # non-zero, true
    # do B
elif x == 2: # bool(x) == int(2), int is non-zero, true
    # do C
elif 5 == 2: # int(5) == int(2), false
    # do D
               # Variant doesn't know how to compare these two types.
elif "2" == 2: # str(2) == int(2), Error! Invalid operands for `==`.
    # do E
elif "2" == "2": # str(2) == str(2), Strings have the same content: true
    # do F
else:
    # do G
```

C++:

There is no built-in Variant wrapper. However, everything is a number. A `bool` always evaluates, as a number, to either `0` (false) or `1` (true).

```cpp
bool x = true;
if (x) {                 // bool value of true: true
    // do A
} else if (2) {          // implicitly casts int to bool. Non-zero: true
    // do B
} else if (x == 2) {     // implicitly casts bool to int. 1 == 2: false
    // do C
} else if (5 == 2) {     // no casting. int values not equal: false
    // do D
} else if ("2" == 2) {   // Error! Cannot compare pointer (char*) with int.
    // do E
} else if ("2" == "2") { // false! Wuuuuut? Cover next lesson
    // do F
} else {
    // do G
}
```

Alternative syntax (placement of curly braces):

> This is the standard for C#.

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

> How to check if pointer is valid? Now we know!
> 
> ```cpp
> Node *child = get_node("Child");
> if (child) {
>     // child is true
>     // child is non-zero
>     // child != 0
>     // child != nullptr
>     print_line(child->get_name()); // Safe! Prints "Child"
> }
> ```

Main differences:

- C++: `else if`

    GDScript: `elif`

- C++: `()` required for expression.

    GDScript: `()` optional/unconventional.

- C++: `{}` recommended for block. No `{}` = only 1 statement (`;`).

    GDScript: indent after colon (`:`) required. Always a block. Block ends with dedent.

## Ternary Operator

Both languages also support ternary operators.

GDScript:

```gdscript
# <true-case> if <expression> else <false-case>
# cases can have different types
print("true" if x == y else ["true"])
```

C++:

```cpp
// <expression> ? <true-case> : <false-case>
// Cases required to be the same type
print_line(x == y ? "true" : "false")
```

Also, it is possible to [nest ternary operators](https://stackoverflow.com/questions/18237432/how-to-rewrite-complicated-lines-of-c-code-nested-ternary-operator).

```cpp
good = m_seedsfilter==0 ? true :
       m_seedsfilter==1 ? newClusters(Sp) :
                          newSeed(Sp);
```

This one is relatively clean and simple. Might be able to get away with it.

But please, don't be this person ([source](https://www.geeksforgeeks.org/c-nested-ternary-operator/)):

```cpp
int a = 2 > 3 ? 2 : 3 > 4 ? 3 : 4; 
```

Usually Godot devs don't like nested ternaries (for readability).

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

In GDScript, you can even explicitly check for different data types, not just different values.

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
// Not supported: float, class/struct instance, *
// Can cast pointers to integral...but why? Either nullptr or some address.
int outside = 0;

switch (value) {
    case 0:
        // `value` is `0`

        int local_var = 1; // Error! Jump to case label crosses init of var.

        // explicitly terminate block
        break;

        // Can mutate declarations outside of switch block.
        // "fallthrough" / "cascade" mechanic if no `break`
    case 1: // `value` is `1`
        outside += 1;
    case 2: // `value` is `1` or `2`
        outside += 1;
    case 3: // `value` is `1`, `2`, or `3`
        outside += 1

        break;

    default: {

        int local_var = 5; // safe! new scope
        outside = local_var;
        
    } break;
}
```

## Loops: While

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
- Can replace `{}` with single statement `;` (not recommended).

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

> The `++` operator is new. That's the "post-increment" operator.
> It returns the current value and then adds 1 to the variable.
> 
> ```cpp
> int x = 5;
> int a = x++; // 5
> int b = x++; // 6
>              // x = 7
> ```
> 
> There's also the "pre-increment", e.g. `++x`, that increases by 1 and *then* returns.
> Decrement versions also exist: `x--` and `--x`
> 
> C++ is the "next iteration" of `c` -> `c++`
> 
> C# is the "next iteration" of `c++` -> `c++++` -> `c#` (4 plus signs)

## Loops: Do-While

Need to change to "do logic, advance, THEN eval itr?"

Do-While Loop:

```cpp
do {
    int x = i;      // logic body
    i++;            // advance towards base case
} while (i < size); // eval itr. At base to quit?
```

`;` after while eval because must conclude statement. Didn't already end with curly braces.

## Loops: For

C++ also has a `for` loop. Bottles up state management into local/consistent format:

```cpp
// syntax:
// for (initializers...; eval expression; advance) {
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

// Equivalent to a `while(true) {}` loop
for (; true;) {

}
```

GDScript also has a `for` loop.

```gdscript
for i in 3:
    print(i) # prints 0, 1, 2
```

However, the number is actually a shortcut for `range(3)`.

```gdscript
for i in 3:
    print(i)
# same as
for i in range(3):
    print(i)
# same as
for i in [0, 1, 2]:
    print(i)
```

As such, GDScript's `for` loop is actually a `for-in` loop, aka a `foreach` loop in C#/PHP/others.

It iterates through every element of a collection with no state management.

```gdscript
var numbers = [0, 1, 2, 3]
for i in numbers:
    print(i)
# equivalent
for i in 4: # implicit array expansion, 4 elements
    print(i)
```

C++11 added "range-based for loops". Collection class must support it.

As of writing, Godot's collections **do not** support them. But if they did...

```cpp
Array arr;
for (Variant elem : arr) {

}
```

Instead, you have to use a for loop:

```cpp
Array arr;
for (int i = 0; i < arr.size(); i++) {

}
```