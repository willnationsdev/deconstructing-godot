# GDScript to C++ Episode 5: Functions

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

### **Aside: Error Enums**

Since I'm showing `main()`...

`0` = `false`, else `true`.

```cpp
bool success = do_something();
```

If `false`...*what* went wrong? Need more info.

1 success case, many error cases. Return `int`? `0` for success, else error.

Human-readable error names -> `enum`.

```cpp
enum Error {
    OK, // 0
    ERR_BAD_PARAM, // 1
    ERR_CANNOT_OPEN_FILE, // 2
    ERR_MISSSPELLLING, // 3
    ERR_FML_WHY_THIS_NO_WORK, // 4
    ERR_ID_10_T // 5
}

int main() {
    // try to open file. If it fails...
    return ERR_CANNOT_OPEN_FILE;
    // else...
    return OK;
}
```

You can see Godot's `Error` enum defined in the `@GlobalScope` API docs.

This can be confusing, even in GDScript, when things like `File.open` return an `Error`:

```gdscript
var f = File.new()
# if 0, i.e. OK, will NOT enter the if statement.
if f.open("data.txt", File.READ):
    print("Operation was 'true'. So...it worked, right? NOPE")
 
var result = f.open("data.txt", File.READ)
if result: # works, but might be confusing
    printerr("Something went wrong")

var err = f.open("data.txt", File.READ)
if err != OK:
    printerr("Result was not OK, therefore a non-zero int Error was returned.")
    printerr("Error: %d" % err)
```

Moving on...
