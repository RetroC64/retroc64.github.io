---
title: 🧭 Overview
---

This overview provides details about the architecture and components of the RetroC64 SDK.


## Architecture

<img src="/img/RetroC64-Architecture.drawio.svg" class="svg-diagram" alt="Architecture Diagram">

The SDK is architecture around 4 main packages:

The [Main package](https://github.com/RetroC64/RetroC64/tree/main/src/RetroC64) that contains:
- The App Layer that manages the life cycle of the build, dotnet watch and live sync with the emulator
- A debugger that implements the [Debug Adapter Protocol](https://microsoft.github.io/debug-adapter-protocol/) protocol that connects to the VICE emulator through a remote interface

The [Core package](https://github.com/RetroC64/RetroC64/tree/main/src/RetroC64.Core) that contains all the core functionalities:
- A Basic compiler
- A C64 assembler with a zero-page allocator
- A fast-loader code
- Packer routines
- Graphics helpers - like the sprite class integrated with SkiaSharp
- Music helpers - that can load, relocate and play SID files
- Storage of D64/PRG files
- And various other things like C64 registers definitions

We have the [Asm6502](https://github.com/xoofx/Asm6502) project that powers:
- The assembler
- The disassembler used by the debugger
- And an emulator used by the <a href="/docs/music//#relocating-sid-files">SID relocator</a>

And finally, [Vice package](https://github.com/RetroC64/RetroC64/tree/main/src/RetroC64.Vice) that implements the custom socket binary protocol to communicate with the VICE emulator.

> [!NOTE]
> The fast-loader code and depacker routines are actually not finished yet and will be hopefully available in a future release.
>
> 1. The packer has been implemented, but the depacker routines are not yet integrated into the SDK. (See issue [#1](https://github.com/RetroC64/RetroC64/issues/1) on GitHub)
> 2. Part of the fast-loader has been ported, but not all of it yet. (See issue [#2](https://github.com/RetroC64/RetroC64/issues/2) on GitHub)
> 3. Lastly, in order to support multi-part loaders, the debugger will also require some improvements. (See issue [#3](https://github.com/RetroC64/RetroC64/issues/3) on GitHub)

