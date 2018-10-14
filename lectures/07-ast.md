# Abstract Syntax
_2018-10-02_ - Lecture 7

1. Syntax Tree
  - Semantic Actions
  - Examples
  - Abstract Grammar
2. Abstract Syntax Tree
  - Internal Representation
  - AST Builder
3. AST Processing
  - Object-Oriented Processing
  - Visitor Processing
  - AST Visualisation

A parser does more than just recognise syntax, it can:
  - evaluate code (interpreter)
  - emit code (simple compiler)
  - build an internal representation of the program (multi-pass compiler)

A parser _generally_ performs semantic actions:
  - **recrusive descent parser**: integrate the actions with the parsing functions
  - **bottom-up parser** (automatically generated): add actions to the grammar

# Syntax Tree

In a multi-pass compiler, the parser builds a syntax tree which is used by the subsequent passes.

A syntax tree can be:
  - **a concrete syntax tree** (or parse tree) if it directly corresponds to the context-free grammar
  - **an abstract syntax tree** if it corresponds to a simplified (or abstract) grammar

The second, an AST, is usually used in compilers.

# Abstract Syntax Tree

The Abstract Syntax Tree (AST) forms the main intermediate representation of the compiler's front-end
