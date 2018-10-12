# An Introduction to MIPS assembly
_2018-10-12_ - Lecture 10

## ToC

1. Overview
2. Registers
3. Instructions
  - Arithmetic
  - Memory
  - Control Structures
  - System Calls

## Assembly program template

`.data`

Data segment: constant and variable definitions go here (including statically allocated arrays)

- format for declarations: `name: storage_type value`
- create storage for variable of specified type with given name and value
- `var1:    .word 3   # one word of storage with initial value 3`
- `array1:  .space 40 # 40 bytes of storage for array1`

`.text`

Text segment: assembly instructions go here


## Components of an assembly program

| Category             | Example                    |
| -------------------- | -------------------------- |
| Comment              | `# Me!`                    |
| Assembler directives | `.data`, `.asciiz`         |
| Operation mnemonic   | `add`, `addi`, `lw`, `bne` |
| Register name        | `$t1`, `$zero`             |
| Address label (decl) | `loop1:`                   |
| Address label (use)  | `loop1`                    |
| Integer constant     | `8`, `-4`, `0xA9`          |
| Character constant   | `'h'`, `'\t'`              |
| String constant      | `"Hello world\n"`          |

## Hello world

```asm
# Description: a simple hello world program

.data

hellostr:   .asciiz "Hello world\n"

.text

li $v0, 4         # setup print syscall
la $a0, hellostr  # argument to print string
syscall           # tell OS to perform syscall
li $v0, 10        # setup exist syscall
syscall           # tell OS to perform syscall
```

## Registers

- 32 general purpose registers
- register preceded by `$` in assembly language
- two formats for addressing (name or number: `$zero` or `$0`)
- holds 32 bits value (= 4 bytes = 1 word)
- stack grows from high memory to low memory
