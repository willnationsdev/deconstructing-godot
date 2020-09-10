# GDScript to C++ Episode 1: Classes and Access Modifiers

- Course Learning:
    - C++ concepts/syntax
    - Basic program memory structure

- Course Requirements:
    - GDScript features and syntax

## Key Principles:

1. C++ thinks *everything* is a *number*.
1. C++ needs to know *exactly* how those numbers are *arranged and used* ahead of time.
1. C++ is *dumb*. It trusts you to painstakingly build whatever you want with almost no supervision.

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
No multiline comment syntax.
Use docstrings instead!
"""
```

## Class Declaration

Declaration = something exists (any language)

- GDScript:
    - File == class, implicit declaration

        ```gdscript
        # some_class.gd (even without this comment)
        ```

    - Must extend an engine `Object` class.

        How To...

        ```gdscript
        extends Node         # Engine class (basetype).
        extends "my_node.gd" # Script file path (path->load->basetype).
        extends MyNode       # Global script class (name->path->load->basetype).

        extends Array        # Error! Array is not an Object!
        ```

        No `extends` == `Reference`. Identical types below.

        ```gdscript
        # ref1.gd

        # ref2.gd
        extends Reference
        ```

- C++:
    - File != class, explicit declaration
    - N top-level classes per file
    - Independently inheritable

Syntax: `<keyword> <identifier> <block> <semicolon>`

```cpp
// some_class.h
class SomeClass {

};
class AnotherClass {
    
};
```

> Usually, `{ ... }` OR `;`, not both. Classes are special. [See more](https://stackoverflow.com/questions/1783465/why-must-i-put-a-semicolon-at-the-end-of-class-declaration-in-c/1783509).

## Access Modifiers And Basic Inheritance

All programming languages have access levels.

"Access Level" == can do `obj.<something>`.

Types: `public`, `protected`, `private`.

Convention: non-`public` content has `_` prefix.

- GDScript:

    - `public`: all content accessible (default) (no keyword, assumed).

        Naming convention communicates intent.

        ```gdscript
        # some_class.gd
        var _x: int # "private" or "protected", but really public
        var z: int  # public
        ```

- C++:

    - `private`: inaccessible (default)
    - `protected`: inherited classes only
    - `public`: all accessible

Syntax: `<keyword><colon>`. Align keyword with class declaration. Affected content indented on newlines.

```cpp
// some_class.h
class SomeClass {
private:
    int _x; // only accessible in SomeClass
protected:
    int _y; // only accessible in SomeClass and its derived classes
public:
    int z; // accessible from any class
private: // can re-declare sections for dependencies
    int _a = z;

protected: int b; // BAD, no newline

int c; // BAD, content not indented

    public: // BAD, modifier not de-dented
};
```

Classes default to `private` access if you omit an access modifier.

```cpp
// some_class.h
class SomeClass {
    int _x; // private
};
```

## C++ Inheritance With Access Modifiers

```cpp
class BaseClass {
public:
    int x;
};
class DerivedClass : public BaseClass {

};
```

- C++ `:` == GDScript `extends`.
- inherit typename (not file)
- Inheritance access modifiers: Imposes limits on inheritance of base class.
    - `public`: No change to `BaseClass` in `DerivedClass`.
    - `protected`: `BaseClass`'s `public` members -> `protected` members.
    - `private`:   `BaseClass`'s `public` and `protected` members -> `private` members.
- If you omit a modifier, it defaults to `private`.

```cpp
class Base {
private:
    int _x;
protected:
    int _y;
public:
    int z;
};
class Private : private Base {}; // _x/_y/z private
class Protected : protected Base {}; // _x private, _y/z protected
class Public : public Base {}; // same as Base
```

C++ *also* has a `struct` keyword. `struct` == `class`, but default is `public`.

```cpp
class BaseClass {
public:
    int x;
}
class DerivedClass : public BaseClass {

}
// equal to:
struct BaseClass {
    int x;
};
struct DerivedClass : BaseClass {

}
```

> Note: In other languages, `struct` is usually something else.

## Multiple Classes Per File

- GDScript:

    - Always 1 top-level class.
    - Supports inner classes ("class constants" via `class` keyword).

        ```gdscript
        # some_class.gd
        extends Reference
        class MyInnerClass:
            extends Node
        
        # usage
        const SomeClass = preload("some_class.gd")
        var my_class = SomeClass.MyInnerClass.new()
        ```

- C++:
    - Supports inner classes.
    - Multiple classes? Be careful, ~ "bad practice". Allowances...
        - Use one and not others? No. Interconnected classes. Use one == all.

            ```cpp
            // `/core/script_language.h`
            class ScriptServer {
                // ScriptLanguage array
            };
            class Script : public Resource {
                // ScriptLanguage instance
                // ScriptInstance instance
            };
            class ScriptInstance {
                // Object instance
                // Script instance
            };
            class ScriptLanguage {
                // Can generate Script template
                // Can debug ScriptInstance
            };
            ```

        - Use inner classes if require custom data structures for class

            ```cpp
            // `/scene/resources/tile_set.h`
            class TileSet : public Resource {
            public:
                // Belongs to TileSet, but...
                // - TileSetEditorPlugin creates them *for* TileSet
                //     - Passes to TileSet to create internal TileData instances.
                // - TileMap accesses them for rendering calculations.
                struct ShapeData {
                    // dimensions, position, etc.
                };
            private:
                // Managed internally by TileSet. Holds all data for a tile.
                struct TileData {
                    // ShapeData array
                };
            };
            ```
