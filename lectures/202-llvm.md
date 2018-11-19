# LLVM - Lecture 2

_2018-11_16_

## Data Flow Analysis

Liveness flows backwards. That's where data-flow comes from.

## Finding and Removing Instructions

`SmallVector<Instruction*. ??>` not `SmallVector<PHINode*, 16>`.

- Run the "search & destroy" loop multiple times. "after destroying some dead code, it might make more dead code!"

First coursework: COPIED FROM WHITEBOARD

```
SmallVector<Instruction*, 16> ul;
  
while changed {
  changed = false
  for (instruction i) {
    if i.isTriviallyDead() {
      changed = true
      ul.push(i)
    }
  }
  
  while (!ul.empty()) {
    node = ul.pop();
    node.replaceAllwith(node.getCurrentValue(0)) // i think?
    nodeblah
}
