LCC-LC3 C Compiler (fmrt-port)
===============================

A port of Ladsaria & Patel's LCC-based C compiler for the LC-3 architecture,
updated for modern 64-bit build hosts and C23 compilers.

## Changes (fmrt-port branch)

### 64-bit Host Compatibility

The original codebase targeted 32-bit hosts.  On 64-bit systems:

- **Removed `-m32` and `-mno-sse` flags** from Makefiles — these are x86-specific
  and break compilation on non-x86 or pure 64-bit environments.

- **Fixed `-Wpointer-to-int-cast` warnings** — `Value` union member `v.p`
  (`void *`, 8 bytes) was cast directly to `unsigned` or `int` (4 bytes) in
  five backend `defconst` functions (`lc3`, `x86`, `mips`, `sparc`, `x86linux`).
  Inserted intermediate casts through `unsigned long` / `long` to match the
  pointer width.  Applied to both the `.md` source templates and the
  `lburg`-generated `.c` files.

- **Fixed ABI notes** (`the ABI of passing union with 'long double' has changed
  in GCC 4.4`) — Changed `long double d` to `double d` in the `Value` union
  (`src/c.h`) and the corresponding `va_arg` call (`src/enode.c`).  The
  compiler only needs 64-bit precision for constant folding.

### C23 Compatibility

- **Renamed `constexpr` → `const_expr`** (`src/c.h`, `src/simp.c`, `src/stmt.c`,
  `src/init.c`) — `constexpr` is a reserved keyword in C23, causing a hard
  compile error.  `const_expr` preserves the semantic meaning without conflicts.

- **Fixed `-Wdiscarded-qualifiers`** — `strchr()` on a `const char *` returns
  `char *`.  Declared receiving pointers as `const char *` in `etc/lcc.c`
  and `cpp/getopt.c`.

### LC-3 Backend Bug Fixes

- **JSRR R7 startup bug** (`src/lc3.md`) — `JSRR` on LC-3 is **write-then-read**:
  it first saves `PC+1` (return address) into the destination register, *then*
  reads that register for the jump target.  The startup code loaded `main`'s
  address into R7 and did `JSRR R7`, causing the return address to overwrite
  `main`'s address before the jump.  Fixed by loading `main` into R0 instead:
  `JSRR R0` saves the return address in R7 and jumps to `main` correctly.

### Build System

- **Stopped tracking `lburg`-generated `.c` files** — `src/alpha.c`,
  `dagcheck.c`, `lc3.c`, `mips.c`, `sparc.c`, `x86.c`, `x86linux.c` are
  generated from `.md` templates and no longer committed to the repository.
  Added to `.gitignore`.

## Description Of Contents

This is a distribution of LCC that supports the LC-3.
Copyright information is in `CPYRIGHT`.  There is absolutely no warranty
for this software.  Installation information is in `INSTALL`.  `TODO`
contains a to-do list.

## Installation

```sh
./configure
make
```

The compiler binaries (`rcc`, `lcc`, `cpp`, `lc3pp`) are built in-tree.
You can run them directly or install with `make install`.

## How To Use

Compiling is similar to a standard C compiler.  Behind the scenes:

1. `.c` files are compiled to pseudo-assembly `.lcc` files
2. `lc3pp` links the `.lcc` files and library files into a single `.asm` file
3. The LC-3 assembler (`lc3as`) assembles the `.asm` into a `.obj` file

See `test/regression/Makefile` for an example.

**Limitations:** Floating-point types are not supported.  Some complex integer
expressions may not generate properly due to the limited LC-3 register set.

## Original Authors

- Ajay Ladsaria
- Sanjay J. Patel (sjp@crhc.uiuc.edu)

## Previous Contributors

- Sean Smith (Dartmouth College)
- Stephen Canon (Mac OS X port)
- Avery Yen (install path fixes)
