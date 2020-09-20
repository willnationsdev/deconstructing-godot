# GDScript to C++ Episode 2: Memory Manipulation

## Variable Memory

Programs -> variables -> data written to memory.

- Where? Variable memory location.
- How to read/write? Data type. Determines algorithm...
    - How many bytes ("byte size")?
    - How are bits arranged in that range?
        
        > The same bits can be interpreted multiple ways!

- How to reference?
    - Map symbol to location via "symbol table".

```cpp

// Memory location
int x; // exists at arbitrary memory address 450130

// Byte size. `int` is 4 bytes. Subsequent vars mem offset by size.
int x; // 450130
int y; // 450134 (+ sizeof(x))

// `int` bit arrangement: leading bit denotes sign. Remaining bits denote value.
int sx = 0;  // 0x00000000
int sy = 1;  // 0x00000001
int sz = -1; // 0xFFFFFFFF

// Data type controls meaningful interpretation of bits.
unsigned int ux = 0;  // 0x00000000
unsigned int uy = 1;  // 0x00000001
unsigned int uz = -1; // 0xFFFFFFFF, becomes 4294967295

// Note: `int` signed by default.
```

## Symbol Tables

Symbol table records variable locations for later reference.

In GDScript, this would be like a Dictionary of strings mapped to integers of array indices.

```gdscript
var symbols = {}
var values = PoolByteArray()

# int x = 2
symbols['x'] = 450130
values[450130] = 0
values[450131] = 0
values[450132] = 0
values[450133] = 2
```

What does this line do?

```cpp
int z = y; 
```

1. Find `y` in symbol table.
1. `y` is type `int`. So, it has 4 bytes.
1. `y` is at location 450134. Grab bytes 450134-450137, inclusive.
1. Interpret value based on `signed int` data type.
1. Find `z` in symbol table.
    - Doesn't exist? Make a new `int` in memory and record address.
1. Copy bits from right-hand value to left-hand memory address.

## Basic Static Casting

Casting from one type to another allows you to control interpretation.


```

## GDScript, Variant, and Union

Required: C++ knows location/size at compile time.

How does GDScript work then?

