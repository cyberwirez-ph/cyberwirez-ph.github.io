---
title: "x86-64 Assembly, Condensed: Easy Notes & References for Learning Assembly"
date: 2026-07-04
description: "Beginner-friendly notes and references covering the fundamentals of x86-64 assembly."
author: "grit"
draft: false
---

*Originally published on [grit8086.github.io](https://grit8086.github.io/posts/condensed-assembly/)*

Notes and materials adapted from OpenSecurity’s x86-64 assembly course, reorganized here in text form since it’s easier for me to recall this way. An easy to absorb reference for understanding x86-64 assembly.

## Numerical Systems

### Decimal

A decimal number system is a base-10 system:


```
0 1 2 3 4 5 6 7 8 9
```


### Binary

“Bi” means two, it’s a base-2 system. **0** represents off, **1** is on. It’s the language computer hardware directly understands.

### Hexadecimal

A base-16 number system (“hexa” = 6, “decimal” = 10, when added it’s 16):


```
0 1 2 3 4 5 6 7 8 9  A  B  C  D  E  F
                     10 11 12 13 14 15
```


### Decimal to Binary (15)

Divide it by 2, write down the remainder, only use integer division, stop when the quotient becomes 0, read the remainders from bottom to top.


```
15 ÷ 2 = 7  r = 1
 7 ÷ 2 = 3  r = 1
 3 ÷ 2 = 1  r = 1
 1 ÷ 2 = 0  r = 1

= 1111b
```


### Decimal to Hexadecimal (15)

Same rules, but divide it by 16:


```
15 ÷ 16 = 0 remainder 15
15 = F in hexadecimal

= 0x0F
```


### Hexadecimal to Decimal (0x100)

List out powers of 16 left to right:

16²| 16¹| 16⁰  
---|---|---  
1| 0| 0  
  
(1 × 16²) + (0 × 16¹) + (0 × 16⁰)


```
= (1 × 256)
+ (0 ×  16)
+ (0 ×   1)
------------
= 256
```


### Hexadecimal to Binary (D)

Convert letters to numbers (if there’s any):


```
D = 13
```


Use this as a base:


```
8 4 2 1
```


8, 4, and 1 add up to 13:


```
8 4 2 1
1 1 0 1

= 1101
```


### Binary to Decimal & Hexadecimal (0001 0011 0011 0111)

Group the bits into four (nibbles), then represent the power of 2, from right to left:


```
+---+---+---+---+  +---+---+---+---+  +---+---+---+---+  +---+---+---+---+
| 0 | 0 | 0 | 1 |  | 0 | 0 | 1 | 1 |  | 0 | 0 | 1 | 1 |  | 0 | 1 | 1 | 1 |
+---+---+---+---+  +---+---+---+---+  +---+---+---+---+  +---+---+---+---+
|2³ |2² |2¹ |2⁰ |  |2³ |2² |2¹ |2⁰ |  |2³ |2² |2¹ |2⁰ |  |2³ |2² |2¹ |2⁰ |
+---+---+---+---+  +---+---+---+---+  +---+---+---+---+  +---+---+---+---+
```


List out the results of the power of 2 only if the corresponding binary digit is 1, and add them if there’s more than 1:


```
+---+---+---+---+  +---+---+---+---+  +---+---+---+---+  +---+---+---+---+
| 0 | 0 | 0 | 1 |  | 0 | 0 | 1 | 1 |  | 0 | 0 | 1 | 1 |  | 0 | 1 | 1 | 1 |
+---+---+---+---+  +---+---+---+---+  +---+---+---+---+  +---+---+---+---+
| X | X | X |2⁰ |  | X | X |2¹ |2⁰ |  | X | X |2¹ |2⁰ |  | X |2² |2¹ |2⁰ |
+---+---+---+---+  +---+---+---+---+  +---+---+---+---+  +---+---+---+---+
|   |   |   | 1 |  |   |   | 2 | 1 |  |   |   | 2 | 1 |  |   | 4 | 2 | 1 |
+---+---+---+---+  +---+---+---+---+  +---+---+---+---+  +---+---+---+---+
(1)                 (3)                (3)               (7)

Hex: 0x1337
```


To get the decimal value, multiply each nibble’s value by its place’s power of 16 and add them up:


```
(1 × 16³) + (3 × 16²) + (3 × 16¹) + (7 × 16⁰)
= 4096 + 768 + 48 + 7
= 4919

Decimal: 4919
```


## Two’s Complement & Negative Numbers

**Two’s complement** is the method computers use to represent negative numbers. To get the negative representation of a positive number, invert all of its bits, then add 1.

### Signed Types

Signed types are data types that can represent negative and positive numbers:


```c
int foo = 1;
```


### Unsigned Types

Unsigned types can only represent 0 and positive values:


```c
unsigned int foo = 1;
```


Both of these types have the same number of bits, but a signed type reserves half of the value range to represent negative numbers:


```
Unsigned:  0 ================================> 255
Signed:   -128 ==============|===============> +127
```


#### Example (0xFF)

Convert hexadecimal to decimal:

F = 15

| 16¹| 16⁰  
---|---|---  
Digit| 15| 15  
Place value| 16| 1  
  
(15 × 16) + (15 × 1) = 255

Convert it to binary:


```
255 ÷ 2 = 127 │ remainder 1
127 ÷ 2 =  63 │ remainder 1
 63 ÷ 2 =  31 │ remainder 1
 31 ÷ 2 =  15 │ remainder 1
 15 ÷ 2 =   7 │ remainder 1
  7 ÷ 2 =   3 │ remainder 1
  3 ÷ 2 =   1 │ remainder 1
  1 ÷ 2 =   0 │ remainder 1

= 11111111b
```


The bits stay the same, only their meaning changes. As an **unsigned char** , **11111111** is **255**. As a **signed char** , the same bits represent **-1**.

### Producing a Negative Value (0x0D)

**D** is 13. Convert it to binary:


```
(READ BOTTOM TO TOP)

13 ÷ 2 = 6  r = 1
 6 ÷ 2 = 3  r = 0
 3 ÷ 2 = 1  r = 1
 1 ÷ 2 = 0  r = 1

= 1101
```


Pad it with leading 0s to fill 8 bits (a **char** is 8 bits wide):


```
00001101
```


Now flip every bit and add 1, this is the two’s complement process that produces the negative value’s representation.

Two’s complement isn’t the process used _every time_ a computer sees a negative number, it’s how the negative number is _stored_ in memory.

**Flip:**


```
11110010
```


**Add 1:**


```
11110011
= -13
```


## Endianness

**Endianness** describes how multi-byte values are stored in memory.

Assume we have a 32-bit value:


```c
uint32_t x = 0xFEEDFACE;
```


Logically, the value is always:

MSB —————————–> LSB

Byte 1 (MSB)| Byte 2| Byte 3| Byte 4 (LSB)  
---|---|---|---  
0xFE| 0xED| 0xFA| 0xCE  
  
The value never changes, only the **order of the bytes in memory** changes.

### Little-Endian (x86/x64)

The least significant byte is stored at the lowest memory address:

Low Address High Address

Addr 0 (Low)| Addr 1| Addr 2| Addr 3 (High)  
---|---|---|---  
0xCE| 0xFA| 0xED| 0xFE  
  
### Big-Endian

The most significant byte is stored at the lowest memory address:

Low Address High Address

Addr 0 (Low)| Addr 1| Addr 2| Addr 3 (High)  
---|---|---|---  
0xFE| 0xED| 0xFA| 0xCE  
  
x86 and x64 processors use **little-endian**.

### Important

Endianness only affects the **order of bytes in memory**. It does **not** change the value stored in a register, or the order of _bits_ inside a byte. For example, **0xCE** is always **11001110** , regardless of endianness, only its position in memory changes.

## Computer Memory Hierarchy


```
                    /----------------\
                   /    Registers     \
                  /--------------------\
                         ↓
              Values needed for execution

                 /------------------------\
                /       CPU Cache          \
               /----------------------------\
                         ↓
             Frequently used data is cached

              /----------------------------\
             /            RAM               \
            /--------------------------------\
                         ↓
               Program is loaded into memory

           /----------------------------------\
          /         Disk (main.exe)            \
         /--------------------------------------\
```


Each layer is slower, cheaper, and larger than the one above it, disk holds everything but is slow, registers hold almost nothing but are instant.

## Assembly Registers

Processor registers are small, volatile storage areas built directly into the CPU, the fastest storage in the system, and where the CPU actually performs operations.

On x86-64, Intel CPUs have **16 general-purpose registers** , plus the instruction pointer (**RIP**), which points to the next instruction to execute. On x86-32 systems, registers are 32 bits wide, and there are only **8 general-purpose registers** plus the instruction pointer.

### Register Evolution and Sub-Registers

Registers were never replaced over time, they were extended. Older, smaller registers still exist as views into the newer, wider ones:

Register| Width| Processor / Notes  
---|---|---  
A| 8-bit| Intel 8008  
AX| 16-bit| Intel 8086 (“A-extended”)  
EAX| 32-bit| Intel 80386  
RAX| 64-bit| x86-64 (AMD Opteron / Intel P4+)  

```
RAX  (64-bit)
 `-- EAX (32-bit)
      `-- AX  (16-bit)
            |-- AL (low 8 bits)
            `-- AH (high 8 bits, legacy)
```


Sub-register access remains available: **AL** is the low 8 bits, **AH** is the high 8 bits (legacy, only exists for AX/BX/CX/DX).

### General-Purpose Registers (x86-64)


```
64-bit   32-bit   16-bit   8-bit

RAX      EAX      AX       AL / AH
RBX      EBX      BX       BL / BH
RCX      ECX      CX       CL / CH
RDX      EDX      DX       DL / DH
```


Disassemblers almost always use these historical names (**RAX** , **RBX** , etc.) rather than numeric identifiers, they’re easier to read and match older documentation.

Registers with conventional roles:


```
64-bit   32-bit   16-bit   8-bit     Convention

RSP      ESP      SP       SPL       Stack pointer
RBP      EBP      BP       BPL       Stack base frame
RSI      ESI      SI       SIL       Source index
RDI      EDI      DI       DIL       Destination index
```


**RSP** is special by ABI convention and should always point to the stack. Older x86 didn’t allow byte-level access to SP, AMD introduced **SPL** in x86-64 to make naming consistent. These registers still count as general-purpose at the hardware level, the compiler can technically repurpose them.

Extra registers added in x86-64, giving compilers more room to keep values in registers:


```
R8  R9  R10 R11
R12 R13 R14 R15
```


Each supports full-width access, e.g. **R8** (64-bit), **R8D** (32-bit), **R8W** (16-bit), **R8B** (8-bit).

**RIP** , the instruction pointer, points to the next instruction to execute. It’s not a general-purpose register, it’s modified indirectly via **call** , **jmp** , **ret** , etc.

### Why Access Smaller Parts of a Register?

Not every value is 64 bits, **char** is 8 bits, **short** is 16 bits, **int** is usually 32 bits. The CPU needs to operate on, store, and load these smaller sizes correctly.

**Example: 32-bit wraparound behavior**


```c
unsigned int x = 1; // On most systems unsigned int is 32 bits
x += UINT_MAX;       // UINT_MAX is 2^32 - 1
if (x)
    printf("Nope");
```


Mathematically, **1 + (2^32 - 1) = 2^32** , which wraps to **0**. If the CPU used a full 64-bit register without truncating, the result would incorrectly be **2^32**. By doing the arithmetic in a 32-bit register (**EAX**), the CPU automatically enforces the correct wraparound. Writing to a 32-bit register also automatically clears the upper 32 bits, this is intentional, and compilers rely on it heavily.

High-byte registers (**AH** , **BH** , **CH** , **DH**) exist mainly for backward compatibility and are rarely used in modern code. Lower-width access (**AL** , **AX** , **EAX**) is still essential and used constantly.

### Conventional Roles

Intel originally suggested usage conventions for registers, these are recommendations, not hard rules, compilers are free to use registers however they want:

  * **RAX** \- Accumulator / return value. Commonly holds function return values.
  * **RBX** \- Base register, originally a pointer to the data section.
  * **RCX** \- Counter register, often used as a loop counter (“C” for counter).
  * **RDX** \- Data / I/O register, historically I/O-related.
  * **RSI** \- Source index, source pointer in string/memory operations.
  * **RDI** \- Destination index, destination pointer, often paired with RSI.
  * **RSP** \- Stack pointer, tracks the most recently pushed value. Has special meaning, shouldn’t be used arbitrarily.
  * **RBP** \- Base pointer, a stable reference point for accessing local variables and saved registers within a function, since RSP moves during execution.
  * **RIP** \- Instruction pointer, points to the next instruction. Not general-purpose, updated automatically by the CPU.



## NOP

**NOP** stands for **No Operation**. It doesn’t change any register or memory, it just advances the instruction pointer and consumes execution time. The CPU executes it, but nothing observable happens.

### What Is NOP Used For?

NOP is commonly used for **instruction alignment** and **padding bytes** between instructions.

### Why Is NOP 0x90?

On x86, the one-byte NOP (**0x90**) is actually encoded as:


```asm
xchg eax, eax
```


**XCHG** swaps the values of two registers. Swapping a register with itself changes nothing, making it a perfect no-op, a real instruction repurposed to do effectively nothing.

### Multi-Byte NOPs

Although **0x90** is the classic 1-byte NOP, Intel also defines multi-byte NOP instructions, ranging from 1 to 9 bytes long. These are used mainly for **alignment** and for **patching code while preserving instruction boundaries** (so a patch doesn’t shift where later instructions start). They still perform no operation, just with different instruction lengths depending on how many padding bytes are needed.

## The Stack

We’re going to walk through how the stack actually works, using a real disassembled program to see it in action.

> **Note on convention:** the example below passes the argument in RDI, which is the **System V AMD64** convention (Linux/GCC). The **Function Parameters & Calling Conventions** section later in this document covers **Microsoft x64** , which passes the first integer argument in RCX instead. These are two different, incompatible ABIs, not two ways of writing the same thing, so don’t expect the register used here to match the register used later.


```c
#include <stdio.h>

void hello(char *name) {
    int age = 42;
    printf("Hello %s your age is %d\n", name, age);
}

int main(void) {
    char *name = "Leonardo";
    hello(name);
    return 0;
}
```

```asm
main:
    push    rbp                       ; save old base pointer
    mov     rbp, rsp                  ; set new base pointer
    sub     rsp, 0x10                 ; allocate 16 bytes for locals

    lea     rax, [leonardo_string]    ; load address of "Leonardo"
    mov     qword [rbp-0x8], rax      ; store pointer in local 'name'

    mov     rax, qword [rbp-0x8]      ; load 'name' pointer
    mov     rdi, rax                  ; move to rdi (1st argument, System V convention)
    call    hello                     ; call hello(name)

    mov     eax, 0x0                  ; return value = 0
    leave
    retn

hello:
    push    rbp
    mov     rbp, rsp
    sub     rsp, 0x20                 ; allocate 32 bytes for locals

    mov     qword [rbp-0x18], rdi     ; store 'name' parameter (from rdi)
    mov     dword [rbp-0x4], 0x2a     ; age = 42

    mov     edx, dword [rbp-0x4]      ; 3rd printf arg: age
    mov     rax, qword [rbp-0x18]     ; load 'name' pointer
    lea     rcx, [format_string]      ; load format string address
    mov     rsi, rax                  ; 2nd arg: name pointer
    mov     rdi, rcx                  ; 1st arg: format string
    mov     eax, 0x0                  ; no floating-point args
    call    printf

    leave
    retn
```


> **Note on stack allocation sizes:** this example is written under System V conventions, which do **not** require the caller to reserve shadow space for the callee. That’s why `main()` only allocates 16 bytes and `hello()` only allocates 32. The mandatory-32-byte shadow space rule covered in **The x64 Stack Allocation Rules** and **Shadow Space** applies specifically to the **Microsoft x64** ABI, not this example, so don’t expect these numbers to satisfy that later rule.

### What Is a Stack Frame?

A stack frame is the region of the stack holding everything one function call needs: the return address, saved registers (especially **RBP**), function arguments, and local variables. Each call gets its own frame, created on entry and destroyed on exit.

**RSP tracks the top of the stack.** In x86-64, the stack grows **downward** , so allocating space means _subtracting_ from RSP:


```
Before:  RSP = 0x7fff1000
sub rsp, 0x10
After:   RSP = 0x7fff0ff0   (16 bytes lower)
```

```
High memory (0x7fff1000)  <- Old RSP
    |
    | Valid, allocated stack memory
    |
    v  Stack grows DOWN
Low memory  (0x7fff0ff0)  <- New RSP
```


**RBP stays fixed for the whole function** , giving you a stable reference point, unlike RSP, which keeps moving as things get pushed and popped. Watch how RBP gets established during **hello()** ’s prologue:


```
Before push rbp:                  After push rbp:               After mov rbp, rsp:
                                   (RSP now points to             (RBP == RSP, both
                                    the saved RBP)                 point to the same slot)

  RSP -> +--------------+            +--------------+ <- RSP        +--------------+ <- RSP, RBP
         |              |            | Saved RBP    |               | Saved RBP    |
         +--------------+            +--------------+               +--------------+
         | main() frame |            | main() frame |               | main() frame |
  RBP -> +--------------+            +--------------+ <- RBP(old)   +--------------+
```


After **sub rsp, 0x20** allocates **hello()** ’s locals, RBP no longer moves, even though RSP does:


```
  RSP -> +--------------+
         |              |
         |  Local vars  |  <- hello()'s frame (32 bytes)
         |              |
  RBP -> +--------------+  <- stays fixed here for the rest of the function
         | Saved RBP    |
         +--------------+
         | main() frame |
         +--------------+
```


This is why local variables are always addressed as **[rbp-N]** , a _fixed_ offset that never shifts, no matter how much RSP moves later in the function:


```asm
mov qword [rbp-0x18], rdi    ; store 'name' at a fixed offset
mov dword [rbp-0x4], 0x2a    ; store 'age' at a different fixed offset
```


If you tried to use RSP for this instead, the offset would silently break the moment anything else got pushed or popped.

**Function prologue** (entry) sets all this up in three steps:


```asm
push rbp          ; 1. Save caller's RBP so we can restore it later
mov  rbp, rsp      ; 2. Establish our own stable base pointer
sub  rsp, N        ; 3. Allocate N bytes of space for locals
```


**Function epilogue** (exit) tears it back down:


```asm
leave              ; Equivalent to: mov rsp, rbp; pop rbp
retn               ; Pop return address into RIP, jump to it
```


**leave** does two things in one instruction, snap RSP back to RBP (instantly freeing all locals), then pop the saved RBP to restore the caller’s frame:


```
Before leave:                       After leave:
  RSP -> +--------------+             +--------------+
         |  Local vars  |             |  (freed)     |
         +--------------+ <- RBP      +--------------+
         | Saved RBP    |       RBP -> | Return addr | <- RSP
         +--------------+             +--------------+
         | Return addr  |
         +--------------+
```


**Why different functions allocate different amounts:** **main()** allocates 16 bytes (8 for the **name** pointer, 8 just for alignment), while **hello()** allocates 32 (8 for its **name** parameter, 4 for **age** , plus padding). The compiler always rounds up to keep the stack 16-byte aligned, even when the actual variables need less.

**Return addresses** work the same way, **call** pushes one automatically:


```asm
call hello    ; pushes the address of the *next* instruction onto the stack
```


When **hello()** finishes and hits **ret** , that saved address is exactly where execution resumes, back in **main()** , right after the **call**.

**Recursion** makes this concrete: each call to the same function still gets its own separate frame, stacked on top of each other:


```
+-----------------------+
|  factorial(1) frame   |
+-----------------------+
|  factorial(2) frame   |
+-----------------------+
|  factorial(3) frame   |
+-----------------------+
```


### PUSH and POP

**PUSH** stores a value on the stack in two steps: decrement RSP by 8, then write the value at the new top.


```asm
push rax
```


is equivalent to:


```asm
sub rsp, 8       ; move stack pointer down first
mov [rsp], rax    ; then write the value
```


Watch it happen:


```
Before push rax:                  After push rax:
  (RAX = 0x3)                       (RAX unchanged, still 0x3)

  Addr        Value                 Addr        Value
  0x...FE8 <- RSP  0x2               0x...FE8    0x2
  0x...FE0    undefined              0x...FE0 <- RSP  0x3   (RAX's value written here)
```


RSP always moves _first_ , before the write, so it never briefly points somewhere invalid.

**POP** does the reverse: read the value at RSP, then increment RSP by 8.


```asm
pop rax
```


is equivalent to:


```asm
mov rax, [rsp]    ; read value at current top
add rsp, 8         ; then move stack pointer up
```


The important detail: **POP** doesn’t erase anything. The value stays sitting in memory, it’s just no longer considered part of the “valid” stack, and the next **PUSH** will overwrite it.

**The golden rule: balance your stack.** Every **PUSH** needs a matching **POP** , and they must undo each other in reverse order (LIFO, last in, first out):


```asm
push rax        ; save rax
push rbx        ; save rbx
; ... do work ...
pop  rbx        ; restore rbx first (reverse order!)
pop  rax        ; then restore rax
```


Getting the order wrong, or forgetting a **POP** entirely, misaligns the stack and corrupts return addresses, which is one of the most common ways handwritten assembly crashes.

### Calculating Stack Offsets

When you see **[rbp-0x8]** or **[rsp+0x18]** in disassembly, here’s how to derive that offset yourself, given a stack layout like this:


```
      HIGH ADDRESSES
+--------------------------+
| 0xb01dface   <- RBP      |
+--------------------------+
| 0xaffab1e    <- TARGET   |
+--------------------------+
| 0x50f7ba11               |
+--------------------------+
| 0x0000b100d              |
+--------------------------+
| 0xb100d1e55  <- RSP      |
+--------------------------+
      LOW ADDRESSES
```


  1. **Count the 8-byte slots** between your starting register and the target. From RBP: 1 slot. From RSP: 3 slots.
  2. **Multiply by 8** to get bytes: RBP -> 8 bytes, RSP -> 24 bytes.
  3. **Determine direction:** moving to a _higher_ address is **+** , moving to a _lower_ address is **-**. The target is below RBP, so **-**. It’s above RSP, so **+**.
  4. **Convert to hex and write it out:** **rbp-0x08** or **rsp+0x18**.



Quick reference for common slot counts:


```
Slots  Bytes  Hex
1        8    0x08
2       16    0x10
3       24    0x18
4       32    0x20
```


## Assembly Syntax Basics

There are two main syntax styles for x86 assembly, they differ mainly in operand order.

**Intel syntax** (used throughout this course): Destination <\- Source, like an assignment (**y = 2x + 1**):


```asm
mov rbp, rsp        ; rbp = rsp
add rsp, 0x14         ; rsp = rsp + 0x14
```


**AT &T syntax** (Unix/GNU): Source -> Destination, like an equation (**1 + 2 = 3**). Registers get a **%** prefix, immediates get a **$** prefix:


```asm
mov %rsp, %rbp        ; rsp -> rbp
add $0x14, %rsp        ; 0x14 + rsp -> rsp
```


### MOV, Copying Data

**MOV** copies a value from one location to another:


```asm
mov rbx, rax           ; register to register
mov rax, [rbx]          ; memory to register
mov [rbx], rax           ; register to memory
mov rbx, imm64            ; immediate (a literal value) to register
mov dword [rbx], imm32     ; immediate to memory
```


**Important restriction:** memory-to-memory moves are not allowed, this is a hard limitation of the x86 architecture. Source and destination can’t _both_ be memory in the same instruction. Register-to-register, register-to-memory, memory-to-register, and immediate-to-memory (or immediate-to-register) are all fine, an immediate written straight into memory doesn’t need a register at all, as shown above and used later in this document (e.g. `mov dword [rbp-0x4], 0x5ca1ab1e`).

### Memory Addressing (the r/m Form)

Memory operands support a flexible calculated address:


```
[base + index*scale + displacement]
```


  * **base** \- a base register, e.g. **rbx**
  * **index** \- an index register, e.g. **rcx**
  * **scale** \- must be 1, 2, 4, or 8 (matching byte/short/int/pointer sizes)
  * **displacement** \- a constant offset


```asm
mov rax, [rbx + rcx*4 + 8]   ; base + index*scale + displacement, all in one instruction
```


This single addressing mode is what lets the CPU compute an array element’s address (**base + index*size**) or a struct field’s address (**base + fixed offset**) without needing separate arithmetic instructions first.

### ADD and SUB

Straightforward addition and subtraction:


```asm
add rsp, 8               ; rsp = rsp + 8
sub rcx, rdx               ; rcx = rcx - rdx
add [rbx+4], 10             ; memory[rbx+4] = memory[rbx+4] + 10
sub rax, [rbx*2]             ; rax = rax - memory[rbx*2]
```


**Operand rules:** the destination can be a register or memory (r/m), the source can be a register, memory (r/m), or an immediate value, but source and destination can’t _both_ be memory, that would be a memory-to-memory operation, which x86 doesn’t allow (same restriction as MOV).

## Function Calls

Now that the basics are in place, here’s how a function call actually looks in assembly.


```c
int func() {
    return 0xbeef;
}

int main() {
    func();
    return 0xf00d;
}
```

```asm
main:
    sub     rsp, 0x28
    call    __main
    mov     eax, 0xf00d
    add     rsp, 0x28
    retn
```


### The CALL Instruction

**CALL** transfers control to another function while remembering where to come back to. It does two things:

  1. **Pushes the return address** \- the address of the instruction right after the **call** \- onto the stack
  2. **Updates RIP** to point at the target function


```asm
call function
```


The target address can be specified as an absolute address, a relative offset, or a register holding the address.

### The RET Instruction

**RET** returns control back to the caller, in two forms:


```asm
ret          ; Pops the top of the stack into RIP
ret 0x8      ; Same, but also adds a constant number of bytes to RSP afterward
```


The second form (**ret N**) shows up often when disassembling Windows APIs, it lets the callee clean up its own stack arguments on the way out, instead of leaving that job to the caller.

## Local Variables on the Stack

### Example 1: A Simple Integer


```c
int func() {
    int i = 0x5ca1ab1e;
    return i;
}

int main() {
    return func();
}
```

```asm
func:
    push    rbp
    mov     rbp, rsp
    sub     rsp, 0x10
    mov     dword [rbp-0x4], 0x5ca1ab1e   ; i = 0x5ca1ab1e
    mov     eax, dword [rbp-0x4]          ; return i
    pop     rbp
    retn

main:
    push    rbp
    mov     rbp, rsp
    sub     rsp, 0x20
    call    func
    pop     rbp
    retn
```


Once **func()** stores **i** , its frame looks like this:


```
     +------------------+
     |   func() frame   |
     |   (12 unused)    |
     +------------------+
     | i = 0x5ca1ab1e   | <- [rbp-0x4]
     +------------------+
     | saved rbp        | <- rbp
     +------------------+ <- rsp
```


**Reading it in raw memory** shows endianness in action:

Address| Hex Value| ASCII  
---|---|---  
0x14FDC0| `5ca1ab1e`| `.«¡\`  
  
The bytes are actually stored in reverse (**1e ab a1 5c**), that’s little-endian, exactly what we covered earlier, just now visible in a real stack dump.

**Size qualifiers matter for memory writes.** A register auto-extends, but memory only ever gets exactly as many bytes as you specify:


```asm
mov qword [rsp+0x10], rax     ; 8 bytes
mov dword [rsp], 0x5C414B1E    ; 4 bytes
mov word  [rsp], ax             ; 2 bytes
mov byte  [rsp], al              ; 1 byte
```


**Why does the compiler allocate more than the variable needs?** **int i** only needs 4 bytes, but the function reserves 16 (**sub rsp, 0x10**). This comes down to the x64 calling convention rules, covered next.

### The x64 Stack Allocation Rules

  * **16-byte alignment** \- RSP must always be aligned to a 16-byte boundary. This is required for SSE/AVX (SIMD) instructions and is part of the ABI.
  * **Shadow space** \- any function that itself calls another function must reserve 32 bytes (**0x20**) for the callee, even if it never uses them. This is a **Microsoft x64** -specific requirement, System V does not require it (see **Microsoft x64 vs System V (GCC) at a Glance** later on).
  * **Padding** \- extra bytes get added wherever needed to keep the total allocation a multiple of 16.



A chain of functions shows this pattern clearly:

Function| Allocation| Bytes| Reason  
---|---|---|---  
func3| 0x10| 16| Minimum alignment  
func2| 0x30| 48| Shadow space + variable + padding  
func| 0x20| 32| Shadow space only  
main| 0x20| 32| Shadow space only  
  
**func2** breaks down as: 32 bytes shadow space + 4 bytes for its **int j** \+ 12 bytes padding = 48 bytes (**0x30**), still a clean multiple of 16.

### Example 2: Multiple Variables, No Padding Needed


```c
int func() {
    long long i = 0xf01dab1ef007ba11;
    long long j = 0x0b57ac1e5;
    return i + j;
}
```

```asm
mov     rax, 0xf01dab1ef007ba11
mov     qword [rbp-0x8], rax     ; i
mov     eax, 0xb57ac1e5
mov     qword [rbp-0x10], rax    ; j
```

```
     | saved rbp        | <- rbp
     +------------------+
     | i (8 bytes)      | <- [rbp-0x8]
     +------------------+
     | j (8 bytes)      | <- [rbp-0x10]
     +------------------+ <- rsp
```


Two 8-byte variables = 16 bytes total, already a multiple of 16, so no padding is needed here.

### Example 3: Arrays, Sign Extension, and Index Math


```c
short a;
int b[6];
long long c;

a = 0xbabe;
b[1] = a;              // short assigned into an int slot
b[4] = b[1] + c;
```


Array element addresses are computed with the same **base + index*scale** form covered earlier:


```asm
mov     eax, 0x1                   ; index = 1
imul    rax, rax, 0x4              ; offset = 1 * 4 (sizeof(int))
movsx   ecx, word [rsp]            ; load 'a', sign-extended
mov     dword [rsp+rax+0x10], ecx  ; store into b[1]
```


**Why movsx here?** **a** is a signed **short** being written into a signed **int** slot, the CPU has to preserve the sign bit while widening it, that’s exactly what sign-extension means. If **a** had been **unsigned** , you’d see **movzx** (zero-extend) instead.

**The takeaway across all three examples:**

  * Local variables aren’t necessarily stored in declaration order, the compiler decides layout.
  * Array access is just **index * element_size** offset math, using the addressing form from earlier.
  * Struct fields follow the same padding rules as standalone variables, laid out in declaration order but padded to keep each field aligned, and the whole frame padded to a multiple of 16.



### IMUL, MOVSX, and MOVZX

These three instructions show up constantly once you’re reading real variable-heavy disassembly.

**IMUL (signed multiply)** has three common forms:


```asm
imul r/m              ; RDX:RAX = RAX * r/m       (result may not fit in one register)
imul reg, r/m          ; reg = reg * r/m            (result truncated to reg's size)
imul reg, r/m, imm      ; reg = r/m * imm
```


Compilers favor **imul** over the unsigned **mul** , since it’s used even for values that could be either signed or unsigned, the two-operand and three-operand forms simply truncate the result either way.

**MOVZX (zero-extend)** and **MOVSX (sign-extend)** move a smaller value into a larger register:


```asm
movzx rbx, ax    ; fills the upper bits with 0
movsx rcx, al    ; fills the upper bits with the sign bit
```


**Both MOVZX and MOVSX only work from 8-bit or 16-bit sources** , neither has a 32-bit-source form. To move a 32-bit value into a 64-bit register with zero-extension, you don’t need a special instruction at all, a plain 32-bit **mov** does it, since writing to a 32-bit register already clears the upper 32 bits of its 64-bit parent (covered earlier in **Why Access Smaller Parts of a Register?**):


```asm
mov    eax, 0xF00DFACE
; rax is now 0x00000000F00DFACE, zero-extension happened automatically
```


To sign-extend a 32-bit value into 64 bits, there’s a dedicated instruction, **MOVSXD** :


```asm
mov    eax, 0xF00DFACE
movsxd rbx, eax    ; rbx = 0xFFFFFFFFF00DFACE, since bit 31 was set (negative)
```


The difference matters: zero-extend always pads with **0** , sign-extend pads with whatever the original sign bit was, to preserve whether the value stays negative.

### A Note on Garbage Values

Stack memory isn’t zeroed out when a program starts, whatever was left over from previous operations, OS setup, or runtime initialization is still sitting there. This is why uninitialized local variables can appear to hold random values before you assign them, completely normal and expected.

## Function Parameters & Calling Conventions

> This section covers the **Microsoft x64** calling convention specifically. It differs from the System V convention used in the earlier Stack examples, most notably in which registers carry the first arguments. See **Microsoft x64 vs System V (GCC) at a Glance** below for the direct comparison.

### A Single Parameter


```c
int func(int a) {
    int i = a;
    return i;
}

int main() {
    return func(0x11);
}
```

```asm
main:
    mov     ecx, 0x11        ; argument goes into ECX
    call    func

func:
    push    rbp
    mov     rbp, rsp
    sub     rsp, 0x10
    mov     dword [rbp+0x10], ecx    ; parameter spilled to stack
    mov     eax, dword [rbp+0x10]
    mov     dword [rbp-0x4], eax     ; i = a
    mov     eax, dword [rbp-0x4]     ; return i
    pop     rbp
    retn
```


The argument is placed in **ECX** before the call, ECX is the register used for the first integer parameter under the Microsoft x64 ABI. The extra **mov** traffic inside **func** (moving it from ECX to the stack and back) is typical of unoptimized builds, the compiler is just being conservative.

### Multiple Parameters


```c
int func(uint64 a, uint64 b, uint64 c, uint64 d, uint64 e) {
    int i = a + b - c + d - e;
    return i;
}

int main() {
    return func(0x11, 0x22, 0x33, 0x44, 0x55);
}
```

```asm
main:
    mov     qword [rsp+0x20], 0x55   ; 5th param, goes on the stack
    mov     r9d, 0x44                ; 4th param
    mov     r8d, 0x33                ; 3rd param
    mov     edx, 0x22                ; 2nd param
    mov     ecx, 0x11                ; 1st param
    call    func
```


This reveals the actual **Microsoft x64** parameter order: the first four integer arguments go in **RCX, RDX, R8, R9** , everything beyond that gets pushed onto the stack.

**Where the parameters land relative to RBP** , once inside **func** :


```
                        Higher addresses
main() frame:
  [rbp+0x30]  arg5 = 0x55           <- 5th arg, plain stack slot
  [rbp+0x28]  arg4 = r9 = 0x44      <- shadow space
  [rbp+0x20]  arg3 = r8 = 0x33      <- shadow space
  [rbp+0x18]  arg2 = rdx = 0x22     <- shadow space
  [rbp+0x10]  arg1 = rcx = 0x11     <- shadow space
  ------------------------------------
  return address to main
  saved rbp
  ------------------------------------
func() frame:
  [rbp-0x4]   i = ffffffef
                        Lower addresses
```


Notice the pattern: **positive** offsets from RBP (**[rbp+N]**) reach _up_ into the caller’s frame to read parameters, **negative** offsets (**[rbp-N]**) reach _down_ into your own frame for locals. Same RBP, two directions, two purposes.

### Shadow Space

The Microsoft x64 ABI requires the _caller_ to always reserve 32 bytes (4 x 8) on the stack for the callee, even if the function takes fewer than 4 parameters. That’s why you’ll see **sub rsp, 0x28** (40 bytes: 32 shadow + 8 alignment) even in functions that take zero arguments.

This exists so the callee has somewhere to “spill” register parameters into memory if it needs to:


```asm
mov     qword [rbp+0x10], rcx    ; spilling rcx into shadow space
mov     qword [rbp+0x18], rdx    ; spilling rdx into shadow space
```


### Caller-Saved vs Callee-Saved Registers

A calling convention also defines who’s responsible for preserving which registers across a call:

  * **Caller-saved (volatile)** \- the callee is free to change these. If the caller still needs the value afterward, _it_ must save it before the call and restore it after: **RAX, RCX, RDX, R8, R9, R10, R11**.
  * **Callee-saved (non-volatile)** \- the caller assumes these survive the call untouched. If the callee wants to use one, it must save and restore it itself: **RBX, RBP, R12-R15**.


```asm
func:
    push    rbx      ; save it, since it's callee-saved
    ; ... function body uses rbx ...
    pop     rbx      ; restore it before returning
    ret
```


### Microsoft x64 vs System V (GCC) at a Glance

Feature| Microsoft x64| System V AMD64  
---|---|---  
1st-4th arguments| RCX,RDX,R8,R9| RDI,RSI,RDX,RCX  
5th/6th arguments| Stack| R8, R9  
7th+ arguments| Stack| Stack  
Shadow space| 32 bytes required| Not required  
Max register args| 4| 6  
  
Both return values in **RAX** (or RDX:RAX for 128-bit results). System V simply fits two more arguments into registers and skips the shadow space requirement entirely, so its stack layout looks a bit leaner for the same function. This is also the table to check back against if you want to see exactly why the earlier Stack section (RDI, no mandatory shadow space) and this section (RCX, mandatory shadow space) look different, they’re the two rows of this table.

### LEA (Load Effective Address)

**LEA** is the one exception to the rule that square brackets **[]** mean “dereference this address.” Instead, it _calculates_ an address using the normal **base + index*scale + displacement** form, and loads that calculated value itself into a register, without ever touching the memory it points to.

Think of it as C’s **&** operator:


```c
int a = 3;
int* pa = &a;   // lea pa, a: gets the address, not the value
```


**Given:**


```asm
lea rax, [rdx + rbx*8 + 5]
```


with **rbx = 2** and **rdx = 0x1000** :


```
rax = 0x1000 + (2 * 8) + 5 = 0x1015
```


**rax** now holds **0x1015** , LEA never reads _from_ address **0x1015** , it just computed that number.

**Why compilers use it for plain math, not just addresses**


```c
int main(int argc, char** argv) {
    int a = atoi(argv[1]);
    return 2 * argc + a;
}
```

```asm
mov     eax, dword [rbp+0x10]   ; argc
lea     edx, [rax+rax]           ; edx = argc + argc = 2 * argc
mov     eax, dword [rbp-0x4]     ; a
add     eax, edx                  ; 2 * argc + a
```


The compiler used LEA to compute **2 * argc** even though nothing here is really “an address”, it recognized that the multiplication fit LEA’s **base + index*scale** form, and LEA is cheaper than a separate **add** /**mul** sequence. This is a common optimization pattern:


```asm
lea eax, [edi + edi]      ; 2 * edi
lea eax, [edi + edi*4]    ; 5 * edi
lea eax, [edi*8 + edi]    ; 9 * edi
```


**Common uses, summarized:**


```asm
lea rax, [rdi + 8]              ; pointer arithmetic, e.g. ptr++
lea rax, [rbx + rcx*4 + 8]      ; address of &array[i]
lea eax, [edi + edi*2]          ; arithmetic optimization: 3 * edi
```


Whenever you see **lea** and the destination clearly isn’t a pointer, that’s usually the compiler doing cheap arithmetic, not addressing.

## Control Flow

Control flow decides which instructions actually execute. There are two types:

  * **Conditional** \- goes somewhere _if_ a condition is met (**if** statements, **switch** , loops)
  * **Unconditional** \- _always_ goes somewhere (function calls, **goto** , exceptions, interrupts)



We’ve already seen function calls manifest as **call** /**ret**. Here’s how **goto** shows up.


```c
int main() {
    goto mylabel;
    printf("skipped!\n");

mylabel:
    printf("mylabel called");
    return 0xb01dface;
}
```

```asm
main:
    sub rsp, 0x28
    jmp mylabel
    lea rcx, [skipped_string]
    call printf

mylabel:
    lea  rcx, [mylabel_string]
    call printf
    mov  eax, 0xb01dface
    add  rsp, 0x28
    ret
```


**goto** is literally just **jmp** to the memory address of the target label.

### The JMP Instruction

**JMP** unconditionally changes RIP to a given address. It can be encoded a few different ways:

  * **Short relative** \- RIP = next instruction’s address + a 1-byte signed displacement. Common in small loops, e.g. **jmp -2** creates an infinite loop. The encoding doesn’t bake in a destination address, it just says “jump N bytes forward/backward from here.”
  * **Near relative** \- same idea, but with a 4-byte displacement, reaching much farther.
  * **Near/far absolute indirect** \- the address comes from a register or is pulled from memory (an r/m form), rather than being a fixed offset.



### if Statements, CMP, and JCC


```c
int main() {
    int a = -1, b = 2;

    if (a == b) return 1;
    if (a > b)  return 2;
    if (a < b)  return 3;

    return 0xdefea7;
}
```

```asm
mov     eax, dword [rbp-0x4]
cmp     eax, dword [rbp-0x8]
jne     .not_equal

    mov  eax, 0x1
    jmp  .end

.not_equal:
    cmp     eax, dword [rbp-0x8]
    jle     .not_greater

    mov  eax, 0x2
    jmp  .end

.not_greater:
    cmp     eax, dword [rbp-0x8]
    jge     .not_less

    mov  eax, 0x3
    jmp  .end

.not_less:
    mov  eax, 0xdefea7
.end:
```


New instructions here: **CMP** (compare), **JNE** (jump if not equal), **JLE** (jump if less than or equal), **JGE** (jump if greater than or equal).

**JCC (jump if condition is met):** if the condition is true, the jump is taken, otherwise execution just falls through to the next instruction. There are dozens of these mnemonics, but many are just synonyms for each other, e.g. **JNE** and **JNZ** both check the exact same thing (zero flag == 0), they just read differently depending on context (equality vs. zero).

### The RFLAGS Register

**RFLAGS** is the 64-bit extension of the older **EFLAGS** register, the upper 32 bits are reserved and unused, the lower 32 bits are EFLAGS itself. It holds single-bit status flags, set automatically by arithmetic instructions:

  * **Zero Flag (ZF)** \- set if the result was zero
  * **Sign Flag (SF)** \- set if the result’s most significant bit is 1 (i.e. negative, for signed values)
  * **Carry Flag (CF)** \- set on unsigned overflow
  * **Overflow Flag (OF)** \- set on signed overflow
  * **Parity Flag (PF)** \- set if the low byte has an even number of 1 bits
  * **Auxiliary Flag (AF)** \- used for BCD arithmetic



#### Some common JCC instructions:


```
JZ/JE    - Jump if ZF == 1                    (Zero / Equal)
JNZ/JNE  - Jump if ZF == 0                    (Not Zero / Not Equal)
JLE/JNG  - Jump if ZF == 1 OR SF != OF        (signed <=)
JGE/JNL  - Jump if SF == OF                   (signed >=)
JBE/JNA  - Jump if CF == 1 OR ZF == 1         (unsigned <=)
JB/JNAE  - Jump if CF == 1                    (unsigned <)
```


You don’t need to memorize these, in practice you’ll be stepping through a debugger and watching **RFLAGS** directly to see whether a jump gets taken.

#### Mnemonic translation:

  * **A** = Above (unsigned notion)
  * **B** = Below (unsigned notion)
  * **G** = Greater than (signed notion)
  * **L** = Less than (signed notion)
  * **E** = Equal (same idea as **Z** , zero flag set)
  * **N** = NOT (e.g. **JNL** = “jump if not less”)



The unsigned/signed distinction matters: **0xFFFFFFFF** is _above_ zero if treated as unsigned, but _not greater than_ zero if treated as signed (it’s actually -1). Same bits, opposite conclusion, depending on which mnemonic family the compiler chose.

### CMP: Setting the Flags

Before a conditional jump can happen, something needs to set the status flags. That’s usually **CMP** , **TEST** , or any instruction with flag-setting side effects (**ADD** , **SUB** , etc.).

**CMP** works by subtracting the second operand from the first, exactly like **SUB** , and setting the same flags (**CF, OF, SF, ZF, AF, PF**). The difference from **SUB** : with **SUB** the result gets stored somewhere, with **CMP** the result is thrown away, only the flags matter.

**Reading comparisons at a glance:**


```asm
cmp 1, 2
jne wow1    ; if (1 != 2)

cmp 1, 2
jle wow2    ; if (1 <= 2), signed comparison ("less", not "below")

cmp 1, 2
jae wow3    ; if (1 >= 2), unsigned comparison ("above", not "greater")
```


**TEST** , like **CMP** , is another instruction that sets flags without storing a result, but instead of subtraction, it uses **bitwise AND**.


```asm
test eax, eax
```


The most common use is `test reg, reg`, this is the standard way to check whether a register is **zero** or **negative** , shorter than `cmp reg, 0`:

  * If the register is zero, **ZF** (zero flag) gets set, so it’s typically followed by **jz** /**jnz**
  * If the register is negative (sign bit = 1), **SF** (sign flag) gets set, so it’s typically followed by **js** /**jns**



You’ll often see this pattern right before a conditional jump or conditional move, like in the signed shift example from **Bit Shifting** :


```asm
test    eax, eax
cmovs   eax, edx    ; conditional move, executes only if the sign flag is set
```


### switch Statements

A **switch** compiles down to essentially the same thing as a chain of **if (x == value)** checks.


```c
switch (a) {
    case 0: return 0;
    case 1: return 1;
    default: return 3;
}
```

```asm
cmp     dword [rbp-0x4], 0x0
jne     .check_one
    mov  eax, 0x0
    jmp  .end

.check_one:
    cmp     dword [rbp-0x4], 0x1
    jne     .default
    mov  eax, 0x1
    jmp  .end

.default:
    mov  eax, 0x3
.end:
```


Writing the equivalent **if** /**else if** /**else** chain by hand produces **nearly identical assembly** , at this level, **switch** and **if** really are the same construct, just different surface syntax.

### Signed vs Unsigned Comparisons

The only substantive thing that changes between signed and unsigned integers is _which conditional jump instructions_ the compiler emits:


```
Unsigned:  JB, JA, JBE, JAE   (Below / Above / Below-or-Equal / Above-or-Equal)
Signed:    JL, JG, JLE, JGE   (Less / Greater / Less-or-Equal / Greater-or-Equal)
```


**Why this matters:** since the compiler emits different jump instructions depending on the variable’s declared signedness, a reverse engineer can use that difference to infer whether a variable was originally **signed** or **unsigned** in the source code, even without seeing the source.

**How the hardware actually handles it:** the CPU itself doesn’t care about signedness when it executes **ADD** or **SUB** , it performs the operation and sets _all_ the status flags (zero, sign, overflow, carry, parity) regardless. It’s entirely up to the _compiler_ to pick the correct flag(s) to check, based on whether the original high-level code declared the type as signed or unsigned.

## Boolean Logic & Bitwise Operations

**Boolean** is just true/false. We’ll use **0** as shorthand for false and **1** for true.


```
AND: true only if both inputs are true      0 AND 0 = 0   1 AND 1 = 1
OR:  true if either input is true            0 OR  1 = 1
XOR: true if only one input is true          1 XOR 1 = 0
NOT: flips the value                         NOT 1 = 0
```


### Logical vs Bitwise in C

**Logical** operators (**& &**, **||** , **!**) evaluate a whole expression as true/false (non-zero counts as true, zero as false).

**Bitwise** operators (**&** , **|** , **^** , **~**) apply the operation independently to each corresponding bit position:


```
output_bit[0] = input1_bit[0] AND input2_bit[0]
output_bit[1] = input1_bit[1] AND input2_bit[1]
...
output_bit[N] = input1_bit[N] AND input2_bit[N]
```

```c
unsigned int i = 0x50da;
unsigned int j = 0xc0ffee;
uint64 k = 0x7ea707a11ed;
k ^= ~(i & j) | 0x7ab00;
return k;
```

```asm
mov     eax, dword [rbp-0x4]
and     eax, dword [rbp-0x8]     ; i & j
not     eax                       ; ~(...)
or      eax, 0x7ab00               ; ... | 0x7ab00
xor     qword [rbp-0x10], rax       ; k ^= ...
```


New instructions here: **AND** , **NOT** , **OR** , and **XOR** , the bitwise counterparts to C’s **&** , **~** , **|** , and **^**.

### AND, Bitwise AND

Corresponds to C’s **&** (not **& &**, that’s logical AND). The destination can be a register or memory (r/m), the source can be a register, memory, or an immediate, but source and destination can’t both be memory.


```asm
and al, bl

   00110011b ( al - 0x33 )
AND 01010101b ( bl - 0x55 )
------------------------
   00010001b ( al - 0x11 )
```


Each bit position is compared independently, 1-to-1 gives 1, anything else gives 0, since **AND** is only true when both inputs are true.

### OR, Bitwise OR

Corresponds to C’s **|**. Same operand rules as **AND**. True if either bit is 1:


```asm
or al, bl

   00110011b ( al - 0x33 )
OR  01010101b ( bl - 0x55 )
------------------------
   01110111b ( al - 0x77 )
```


### XOR, Bitwise Exclusive OR

Corresponds to C’s **^**. True only if exactly one input bit is 1:


```asm
xor al, al

   00110011b
XOR 00110011b
------------
   00000000b
```


XORing anything with itself always produces **0** , which is why compilers commonly generate **xor eax, eax** to zero a register, it’s faster than an equivalent **mov eax, 0**.

### NOT, One’s Complement Negation

Corresponds to C’s unary **~** (not **!** , that’s logical NOT). Takes a single register or memory operand, and simply flips every bit, that’s the entire operation.

## For Loops


```c
int main() {
    int i;
    for (i = 0; i < 10; i++) {
        printf("i = %d\n", i);
    }
    i--;
}
```

```asm
main:
    mov     dword [rbp-0x4], 0x0      ; i = 0
    jmp     .check

.body:
    mov     eax, dword [rbp-0x4]
    lea     rcx, [format_string]       ; "i = %d\n"
    mov     edx, eax
    call    printf
    add     dword [rbp-0x4], 0x1       ; i++

.check:
    cmp     dword [rbp-0x4], 0x9
    jle     .body                      ; loop while i <= 9

    sub     dword [rbp-0x4], 0x1       ; i--
    mov     eax, 0x0
    retn
```


The shape here is familiar: a comparison (**CMP**) feeding a conditional jump (**JLE**), exactly like the **if** statements from before. A **for** loop really is just an **if** -style check that jumps back to itself instead of falling through.

### INC/DEC (Increment/Decrement)

**INC** and **DEC** take a single register or memory operand and increase or decrease its value by 1:


```asm
xor rax, rax   ; rax = 0
inc rax        ; rax = 1
```

```asm
mov rax, 0
dec rax        ; rax = 0xFFFFFFFFFFFFFFFF, wraps around (unsigned underflow)
```


You’d expect to see **inc** /**dec** for `i++` and `i--`, but many compilers actually favor **add** /**sub** instead, following Intel’s own optimization guidance. So seeing **add dword […], 0x1** where you’d expect **inc** is normal in optimized output, seeing **inc** /**dec** directly can sometimes hint that the code is hand-written or unoptimized.

**INC and DEC modify OF, SF, ZF, AF, and PF, but leave CF (the carry flag) untouched.** This is the actual reason the two instructions exist separately from `add reg, 1` / `sub reg, 1`, which do affect CF. It lets you increment or decrement a counter in the middle of a longer arithmetic sequence without destroying a carry flag that a surrounding operation still depends on.

## Repeatable String Instructions

x86 provides instructions that can repeat themselves automatically using the **rep** prefix, driven by a counter in **RCX**. Each iteration, RCX decrements, once it hits 0, execution moves to the next instruction.

### REP STOS, Filling Memory

**REP STOS** (Repeat STore String) fills memory with a single repeated value, essentially a hardware-accelerated **memset()**.


```asm
mov rdi, [address]   ; RDI = where to start writing
mov rax, [value]      ; value to write
mov rcx, [count]       ; how many times to repeat
rep stosb               ; fill memory, byte by byte
```


Each iteration stores **AL/AX/EAX/RAX** at **[RDI]** , then **RDI** automatically increments to the next position:


```
STOSB - fill 1 byte  at [RDI] with AL
STOSW - fill 2 bytes at [RDI] with AX
STOSD - fill 4 bytes at [RDI] with EAX
STOSQ - fill 8 bytes at [RDI] with RAX
```


**Real example, zeroing a buffer:**


```c
short a;
int b[6];
long long c;

a = 0xbabe;
b[1] = a;
b[4] = b[1] + c;
```

```asm
push    rdi
sub     rsp, 0x90

lea     rax, [rsp + 0x10]   ; rax = address of buffer
mov     rdi, rax             ; rdi = destination for STOS

xor     eax, eax              ; value to fill with = 0
mov     ecx, 0x80              ; 128 bytes
rep     stosb                  ; equivalent to memset(buffer, 0, 128)

mov     dword [rsp], 0x05EAF00D   ; write a value elsewhere on the stack

mov     eax, 0x1                     ; index = 1
imul    rax, rax, 0x2                ; offset = 1 * 2 (sizeof(int) element here is stored as a 16-bit slot)
movzx   ecx, word [rsp]               ; ecx = lower 2 bytes of 0x05EAF00D = 0xF00D
mov     word [rsp + rax + 0x10], cx   ; buffer[1] = 0xF00D

mov     eax, 0x1
imul    rax, rax, 0x2
movzx   eax, word [rsp + rax + 0x10]  ; read it back, eax = 0xF00D (return value)

add     rsp, 0x90
pop     rdi
ret
```


**What’s happening:** **rep stosb** zeroes a 128-byte buffer in a single instruction, equivalent to `memset(buffer, 0, 128)`. Afterward, a value gets written elsewhere on the stack, its lower 2 bytes get copied into the buffer using the same **index * scale** addressing you’ve already seen, and finally that same value gets read back out.

**Why REP STOS is fast:** writing this manually as `for (int i = 0; i < 128; i++) buffer[i] = 0;` requires the CPU to fetch, decode, and execute a comparison and jump every single iteration. **REP STOS** handles the looping internally in hardware, no per-iteration branch overhead.

**Common use cases:** zeroing memory (**memset** -style), filling arrays with a repeated value (e.g. **0xFF**), initializing buffers before use.

**Quick reference:**


```asm
rep stosb    ; Fill RCX bytes with AL
rep stosw    ; Fill RCX words (2 bytes) with AX
rep stosd    ; Fill RCX dwords (4 bytes) with EAX
rep stosq    ; Fill RCX qwords (8 bytes) with RAX
```


**RDI** = destination, **RAX/EAX/AX/AL** = value to fill, **RCX/ECX/CX** = count.

### REP MOVS, Copying Memory

**REP MOVS** copies memory from one location to another, source to destination, essentially a hardware-accelerated **memcpy()**. Unlike a regular **MOV** , **MOVS** can move memory to memory directly, but only between **RSI** (source) and **RDI** (destination).


```asm
rep movsb    ; copy RCX bytes   from [RSI] to [RDI]
rep movsw    ; copy RCX words   from [RSI] to [RDI]
rep movsd    ; copy RCX dwords  from [RSI] to [RDI]
rep movsq    ; copy RCX qwords  from [RSI] to [RDI]
```


**RSI** = source, **RDI** = destination, **RCX/ECX/CX** = count.

### The Direction Flag (DF)

**DF** controls which way **REP MOVS** (and similar string instructions) copy, forward (incrementing RSI/RDI) or backward (decrementing them).


```asm
cld    ; Clear direction flag, forward (usually what you want)
std    ; Set direction flag, backward
```


**Why this matters for security:** if an attacker can influence **DF** and flip a copy from forward to backward when the programmer expected forward, that can lead to memory corruption, worth knowing which direction a copy runs when auditing code. Most of the time, **cld** is the correct choice.

## Bit Shifting


```c
unsigned int a, b, c;
a = 0x5;
b = a << 4;
c = b >> 3;
```

```asm
mov     eax, dword [rbp-0x4]
shl     eax, 4      ; a << 4
mov     eax, dword [rbp-0x8]
shr     eax, 3      ; b >> 3
```


### SHL, Shift Logical Left

Corresponds to C’s **«** operator. The first operand (source and destination) is a register or memory, the second is either **cl** (the lowest byte of RCX) or a 1-byte immediate, specifying how many places to shift.

Each shift left multiplies the value by 2, and it’s more efficient than an actual multiply instruction. Bits pushed off the left edge move into the carry flag (**CF**), and zeros fill in on the right (the least significant bits):

5 in binary:

128| 64| 32| 16| 8| 4| 2| 1  
---|---|---|---|---|---|---|---  
0| 0| 0| 0| 0| 1| 0| 1  
  
= 5

Shifted left by 1 (`x << 1`):

128| 64| 32| 16| 8| 4| 2| 1  
---|---|---|---|---|---|---|---  
0| 0| 0| 0| 1| 0| 1| 0  
  
= 10

Every bit visibly moved one position to the left, doubling the value from 5 to 10.

### SHR, Shift Logical Right

Corresponds to C’s **»** operator (for unsigned values). Same operand rules as **SHL**. Each shift right divides the value by 2, more efficient than an actual divide. Bits pushed off the right edge move into **CF** , and zeros fill in on the left (the most significant bits):


```
00110011b (51), shifted right by 2:

Before: 0 0 1 1 0 0 1 1  = 51
After:  0 0 0 0 1 1 0 0  = 12
```


**CF** ends up set to **1** here, because the last bit that fell off the right edge was a **1**.

### Where Did the Multiply/Divide Go?

You’ll often see **shl** /**shr** in disassembly even when the C source never wrote a shift at all:


```c
unsigned int a, b, c;
b = a * 8;
c = b / 16;
```

```asm
shl     eax, 0x3     ; a * 8  ->  a << 3
shr     eax, 0x4      ; b / 16 ->  b >> 4
```


When a multiply or divide by a power of 2 appears in C, an optimizing compiler frequently converts it into a shift instead, since shifting is cheaper than actual multiplication or division hardware-wise. Seeing shifts here doesn’t mean the programmer wrote shifts, it usually means the compiler optimized an ordinary multiply/divide.

### Signed vs Unsigned Shifting: SAR

Whether a value is **signed** or **unsigned** changes which right-shift instruction the compiler emits:


```c
unsigned int a, b, c;   // unsigned -> shr
         int a, b, c;    // signed   -> sar
```


For a **signed** divide by a power of 2, the compiler emits **SAR** instead of **SHR** :


```asm
shl     eax, 0x3
lea     edx, [rax+0xf]
test    eax, eax
cmovs   eax, edx
sar     eax, 0x4      ; signed shift, note: SAR here, not SHR
```


The extra **lea** /**test** /**cmovs** sequence beforehand exists to correctly round negative values toward zero, unsigned division doesn’t need this adjustment, which is another subtle hint that a variable was declared signed.

**SAR (Shift Arithmetic Right)** behaves like **SHR** , except the bits shifted in from the left are filled with the **sign bit** , not zero, preserving whether the value stays negative or positive:


```
10110011b (signed, sign bit = 1), shifted right by 1:

Before: 1 0 1 1 0 0 1 1
After:  1 1 0 1 1 0 0 1
        ^
        MSB filled with the sign bit (1), not 0
```


If the value had started positive (sign bit = 0), the vacated bit would be filled with **0** instead, exactly like **SHR**. This is why **SAR** exists as a separate instruction: it needs an extra decision (check the sign) that **SHR** doesn’t.

### SAL, Shift Arithmetic Left

**SAL** behaves identically to **SHL** , same operand rules, same “multiply by 2 per shift” behavior, same bits shifted into **CF**. There’s no separate “arithmetic” concern on the left shift, since there’s no sign bit to preserve when shifting away from the least significant end.

## Multiplication and Division


```c
#define uint64 unsigned long long

uint64 main() {
    uint64 a = 0xdefec7ed;
    a *= 0xde7ec7ab1e;
    a /= 0x2bad505ad;
    return a;
}
```

```asm
mov     rax, qword [rbp-0x8]
imul    rax, qword [rbp-0x10]    ; a *= ...
mov     qword [rbp-0x8], rax      ; store the product back into 'a'
mov     rcx, qword [rbp-0x18]
mov     rax, qword [rbp-0x8]
mov     edx, 0                     ; zero out edx before dividing
div     rcx                          ; a /= ...
```


### IMUL, Signed Multiply

**IMUL** was already introduced back in the Local Variables section, this example shows it in a more complete multiply-then-divide flow. As a refresher, its common forms:


```asm
imul r/m              ; RDX:RAX = RAX * r/m
imul reg, r/m           ; reg = reg * r/m (truncated)
imul reg, r/m, imm       ; reg = r/m * imm
```


Note the added `mov qword [rbp-0x8], rax` above: the two-operand `imul` only updates the register, it doesn’t write back to memory on its own, so the product has to be stored back to `a`’s stack slot before the code reloads `rax` from `[rbp-0x8]` for the division. Without that store, the reload would fetch the original, un-multiplied value.

### DIV, Unsigned Divide

**DIV** comes in three widths, and always divides a _double-width_ dividend by the given operand:


```
Divide AX by r/m8        -> AL = quotient, AH = remainder
Divide EDX:EAX by r/m32   -> EAX = quotient, EDX = remainder
Divide RDX:RAX by r/m64    -> RAX = quotient, RDX = remainder
```


Notice the dividend spans **two** registers (e.g. **EDX:EAX** together form a 64-bit dividend for a 32-bit division). If your actual value only needs 32 or 64 bits, the compiler zeroes **EDX** /**RDX** first, that’s exactly what **mov edx, 0** is doing in the example above, before the division even happens.

If the divisor is **0** , a divide-by-zero exception is raised.

**Example (8-bit form):**


```
Before:  ax = 0x8, r/m8 (cx) = 0x3
Operation: div cx
After:   ah (remainder) = 0x2, al (quotient) = 0x2
```


**Example (64-bit form):**


```
Before:  rdx = 0x0, rax = 0x8, r/m64 (rcx) = 0x5
Operation: div rcx
After:   rdx (remainder) = 0x3, rax (quotient) = 0x1
```


### IDIV, Signed Divide

Same structure as **DIV** , but for signed operands, same three widths, same double-width dividend convention, same divide-by-zero exception on a zero divisor:


```
Divide AX by r/m8        -> AL = quotient, AH = remainder
Divide EDX:EAX by r/m32   -> EAX = quotient, EDX = remainder
Divide RDX:RAX by r/m64    -> RAX = quotient, RDX = remainder
```


**Example:**


```
Before:  ax = 0xFFFE, r/m8 (cx) = 0x2
Operation: idiv cx
After:   ah (remainder) = 0x0, al (quotient) = 0xFF
```


Here **ax = 0xFFFE** is **-2** as a signed 16-bit value, so **-2 / 2 = -1** remainder **0** , and **-1** as a signed byte is **0xFF** , matching the quotient shown. The key practical difference from **DIV** : since the operands are interpreted as signed, negative dividends behave differently, this is exactly the same **DIV vs IDIV** / **SHR vs SAR** distinction you’ve already seen elsewhere, unsigned and signed operations need separate instructions because the hardware has to know which interpretation to apply.
