# Interfaces: GDScript to C++

A practical explanation of interfaces for beginners.

## Concepts

What is an interface?

It's a contract where a class agrees to define a set of function signatures.

What is a function signature?

It's the combination of the function's...

- name
- return data type
- number of parameters
- parameter data types

## Purpose

The reason you would want to rely on interfaces in software architecture is because it keeps your code "loosely coupled" to other parts of the codebase. This makes it easier to adapt to changes in requirements or usage. Your code can change shape without breaking the program and without creating bugs.

This means you can make changes faster. Iteration increases and you save time and money. It also makes it easier to work *with* your code so that others feel more comfortable joining your team. Not driving yourself insane is another plus.

## Example

Let's say we want to store data for our game. We like the tree structure of nodes to organize our data, but Nodes are big. They use more memory than Objects and Resources. Can we add a Node-like structure to other types?

Godot Engine supports a similar idea with its `Tree` class and its internal `TreeItem` Object types. It isn't an interface, but it *is* a non-Node Object that supports a hierarchical structure.

For our case, we could have an IParentable interface that defines the methods below:

```gdscript
func get_parent() -> Object:
    pass
func get_children() -> Array:
    pass
```

We have a "set of function signatures" that must be defined in a class.

We've detailed the names they must have, what parameters they must accept, and what data type they must return.

Note that the sample is just a pair of methods. It is *not* an actual script file.

Now for concrete implementations of the interface, they just have to implement those same methods.

```gdscript
# duck-typed interface. Works off function names.
# The existence of the interface is implied by the functions' use.

# parentable_object.gd
extends Object

func get_parent() -> Object:
    return null


func get_children() -> Array:
    return []

# parentable_resource.gd
extends Resource

export var _parent: Object = null
export var _children := []

func get_parent() -> Object:
    return _parent
    

func get_children() -> Array:
    return _children

# uses_parentable.gd
extends Node

func my_func() -> void:
    var data = DataStoreSingleton.get_data()

    # BAD - we've filtered out the Object version!
    if data is Resource:
        var p = data.get_parent()
        var c = data.get_children()

    # Note: cannot verify signature, only name.
    var is_iparentable = (data.has_method("get_parent")
                         and data.has_method("get_children"))

    # If error, silently fails, but continues
    if is_iparentable:
        var p = data.get_parent()
        var c = data.get_children()
    
    # If error, loudly fails, but continues
    if is_iparentable:
        var p = data.get_parent()
        var c = data.get_children()
    else:
        push_error("'data' is does not implement the IParentable interface")

    # If error, loudly fails and crashes
    var p = data.get_parent()
    var c = data.get_children()
```

GDScript has different degrees to which it can satisfy an interface. However, nothing is guaranteed at parse-time. You are limited to basic inheritance with static typing or checking method names.

---

In C++, data types and method signatures are *far* more explicit. There's a lot of clutter on the page, but we'll break it down step by step and then explore the practical example.

```cpp
// some_class.h
class SomeClass {
public:
    virtual ISomeType *get_value() const = 0;
    virtual ISomeType *get_value() const override { return nullptr; }
};

// class declaration
class Some Class { ... };

// "access modifier": whether other classes can access the variable/method.
class Some Class {
private: 
    int _x; // only available inside class. GDScript just uses `_` convention.
protected: 
    int _y; // only available in this or extended classes.
public: 
    int z; // available everywhere. This is the only option in GDScript.
};

// basic function signature:
// <ReturnType> <method_name>(<param list...>);
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
    virtual IParentable *get_parent() const override {}
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
            r_children->push_front(E->get());
        }
    }
};

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
