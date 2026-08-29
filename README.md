# GT1: Redlined

**A static recompilation of *Gran Turismo* (1997, PS1, NTSC-U) into a native, emulator-free PC executable.**

Every original MIPS R3000 instruction is translated ahead of time into equivalent C++, with BIOS, GPU, and CD-ROM behavior modeled directly in a hand-written runtime harness — the same technique behind [N64Recomp](https://github.com/N64Recomp/N64Recomp) (Zelda 64: Recompiled) and [PS1Recomp](https://github.com/PS1Recomp/ps1-recomp), applied here to a title reverse-engineered from scratch.
<img width="629" height="470" alt="image" src="https://github.com/user-attachments/assets/84338e8c-cc17-4cd1-a760-d188a7f5a707" />


---

## Status

**Pre-boot. Toolchain complete; reverse engineering is the current phase.**

| Milestone | State |
|---|---|
| Disc verified against redump (SCUS-94194) | ✅ |
| Ghidra + `ghidra_psx_ldr` (PSX-EXE / MIPS-LE loader) | ✅ |
| `mkpsxiso`/`dumpsxiso` built | ✅ |
| Executable extracted from disc | ⬜ |
| Function boundaries mapped in Ghidra | ⬜ |
| First TOML config / recompiler run | ⬜ |
| BIOS HLE + threading harness | ⬜ |
| First GPU command stream logged | ⬜ |
| First rendered frame | ⬜ |
| Boots to main menu | ⬜ |

This README will get more triumphant as that table fills in.

---

## Why this game, and why it's harder than it looks

*Gran Turismo* is a first-party Sony title (`SCUS-94194`, the Sony-studio prefix rather than the third-party `SLUS` one), built against an internal PsyQ SDK toolchain, and leans on the PS1's GTE (geometry coprocessor) heavily for its physics and rendering — a step further than *Rayman* or *Crash Bandicoot*, the two titles the closest existing recompilation runtime has actually validated against. It also streams per-track code through the console's overlay mechanism continuously during play, rather than only at load screens. Every function boundary, overlay window, and PsyQ signature match here is being established for the first time, straight from raw disassembly — this appears to be the first public reverse-engineering effort applied to GT1.

## Architecture

```
GT1 disc (.bin/.cue)
        │
        ▼
dumpsxiso ──► SYSTEM.CNF, boot executable (PSX-EXE)
        │
        ▼
Ghidra + ghidra_psx_ldr ──► function boundaries, overlay map,
        │                    GTE macro segments (manual + signature-assisted)
        ▼
Recompiler config (TOML) ──► per-segment memory layout, overlay table,
        │                    function list, hand patches
        ▼
Static recompiler ──► one C++ function per original MIPS routine,
        │              signature: fn(uint8_t* rdram, recomp_context* ctx)
        ▼
Runtime harness (this repo's real code) ──► BIOS/HLE syscalls, GPU command
        │                                    interpreter → OpenGL, CD-ROM
        │                                    state machine, SPU, SDL2 host
        ▼
Native binary — macOS / Linux / Windows
```

The recompiled function bodies are machine-generated; corrections flow through the Ghidra analysis or the TOML config and regenerate from there, keeping the pipeline reproducible from source disc to binary. The runtime harness above it is the real C++ this repo is about: a from-scratch hardware simulation layer standing in for what used to be silicon.

## Toolchain

- **Reverse engineering:** Ghidra 12.1.2 + [`ghidra_psx_ldr`](https://github.com/lab313ru/ghidra_psx_ldr) (little-endian MIPS R3000, PSX-EXE header parsing, GTE macro decompilation, overlay support)
- **Disc tooling:** [`mkpsxiso`/`dumpsxiso`](https://github.com/Lameguy64/mkpsxiso)
- **Recompiler:** TOML-configured static recompiler in the [N64Recomp](https://github.com/N64Recomp/N64Recomp)/[PS1Recomp](https://github.com/PS1Recomp/ps1-recomp) lineage
- **Runtime harness:** C++17, SDL2 (window/input/audio), OpenGL 3.3 (VRAM/GPU command rendering)
- **Host:** developed on macOS (Apple Silicon), targeting cross-platform

## Legal

This repository's version control stays limited to source and configuration — disc images, extracted executables, and game assets stay local only, enforced via `.gitignore`. Building or running anything here requires your own legally-owned copy of *Gran Turismo* (SCUS-94194, NTSC-U). This is an independent, fan-driven reverse-engineering project; Sony Interactive Entertainment and Polyphony Digital own *Gran Turismo* and hold all associated rights.

## Static recompilation vs. emulation

An emulator interprets or JIT-compiles PS1 instructions at runtime, every run, forever. A static recompiler performs that translation once, ahead of time, producing an ordinary native executable that expresses the game's original logic directly as compiled C++. The tradeoff is upfront cost: every function boundary, every overlay, every piece of hardware the game touches has to be correctly identified and modeled before anything runs. That upfront cost is most of what this repository represents.
