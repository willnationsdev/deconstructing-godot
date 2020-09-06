# GDScript to C++ Episode 1: Comments and Classes

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
A multiline comment syntax doesn't exist.
Use docstrings instead!
"""
```

## Class declaration

- GDScript:
    - file == class, implicit declaration

        ```gdscript
        extends Node         # an engine class
        extends "my_node.gd" # a file path to another script
        extends MyNode       # the name of a global script class
        ```

    - must extend an engine class. No `extends` == `Reference`.

        These two classes are identical in their inheritance/structure

        ```gdscript
        # ref1.gd

        # ref2.gd
        extends Reference
        ```

- C++:
    - file != class, explicit declaration
    - N top-level classes per file
    - independently inheritable

Syntax: `<keyword> <identifier> <block> <semicolon>`

```cpp
// some_class.h
class SomeClass {

};
class AnotherClass {
    
};
```

> Note: *Usually*, if you have curly braces, then you WILL NOT need a semicolon. Classes are a special case.

## Access Modifiers And Basic Inheritance

Every language's classes have "access levels". These determine whether content is visible to another class. Example: "public", "private", etc. By convention, non-public properties and methods are prefixed with an underscore.

- GDScript:

    - public: all content accessible (default) (no keyword, assumed).

        Naming convention communicates intent.

        ```gdscript
        # some_class.gd
        var _x: int # "private", but really public
        var z: int  # public
        ```

- C++:
    - `private`: inaccessible (default)
    - `protected`: accessible only for inherited classes
    - `public`: all content accessible
    - `struct`: class that defaults to `public` (detailed below)

Syntax `<keyword><colon>`. Align keyword with class declaration. Affected content indented on newlines.

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

C++ Inheritance:

```cpp
class BaseClass {
public:
    int x;
};
class DerivedClass : public BaseClass {

};
```

- `:` == GDScript `extends`.
- Only inherit by typename (not file).
- Inherited with access modifier. Imposes limits on inheritance of base class.
    - `public`: No change to `BaseClass` in `DerivedClass`. *Most of the time*, this is what you will see.
    - `protected`: `BaseClass`'s `public` members -> `protected` members.
    - `private`:   `BaseClass`'s `public` and `protected` members -> `private` members.
- If a modifier is omitted, it defaults to `private`.

C++ *also* has a `struct` keyword.

```cpp
struct BaseClass {
    int x;
};
struct DerivedClass : BaseClass {

}
```

`struct`s default to `public` access for content and inheritance. Above code snippet is equivalent.

For more examples, visit [this StackOverflow page](https://stackoverflow.com/questions/860339/difference-between-private-public-and-protected-inheritance).

## Multiple Classes Per File

- GDScript:

    - Not possible. Always 1 top-level class. Can have nested classes ("class constants" via `class` keyword)

        ```gdscript
        # some_class.gd
        extends Reference
        class InnerClassA:
            extends Node
        class InnerClassB extends KinematicBody2D:
            pass
        
        # usage
        const SomeClass = preload("some_class.gd")
        var a = SomeClass.InnerClassA.new()
        var b = SomeClass.InnerClassB.new()
        ```

- C++:
    - Be careful, ~ "bad practice".
    - Would it make sense to include one class and not the other?

        ```cpp
        // `/core/script_language.h`
        class ScriptServer {
            // ScriptLanguage array
        };
        class Script : public Resource {
            // ScriptLanguage instance
            // ScriptInstance instance
        };
        class ScriptInstance : public Button {
            // Object instance
            // Script instance
        };
        class ScriptLanguage {
            // Can generate Script template
            // Can debug ScriptInstance
        };
        ```

        Interconnected classes. Use 1, you use them all.

    - Use inner classes if require custom data structures for class

        ```cpp
        class TileSet : public Resource {
        public:
            // Belongs to TileSet, but...
            // - is built by TileSetEditorPlugin.
            // - is accessed by TileMap for rendering calculations.
            // Passed to TileSet for configuring TileData instances.
            struct ShapeData {
                // dimensions, position, etc.
            };
        private:
            // Managed internally by TileSet. Holds all data about a tile.
            struct TileData {
                // ShapeData array, name, collision/occlusion/navigation polygon
            };
        };
        ```
