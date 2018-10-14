## Introduction to Lexical Analysis
_2018-09-21_ - Lecture 3

**The lexer: a recap**

- Maps a character stream into words, it's the basic unit of syntax
- Assigns a syntactic _category_ to each word (POS, part of speech)
  - `x = x + y;` becomes `ID(x) EQ ID(x) PLUS ID(y) SC`
  - word ~= lexeme (~= kind of equals)
  - syntactic category ~= part of speech
  - In casual speech, we call the pair a token
- Typical token: number, ID, +, -, new, while, if, ...
- Scanner eliminates white space (include comments)

### Context-free language

Context-free syntax is spec'd with a grammar

- `SheepNoise -> SheepNoise baa | baa`
- This grammar defines the set of noises that a shee; makes under normal circumstances
- It's written in a variant of BNF (Backus-naur form)

Formallay, a grammar `G = (S,N,T,P)`
- S is the start symbol
- N is a set of non-terminal symbols
- T is a set of terminal symbols or words
- P is a set of productions or rewrite rules (P:N -> N ∪ T)

**Exercise: write the grammar of a signed natural number**

My maybe correct solution:

```
number :: [sign] digit+
digit :: 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9
sign :: + | -
```
