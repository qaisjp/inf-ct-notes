# LLVM lecture 3

_2018-11-20_

**Do not optimise STOREs anywhere.**

## DCE and Volatile Variables

Volatile is bad. It tells the compiler not to optimize an expression and to keep it in memory (on the stack or in the heap).

The compiler doesn't know why the programmer declared the variable volatile, and must be conservatives.

**There's gonna be some volatility tests — don't remove those lines!!!!!**

```
int foo(int x, int y) {
  volatile int a = x + y; // !!!!
  b = a;
  return x;
}
```

Keep the `load volatile` & `store volatile`. Use `llvm::instruction::mayHaveSideEffects()`.

**Do not optimise STOREs anywhere. Volatile or not, it needs a different process.**

> Volatile is nasty. But where's

## DCE and Control Flow

`-simplifycfg` will eliminate "dead basic blocks" but it's not this coursework. It's a different process!

```
int foo(int x, int y) {
  int a = x + y;
  if (x > 0)
    a = 1; // !!!!
  return y;
}
```

## Eliminating Dead Code in LLVM

> This section is copied from the slides


- Look at instruction.def for the possible instructions
  - https://github.com/llvm-mirror/llvm/blob/master/include/llvm/IR/Instruction.def
- Do not remove
  - control flow (ReturnInst, SwitchInst, BranchInst, IndirectBrInst, CallInst)
    - llvm::Instruction::IsTerminator()
  - stores (StoreInst)
    - llvm::Instruction::mayHaveSideEffects()
- Do remove
  - AllocaInst, LoadInst, GetElementPtrInst
  - SelectInst, ExtractElementInst, InsertElementInst, ExtractValue, InsertValue
  - binary instructions
  - comparisons
  - casts
- How to eliminate dead code?
  - Compute the OUT set for each instruction (liveness)
  - If the virtual register defined by an instruction is not in the OUT set, remove it!
    - llvm::Instruction::eraseFromParent()
  - Iterate until there are no changes


```
x = 1;
^
|
DEF(X)
~~OUT(phi or sigma or something i can't read greek symbol)~~
OUT(X)
```

```
A = 1     D(A)    O(A)
B = A     D(B)    O(B)
C = B     D(C)    O(?)
```

Backwards is better for some reason, "it will converge faster".

