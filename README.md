
### Pointer Dereferencing vs Direct Indexing — Performance and Assembly Analysis

### Overview

This project explores the differences between **pointer arithmetic (dereferencing)** and **direct indexing** in nested loops accessing array elements. The goal was to observe whether modern compilers generate different assembly code—especially regarding SIMD Register_Usage_Opt—and how aliasing assumptions influence optimization.

---
In this project:

* Implemented two functions performing element-wise addition on integer arrays:

  1. `add` — uses **direct indexing** (`a[i] = b[i] + c[i]`)
  2. `add_vectorized` — uses **pointer arithmetic** (`*(a + i) = *(b + i) + *(c + i)`)

* Compiled both with aggressive optimization flags (e.g., `-O3` or `/O2`) to enable vectorization.

* Examined generated assembly code to check for SIMD instructions and register usage.

* Measured runtime performance to compare efficiency.

---

## Key Findings


* **Both versions use SIMD instructions** — notably XMM (SSE) registers — confirming that compilers aggressively optimize both pointer and index accesses.

* We look for **evidence of vectorization** 

In the disassembly output, this looks like:

```asm
  000000000000006B: 66 0F FE C8        paddd       xmm1,xmm0
  000000000000006F: F3 0F 6F 44 10 C0  movdqu      xmm0,xmmword ptr [rax+rdx-40h]
  0000000000000075: F3 0F 7F 4C 07 B0  movdqu      xmmword ptr [rdi+rax-50h],xmm1
  000000000000007B: 66 0F FE D0        paddd       xmm2,xmm0
  000000000000007F: F3 0F 6F 44 10 D0  movdqu      xmm0,xmmword ptr [rax+rdx-30h]
  0000000000000085: F3 0F 6F 48 D0     movdqu      xmm1,xmmword ptr [rax-30h]
  000000000000008A: F3 0F 7F 54 07 C0  movdqu      xmmword ptr [rdi+rax-40h],xmm2
```

* The **presence of XMM registers** in assembly indicates that loops are vectorized for parallel data processing, regardless of pointer vs direct indexing syntax.

* Modern compilers use advanced alias analysis and heuristics to **assume no harmful aliasing** or insert runtime checks, enabling vectorization even when pointers are involved.

* Differences in syntax (`a[i]` vs `*(a + i)`) have **minimal to no impact** on the quality of generated machine code when optimizations are enabled.

---


## Why This Matters

* Compiler optimizations focus more on **aliasing assumptions** than pointer syntax.

* Using `restrict` (or compiler-specific equivalents) can help the compiler optimize more aggressively by explicitly stating no pointer aliasing.

* For developers, writing clean, simple loops and enabling optimization flags often results in efficient SIMD-accelerated code regardless of pointer usage.

---


## 🔍 Parsing the Assembly

After the build, the CMake script generates an assembly dump using:

* **Linux/macOS**: `objdump -d -M intel`
* **Windows (MSVC)**: `dumpbin /DISASM`

The C++ program takes the path to the assembly file and searches for the function symbols (mangled names) for `add` and `add_vectorized`.

```bash
./Register_Usage_Opt analysis/assembly.txt
```

It then prints the extracted assembly instructions for manual inspection.

---
## Build & Run Instructions (Local)

```bash
cmake -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build
./build/Register_Usage_Opt build/analysis/assembly.txt
```


## Conclusion

This experiment confirms that **pointer dereferencing and direct indexing are treated equivalently by modern optimizing compilers**, producing vectorized assembly code using XMM registers. The **aggressive use of SIMD instructions** occurs even when aliasing might theoretically exist, demonstrating the sophistication of compiler optimization and alias analysis.

---
