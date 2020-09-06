# Comparing C++ and GDScript Syntax

## Outline

What you will learn:

Basic C++ concepts/syntax and the fundamentals of a computer program's memory structure.

Expected Knowledge:

GDScript features and syntax.

## Introduction

Key Principles:

1. C++ thinks *everything* is a *number*.
1. C++ needs to know *exactly* how those numbers are *arranged and used* ahead of time.
1. C++ is *dumb*. It gives you the tools to painstakingly construct whatever you want and trusts that you know what you are doing.

## Comments

C++:

```cpp
// Comments use double-slashes
int x; // a comment after a statement
/* This is a
   multiline
   comment */
```

GDScript:

```gdscript
# Comments use pound
var x # a comment after a statement
"""
A multiline comment syntax doesn't exist.
Use docstrings instead!
"""
```

## Class declaration

In GDScript, the starting point of knowledge is class declarations. Every `.gd` file declares the existence of a class, even if it is empty (inheritance defaults to `Reference`).

In C++, it isn't enough to have a file. You must explicitly declare the existence of the class using the `class` keyword, followed by its name, the block of code associated with the class, and a terminating semicolon.

```cpp
// some_class.h
class SomeClass {

};
```

Notice the lack of inheritance. Unlike GDScript, C++ classes can exist independently.

## Access Modifiers And Basic Inheritance

In C++, external classes can only access properties and methods (aka members) under certain conditions. By default, members are `private`, i.e. hidden. The class must explicitly specify a `protected` or `public` access level for a region of code if desired.

By convention, non-public properties and methods are prefixed with an underscore.

```cpp
// some_class.h
class SomeClass {
private:
    int _x; // only accessible in SomeClass
protected:
    int _y; // only accessible in SomeClass and its derived classes
public:
    int z; // acessible from any class
};
```

```gdscript
# some_class.gd
# Everything is public. Uses conventions to communicate intended access.
var _x: int
var z: int
```

In GDScript, every class must extend an existing engine class. It can be done in one of 3 ways:

```gdscript
extends Node         # an engine class
extends "my_node.gd" # a file path to another script
extends MyNode       # the name of a global script class
```

In every case, the inheriting class gains unrestricted access to all of the base class's content.

Here's an example of inheritance in C++:

```cpp
class BaseClass {
public:
    int x;
};
class DerivedClass : public BaseClass {

};
```

The `:` after the class name is C++'s `extends` syntax.

The `BaseClass` is the class name to be inherited.

Inheriting classes apply an access modifier to their base class. This imposes a limitation on the access levels inherited by the base class.

`public` inheritance makes no change to `BaseClass` in `DerivedClass`. *Most of the time*, this is what you will see.

`protected` inheritance converts all of `BaseClass`'s `public` members to `protected` members.

`private` inheritance converts all of `BaseClass`'s `public` and `protected` members to `private` members.

If a modifier is omitted, it defaults to `private`.

C++ *also* has a `struct` keyword.

```cpp
struct BaseClass {
    int x;
};
struct DerivedClass : BaseClass {

}
```

The only difference between a `class` and a `struct` is the default access level which is `public` rather than `private`. Therefore, the two previous code segments are functionally equivalent.

For more examples, visit [this StackOverflow page](https://stackoverflow.com/questions/860339/difference-between-private-public-and-protected-inheritance).

## Symbols: Variables as Memory Locations

A "symbol" is a string of text within a language's source code that identifies a language entity (something the language tracks).

```gdscript
extends Node
var x = 0 # `x` is a symbol
```

All languages have the concept of a "symbol table". It tracks each symbol's memory location.

A language's internal symbol table might look something like a string-to-int Dictionary:

```gdscript
var symbols = {}

# `var x = 10`

symbols["x"] = 10484892920
```

However, for simplicity, most memory address information is hidden in GDScript.

## Dynamic Variant vs Static Data Types

When a program creates a variable, it needs to know how many bytes each variable is. When declaring variables, the program writes their values into memory sequentially, beside one another. However, how it does so is determined by data types.

Data types have a *byte size* which can affect the byte offset between variables.

Data types tell the computer *how* to write their data to memory.

### GDScript

Now, GDScript is dynamic. How does it achieve this? It is built on Godot's Variant. Variant can store a subset of data types. The largest of these is 16 bytes. It combines these 16 bytes with a 4-byte enum to keep track of a data type. With these combined 20 bytes, it has some tricks available to it:

- It stores a *size* of 20 bytes for everything, even if the data needs less than that. No need to calculate sizes at compile time.
- It freely *interprets* data types on a whim based on the enum value.
    - `int(5)` to `bool(1)`?
        1. 4-byte `TYPE_INT` -> `TYPE_BOOL`.
        2. 16-byte 0x0000000000000005 -> 16-byte 0x0000000000000001.

> See [@GlobalScope docs](https://docs.godotengine.org/en/latest/classes/class_@globalscope.html) for the `Variant.Type` enum to see all available types.

### C++ Data Types

C++ is a natively-compiled, statically-typed language. Because it is compiled ahead of execution for a native target, it must know the exact structure of everything ahead of time. How?

C++ only comprehends numbers but applies them in many ways...

C++ qualifies numbers:

- sizes (usually 8, 16, 32, 64)
- signage
- byte size qualifiers (`short`, `long`)
- mutability (default true, `const` false)
- reference to existing number (`&`)?
- memory address (`*`)?

C++ interprets numbers (a "data type"):

> Note: Byte sizes are not final! Can be adjusted with size qualifiers and may be compiler dependent.

> C++ believes that *every* type can be boolean (is it zero?). There are many values of 0x0000 which can be expressed as different data types. See the 'False' sections below.

- void: `void` (0 bytes)
    - Nothing/Unknown.
- boolean: `bool` (1 byte)
    - Truth-ness.
    - True: `true`
    - False: `false`
- integer: `int` (4 bytes)
    - Positive or negative whole numbers.
    - True: `1000`, `-3`
    - False: `0`
- character: `char` (1 byte)
    - Letters to be displayed **or** *small* positive or negative whole numbers.
    - True: `'a'`, `'\n'`
    - False: `'\0'` ("null terminator")
- floating-point: `float` (4 bytes)
    - Imprecise representations of fractions and decimals. Leave a margin of error in comparisons for consistency.
    - True: `3.14`, `-0.0000001`
    - False: `0.0`
- pointer: `*` (4 bytes x32 | 8 bytes x64)
    - Stores a memory address to a specific type
    - True: `504891358`
    - False: `NULL` or, after C++11, `nullptr`

> See the [C++ reference](https://en.cppreference.com/w/cpp/language/types) for a more detailed breakdown type representations.

Computers consider data types with distinct qualifiers to be *completely different data types*.

A function marks the starting memory location of executable instructions. It has a *number*.

A class or struct is a named, sequential list of varying types of numbers.

Inheritance is more or less a named, sequential list of classes.

> C++ supports *multiple inheritance*. This can lead to inheriting duplicate classes. See the [Diamond Problem](https://www.geeksforgeeks.org/multiple-inheritance-in-c/) for more information.

### C++ Example

```cpp
struct VerifiablePoint {
    bool isVerified;
    int x;
    int y;
}
```

Depending on the compiler, an instance of `VerifiablePoint` in memory could look like this:

```cpp
VerifiablePoint p;
p.isVerified = false
p.x = 10
p.y = 4

// memory   : value
// -----------------
// 00000000 : 0x0000
// 00000001 : 0x000A
// 00000005 : 0x0004
// 00000009 : 
```

## Values vs. References

In GDScript, most all data is "passed by value". These variables make a copy during assignment.

```gdscript
var v1 = Vector2()
var v2 = v1 # copied!
v1.x = 10
# v2.x is still 0
```

However, every Object, Array, and Dictionary is "passed by reference". These variables create a shared "reference" to the same data during assignment.

```gdscript
var n1 = Node.new()
var n2 = n1 # referenced!
n1.name = "MyNode"
# n2.name is also "MyNode" now
```

In C++, the distinction between a value and a reference is not based on the data type, but rather the syntax. Furthermore, different data types are responsible for handling references versus values.

```cpp
class SomeClass {};

SomeClass s1; // data type is SomeClass, a value
SomeClass &s2 = s1; // data type is SomeClass&, a reference
```

The `&` modifies the type to be a reference to a SomeClass instance. C++ allows you to create these references for *any* data type, including integers.

```cpp
int x = 0;
int &y = x; // references x directly
```

`&` creates a reference to an *existing* value. It does not store the value's memory address, but it also does not work with literals which have no memory address.

```cpp
int &x = 0; // error! References must be assigned an addressable value
```

References are *constant*. You can't make them reference something else once they've been initialized! Why is that? Because of the Stack...

```cpp
int x = 0;
int y = 0;
int &z = x;
z = &y; // error! References cannot be reassigned.
```

## The Stack

Whenever you use a function (`void func() { ... }`) or control flow (`if/else/for/do/while/switch/case { ... }`)

```cpp
<something?> {

}
```



Whenever you enter a new block of code, the computer sets aside a region of memory for you to use in that block called the Stack. As we discussed earlier, every GDScript variable is 20 bytes, so...

```gdscript
var x = 1
var y = 16
var z = 256
```

...looks like this in the Stack:

```
184728300: 0x00001
184728320: 0x00010
184728340: 0x00100
```

The compiler knows that every GDScript variable is 20 bytes. Therefore, it always knows at which memory address to write the next variable's data..

Once you've established that a reference is refering to a symbol's memory address, it won't allow you to reset it to look at a different location, lest you change it and corrupt memory.

The most important thing about memory, however, is how it gets reused!

Say you define some variables in a block of code. When the block ends and you start a new block, your new segment of the Stack may overlap with the previous block's. The data written in memory for the old variables *may still be there*.

In GDScript, everything is initialized for you, setting the old data to a default value.

```gdscript
var x      # always defaults to null
var y: int # always defaults to 0
```

In C++, uninitialized variables will contain arbitrary trash from previous contexts. C++ will attempt to interpret those bits as if they were the variable's type, no matter what data type they were originally part of.

```cpp
int x;     // value is ?????. Was it an object? Who cares? Now it's an `int`.
int y = 0; // value is 0
```

Therefore, for safety in C++, you must always initialize your data manually.

## Arrays

In GDScript, `Array`s are a built-in class with their own methods. They can dynamically resize themselves to grow or shrink as needed. They can also store multiple data types.

```gdscript
var arr := []         # Assignment using Array literal.
var arr2 := Array()   # Assignment using Array constructor.
if arr == arr2:       # They know how to compare against each other.
    print(arr.size()) # size() returns 0

arr.append(10)        # now size() returns 1

arr.append("hello")   # now contains integers AND strings

```

Not so in C++. Arrays are a primitive (non-class) data type. More importantly, the Stack needs to know, at compile time, where it needs to write the next variable's data. This means you have to declare ahead of time how much data is needed for every variable.

In order to know the size of the array, you need to know the size of its elements. So, every element must be the same data type and there must be a fixed total capacity for the array.

```cpp
// sizeof(int) == 4
// capacity of array is 10
// 4 bytes * 10 elements = 40 total bytes
// Therefore, the 5 in `x` is written 40 bytes ahead of `arr`.
int arr[10]; // type = `int[]`; capacity = 10; size = 0;
arr[0] = 3;  // size = 1
int x = 5;
```

Unlike an `Array`, C++ arrays are given existing memory on the Stack.

If you attempt to access data outside the bounds of the array, it doesn't loop around. Instead, accessing memory that doesn't belong to you will result in a fatal crash called a "segmentation fault".

GDScript produces a runtime script error if you try to access a too-high index. But, a too-low index is interpreted as an intent to count backwards from the end.

```gdscript
var arr := [4, 5, 6]
arr[0] # 4
arr[3] # error!
arr[-1] # 6
```

```cpp
int arr[3];
arr[-1]; // crash
arr[0]; // safe
arr[1]; // safe
arr[2]; // safe
arr[3]; // crash
```

Because the exact size must be known at compile time, only literals and constants may be used to define the length of an array.

```cpp
const int CAPACITY = 10; // `const int`. Immutable value (value is constant).
int capacity = 10;       // `int`. Mutable value (value can change).

int arr[10];             // literal, compiles.
int arr[CAPACITY];       // variable with immutable data type, compiles.
int arr[capacity];       // variable with mutable data type, does not compile.
```

But wait! That can't be all there is, right? GDScript's `Array` can change its capacity. Why is that? Because of the Heap...

## Heap, and Pointers (TODO)

The Heap is a region of memory that exists elsewhere in the computer. Where *exactly* the memory is generally unknown since the operating system micromanages it. An app can request memory from the Heap whenever it wants to at runtime.

The Heap is not tied to a particular block of code. The memory will persist between code blocks until we manually free the memory.

The good news? You can request a variable amount of data. Also, the memory is not part of a block, so the Stack can't remove it.

The bad news?




> Note: there is some ambiguity surrounding the term "reference".
>     - is a reference (reference)
>     - references (reference or pointer)
>     - pass by reference (reference or pointer)




The `*` means we have a `uint64_t` (an unsigned 64-bit integer) which stores a memory address for a `SomeClass` instance.


C++ calls these data types "pointers"

## Method Definitions (Basic) (TODO) (MOVE)

In GDScript, a return value's data type and the parameters' data types are optional. In C++, they are required. Because they are required, they go before a parameter's symbol, not after.

```cpp
class SomeClass {

    void print_hello() {
        print_line("Hello World!");
    }

    int add(int a, int b) {
        return a + b;
    }
};
```

```gdscript
# some_class.gd
func print_hello() -> void:
    print("Hello World!")


func add(a: int, b: int) -> int:
    return a + b
```

## Declarations vs Definitions

In GDScript, every symbol is both a declaration that *a thing exists* and a definition of *what that thing is*.

```gdscript
# node_a.gd, file exists (declaration), its content details definition
extends Node

# node_b.gd, file exists (declaration), its content details definition
extends Node
```

But C++ is ultimately a procedural language. Definitions occur in order.

```cpp
class A {
    B b; // error! Do not recognize class 'B'.
}

class B {
    A a;
};
```

The compiler will need to know how many bytes an instance of A uses. So it looks up the size of B. But if we haven't arrived at B's declaration, then we don't know what B is.

Let's first "declare" that these types exist using a "forward declaration". Then we can "define" what they are later. This will tell the compiler to worry about how big they are until after it has catalogued the size of everything else.

```cpp
// forward declarations
class A;
class B;

class A {
    B b;
}

class B {
    A a;
};
```

This is how C++ handles circular dependency issues which GDScript 1.0 in Godot 3.x struggles with.

## Scopes

A "scope" is a region of code where certain symbols exist.

### Local Scope

    Local variables only exist in a specific block and its nested blocks. This can often make the terms `scope` and `block` interchangeable *within local scopes*.

    C++ can create a new scope any time it wants with curly braces.

    ```cpp
    {
        // `x` symbol does not exist
        int x = 10;
        // `x` symbol now exists
        {
            int y = x;
            // `y` symbol now exists
        }
        // `y` symbol no longer exists
    }
    // `x` symbol no longer exists
    ```

    GDScript uses indents to define new local scopes. It cannot create them directly. You must declare a method or use control flow to establish a new scope.

    ```gdscript
    func do():
        # `x` symbol does not exist
        var x = 10
        # `x` symbol now exists
        if true:
            var y = x
            # `y` symbol now exists
        # `y` symbol no longer exists
    # `x` symbol no longer exists
    ```

### Class Scope (TODO)

    Symbols which are available throughout a class.

    ```cpp
    class SomeClass {
    
    };
    ```

    ```gdscript
    var x = 10
    func do_one():
        x += 5 # refers to `x` property
    func do_two():
        x += 10 # also refers to `x` property
    ```

### Global Scope

Symbols which are available everywhere.

GDScript does not give users write access to its global scope, but it does create a few read-only variables for users.

- `GDNativeClass`, e.g. `Node`, `Resource`, `Object`
    - `Object` itself doesn't have a `.new()` method. GDScript creates fake classes with `.new()` which creates instances for users.
- Engine Singletons, e.g. `Engine`, `OS`, etc.
- Autoload Singletons (Node-extending scripts or scenes you configure in ProjectSettings)
- Script Classes (scripts using the `class_name` keyword)

In C++, everything is in global scope by default. To avoid putting things in global scope, one must wrap them in different scope. (Note: we'll go over function syntax later).

C++ has a special term for the collection of symbols present certain scopes: a "namespace".

```cpp
// A C++ function with a global variable.
int x = 0;
void do_nothing() {
    return 0;
}
```

If a symbol occupies the global namespace, then C++ will not allow something else to use that symbol. This can quickly become a problem, so you should try to avoid it as much as possible.

To not pollute the global namespace, we must move the global variable inside the function.

```cpp
// A C++ function with a local variable.
void do_nothing() {
    int x = 0;
}
```

Uh oh! The `do_nothing` function is still in the global namespace. Put it in a class!

```cpp
// A C++ class with a member function, i.e. a method.
class SomeClass {
public:
    void do_nothing() {
        int x = 0;
    }
};
```

That's better. But now `SomeClass` is in the global namespace. Can't we do something?

Thankfully, C++ lets you create namespaces too.

```cpp
// A C++ function with a local variable
namespace Some {
    class SomeClass {
    public:
        void do_nothing() {
            int x = 0;
        }
    };
}
```

Now, let's access our `SomeClass`.

```cpp
namespace Some {
    // ...
}

SomeClass some; // error! Do not recognize identifier 'SomeClass'.
```

C++ gives us an error saying it doesn't know of a `SomeClass` class. This is because we are doing so outside of the namespace. Our mention of `SomeClass` tries to find it in the global namespace. To look inside the `Some` namespace, we must use the `::` "scope operator".

```cpp
Some::SomeClass some;
```

This is analogous to trying to access an inner class from outside of a GDScript.

```gdscript
# list.gd
class Element:
    pass

# some_class.gd
const List = preload("list.gd")
var element = Element.new() # error! Do not recognize 'Element'.
var element = List.Element.new() # works!
```

## Static (TODO)

You use the scope operator whenever you must access something in a "static" context. That is, a context belonging to a namespace or class.

## What's an API? Source Files vs. Header Files

There's another reason that C++ might use declarations in place of definitions.

The bigger an application gets, the longer you have to wait for C++ code to compile in order to test your code. But what if a change doesn't affect other parts of your code?

```cpp
// number.cpp

int get_number() {
    return 5;
}

// double.cpp
#include "number.cpp"
int double_number() {
    return 2 * get_number();
}
```

> We'll get to `#include` later. For now, know that it copy/pastes the other file's contents into the current file.

If you compile the `double.cpp` file, it will have both the `get_number` and `double_number` functions.

If you change `get_number` to return 10 instead of 5, the compiler *still* recompiles the entire file, including the `double_number` function. But, `double_number`'s logic is completely unchanged. Can we avoid that?

Languages expose a collection of "language entities":

- global variables and functions
- namespaces
- classes
    - properties (class variables)
    - methods (class functions)

The sum of all language entities which enable a programmer to use a piece of software are called the Application Programmer Interface or API.

When it comes to functions in an API, there is a big distinction between declarations and definitions. If the function signature is unchanged, then changing the implementation of that function's definition has no effect on the API. Therefore, only the implementation needs to be recompiled.

```cpp
// number.h
int get_number(); // forward declare that we want this function here in memory.

// number.cpp
#include "number.h"
int get_number() { // define what the function should do
    return 10;
}

// double.cpp
#include "number.h"
int double_number() {
    return 2 * get_number(); // still references same function start in memory.
}
```

When you call a function, the CPU must jump to that location in memory and begin executing the logic there. But if the memory address doesn't need to recompile, then the things *referencing* that function's memory address also do not need to recompile.

`double.cpp` only includes the `.h` header file containing the declarations. As a result, so long as those don't change, recompiling `double.cpp` is unnecessary. When executing `double_number`, the compiler will still need to 

When you do the same thing for classes, you must combine header file and source file separation with the lessons you learned about scope access.

```cpp
// number.h
class Number {
public:
    int get_value();
}

// number.cpp
#include "number.h"
int Number::get_value() { // symbol 'get_value' belongs to the 'Number' scope
    return 10;
}

// double.cpp
#include "number.h"
int double_number(Number n) {
    return 2 * n.get_value();
}
```

## Virtuals, Abstract Classes, and Overrides (TODO)

```cpp

```

```gdscript

```

## Function Overloads (TODO)

```cpp

```

```gdscript

```

## Control Flow: If/Else (TODO)

## Control Flow: Switch/Case (TODO)

## Control Flow: While/Do-While Loop (TODO)

## Control Flow: For Loop (Array) (TODO)

```cpp

```

```gdscript

```

## Control Flow: For Loop (Vector) (TODO)

```cpp

```

```gdscript

```

## ControlFlow: For Loop (List) (TODO)

```cpp

```

```gdscript

```

(WIP, section below is obsolete. Keeping temporarily for reference)

```cpp
// some_class.h
class SomeClass {
public:
    virtual ISomeType *get_value() const = 0;
    virtual ISomeType *get_value() const override { return nullptr; }
};

// class declaration
class SomeClass { ... };

// "access modifier": whether other classes can access the variable/method.
class SomeClass {
private: 
    int _x; // only available inside class. GDScript just uses `_` convention.
protected: 
    int _y; // only available in this or extended classes.
public: 
    int z; // available everywhere. This is the only option in GDScript.
};

// basic function signature:
// <ReturnType> <method_name>(<ParamType> param1, <ParamType> param2);
// ReturnType: ISomeType *
// MethodName: get_value
// ParameterList: ()
ISomeType *get_value();

// advanced function signature:
// `virtual`: can be overridden by a derived class. Always in GDScript.
// `*`: handling a data type by reference. Similar to Object/Array/Dictionary.
// `const`: This function makes no changes to the Object it belongs to.
// `= 0`: There is no implementation of this function. Class is abstract. Cannot instantiate.
virtual ISomeType *get_value() const = 0;

// advanced function signature (continued):
// `virtual`: part of an overridden method hierarchy. Always in GDScript.
// `override`: this method IS overriding an inherited method.
// `nullptr`: This is GDScript's `null`. The `ptr` stands for "pointer".
virtual ISomeType *get_value() const override { return nullptr; }
```

Now, for the actual example.

```cpp
// iparentable.h
class IParentable {
public:
    virtual IParentable *get_parent() const = 0;
    virtual void get_children(List<IParentable *> *r_children) const = 0;
};

// parentable_object.h
#include "iparentable.h"
class ParentableObject : public Object, public IParentable {
    // <insert Godot Object stuff here>
public:
    virtual IParentable *get_parent() const override { return nullptr; }
    virtual void get_children(List<IParentable *> *r_children) const override {}
};

// parentable_resource.h
#include "iparentable.h"
class ParentableResource : public Resource, public IParentable {
    IParentable *_parent;
    List<IParentable *> _children;
public:
    virtual IParentable *get_parent() const override {
        return _parent;
    }
    virtual void get_children(List<IParentable *> *r_children) const override {
        for (const List<IParentable *>::Element *E = _children.front(); E; E = E->next()) {
            r_children->push_back(E->get());
        }
    }
};
// # list.gd
// class_name List
// class Element:
//     extends Reference
// for (starting condition; iteration condition; iteration operation) {

// }
// for elem in list.children:

// uses_awesome_parent.h
class UsesAwesomeParent : public Node {
public:
    void my_func() {
        Node *n = get_node("/root/DataStoreSingleton");
        if (!n) {
            return;
        }

        DataStore *dataStore = Object::cast_to<DataStore>(n);
        if (!dataStore) {
            return;
        }

        IParentable *data = Object::cast_to<IParentable>(dataStore->get_data());
        if (!data) {
            return;
        }

        IParentable *p = data->get_parent();

        List<IParentable *> c;
        data->get_children(&c);
    }
};
```

Note that, with C++, the relationship with the interface must be explicit. Just because the `Node` class has `get_parent` and `get_children` methods doesn't mean it conforms to the `IParentable` interface. You would need to modify the definition of the `Node` class to explicitly have it implement the `IParentable` interface.

In contrast, because GDScript is duck-typed, it would allow a `Node` to be used in place of a `ParentableObject` / `ParentableResource` for these purposes just fine.

Note though that, in practice, making nodes an `IParentable` or using `IParentable`s where a `Node` is expected would be problematic. After all, `Node` can do a lot more, so things expecting a `Node` would want more from an `IParentable`. However, the idea of exposing a subset of class features via an interface is useful in contexts where the code *only cares* about *that interface* and no other features.

To have a truly extensible codebase, you'd have to redesign `Node` so that each of its various features were bound to an interface which any given class could implement. Why doesn't Godot do this? Because breaking down `Node`'s features to such a degree of precision would add unnecessary complication to the engine's design.
