# GDScript to C++ Episode 5: Control Flow

## Conditionals

In GDScript:

```gdscript
var x := 0
if x == 10:
    # do A
elif x == 15:
    # do B
elif x: # implicitly cast to bool. 0 -> false.
    # do C
else:
    # do D
```

In C++:

```cpp
int x = 0;
if (x == 10) {
    // do A
} else if (x == 15) {
    // do B
} else if (x) { // implicitly cast to bool. 0 -> false.
    // do C
} else {
    // do D
}
```

- `elif` -> `else if`
- Parentheses (`()`) around expressions
- For blocks, curly brances (`{}`) are required. C++ does not rely on indentation.

Another supported style (standard for C#, Java, etc.):

```cpp
if (...)
{
    // do A
}
else if (...)
{
    // do B
}
else
{
    // do C
}
```

Coding convention decides.

Syntax also supports just statements, no `{}` blocks.

```cpp
if (true)
    print_line("hello");
```

However, these are often considered a code smell.

```cpp
if (false)
    print_line("hello ");
    print_line("world!");
// same as
if (false)
    print_line("hello ");
print_line("world!");
// same as
if (false) {
    print_line("hello ");
}
print_line("world!");
```

Also, if have `{}` already, no need to *add* them for later edits.

Statements (`;`) force a single line of code. Blocks (`{}`) give flexibility for 0+ lines of code.

> So, why then do class declarations need `{}` *and* `;`?
> 
> Actually, they can post-declare instance symbols.
> 
> ```cpp
> class A {} a;
> // equal to...
> class A {};
> A a;
> ```

🤮 Blegh! Who's gonna pay attention to that?

Can also inline statements. Even more nasty.

```cpp
if (some_test()) do_something();
else if (1 == 2) do_something_else();
else otherwise_do_this();
```

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

Main Differences Recap:

- C++: `else if`

    GDScript: `elif`

- C++: `()` required for expression.

    GDScript: `()` optional/unconventional.

- C++: `{}` recommended for block. No `{}` = only 1 statement (`;`) (BAD).

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