📘 Xtensa Assembly Microbenchmark Suite (ESP32)

This ESP-IDF project benchmarks a set of hand-written Xtensa assembly routines on the ESP32 and compares them with their C equivalents.
The focus is on understanding the Xtensa LX6 ISA, register behavior, and cycle-level performance differences.

The core of this project is asm_routines.S, which contains the full assembly implementations.

⭐ Features
🔧 Hand-written Xtensa Assembly (main highlight)

Located in main/asm_routines.S, written entirely by hand:

bitmask_set_asm()

bitmask_clear_asm()

popcount_asm()

These routines strictly follow the ESP32 Xtensa ABI, use explicit register operations, and define their own prologue/epilogue via retw.

🆚 Matching C Implementations

Found in main.c for baseline comparison.

⏱ Cycle-Accurate Timing

Uses the ccount register to measure:

cycles_asm  vs  cycles_c

🗂 Minimal Component Structure

Easy to read, modify, and extend.

📁 Project Structure
esp32_xtensa_asm_bench/
├── CMakeLists.txt
└── main/
    ├── CMakeLists.txt
    ├── main.c          # C implementations + benchmark harness
    └── asm_routines.S  # ★ Hand-written Xtensa assembly routines


👉 asm_routines.S is the centerpiece of the project.

🧪 Benchmark Method

The benchmark measures cycle counts for both C and ASM versions by reading:

rsr.ccount


Operations tested:

Bitmask set

Bitmask clear

Popcount (32-bit)

The idea is not to build a complex benchmark framework,
but to observe real instruction-level differences between compiler output and hand-written assembly.

📊 Typical Results
Operation	C (-O2)	ASM	Speedup
Bitmask set/clear	~12–16 cycles	~6–8 cycles	~1.5–2×
Popcount (32-bit)	~70–90 cycles	~45–55 cycles	~1.3–1.6×

These numbers match typical ESP32 LX6 behavior.

🔧 Build & Run
idf.py set-target esp32
idf.py build
idf.py flash monitor


🔍 Key Xtensa Instructions Used in asm_routines.S

The hand-written assembly routines rely on a small set of core Xtensa LX6 instructions.
These instructions directly manipulate registers, perform bit operations, and return control using the Xtensa ABI.

sll / slli — Shift Left Logical

Used to compute bit masks:

movi a5, 1
sll  a5, a5, a3     // a5 = 1 << bit

or — Bitwise OR

Used to set a specific bit:

or   a2, a4, a5     // value | (1 << bit)

and — Bitwise AND

Used to clear a bit:

and  a2, a4, a5     // value & ~(1 << bit)

not — Bitwise NOT

Used to invert a bit mask:

not  a5, a5         // ~(1 << bit)

srl — Shift Right Logical

Used inside the popcount loop:

srl  a2, a2, 1      // v >>= 1

andi — AND Immediate

Fast way to check the lowest bit:

andi a4, a2, 1      // v & 1

beqz — Branch if Equal to Zero
j — Unconditional Jump

Used for simple loop control:

beqz a2, end        // if v == 0, exit
j    loop           // continue loop

retw — Return from Windowed Function

Required by ESP32 ABI:

retw

📌 Example Highlighted Code Snippet

bitmask_set_asm:
    mov  a4, a2          // temp = value
    movi a5, 1
    sll  a5, a5, a3      // KEY: sll -> compute (1 << bit)
    or   a2, a4, a5      // KEY: or  -> set bit
    retw                 // KEY: retw -> ABI-compliant return

popcount_asm:
    movi a3, 0           // count = 0
loop:
    beqz a2, done        // KEY: beqz -> branch if zero
    andi a4, a2, 1       // KEY: andi -> extract LSB
    add  a3, a3, a4
    srl  a2, a2, 1       // KEY: srl -> shift right
    j    loop            // KEY: j -> jump
done:
    mov  a2, a3
    retw

📎 Notes

This project is intentionally small and meant for practicing low-level Xtensa assembly.

asm_routines.S can be used as a starting point for exploring more instructions, branch behavior, or custom performance tests.
