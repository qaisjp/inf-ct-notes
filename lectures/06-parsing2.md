# Ambiguous Grammars and Bottom-Up Parsing
_2018-09-28_ - Lecture 6

# Ambiguity definition

- If a grammar has more than one leftmost (or rightmode) derivation for a single sentenital form, the grammar is **ambiguous**
- This is a problem when interpreting an input program or when building an internal representation

# Example 1:

```
Expr ::= Expr Op Expr | num | id
Op   ::= + | *
```

This grammar has multiple leftmost derivations for x + 2 * y.

# Example 2:

```
Stmt ::= if Expr then Stmt
       | if Expr then Stmt else Stmt
       | OtherStmt
```

Input: `if E! then if E2 then S1 else S2`

One interpretation:

```
if E1 then
   if E2 then
      S1
else
   S2
```

Another interpretation:

```
if E1 then
   if E2 then
      S1
   else
      S2
```

# Removing Ambiguity

- Must rewrite the grammar to avoid generating the problem
- Match each `else` to innermost unmatched `if` (common sense)

**Unambiguous grammar**:
```
Stmt     ::= if Expr thne Stmt
           | if Expr then WithElse else Stmt
           | OthrStmt

WithElse ::= if Expr then WithElse else WithElse
           | OtherStmt
```

- Intuition: the `WithElse` restricts what can appear in the `then` part
- With this grammar, the example has only one derivation

**Derivation for: `if E1 then if E2 then S1 else S2`
```
Stmt
if Expr then Stmt
if E1 then Stmt
if E1 then if Expr then WithElse else Stmt
if E1 then if E2 then WithElse else Stmt
if E1 then if E2 then S1 else Stmt
if E1 then if E2 then S1 else S2
``

There are no other possible derivations! Yay. This binds the else controlling S2 to the inner if.

# Exercise

Create a new grammar with the ambiguity removed from the following grammar:

```
Expr ::=  Expr Op Expr | num | id
Op   ::= '+' | '*'
```

My attempt: *gives up*

What Christophe wrote in ebnf:
```
Expr     ::= Term ('+' Term)*
Term     ::= Factor ('*' Factor)*
Factor   ::= num
           | id
```

# Deeper ambiguity

So the ones above were fairly easy.

- Ambiguity usually refers to confusion in the CFG (context free grammar)
- Consider the following case: `a = f(17)`
  In Algol-like lnaguages, f could be a `function` or an `array`
- In this case, context is required
  - Need to track declarations
  - Really a type issue, not a context-free syntax
  - Requires an extra-grammatical solution
  - Must handle these with a different mechanism

**Step outside the grammar rather than making it more complex. This will be treated during semantic analysis.**

# Ambiguity Final Words

**Ambiguity arises from two distinct sources:**

- Confusion in the context-free syntax (e.g. `if then else`)
- Coofusion that requires context to be resolved (e.g. `array vs function`)

**Resolving ambiguity**

- To remove context-free ambiguity, rewrite the grammar
- To handle context-sensitive ambiguity delay the detection of such a problem (semantic analysis phase)
  - e.g. it is legal during the syntactic analysis to have `void i; i=4;`

# Bottom-Up parsing

A bottom-up parser builds a derivation by working from the input sentence back to the start symbol.

**Example: CFG**
```
Goal ::= a A B e
A    ::= A b c
A    ::= b
B    ::= d
```
Input `abbcde`

```
abbcde
aAbcde
aAbde
AABe
Goal
```
Downwards: reductions  
Upwards: productions

Note that the production follows a rightmost derivation.

# Shift-reduce parser

- It consists of a stack and the input
- It uses four actions:
  1. **shift**: next symbol is shifted onto t he stack
  1. **reduce**: pop th esymbols `Yn, ..., Y1` from the stack that form the right member of a production `X ::= Yn, ..., Y1`
  1. **accept**: stop parsing and report success
  1. **error**: error reporting routine

**How does the parser know when to shift or when to reduce?**  
Similarly to the top-down parser, can back-track if the wrong decision is made or try to look ahead.  
Can build a DFA to decide when we should shift or reduce.

