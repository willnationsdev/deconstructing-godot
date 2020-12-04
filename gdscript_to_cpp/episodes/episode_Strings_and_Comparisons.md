# GDScript to C++ Episode 6: Strings and Comparisons

## Object Comparisons

In GDScript, you can compare objects.

```gdscript
var obj1 = Object.new()
var obj2 = Object.new()
if obj1 == obj2:
    pass # always false. But why?
```

Because the Variant class, when dealing with Objects, deals with an `Object*` internally.

Similar to the C++ code:

```cpp
Object *obj1 = memnew(Object); // Object instance at memory address 450130
Object *obj2 = memnew(Object); // Object instance at memory address 624095
if (obj1 == obj2) {
    // always false. 450130 != 624095
}
// Error! No operator overload `==` for types Object and Object.
if (*obj == *obj2) {
}
```

But...there *are* non-primitive types that compare safely in GDScript.

```gdscript
var a1 = [1, 2, 3]
var a2 = [1, 2, 3]
var d1 = { x: 1, y: 2 }
var d2 = { x: 1, y: 2 }
if a1 = a2:
    print("a1 and a2 have the same content") // Works!
if d1 = d2:
    print("d1 and d2 have the same content") // Works!
```

What's the difference here?

Reminder: C++ is *dumb*.

```cpp
// I know nothing. What even is I? Self do wat?
class Object {};
```

You know how in GDScript, you can override methods?

```gdscript
# base.gd


func print_name():
    print("base")

# derived.gd
extends "base.gd"


func print_name():
    print("derived")

# my_node.gd
extends Node


func _ready():
    var base = load("base.gd").new()
    base.print_name() # prints "base"

    var derived = load("derived.gd").new()
    derived.print_name() # prints "derived"
```

**What if** GDScript had a method to compare Objects?

What if this were a kind of engine notification that you could override like `_ready()`?

```gdscript
# reference_vector2.gd
extends Reference


var x := 0.0
var y := 0.0


func _equals(v1, v2) -> bool:
    return v1.x == v2.x and v1.y == v2.y
```

C++ supports this with many predefined `operator<symbol>()` functions:

(again, cover functions later)

```cpp
bool Array::operator==(const Array &otherArray) const {
    // `==` operator belonging to Array class
    // Comparing against some other Array called `otherArray`

    // Add logic to manually compare content, not memory address
}
```

The `Variant` class then does something like this in GDScript:

```gdscript
func _equals(other) -> bool:
    match typeof(other):
        TYPE_ARRAY:
            return (self as Array) == (other as Array)
        TYPE_DICTIONARY:
            return (self as Dictionary) == (other as Dictionary)
        # etc.
```

That *forces* a call to the type-specific `operator==` implementation.

So, why is there a difference?

`Object` is too general-purpose. Use truest comparison (instances).

`Array` and `Dictionary` are collections. Use convenient comparison (content).

## String Comparisons

Strings are more complicated. Every string is a c-string that is a pointer to some sequence of characters in memory.

`const` keyword is a type qualifier that means the variable cannot be mutated. It is read-only.

A `const char*` is a *completely different data type* than a `char*`.

Despite this, assignment (`=`) still works (will cover more later).

String literal = `const char*` since value cannot change.

```cpp
// Cover this later. Gives access to `strcmp()` function.
#include <cstring>

const char *s1 = "hello";
const char *s2 = "hello";
const char *s3 = s1;

// const char * == int
// Error! Cannot compare pointer with int.
"2" == 2;

// (long long <- const char*) == int
// 450130 == 2
(long long)"2" == 2;

// const char* == const char*
// Literals directly written into compiled binary.
// Each literal address is different: false
"2" == "2";

// Comparing *addresses*. Not same c-string: false
s1 == s2;

// Comparing *addresses*. Same: true
s1 == s3;

// s1 first char 'h'. s2 first char 'h'.
// 'h' == 'h' -> 0x68 == 0x68: true
*s1 == *s2;

// strcmp; Compares char at each index until \0.
// Returns 0 if same: true
int result = strcmp(s1, s2);
if (result < 0) {
    // s1 has earlier char with lesser numeric value
} else if (result > 0) {
    // s1 has earlier char with greater numeric value
} else if (result == 0) {
    // s1 and s2 have equal char values all the way through.
}
```