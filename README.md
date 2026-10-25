# The Frankenstein Computer — SCUFF32

**A modular RISC-V FPGA computer built from the hardware up.**

The Frankenstein Computer is an independent hardware/software co-design project to turn a custom RISC-V processor into a functioning computer. The long-term vision is to integrate interchangeable CPU microarchitectures, a lightweight operating system, peripherals, hardware accelerators, security experiments, and a custom FPGA motherboard.

**Current focus:** **SCUFF-DOS**, a lightweight DOS-like environment running on an existing **dual-issue, in-order RV32I superscalar CPU** designed in SystemVerilog. GPU, NPU, out-of-order CPU, and custom-motherboard work are planned ideas—not completed features or committed deadlines.

## Project status

| Component | Status | Scope |
| --- | --- | --- |
| 2-way superscalar RV32I CPU | **Implemented / under verification** | In-order RTL processor |
| RV32I C cross-compilation | **Demonstrated in RTL simulation** | Startup assembly, linker script, GCC-built C program |
| SCUFF-DOS | **In development** | Boot firmware, UART console, shell, loadable applications |
| Basys 3 FPGA SoC | **Planned** | Physical memory, UART, clock/reset, board integration |
| Custom FPGA motherboard | **Planned** | KiCad PCB with FPGA mounted directly |
| Tiny-GPU / `andrewCompute` | **Future idea** | Rendering, compute kernels, GPU runtime |
| NPU / computer vision | **Future idea** | INT8 inference, optional camera interface |
| OoO CPU / hardware security | **Future ideas** | Alternate CPU and controlled security experiments |

**Verified baseline:** A GCC-generated RV32I C program containing calls, stack operations, loops, and RAM writes completed in RTL simulation at one-cycle memory latency: **336 cycles**, with `RAM[0x1000] = 0x0000002d` (45). A two-cycle memory-latency run timed out; instruction/data-memory handshakes and pipeline stalls remain active debugging work. The existing CPU has also been taken through a SKY130 OpenLane physical-design flow, with **90.9 MHz reported post-route timing**; this is a tool result, **not a measurement of fabricated silicon**.

## Repository organization

```text
SCUFF32/
├── 0-kicad/                         # KiCad motherboard design (future)
│   ├── billofmaterials/
│   ├── board_layout/
│   ├── fabrication/`                # Gerbers, BOM, assembly outputs (when ready)
│   └── schematics/              
├── 1-verilog_src/                   # Synthesizable SystemVerilog
│   ├── riscv_cores/
│   │   ├── core_riscv_v1/           # Existing 2-way superscalar core
│   │   └── core_riscv_v2/           # Future 2-way OoO core
│   ├── soc_top/                     # SoC top-level and address decoding
│   ├── interconnect/                # Bus adapters and arbitration
│   ├── peripherals/                 # UART, GPIO, timer, video, etc.
│   ├── gpu_core/                    # Future graphics/compute accelerator
│   └── tiny-npu/                    # Future inference accelerator
├── 2-software/
│   ├── boot/                      # Startup assembly and linker scripts
│   ├── dos/                       # Kernel, shell, and drivers
│   ├── lib_gpu/                   # Future GPU C API and runtime, andrewCompute
│   ├── lib_npu/                   # Test ML/inference workloads 
│   ├── libc/                      # Minimal freestanding C utilities
│   └── programs/                  # OS startup, Snake, security attacks
├── 3-verification/
│   ├── cpu/                       # Core testbench and CPU regressions
│   ├── soc/                       # End-to-end memory/MMIO/boot tests
│   ├── peripherals/               # UART and other peripheral tests
│   └── accelerators/              # Future GPU/NPU tests
├── 4-fpga/
├── 5-tools/
│   ├── image_builder/             # ELF/bin -> $readmemh images
│   ├── uart_loader/               # Future executable transfer utility
│   └── scripts/                   # Reproducible build/run scripts
├── .gitignore
├── Makefile                      
└── README.md
```

## 0 — PCB design (KiCad)

The eventual **SCUFF32 motherboard** will be a custom PCB with a Xilinx/AMD FPGA soldered directly to the board. The PCB will carry the hardware needed to make the FPGA a standalone computer rather than a development-board-only demonstration.

**Proposed hardware:** FPGA power supplies and decoupling, oscillator/reset, configuration flash and JTAG, external RAM, UART/USB-UART, keyboard interface, VGA or HDMI output, storage, and expansion headers. An NPU camera connector may be added if useful.

**Design workflow:** Choose the FPGA **after** estimating the combined LUT/FF/BRAM/DSP requirements of the CPU and potential accelerators; plan FPGA I/O banks and pin assignments; design and review the schematic; route the board against the chosen stack-up and assembly capabilities; manufacture, assemble, and bring up subsystems individually.

**Status:** Planned. The Basys 3 will serve as the initial prototype before committing to a BGA FPGA PCB.

## 1 — Verilog / SystemVerilog RTL

The hardware foundation is an existing **two-way, in-order RV32I superscalar processor**. Its RTL includes pipelined instruction processing, dual-issue/dispatch control, hazard and forwarding logic, and branch prediction. The CPU will eventually be integrated with a memory subsystem, address decoder, and memory-mapped peripherals in a reusable SoC wrapper.

**Architectural goals:**

- Keep CPU implementations interchangeable behind a stable memory/bus interface.
- Make instruction and data accesses work with realistic synchronous FPGA memories and variable response latency.
- Provide a consistent physical address map for boot ROM, RAM, peripheral registers, and future accelerators.
- Extend the SoC later with GPU, NPU, DMA, and potentially an out-of-order CPU without requiring a redesign of the existing software interface.

The first milestone is **reliable memory-ready/stall behavior** in the existing processor before adding more hardware.

## 2 — Software / SCUFF-DOS

**SCUFF-DOS** is a lightweight, DOS-like bare-metal environment written primarily in **C and RISC-V assembly**. It is not intended to be MS-DOS compatible.

The initial software stack consists of startup assembly, a linker script, C runtime initialization, UART routines, and an interactive shell. The first commands will be simple utilities such as `help`, `echo`, `ver`, and `mem`.

As the platform matures, SCUFF-DOS may add RAM-loaded programs, a basic executable format or ELF loading, simple games, timers, persistent storage, and experimental security programs. The first applications can be **cross-compiled on a Mac**; an on-device editor or compiler is optional, later work.

The **`andrewCompute`** C library is a future GPU programming interface. Its proposed API includes `andrewMalloc`, `andrewMemcpy`, `andrewLaunch`, `andrewSynchronize`, and `andrewFree`. Initial implementations could control fixed-function hardware operations or predefined compute kernels—no custom programming language or C compiler is required to start.

## 3 — Verification — why it matters

A processor that passes one idealized testbench is not yet a reliable computer. **Verification is the bridge between apparently correct RTL and software that continues to work with real hardware timing.** Stalls, flushes, branches, memory side effects, and dual-issue replay must behave correctly even when memory or peripherals respond later than expected.

Verification priorities:

- Run directed RV32I instruction tests and GCC-generated C programs.
- Check architectural register/memory results against expected outcomes.
- Vary instruction- and data-memory latency and verify transaction acceptance exactly once.
- Exercise branches, jumps, function calls, loads/stores, byte enables, and pipeline flushes.
- Add SoC tests for boot, RAM, UART registers, address decoding, and (later) DMA/accelerators.
- Maintain repeatable regressions so hardware changes do not silently break SCUFF-DOS.

**Known issue:** The current C test passes at `+lat=1`, but `+lat=2` times out. Resolving the memory handshake/pipeline-stall interaction is the immediate verification task.

## 4 — FPGA implementation

The initial hardware target is the **Digilent Basys 3 (Artix-7)**, using **Vivado** to synthesize, implement, and generate the FPGA bitstream. The Basys 3 provides a way to test clock/reset, BRAM-backed instruction/data storage, and physical UART communication before designing the custom PCB.

Planned progression:

1. Map the processor and memory controller into an FPGA top-level.
2. Boot initialized firmware from FPGA memory and execute real compiled C code.
3. Add a memory-mapped UART and communicate with a Mac serial terminal.
4. Boot into a working `SCUFF-DOS>` shell.
5. Port the validated SoC to the future SCUFF32 motherboard, with its external memory and peripherals.

The eventual FPGA device must be selected from measured resource usage and bandwidth needs—not solely from the current Basys 3 design.

## 5 — Toolchain and development utilities

| Tool | Role |
| --- | --- |
| **SystemVerilog** | CPU, memory, peripheral, and accelerator RTL |
| **Icarus Verilog (`iverilog` / `vvp`)** | Existing RTL simulation and C-program execution tests |
| **RISC-V GNU cross-toolchain** | Compile C and assembly to RV32I machine code |
| **GNU linker + linker scripts** | Resolve symbols and place code/data into the SoC address map |
| **`objcopy` / `objdump`** | Extract binary images and inspect RISC-V machine code |
| **Python image conversion** | Generate `$readmemh` instruction/data images from linked binaries |
| **Vivado + XDC constraints** | FPGA synthesis, implementation, timing, and bitstream generation |
| **KiCad** | Future SCUFF32 schematic and PCB layout |
| **Mac serial terminal** | UART-based communication with running firmware |

Typical compiler settings for the current core include `-march=rv32i -mabi=ilp32` with a freestanding runtime and a project-specific linker script. The full build process will be documented alongside the working scripts as SCUFF-DOS development continues.

## Development priorities

**Now:** Fix variable-latency memory behavior → integrate ROM/RAM and UART → boot a C-based SCUFF-DOS shell → load and run a separate program.

**Later:** Security experiments, tiny GPU and `andrewCompute`, custom FPGA motherboard, NPU/camera integration, and an interchangeable two-way OoO CPU. These are exploratory directions, not promised deliverables.

## Project independence

This is a personal, independently developed project. Development should remain separate from employer systems, confidential information, work products, and intellectual property.

## References

List relevant articles, docs, research papers, YouTube videos here in IEEE format.

---

*The Frankenstein Computer: taking a processor that works in simulation and making it into a real computer.*
