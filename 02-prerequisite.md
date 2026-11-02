
## 2. Prerequisite: CPU Architecture
 
> Concept only. This explains why a Docker image built for one CPU may not run on another.
 
### 2.1 What does a CPU do?
 
The CPU repeats a loop forever: **Fetch -> Decode -> Execute -> Store**.
 
```mermaid
flowchart LR
    F["1. Fetch<br/>get next instruction<br/>from memory"] --> D["2. Decode<br/>understand the<br/>instruction"]
    D --> E["3. Execute<br/>do the work<br/>(ALU)"]
    E --> S["4. Store<br/>write result to<br/>register or memory"]
    S --> F
```
 
### 2.2 Inside a Modern CPU
 
```mermaid
flowchart TD
    subgraph CPU["CPU Package"]
        subgraph C0["Core 0"]
            R0["Registers"]
            ALU0["ALU / FPU"]
            L1a["L1 Cache"]
        end
        subgraph C1["Core 1"]
            R1["Registers"]
            ALU1["ALU / FPU"]
            L1b["L1 Cache"]
        end
        L2["L2 Cache"]
        L3["L3 Cache (shared)"]
    end
    RAM["RAM (main memory)"]
    DISK["Disk / SSD"]
 
    L1a --> L2
    L1b --> L2
    L2 --> L3
    L3 --> RAM
    RAM --> DISK
```
 
| Term | Meaning |
|---|---|
| **Core** | An independent processing unit. More cores = more tasks in parallel. |
| **Thread** | A path of execution. One core may run 2 threads (hyper-threading / SMT). |
| **Registers** | Tiny, fastest storage inside the core. |
| **Cache (L1, L2, L3)** | Small, fast memory that stores frequently used data. |
| **ALU** | Arithmetic Logic Unit, performs calculations. |
 
Speed order (fast to slow): **Registers -> L1 -> L2 -> L3 -> RAM -> SSD -> Network**.
 
### 2.3 Instruction Set Architecture (ISA)
 
The **ISA** is the "language" a CPU understands. A program is compiled into machine code for **one specific ISA**.
 
```mermaid
flowchart TD
    SRC["Source code (C, Go, Rust, ...)"] --> COMP["Compiler"]
    COMP --> X86["Machine code for x86-64<br/>(Intel / AMD)"]
    COMP --> ARM["Machine code for ARM64<br/>(Apple M-series, Graviton, Raspberry Pi)"]
    X86 --> CPU1["x86-64 CPU runs it"]
    ARM --> CPU2["ARM64 CPU runs it"]
    X86 -. cannot run directly .-> CPU2
    ARM -. cannot run directly .-> CPU1
```
 
### 2.4 Common Architectures
 
| Architecture | Other names | Type | Found in |
|---|---|---|---|
| **x86-64** | `amd64`, `x64` | CISC | Intel / AMD PCs, laptops, most servers |
| **ARM64** | `arm64`, `aarch64` | RISC | Apple Silicon (M1-M4), AWS Graviton, Raspberry Pi 4/5, phones |
| **ARMv7** | `arm/v7`, `armhf` | RISC (32-bit) | Older Raspberry Pi, IoT |
| **RISC-V** | `riscv64` | RISC (open) | Emerging boards and chips |
 
- **CISC**: many complex instructions (x86).
- **RISC**: fewer, simpler instructions; usually power-efficient (ARM).
### 2.5 Why This Matters for Docker
 
A container runs machine code **directly on the host CPU** (no translation). So:
 
```mermaid
flowchart LR
    IMG_AMD["Image built for<br/>linux/amd64"] -->|runs natively| H_AMD["amd64 host<br/>(Intel / AMD)"]
    IMG_ARM["Image built for<br/>linux/arm64"] -->|runs natively| H_ARM["arm64 host<br/>(Apple M-series / Graviton)"]
    IMG_AMD -.->|needs QEMU emulation, slower| H_ARM
    IMG_ARM -.->|needs QEMU emulation, slower| H_AMD
```
 
Check your CPU architecture:
 
```bash
uname -m
# x86_64  -> amd64
# aarch64 -> arm64
```
 