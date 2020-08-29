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

GDScript has different degrees to which it can satisfy an interface. However,
nothing is guaranteed at parse-time. You are limited to basic inheritance with static
typing or checking method names.

---

```cpp
// iparentable.h
class IParentable {
public:
    virtual IParentable *get_parent() const = 0;
    virtual void get_children(List<IParentable *> *r_children) const = 0;
}

// parentable_object.h
#include "iparentable.h"
class ParentableObject : public Object, public IParentable {
    // <insert Godot Object stuff here>
public:
    virtual IParentable *get_parent() const override {}
    virtual void get_children(List<IParentable *> *r_children) const override {}
}

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
}

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
}
```

Note that, with C++, the relationship with the interface must be explicit.
Just because the `Node` class has `get_parent` and `get_children` methods
doesn't mean it conforms to the `IParentable` interface.

In contrast, because GDScript is duck-typed, it would allow a Node to be used
in place of a ParentableObject / ParentableResource for these purposes just
fine.
