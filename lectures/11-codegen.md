# Introduction to Code Generation
_2018-10-16_ - Lecture 11

1. Introduction
  - Overview
  - The Backend
  - The Big Picture
2. Code Generation

# Overview

**Frontend**  
- Lexer
- Parser
- AST builder
- Semantic Analyser

**Middle-end**  
- Optimizations (Compiler Optimizations course)


**Backend**  
- Translate IR into target machine code
- Choose insturctions to implement each IR operation
- Decide which value to keep in registers
- Ensure conformance with system interfaces
- Automation has been less successful in the backend

# Instruction Selection

- Mapping the IR into assembly code (in our case AST to MIPS assembly)
- Assumes a fixed storage mapping & code shape
- Combining operations, using address modes

# Register Allocation

- Deciding which value resides in a register
- Minimise the amount of spilling

**Spilling**

If we have a lot of different values we might not have enough registers and will need to put things onto the stack. 

# Instruction Scheduling

- Avoid hardware stalls and interlocks
- Reordering operations to hide latencies
- Use all functional units productively

**Instruction scheduling is an optimisation**  
_It improves code quality and isn't strictly required_

# The Big Picture

_How hard are these problems?_

- Instruction Selection
  - Can make locally optimal choices with an automated tool
  - Global optimality is NP-Complete
- Instruction scheduling
  - Single basic block => heuristic work quickly
  - General problem, with control flow => NP-Complete
- Register allocation
  - Single basic block, no spilling => linear time
  - Whole procedure is NP-Complete (graph colouring algorithm)

**NP-Complete**
> A problem is called NP (nondeterministic polynomial) if its solution can be guessed and verified in polynomial time;
> nondeterministic means that no particular rule is followed to make the guess.
>
>If a problem is NP and all other NP problems are polynomial-time reducible to it, the problem is NP-complete.

**These three problems are tightly coupled!**  
However, conventional wisdom says we lose little by solving these problems independently.

_How do we solve these problems?_

- Instruction Selection
  - Use some form of pattern matching
  - Assume enough registers or target _"important"_ values
- Instruction scheduling
  - Within a block, list scheduling is _"close"_ to optimal
  - Across blocks, build framework to apply list scheduling
- Register allocation
  - Start from virtual registers & map _"enough"_ into `k`
  - With targeting, focus on _"good"_ priority heuristic

**Approximate solutions**  
Will be important to define good metrics for "close", "good", "enough", ...

Some compilers focus on specific things or have things hidden behind flags. Think "minimise RAM use", "minimise instructions", "less power intensive", etc.

# Generating Code for Register-Based Machine

The key code quality issue is holding values in registers

- when can a value be safely allocated to a register?
  - When only 1 name can reference its value
  - Pointers, parameters, aggregate & arrays all cause trouble
  - **Usually scalars: ints and floats, but not pointers!**
- when should a value be allocated to a register?
  - when it is both _safe_ & _profitable_

Encoding this knowledge into the IR
  - assign a virtual register to anything that go into one
  - load or store the others at each reference

**Register allocation is key**  
All this relies on a strong register allocator.

# Register-based machines

- Most real physical machines are register-based (atleast for recent architectures)
- Instructions operate on registers
- The number of architecture register available to the compiler can vary from processor to processors
- The first phase of code generation usually assume an unlimited numbers of registers (virtual registers)
- Later phases (register allocator) converts these virtual register to the finite set of available physical architectural registers (more on this in the lecture on register allocation)

**In our coursework we'll always assume we have enough registers. You're welcome to attempt to implement a register allocator, but this _will not_ be tested.**

# [Generating code for Register-based machines](https://www.inf.ed.ac.uk/teaching/courses/ct/18-19/slides/11-codegen.pdf#page=12)

Whiteboard work.

# Exercise

**Write down the list of assembly instructions for `x+(y*3)`**

```asm
loadI @x      -> r1 // r1 = &x
loada r1      -> r1 // r1 = *r1

loadI @y      -> r2 // r2 = &y
loada r2      -> r2 // r2 = *r2

loadI 3       -> r3 // r3 = 3
mul   r2, r3  -> r2 // r2 = r2 * r3 (y*3)

add   r1, r2  -> r1 // r1 = r1 + r2 (x + (y*3))
```

# Exercise
**Assuming you have an instruction muli (_mul_tiply _i_mmediate), rewrite the previous example)

The last three instructions can be merged into one! It's hard to notice that `(y * 3)` can be just one instruction. You need the context! There's a whole lecture on this operation: **instruction selection**.

As far as our compiler is concerned we only want it to _work_ so we don't really need to worry about this.

# Visitor for Code Generation

After traversing a subtree, the visitor return the register that contains the results of evaluating the subtree.

The visitor uses two functions to manage registers:

- `Register nextRegister()`: returns the next available unused register
- `void freeRegister(Register r)`: frees up the register `r`

# Visitor impl for _binary operators_

_a naive implementation_

```java
// Imagine x + (y*3)
Register visitBinOp(BinOp binOp) {
  Register lhsReg = binOp.lhs.accept(this); // visit x
  Register rhsReg = binOp.rhs.accept(this); // visit y
  Register result = nextRegister();
  
  switch (binOp.op) {
    case ADD:
      emit(add lhsReg.id rhsReg.id -> result.id);
      break;
    ...
  }
  
  freeRegister(lhsReg, rhsReg);
  
  return result;
}
```

_Why free registers if we have an infinite amount of them?_  
In our implementation we won't have a register pass so we can't just use them all. The program given in tests will never exceed ~30 registers, it'll always have less. We would quickly run out of registers in the real architecture. Always try to free the register as soon as you can (no cost to this).

# Visitor impl for variables

```asm
loadl @x    -> r1 // load the address of x into r1
loadA r1    -> r2 // now teh value of x is in r2
```

```c
int foo(int x) {
  // use x somewhere
}
```

vs.

```c
int x;

int foo() {
  // use x somewhere
}
```

So, `x` could be somewhere on the stack (an offset from the frame pointer)... or it could be a register. Is it statically allocated (remember label)?

_a naive implementation_
```java
Register visitVar(Var v) {
  Register addrReg = nextRegister(); // alloacte a register
  Register result = nextRegister();
  emit(loadl v.address -> addrReg.id); // some details omitted here)
  emit(loadA addrReg.id -> result.id);
  
  freeRegister(addrReg);
  return result;
}
```

# Visitor impl for int literals

```asm
loadl 3   -> r1
```

```java
Register visitIntLiteral(IntLiteral it) {
  Register result = nextRegister();
  emit(loadl it.value -> result.id);
  return result;
}
```
