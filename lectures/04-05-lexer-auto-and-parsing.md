## Automatic Lexer Generation and Top-down Parsing
_2018-09-25_ - Lecture 4, 5

Starting from a collection of regular expressions we can automatically generate a Lexer. An FSA is used for the construction.

It is possible to systematically generate a lexer for any regex. Here are the three steps:

1. regular expression (RE) -> non-deterministic finite automata (NFA)
2. NFA -> DFA

   The Subset Construction algorithm (in English)
    - Start from the start state `s0` of the NFA, compute its ϵ-closure
    - Build subset from all states reachable from `q0` for character `α`
    - Add this subset to the transition table/function `δ`
    - If the subset has not been seen before, add it to the worklist
    - Iterate until no new subset are created
3. DFA -> generated lexer
