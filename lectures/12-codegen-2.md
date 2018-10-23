# Code generation
_2018-10-23_ - Lecture 12

1. Code Shapes
  - Boolean and Relational Values
  - Control-Flow
2. Memory management
  - Static vs Dynamic
  - Data structures
3. Function calls

# Boolean and Relational Values

Representing `(x < 10 && y > 3)` depends on the target machine.

Several approaches:

- numerical representation
- positional encoding (e.g MIPS assembly)
- conditional move and predication

Correct choice depends on both context and ISA (Instruction Set Architecture)

# Numerical Representation

- Assign values to `true` and `false`, usually 1 and 0
- Use comparison operator from the ISA to get a value from a relational expression

`x < y` is `cmp_LT rx, ry   -> r1`

So for:

``c
if (x < y)
  stmt1
else
  stmt2
```

The equivalent is:

```asm
    cmp_LT  rx, ry   -> r1
    cbr     r1       -> L1
    stmt2
    br               -> Le
L1: stmt1
Le: 
```

# Positional Encoding

**What if the ISA doesn't provide comparison operators that return a value?**

- Must use conditional branch to interpret the result of a comparison
- Necessitates branches in the evaluation
- This is the case for MIPS assembly (and Java ByteCode for instance)

**Example: `x < y`**

```asm
    br_LT rx, ry    -> Lt
    loadl 0         -> r1
    br              -> Le
Lt: loadl 1         -> r1
Le: ...
```

If the result is used to control an operation, then positional encoding isn't terrible!

For example:

```c
if (x < y)
  a = c + d;
else
  a = e + f;
```

With boolean comparison (6 lines):

```asm
    cmp_LT  rx, ry    -> r1
    cbr     r1        -> Lt
    add     re, rf    -> ra
    br                -> Le
Lt: add     rc, rd    -> ra
Le: ...
```

Positional encoding (5 lines!):

```asm
    br_LT   rx, ry    -> Lt
    add     re, rf    -> ra
    br                -> Le
Lt: add     rc, rd    -> ra
Le: ...
```

We've saved on a register, which is great!

# Conditional Move and Predication

Conditional move (`cmov`) and predication (instructions run based on a predicate) can simplify this code.

**With conditional move:**

```asm
    cmp_LT  rx, ry    -> r1
    add     rc, rd    -> r2
    add     re, rf    -> r3
    cmov  r1, r2, r3  -> ra
```

So `cmov r1, r2, r3` is:

```
if r1
  ra = r2
else
  ra = r3
```

**With predicated execution:**
```asm
        cmp_LT  rx, ry  -> r1
(r1)?   add     rc, rd  -> ra
(!r1)?  add     re, rf  -> ra
```

## Finally, consider `x = (a < b) & (c < d)`.

**Positional encoding**
```markdown
    Positional encoding     |  Boolean comparison
---------------------------------------------------
    br_LT   ra, rb  -> L1   |
    br              -> L2   |
L1: br_LT   rc, rd  -> L3   |  cmp_LT ra, rb  -> r1
L2: loadl   0       -> rx   |  cmp_LT rc, rd  -> r2
    br              -> Le   |  and    r1, r2  -> rx
L3: loadl   1       -> rx   |
Le: ...                     |
```

Here's the boolean comparison produces much better code, obviously!

**Best choice depends on two things**
- Context
- Hardware

# Control Flow

## Subsections
- If-then-else
- Loops (for, while, ...)
- Switch/case statements

## If-then-else
_Follow the model for evaluating relational and boolean with branches._

Branching versus predication (e.g. IA-64, ARM ISA) trade-off:

- Frequency of execution:
  uneven distribution, try to speedup common case
- Amount of code in each case:
  unequal amounts mean predication might waste issue slots
- Nested control flow:
  any nested branches complicates the predicates and makes branching attractive

## Loops

<table>
<tr>

<td>
<strong>The basic pattern</strong><br>
<pre>
          |
          v
p----[ Pre-test ]
|         |
|         v
|    [ Loop body ] <--p
|         |           |
|         v           |
|    [ Post-test ]----b
|         |
|         v
b--> [ Next block ]  
          |
          v
</pre></td>

<td>
  <ul>
    <li>evaluate condition before the loop (if needed)</li>
    <li>evaluate condition after the loop</li>
    <li>branch back to the top (if needed)
  </ul>
  <br>
  <strong>while</strong>, <strong>for</strong> and <strong>do while</strong> loops all fit this model.
</td>

</tr>
</table>

**Example: for loop**
```asm
for (i = 1; i < 100; i++) {
  body
}
next stmt
```

**Corresponding assembly**

```asm
    loadl 1       -> r1
    loadl 100     -> r2
    br_GE r1, r2  -> L2
L1: body
    addl  r1, 1   -> r1
    br_LT r1, r2  -> L1
L2: next  stmt
```

**Exercise**

Write the assembly code for the following while loop:

```asm
... subbed lecturer is bad
```

Most modern programming languages include a `break` statement.

- Exists from the innermost control-flow statement
  - Out of the innermost loop
  - Out of a case statement
- Solution:
  - use an unconditional branch to the next statement following the control-flow construct (loop or case statement)
  - `skip` or `continue` statement branch to the next iteration (start of the loop)

## Case Statement (switch)

**Case statement**
```c
switch (c) {
  case 'a': stmt1;
  case 'b': stmt2;  break;
  case 'c': stmt3;
}
```

1. Evaluation to the controlling expression
2. **Branch to the selected case** (important!)
3. Execute the code for that case
4. Branch to the statement after the case

Stategies:
- Linear search (nested if-then-else)
- Build a table of case expressions and use a binary search on it
- Directly compute an address (requires dense case set)
