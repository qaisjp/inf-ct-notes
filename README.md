# inf-ct on [inf.ed.ac.uk](https://www.inf.ed.ac.uk/teaching/courses/ct/18-19/)

## Edge cases to account for:

- empty files
- lexer:
  - integer literals include multiple integers, like `00`
  - string literals shouldn't go onto multiple lines
  - `returner`/`tif`/`felser` should not trigger `return`/`if`/`else`
  - `2+2` and `2 +2` and `2 + 2` are the same
  - Watch out, `2 >! false` and `2>!0` are valid. (it's `2 > 1`)

# Notes

## Lecture 1: Introduction

_2018-09-18_

Corner case reported

## Lecture 2: The View from 35,000 Feet

_2018-09-18_

### High level view of a compiler

```
Source code -> [Compiler] -> Machine code
                   |
                   |---> Errors
```

Must:

- recognise legal/illegal programs
- generate "correct" code
- storage of all variables (and code)
- Agree with OS & linker on format for obj code

It's a big step-up from assembly language so you can use higher level notations

### Traditional two-pass compiler

```
Source -> Frontend --(IR)--> Backend -> Machine code
             |                |
             |----------------|-----> Errors
```

- **IR**: intermediate representation
- **Frontend**: map legal src code to IR
- **Backend**: IR to machine code

Multiple frontend to the backend.

### Frontend

```
                     Lexer
            _______________________
            |                     |
Source -> Scanner --(char)--> Tokeniser --(token)--> Parser --(AST)--> Semantic Analyser --(AST)--> IR Generator --(IR)-->

Each stage can generate errors
```

**Lexer** for lexical analysis:
- Recognises words in a char stream
- Produces tokens (words, such as: `number` `identifier` `+` `-` `new` `while` `if`) from lexeme
- Collect identifier info
- Eliminates whitespace and comments
- Example:
  - `x=y+2` becomes
  - `IDENTIFIER(x) EQUAL IDENTIFIER(y) PLUS CST(2)`

**Parser**:
- Recognises **context-free** syntax & reports errors
- Hand-coded parsers are fairly easy to build
- Most books advocate using automatic parser generators

Mapping source like `gravity * 2` into an abstract syntax tree.

**Semantic analyser**:
- Semantic = context-sensitive
- Example: checks var/func declared before use
- Example: type checking

**IR generator**:

Generates the IR used by the rest of the compiler. Sometimes the AST is the IR.

### Backend (IR to machine code)

```
--(IR)--> Instruction Selection --(IR)--> Register Allocation --(IR)--> Instruction Scheduling --(Machine code)-->

Each stage can produce an errors.
```

- Choose instructions to implement each IR operations
- Decide which values to keep in registers
- Ensure conformance with system interfaces

Automation has been less succesful in the backend :(

**The coursework will only contain Instruction Selection. The coursework doesn't care about the other two here.** The aim is to produce _correct_ machine code. **The time limit is 10 seconds for your compiler.**

**Instruction Selection**:

- Produce fast, compact code.
- Take advantage of target features like addressing modes
- Usually viewed as a _pattern matching_ problem
- adhoc methods, pattern matching, dynamic programming
- Example: madd instruction

##### madd instruction

multiplier accumulator. can multiply and add at the same time (add is free!!!!)

`x * y + z` is just ONE op :)

It takes three parameters.

But if you have `x * y` somewhere else, you might choose to figure out `x * y` once and just add `z` that one time.

Of course in this particular case both ways will result in just two operations. (`x * y + z`, `x * y` VS `x * y`, `+ z`)

**Register Allocation**:

- Have each val in a register when used.
- Manage a limited set of resources.
- Changes instruction choices, inserts `LOAD`s and `STORE`s (this is called spilling).
- Optimal allocation is NP-Complete (1 or k registers). Compiler approximate solutions to this problem.

**Instruction Scheduling**:

- Avoid hardware stalls and interlocks
- Optimal scheduling is NP-Complete in nearly all cases
- There are well developed techniques
- Can increase lifetime of vars by changing the allocation.

### Between the frontend and backend

This is the subject of UG4 Compiler Optimisation.

- Optimisation.
- Analyses IR and rewrites/transforms IR
- Reduce running time (or improve space, power consumption)
- **Must preserve meaning of the code** ("measured by values of named variables")

### Optimiser (after backend)

Optimises away branches like

```go
if (false) {
    // do something
}
```

Constant expressions

```cpp
int x = 3

// some code that doesn't touch x

int y = x + 2 // we know this is 5, compile-time

run(y + 4) // we know this is 9, compile-time
```

Reduce cost of operations

```cpp
int x, y;

y = x^2 // becomes x * x

y = 2 * x // becomes x << 1
```

