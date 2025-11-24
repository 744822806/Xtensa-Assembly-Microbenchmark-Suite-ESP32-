Xtensa Assembly Microbenchmark Suite (ESP32)

This project contains a few small Xtensa assembly routines for the ESP32 and a simple benchmark that compares them against equivalent C implementations.
The goal is to get familiar with the Xtensa LX6 ISA, calling conventions, register usage, and observe the performance differences at the instruction level.

Features

Bitmask operations written in hand-crafted Xtensa assembly

A simple popcount (32-bit) routine in assembly

Matching C versions for baseline comparison

Cycle timing using the ccount register

Minimal ESP-IDF component layout

Directory Structure
esp32_xtensa_asm_bench/
├── CMakeLists.txt
└── main/
    ├── CMakeLists.txt
    ├── main.c
    └── asm_routines.S


asm_routines.S contains the assembly functions.
main.c includes the C versions and a very basic benchmark loop.

Benchmark Method

The benchmark reads the CPU cycle counter (rsr.ccount) before and after running each routine in a tight loop.

Tested operations:

bitmask_set(value, bit)

bitmask_clear(value, bit)

popcount(v)

This isn’t a full microarchitectural study—just a straightforward way to compare instruction count and compiler overhead.

Typical Results (based on LX6 behavior)

On a standard ESP32 (240 MHz), hand-written assembly usually performs slightly better than optimized C:

Operation	C (-O2)	ASM	Speedup
Bitmask set/clear	~12–16 cycles	~6–8 cycles	~1.5–2×
Popcount (32-bit)	~70–90 cycles	~45–55 cycles	~1.3–1.6×

Exact numbers vary depending on toolchain and optimization flags.

Build & Run
idf.py set-target esp32
idf.py build
idf.py flash monitor


Benchmark results will be printed to the serial console.

Notes

This is a minimal example intended for learning and experimentation.

It can be extended to test other instructions, branching patterns, register pressure, or compiler-generated code for comparison.
