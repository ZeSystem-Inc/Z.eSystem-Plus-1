# ZeSystemPlus 🚀 (v1.0.0 - Base Kernel Architecture)

An independent, from-scratch x86 operating system written in Assembly and C++. This repository represents the initial foundation of **ZeSystemPlus**, focusing on establishing a clean, modular environment without external dependencies or commercial bloat.

This version forms the fundamental baseline before advanced features like custom filesystems (ESFS) or dynamic hardware scanning (PCI) are introduced.

## 🛠️ Architecture & Components

The initial system architecture is split into three core pillars:

1. **Custom Bootloader (`boot.asm`)**
   - Written in 16-bit x86 Real Mode Assembly.
   - Handles the initial BIOS execution sequence at `0x7C00`.
   - Disables interrupts, enables the A20 line for extended memory access.
   - Sets up a basic **Global Descriptor Table (GDT)**.
   - Performs the critical transition into **32-bit Protected Mode** by setting the PE (Protection Enable) bit in the `CR0` register.
   - Performs a far jump to clear the pre-fetch queue and hand control over to the C++ kernel.

2. **Modular C++ Kernel (`kernel.cpp`)**
   - A bare-metal C++ runtime environment running natively in 32-bit Protected Mode.
   - Contains the main entry function (`kernel_main`) called directly by the bootloader.
   - Direct memory-mapped I/O configuration for basic text display using the VGA text buffer (`0xB8000`).

3. **Linker Definition (`linker.ld`)**
   - Instructs the linker (`ld`) on how to stitch the assembly bootstrap code and compiled C++ objects together.
   - Explicitly ensures correct section alignment (`.text`, `.data`, `.bss`) to guarantee memory safety during low-level execution.

## 🚀 Getting Started

### Prerequisites
To build and compile this operating system from source, you will need a cross-compilation environment to target `i686-elf` (to prevent your host machine's OS headers from polluting the standalone binary):
- **NASM** (Netwide Assembler)
- **i686-elf-gcc / i686-elf-g++** (Cross-Compiler)
- **i686-elf-ld** (Linker)
- **QEMU** or **Bochs** (Emulators for testing)

### Building the OS Image
To assemble the bootloader, compile the kernel, and link them into a single bootable raw disk image (`os.img`), execute the following commands:

```bash
# 1. Assemble the bootloader
nasm -f bin boot.asm -o boot.bin

# 2. Compile the C++ kernel source
i686-elf-g++ -c kernel.cpp -o kernel.o -ffreestanding -O2 -Wall -Wextra -fno-exceptions -fno-rtti

# 3. Link the objects using the linker script
i686-elf-ld -T linker.ld -o kernel.bin kernel.o --oformat binary

# 4. Create a unified bootable raw image (combining boot sector and kernel)
cat boot.bin kernel.bin > os.img
