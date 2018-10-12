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

[See full list here](https://www.inf.ed.ac.uk/teaching/courses/ct/18-19/slides/10-mips-assembly.pdf#page=7).

## Arithmetic instructions

- Most use three operands
- All operands are registered (no memory access)
- All operands are 4 bytes (word)

[See many examples here](https://www.inf.ed.ac.uk/teaching/courses/ct/18-19/slides/10-mips-assembly.pdf#page=9).

## Load/Store Instructions

- Memory access only allowed with explicit load and store instructions (load/store architecture)
- All other instructions use register operands
- Load
  - `lw   register_denitation, mem_source`
    copy a word (4 bytes) at source memory to destination register
  - `lb   register_destination, mem_source`
    copy a byte to low-order byte of destination register (sign extend higher-order bytes)
  - `li   register_destination, value`
    load immediate value into destination register
- Store
  - `sw   register_source, mem_destination`
    store a word (4 bytes) from source register to memory location
  - `sb   register_source, mem_destination`
    store a byte (low-order) from source register to memory location

_Example_

```asm
.data
var1: .word 23  # declare storage for var1; initial value 23

.text
lw $t0, var1    # load contents of memory location into register $t0: $t0 = 23
li $t1, 5       # $t1 = 5 (load immediate)
sw $t1, var1    # store contents of $t1 into mem: var1 = 5
```
## Indirect and Based Addressing

- load address
  - `la $t0, var1`
    copy memory address of var1 into register `$t0`
- indirect addressing:
  - `lw $t1, ($t0)`
    load word at memory address contained in `$t0` into `$t2`
  - `sw $t2, ($t0)`
    store word in register `$t2` into memory at address contained in `$t0`
- based/indexed addressing (useful for **field access in struct**):
  - `lw $t2, 4($t0)`
    load word at memory address (`$t0+4`) into register `$t2`
  - `sw $t2, -12($t0)`
    store content of register `$t2` into memory at address `($t0-12)`

## Exercise

Write the assembly program corresponding to the following C code:

```asm
struct point_t {
    int x ;
    int y ;
}

struct point_t p ;
int arr [12];

void main () {
    p.x = 2;
    p.y = 4;
    arr [3] = 6;
}
```

### My solution

```asm
.data

arr:        .space  48  # allocate 12 integers, 12*4 
point_t_p:  .space  8   # allocate two integers (first is x, first is y)

.text
                        # p.x = 2;
la $t0, point_t_p       # $t0 = &point_t_p
li $t1, 2               # $t1 = 2
sw $t1, ($t0)           # *$t0 = $t1

                        # p.y = 4;
li $t1, 4               # $t1 = 4
sw $t1, 4($t0)          # *($t0+4) = $t1

                        # arr[3] = 6;
la $t0, arr             # $t0 = &arr        (*)
li $t1, 6               # $t1 = 6
sw $t1, 12($t0)         # *($t0+12) = $t1

# Note(*)
# We reuse $t0 because we are smart humans.
# The compiler won't know, and so might want to use another register.
```
