---
description: What Happens at Every Stage
---

# From C++ Source Code to Execution

## From C++ Source Code to Execution: What Happens at Every Stage

One of the best ways to understand C++, operating systems, reverse engineering, and exploit development is to understand what actually happens between writing a `.cpp` file and the CPU executing your program.

Consider the following program:

```cpp
#include <iostream>

#define PI 3.14

int square(int x) {
    return x * x;
}

int main() {
    int a = 5;
    std::cout << square(a) << std::endl;
}
```

Although it looks simple, this program passes through multiple stages before it runs.

***

## The Complete Pipeline

```
              Human Readable Source Code
                        |
                        |
                  hello.cpp
                        |
                        V
              --------------------
              |  Preprocessor    |
              --------------------
                        |
                   hello.i
                        |
                        V
              --------------------
              |    Compiler       |
              --------------------
                        |
                   hello.s
                        |
                        V
              --------------------
              |    Assembler      |
              --------------------
                        |
                   hello.o
                        |
                        V
              --------------------
              |      Linker       |
              --------------------
                        |
                    hello
                        |
                        V
              --------------------
              | Operating System  |
              --------------------
                        |
                        V
                  CPU Executes
```

Each stage has a completely different responsibility.

***

## Stage 1 - Preprocessing

Command:

```bash
g++ -E hello.cpp -o hello.i
```

### What is the Preprocessor?

The preprocessor is **not** the compiler.

Instead, it performs simple text processing before compilation begins.

It handles directives such as:

```cpp
#include
#define
#ifdef
#ifndef
#if
```

For example:

```cpp
#define PI 3.14

std::cout << PI;
```

becomes

```cpp
std::cout << 3.14;
```

Notice that the preprocessor simply replaces text.

It does **not** understand C++.

***

### Header Expansion

Suppose your program contains:

```cpp
#include <iostream>
```

The preprocessor literally opens the `iostream` header and copies its contents into your file.

That header itself includes other headers.

Those headers include more headers.

Eventually your tiny program may expand into tens of thousands of lines.

Conceptually:

```
hello.cpp

↓

Copy iostream

↓

Copy everything iostream includes

↓

Copy everything those headers include

↓

Produce hello.i
```

***

### Comment Removal

Comments are removed completely.

```cpp
// This comment disappears
```

The compiler never sees comments.

***

### Conditional Compilation

The preprocessor can also include or exclude parts of your program.

```cpp
#ifdef DEBUG
std::cout << "Debug Mode";
#endif
```

If `DEBUG` is not defined, this code simply disappears.

***

### Output

The result is:

```
hello.i
```

This is still C++ source code, but with:

* Headers expanded
* Macros replaced
* Comments removed
* Conditional compilation completed

***

### Important

The preprocessor does **not** understand:

* Variables
* Functions
* Classes
* Templates
* Objects

It is essentially a sophisticated text replacement engine.

***

## Stage 2 - Compilation

Command:

```bash
g++ -S hello.i -o hello.s
```

Now the actual compiler begins.

This is where the compiler understands the C++ language.

Internally, several important steps occur.

***

### Parsing

Suppose you write:

```cpp
int x = 5;
```

The compiler no longer sees plain text.

Instead, it constructs an Abstract Syntax Tree (AST).

Conceptually:

```
VariableDeclaration
    Type = int
    Name = x
    Initializer = 5
```

This structured representation makes it easier for the compiler to analyze your program.

***

### Semantic Analysis

The compiler verifies whether your program is valid.

Examples:

Valid:

```cpp
std::cout << x;
```

Invalid:

```cpp
x + "hello";
```

Invalid:

```cpp
return "abc";
```

from an `int` function.

This stage checks:

* Types
* Function signatures
* Scope
* Variable declarations
* Language rules

***

### Optimization

When optimization is enabled , the compiler attempts to improve performance.

Example:

```cpp
int x = 5;
int y = x + 0;
```

can become:

```cpp
int y = 5;
```

Small functions may also be inlined.

Instead of:

```cpp
square(5);
```

the compiler may directly generate:

```cpp
5 * 5
```

without making a function call.

***

### Code Generation

Finally, the compiler converts your C++ program into assembly language.

Example:

```cpp
int square(int x) {
    return x * x;
}
```

might become:

```asm
square:
    mov eax, edi
    imul eax, edi
    ret
```

The exact instructions depend on your CPU architecture.

***

### Output

The compiler produces:

```
hello.s
```

This is human-readable assembly language.

***

## Stage 3 - Assembly

Command:

```bash
g++ -c hello.s -o hello.o
```

Now the assembler takes over.

The assembler converts assembly instructions into machine code.

For example:

```asm
mov eax, 5
```

becomes binary instructions similar to:

```
B8 05 00 00 00
```

The CPU only understands binary machine instructions.

It does **not** understand:

* C++
* Variables
* Functions
* Classes
* Assembly mnemonics

***

### The Object File

The assembler creates:

```
hello.o
```

An object file contains much more than machine code.

Typical sections include:

#### `.text`

Machine instructions.

***

#### `.data`

Initialized global variables.

Example:

```cpp
int x = 5;
```

***

#### `.bss`

Uninitialized global variables.

Example:

```cpp
int counter;
```

***

#### `.rodata`

Read-only data.

Example:

```cpp
"Hello World"
```

String literals usually live here.

***

### Symbols

Suppose you call:

```cpp
square();
```

but the implementation exists in another file.

The object file records:

> I need a function named `square`.

It does **not** know where that function lives.

That responsibility belongs to the linker.

***

## Stage 4 - Linking

Command:

```bash
g++ hello.o -o hello
```

The linker combines everything together.

Imagine two files:

**main.cpp**

```cpp
int square(int);

int main() {
    square(5);
}
```

**math.cpp**

```cpp
int square(int x) {
    return x * x;
}
```

Each source file is compiled independently.

The linker connects them.

Conceptually:

```
main.o
Needs:
square()
↓
math.o
Provides:
square()
↓
Linker
↓
Executable


```

#### Static vs Dynamic Linking

```
Static

Executable
|
+-- printf()
+-- cout()
+-- malloc()

Everything copied inside
----------------------------
Dynamic

Executable
|
+-- libc.so
+-- libstdc++.so

Resolved by loader
```

***

### Libraries

Your program also uses:

```cpp
std::cout
```

Where is it defined?

Not inside your source code.

It lives inside the C++ Standard Library.

The linker connects your executable with that library.

Without linking you would see errors such as:

```
undefined reference to std::cout
```

***

### Relocation

During compilation, the compiler does not know where functions will be placed.

For example:

```
call square
```

The linker eventually decides that:

```
square

↓

0x401240
```

and updates every function call accordingly.

This process is called **relocation**.

***

### Output

The linker produces:

```
hello
```

A complete executable.

***

## Stage 5 - Program Execution

Command:

```bash
./hello
```

Now the operating system takes control.

***

### The Loader

The operating system loads your executable into memory.

Different sections are mapped into different memory regions.

```
.text     -> Executable instructions
.data     -> Writable initialized variables
.bss      -> Writable zero-initialized variables
.rodata   -> Read-only constants
```

***

### Runtime Initialization

Before `main()` executes, the C++ runtime performs initialization.

This includes:

* Global variables
* Static variables
* Global constructors
* Exception handling
* Heap initialization
* Thread-local storage
* Standard library initialization

Only after this does your program reach:

```cpp
int main()
```

***

### CPU Execution

The CPU repeatedly performs the following cycle:

```
Fetch Instruction

↓

Decode Instruction

↓

Execute Instruction

↓

Repeat
```

For example:

```asm
mov eax, 5
add eax, 2
ret
```

The CPU executes one instruction after another.

At this point, there is no C++ anymore.

Only machine instructions remain.

***

## Putting Everything Together

```
hello.cpp
```

Human-readable C++ source code.

↓

```
hello.i
```

Preprocessed source code.

Headers expanded.

Macros replaced.

Comments removed.

↓

```
hello.s
```

Assembly generated by the compiler.

↓

```
hello.o
```

Machine code with:

* Symbols
* Relocations
* Metadata
* Debug information (optional)

↓

```
hello
```

Fully linked executable.

↓

```
Operating System Loader
```

Creates a process.

Maps memory.

Loads required libraries.

Initializes the runtime.

↓

```
main()
```

Execution begins.

***

## Why This Matters

Understanding each stage is essential for systems programming and cybersecurity.

| Stage        | Why It Matters                                                                  |
| ------------ | ------------------------------------------------------------------------------- |
| Preprocessor | Macro expansion, conditional compilation, source-level obfuscation              |
| Compiler     | Optimizations, debugging behavior, code generation                              |
| Assembler    | Relationship between assembly and machine code                                  |
| Object Files | Symbols, sections, relocations used during reverse engineering                  |
| Linker       | Static vs dynamic linking, imports, exports, relocation                         |
| Loader       | Process creation, virtual memory, ASLR, DEP/NX                                  |
| Runtime      | Stack initialization, heap setup, global constructors, transition into `main()` |

