# inf-ct on [inf.ed.ac.uk](https://www.inf.ed.ac.uk/teaching/courses/ct/18-19/)

## Corner cases to account for:

- empty files

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
--(IR)--> Instruction Selection --(IR)--> Register allocation --(IR)--> Instruction scheduling --(Machine code)-->

Each stage can produce an errors.
```

- Choose instructions to implement each IR operations
- Decide which values to keep in registers
- Ensure conformance with system interfaces

Automation has been less succesful in the backend :(

**The coursework will only contain Instruction Selection. The coursework doesn't care about the other two here.**

**The aim is to produce _correct_ machine code.**
