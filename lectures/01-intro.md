# Introduction 
_2018-09-18_ - Lecture 1

## Edge cases to account for:

**In general, check against gcc.***

- empty files
- lexer:
  - integer literals include multiple integers, like `00`
  - string literals:
    - shouldn't go onto multiple lines (shouldn't contain raw the single character `\n`)
    - Escape sequences are fixed: https://docs.oracle.com/javase/tutorial/java/data/characters.html
  - `returner`/`tif`/`felser` should not trigger `return`/`if`/`else`
  - `2+2` and `2 +2` and `2 + 2` are the same
  - Watch out, `2 >! false` and `2>!0` are valid. (it's `2 > 1`)
  - `/*/*/` is entirely one comment. `/* */` and `/* _*/` need checks (odd/even problem)
  - line number reporting:
    - `#incLUDE "minic.h"` should mark an error at `#` (`incLUDE` will be an ID)
