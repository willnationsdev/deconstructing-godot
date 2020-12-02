# GDScript to C++ Episode 4: Scopes and Statics

"scope" == layer of symbols.

Types of scopes:

- global: symbols are universally available everywhere
- class: symbols persist globally within a class. Outside the class, symbols must be accessed *through* the class.
- block: symbols created and persist within the block
    - also known as "local" scope.
    - function: a type of block scope + predefined symbols for parameters

## GDScript

### **Global Scope**

Engine Symbols: API docs symbols defined by engine binary.

```gdscript
# fps_label.gd
tool
extends Label # 'Label' = engine *class* symbol
func _physics_process(p_delta):
    # 'Engine' = engine singleton *instance* symbol
    text = str(Engine.get_frames_per_second())
```

[Autoload](https://docs.godotengine.org/en/stable/getting_started/step_by_step/singletons_autoload.html): "singletons" defined by ProjectSettings > Autoloads.

![Autoload GUI with name mapped to Node/PackedScene resource path.](https://docs.godotengine.org/en/stable/_images/autoload_example.png)

How it appears in your `project.godot` file:

> The `*` marks whether the autoload is enabled or not.

```ini
[autoload]

TestNode2D="*res://Node2D.gd"
```

[Script Classes](https://docs.godotengine.org/en/stable/getting_started/step_by_step/scripting_continued.html#register-scripts-as-classes): User-defined names for scripts.

```gdscript
# player.gd
extends KinematicBody2D
class_name Player # `class_name` keyword to define global name.
```

You can also define names for GDNative classes on the NativeScript resouce:

![A NativeScript resource with the `script_class_name` property set to `PlayerCpp`.](img/player_cpp.png)

How it appears in your `project.godot` file:

```ini
_global_script_classes=[ {
"base": "KinematicBody2D",
"class": "Player",
"language": "GDScript",
"path": "res://player.gd"
}, {
"base": "",
"class": "PlayerCpp",
"language": "NativeScript",
"path": "res://player_cpp.gdns"
} ]
_global_script_class_icons={
"Player": "",
"PlayerCpp": ""
}
```

You can fetch these values at runtime with...

```gdscript
var script_class_list = ProjectSettings.get_setting("_global_script_classes")
var script_class_map = {}
for a_dict in script_class_list:
    script_class_map[a_dict["class"]] = load(a_dict["path"])

var player = script_class_map["Player"].new()
```

GDScript generates global variables for each of these.

More script class support is planned in the future (Godot 4.0+).

### **Class Scope**

Created with `class` keyword, a class name, and a colon (`:`) with indents.

```gdscript
# player.gd
extends KinematicBody2D
class_name Player

var data # class scope

func print_data():
    print(data) # prints the data from Player

class Helper:
    extends Reference

    var data # class scope, a different 'data'

    # has no awareness of Player.data

    func print_data():
        print(data) # prints the data from Helper
```

### **Block Scope**

Created with any *other* case that uses a colon (`:`) with indents.

```gdscript
if true:
    # new block via conditional
    pass

match self.get_class():
    # new block via match
    pass

for i in 10:
    # new block via loop
    pass

func _ready():
    # new block via function
    pass
```

Unlike class scope, block scopes form a tree of nested symbols that can conflict with one another.

```gdscript
# the_class.gd
extends Node

var data = "test" # class scope: `the_class.gd`

func _ready(): # `the_class.gd` > `_ready`

    var data = "hi" # Error! Conflicts with `data` in `the_class.gd`
    var x = 2       # Safe! New symbol `x`

    if true: # `the_class.gd` > `_ready` > _
        var x = 0 # Error! Conflicts with `x` in previous scope `_ready`
        var y = 0 # Safe!

        print(data) # Safe! Go up tree, find `data` at `the_class.gd`.
        print(x)    # Safe! Go up tree, find `x` at `_ready`.
        print(y)    # Safe! find `y` locally.

    # `y` is unloaded

    if true: # different: `the_class.gd` > `_ready` > _
        print(data) # Safe!
        print(x)    # Safe!
        print(y)    # Error! `y` symbol doesn't exist!

# `x` symbol unloaded
# `data` symbol persists

func _process(p_delta): # `the_class.gd` > `_process`
    # `p_delta` symbol defined

    print(data)    # Safe!
    print(p_delta) # Safe!

# `p_delta` symbol unloaded
```

## C++

### **Global vs. `{}` Scope**

By default, in global scope.

New scopes only require `{}`. No conditional/loop/etc. needed.

```cpp
bool x = true;
{
    bool y = true;

    // both exist
}
// `y` is unloaded
```

C++ entry point: global `main` function (covering function syntax later)

```cpp

bool x = true; // global variable

int main() {
    x; // Safe! Global variable `x`

    int y = 10; // block scope `y`

    return 0;
}
// `y` is unloaded
```

All C++ control flow (conditionals/switches/loops/functions) make use of curly braces and work the same way.

### Shadowing

In GDScript, when we redeclared variables in a nested block, we got name conflicts.

In C++, this doesn't happen. Nested symbol duplication results in *shadowing* that hides the original symbol.

```cpp
bool x = true;
{
    int x = 2; // Works! `bool x` doesn't exist now!

    // all references to `x` now get `x` from local scope.
    int sum = x + x; // 4
}
{
    bool x = false; // Works! Data type doesn't matter.
}
int x = 2; // Error! Redeclaring `x` in same scope.
```

> Some languages, like F# or Rust, do support this.
> ```fsharp
> // F#
> let x = 10     // x = 10
> let x = x + 10 // x = 20
> let x = x + x  // x = 40
> ```

### **Declarations and Definitions**

In C++, there's a difference between *declaring* that a symbol exists vs. *defining* what that symbol is.

```cpp
// Can declare type with a definition (all at once)
struct Vector2 {
    float x = 0.0f;
    float y = 0.0f;
}

// Can separate declaration and definition
struct Vector3;
struct Vector3 {
    float x = 0.0f;
    float y = 0.0f;
    float z = 0.0f;
};
```

Why do this?

C++ ~= C with classes.

A *procedural* language + Object-Orientation. Types defined one after another, in order, not all at once.

```cpp
class Node {
    SceneTree *_tree; // Error! `SceneTree` not defined
};
class SceneTree {
    Node *_root;      // Error! `Node` not defined (because of error above)
};
```

How to fix? Use *forward declarations*.

```cpp
class SceneTree;

class Node {
    SceneTree *_tree; // SceneTree exists! Figure out details later.
};
class SceneTree {
    Node *_root;      // Node exists!
};
```

> Note: C#/Java are fully Object-Oriented and will automagically figure out dependencies to load things in the proper order.

### **Static And The Scope Operator (`::`)**

How do we answer the question, "How many instances of a class have we made?"

One answer: use a `static` variable.

"Static" means a *class* owns a symbol, not *instances* of the class. All instances *share* access to something belonging to the class.

```cpp
struct Counter {
    static int count; // variable declaration
    Counter() { count += 1; } // constructor increases num by 1
};

int Counter::count = 0; // variable definition, initialization

int main() {
                // Counter::count == 0
    Counter c;  // Counter::count == 1
    Counter c2; // Counter::count == 2

    int count = c.count         // Error! `c` has no `count` member.
    int count = Counter.count   // Error! Cannot use `.` with data type.
    int count = Counter::count; // Works!

    return 0;
}
```

- Add `static` keyword before data type in declaration.

- Use constructor function which gets called during instantiation (cover more later).

- Use the scope operator (`::`) to access `static` declarations.

The static member variable must be *defined* (initialized) outside of the class definition.

Pay special attention to our *definition* of the `count` member variable.

```cpp
int Counter::count = 0;
```

This is *no different* than initializing a typical variable.

```cpp
int count = 0;
```

The syntax has *not* changed.

```xml
<DataType><Symbol><AssignmentOperator><ValueExpression><Semicolon>
```

By default, in the global *namespace*.

A `namespace` is a kind of scope for typenames: Classes, structs, enums, etc.

The symbol we want, `count`, is part of `Counter`, so we must enter its space with `::` before the symbol becomes visible to use from the global namespace.

But, why must we define it separately?

```cpp
struct Counter {
    static int count; // declare
    // ...
};
int Counter::count = 0; // define
```

When `Counter` is created, so is `count`.

`Counter` is defined in the global namespace. So too must `count` be so defined.

It just so happens that `count` belongs to `Counter`, not the global namespace, hence the `Counter::count`.

Can also access global namespace manually using the scope operator (`::`) with no prefix.

```cpp
int count = 10;
struct Counter {
    static int count;
    Counter() { count += ::count; }
}
int Counter::count = 0;
```

Every new instance of `Counter` now increases `Counter::count` by `10`.

It's even possible to create your own namespaces manually.

```cpp
int global_int = 10;

namespace data {
    namespace integers {
        namespace stuff {
            int x = 5;
        }
    }
    int y = 200;
    int z = 100;
}
global_int += 5; // 15
data::integers::stuff::x += 20; // 25

// Too long. Let's make an alias:
namespace dis = data::integers::stuff;
int sum = dis::x + global_int; // 40

// Getting tired of writing new aliases all the time...
using data;
int y_and_z = y + z; // 300
```

> See more about differences between [static class members and namespaces](https://stackoverflow.com/questions/1434937/namespace-functions-versus-static-methods-on-a-class). Note that [templates also come into play](https://stackoverflow.com/questions/14361408/using-a-namespace-in-place-of-a-static-class-in-c) (we'll cover those later).

### Statics in GDScript

Here's a thought exercise...

Godot specifically *does not support* static data in scripting for thread-safety reasons.

*However*, Godot's scripted classes != C++ classes. They are `Script` resource *instances*.

That's why GDScript lets you alias them with variables. They are *values*.

```gdscript
const SomeClass = preload("some_class.gd")
const MyAlias = SomeClass # can assign "class" to a different variable
```

All Godot `Object` types, including `Resource` and `Script`, support runtime metadata read/write with `set_meta(key, value)` and `get_meta(key)`.

You can combine this with script classes for pseudo-static data.

```gdscript
# static.gd
extends Reference
class_name Static

# some_class.gd
extends Node
func _ready():
    Static.set_meta("some_data", { x: 1, y: 2})

# another_class.gd
extends Node
func _ready():
    var data = Static.get_meta("some_data")
    if data:
        print(Vector2(data.x, data.y))
```

However, this is similar to creating an Autoload:

```gdscript
# static.gd, autoload name "Static"
extends Node
var meta = {}

# some_class.gd
extends Node
func _ready():
    Static.meta.some_data = { x: 1, y: 2 }

# another_class.gd
extends Node
func _ready():
    var data = Static.meta.some_data
    if data:
        print(Vector2(data.x, data.y))
```

Which should you choose? Well, Autoloads store data on a Node instance, not a resource's metadata, so you have access to more features:

- non-static members, methods, and signals
- `setget` keyword
- statically-typed data (metadata is inherently untyped)
- `get_tree()`, the SceneTree and all that entails...
    - the Node tree
    - Node groups
    - delta-time
    - input events
- data serialization support (saving/loading)

So, while pseudo-static data is *possible* in GDScript, it's just not preferable to the alternative.

### More Statics: Functions and Classes

GDScript does, however, support static *functions*.

```gdscript
extends Reference
class_name Util

static func get_author() -> String:
    return "willnationsdev"
```

No need to create a Util instance.

```gdscript
var util = Util.new() # why bother?
var name = Util.get_author() # can directly call on Script
```

C++ supports the same (again, we'll cover functions in more detail later).

```cpp
struct Util {
    static const char* get_author() {
        return "willnationsdev";
    }
}
Util util;
const char* name = util.get_author();  // Error! get_author is static!
const char* name = Util::get_author(); // Works!
```

However, it's frustrating that people can even make a Util instance. Let's prevent that.

```cpp
static struct Util {
    // same...
}
Util util; // Error! Static class cannot be instantiated!
```

Booyah.

### To Be Continued

Unfortunately, that is not the end of scopes in C++. When we learn Inheritance, there will be yet more to discover (`this`, `Base1::x` vs. `Base2::x`), but alas, those are for another day.
