## Machine Program

### Architecture (x86, ARM, …)

(also ISA: instruction set architecture) The parts of a processor design that one needs to understand for writing correct machine/assembly code.

- Machine Code: The byte-level programs that a processor executes
- Assembly Code: A text representation of machine code
- Microarchitecture: Implementation of the architecture
    - Examples: cache sizes and core frequency

### CPU

- Register file
- PC(Program counter)
- Condition Codes (flags)

### Memory

- Byte addressable array
- Code and user data
- Stack to support procedures

### Assembly

- Constants
    - `$-15213` (decimal), `$0x3b6d` (hex)
- Registers
    - `%rax`
- Memory address
    - `(%rax)` : addr of rax
    - `0x1c(%rax)` : addr of rax + 0x1c
    - `0x4(%rcx, %rdi, 0x3)` : value of rcx + value of rdi * 0x3 + 0x4

### x86-64 Registers

| 64-bit | 32-bit | 16-bit | 8-bit | Special Purpose for functions | When calling a function | When writing a function |
| --- | --- | --- | --- | --- | --- | --- |
| rax | eax | ax | ah,al | Return Value | Might be changed | Use freely |
| rbx | ebx | bx | bh,bl |  | Will not be changed | Save before using! |
| rcx | ecx | cx | ch,cl | 4th integer argument | Might be changed | Use freely |
| rdx | edx | dx | dh,dl | 3rd integer argument | Might be changed | Use freely |
| rsi | esi | si | sil | 2nd integer argument | Might be changed | Use freely |
| rdi | edi | di | sil | 1st integer argument | Might be changed | Use freely |
| rbp | ebp | bp | bpl | Frame Pointer | Maybe Be Careful | Maybe Be Careful |
| rsp | esp | sp | spl | Stack Pointer (Current Stack Top) | Be Very Careful! | Be Very Careful! |
| r8 | r8d | r8w | r8b | 5th integer argument | Might be changed | Use freely |
| r9 | r9d | r9w | r9b | 6th integer argument | Might be changed | Use freely |
| r10 | r10d | r10w | r10b |  | Might be changed | Use freely |
| r11 | r11d | r11w | r11b |  | Might be changed | Use freely |
| r12 | r12d | r12w | r12b |  | Will not be changed | Save before using! |
| r13 | r13d | r13w | r13b |  | Will not be changed | Save before using! |
| r14 | r14d | r14w | r14b |  | Will not be changed | Save before using! |
| r15 | r15d | r15w | r15b |  | Will not be changed | Save before using! |

### Operations

[[Cheatsheet](https://www.cs.cmu.edu/afs/cs/academic/class/15213-s20/www/recitations/x86-cheat-sheet.pdf)]

1. Move Data: `movq Source, Dest` 
    - `mov`  vs. `movq`
        - `movq` move 8 bytes (64 bits) data
        - `mov` move data which the size based on data
    - `movq (%rdi), %rax` : Store the memory address of `%rdi` (not value) into `%rax`
2. Computes an address and stores it in a register: `leaq Src, Dst` 
    - `leaq (%rdi), %rax` is same as `rax = rdi;`
    - `Src`  should be an address.
3. `addq Src,Dest Dest` = Dest + Src
4. `subq Src,Dest Dest` = Dest − Src
5. `imulq Src,Dest Dest` = Dest * Src
6. `shlq Src,Dest Dest` = Dest << Src Synonym: salq
7. `sarq Src,Dest Dest` = Dest >> Src Arithmetic
8. `shrq Src,Dest Dest` = Dest >> Src Logical
9. `xorq Src,Dest Dest` = Dest ^ Src
10. `andq Src,Dest Dest` = Dest & Src
11. `orq Src,Dest Dest` = Dest | Src

### C to Object code

![c-to-object-code](/machine-program-data/images/c_to_object_code.png)

### Condition Code

Single bit registers

- CF (Carry Flag (for unsigned))
- ZF (Zero Flag)
- SF (Sign Flag (for signed))
- OF (Overflow Flag (for signed))

## Machine Data

### Primitive Types

**64bit machine**

- `char` = 1 byte
- `short` = 2 bytes
- `int` = 4 bytes
- `long` = 8 bytes
- `double` = 8 bytes
- `pointer` = 8 bytes

**32bit machine**

- `char` = 1 byte
- `short` = 2 bytes
- `int` = 4 bytes
- `long` = 4 bytes
- `double` = 8 bytes
- `pointer` = 4 bytes

### Compound Types

- Alignment required.
- **Initial address** and **size** must both be multiples of the alignment requirement.

**struct**

- Alignment requirement: **Initial address** and **size** must both be multiples of the size of largest member.
- The initial address of member need to be multiples of the size of itself.
```c
struct {
char s;         // 1 byte
			    // padding 1 byte
short sa[4];    // 4 × 2 = 8 bytes
char c;         // 1 byte
                // padding 1 byte
} exam;

struct {
  short s;  // 2-byte type
            // padding 2 bytes
  int i;    // 4-byte type
  short sa[4];
  char c;   // 1-byte type
            // padding 3 bytes
} exam;
```
### union

- Alignment requirement: **Initial address** and **size** must both be multiples of the size of largest member.
- Total size = size of largest member.