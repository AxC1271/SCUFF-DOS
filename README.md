# SCUFF32 — Project Ideas & Roadmap

Current priority: SCUFF-DOS on the existing two-way superscalar, in-order RV32I CPU. Other projects are parked ideas, not commitments or deadlines.

## Long-term vision

Build a modular, custom FPGA-based computer from the CPU microarchitecture upward: RTL, SoC interconnect, firmware, OS, peripherals, accelerators, security experiments, and eventually a custom motherboard with an FPGA soldered directly onto the PCB. This is an independent personal project, separate from internship/employer work and IP.

---

## Idea backlog

1. SCUFF-DOS (NOW): Use the existing 2-way superscalar in-order RV32I processor. Build a lightweight DOS-like environment using the RV32I GCC/assembler/linker toolchain, startup code, bootable memory image, UART connection to a Mac, shell, and eventually loadable C programs and simple games.
2. Tiny-GPU / andrewCompute: Develop a SystemVerilog graphics/compute accelerator, initially with simple programmable kernels or a blitter/SIMD engine. Expose it through a deliberately playful C API: andrewMalloc, andrewMemcpy, andrewLaunch, andrewSynchronize, andrewFree. Consider DMA and a shared memory interface.
3. NPU / computer vision (lower priority): Add an INT8 inference accelerator, camera interface, and a demonstration of facial-recognition-gated shell access. Treat this as a demo, not secure authentication without spoof resistance, secure boot, and protected credentials.
4. Custom FPGA motherboard: Design and assemble a PCB with the FPGA chip mounted directly, peripherals such as UART, keyboard interface, VGA/HDMI, external RAM, flash/storage, JTAG, clocks, and power regulation. Choose FPGA after evaluating aggregate CPU/GPU/NPU resource demands; prototype on Basys 3 first. Keep flexibility for architectural swaps.
5. Two-way superscalar out-of-order CPU (lower priority): Future CPU replacement using renaming, reorder buffer, scheduling, and precise retirement while maintaining the RV32I software ecosystem. Benchmark against existing in-order core.
6. Hardware security research: Write intentionally vulnerable programs on SCUFF-DOS to study binary exploitation (e.g., stack overflow, ret2win, ROP), then implement and benchmark defenses such as stack canaries, hardware shadow stacks, control-flow integrity, and memory/privilege protection.

## Current proven baseline

- Existing two-way superscalar, in-order RV32I RTL core.
- SKY130 OpenLane post-detailed-route timing closure reported at 90.9 MHz (physical-design flow result, not measured fabricated silicon).
- GCC-generated RV32I C program with calls, stack operations, loops and RAM writes passed RTL simulation at +lat=1: 336 cycles; RAM[0x1000] = 0x2d (45).
- Simulation with +lat=2 timed out; fetch/data-memory handshake and stall handling need investigation.
- Toolchain includes riscv64-unknown-elf-gcc, linker script, startup assembly, ELF-to-binary conversion, and $readmemh instruction/data images.

## Immediate SCUFF-DOS execution plan

### Milestone A — Robust CPU/memory behavior
- Diagnose +lat=2 failure by isolating instruction-ready stalls versus data-ready stalls.
- Ensure each instruction and memory request is accepted exactly once; verify stalls, flushes, and dual-issue replay.
- Run multiple compiled C tests under varied memory latencies; check architectural outputs.

### Milestone B — Minimal FPGA SoC
- Integrate the existing CPU into a SoC wrapper with a consistent address map, boot ROM/BRAM and writable RAM.
- Address instruction/data access to shared memory regions, including .rodata, stack, and future RAM-resident executables.
- Add memory-mapped UART TX/RX/status and basic GPIO diagnostics.
- Prototype on the Basys 3 before designing the custom motherboard.

### Milestone C — First real DOS boot
- Boot startup assembly into a C firmware/kernel entry point.
- Provide UART putchar, getchar, puts, and a simple line editor.
- Show a boot banner and command prompt on the Mac serial terminal.
- Add shell commands such as help, echo, ver, and mem.

### Milestone D — Programs and growth
- Define a simple program ABI and linked memory regions.
- Load/run a separate C program from RAM, then return to the shell.
- Add a small text-mode game and eventually timer, storage, and security demos.

### Architecture guiding principles
- Keep CPU-specific logic behind stable bus interfaces so future cores can be swapped.
- Reserve memory-map space for future UART, GPIO, timer, DMA, GPU, NPU, video, and storage.
- Keep project fun and modular: finish one useful milestone at a time.
- Don't assume all future accelerators fit in the Basys 3; select the eventual PCB FPGA after actual Vivado resource and bandwidth estimates.

### Suggested names
- SCUFF32: overall FPGA computer/SoC platform.
- SCUFF-DOS: bare-metal operating environment.
- andrewCompute: experimental GPU compute runtime.
- SCUFF-HACK: hardware/software security research track.

---

Last captured: October 25, 2026.