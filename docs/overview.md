---
title: 🧭 Overview
---

This page outlines the architecture and components of the RetroC64 SDK.

## Architecture

<img src="/img/RetroC64-Architecture.drawio.svg" class="svg-diagram" alt="Architecture Diagram">

The SDK is organized into four packages:

The [Main package](https://github.com/RetroC64/RetroC64/tree/main/src/RetroC64) contains:
- App layer that manages the build lifecycle, `dotnet watch`, and live sync with the emulator
- A debugger that implements the [Debug Adapter Protocol](https://microsoft.github.io/debug-adapter-protocol/) and connects to the VICE emulator via its remote interface

The [Core package](https://github.com/RetroC64/RetroC64/tree/main/src/RetroC64.Core) provides:
- A BASIC compiler
- A C64 assembler with a zero-page allocator
- Fast-loader code
- Packer routines
- Graphics helpers, including a sprite class integrated with SkiaSharp
- Music helpers that load, relocate, and play SID files
- D64/PRG file handling
- C64 register definitions and other utilities

The [Asm6502](https://github.com/xoofx/Asm6502) project was specifically created to provide:
- The assembler
- The disassembler used by the debugger
- An emulator used notably for the <a href="/docs/music/#relocating-sid-files">SID relocator</a>

Finally, the [VICE package](https://github.com/RetroC64/RetroC64/tree/main/src/RetroC64.Vice) implements a custom binary socket protocol to communicate with the VICE emulator.

> [!NOTE]
> The fast-loader and depacker routines are not finished yet and will be available in a future release.
>
> 1. The [packer](https://github.com/RetroC64/RetroC64/tree/main/src/RetroC64.Core/Packers) is implemented, but depacker routines are not yet integrated into the SDK. (See issue [#1](https://github.com/RetroC64/RetroC64/issues/1))
> 2. Part of the [fast-loader](https://github.com/RetroC64/RetroC64/tree/main/src/RetroC64.Core/Loader) is ported; the rest is pending. (See issue [#2](https://github.com/RetroC64/RetroC64/issues/2))
> 3. To support multi-part loaders, the debugger also needs improvements. (See issue [#3](https://github.com/RetroC64/RetroC64/issues/3))


## Multi-Disk Support

RetroC64 provides a flexible multi-disk support system that allows developers to create multiple disk images within their projects.

After following the [steps to create your first RetroC64 application](/docs), replace the content of `Program.cs` with the following code:

<a href="https://github.com/RetroC64/RetroC64-Examples/tree/main/HelloMulti" class="btn btn-primary"><span class="bi bi-github"></span> Code on GitHub ></a>

```csharp
// Copyright (c) Alexandre Mutel. All rights reserved.
// Licensed under the BSD-Clause 2 license.
// See license.txt file in the project root for full license information.
using Asm6502;
using RetroC64;
using RetroC64.App;
using static RetroC64.C64Registers;

return await C64AppBuilder.Run<HelloMulti>(args);

/// <summary>
/// Represents a multi-disk Commodore 64 application that demonstrates basic 
/// and assembly program examples across multiple disks.
/// </summary>
/// <remarks>This class adds two disks to the application, each containing sample 
/// programs. The first disk includes BASIC programs that print messages, while 
/// the second disk includes additional BASIC programs and an assembly program 
/// that modifies display colors.
/// </remarks>
internal class HelloMulti : C64App
{
    protected override void Initialize(C64AppInitializeContext context)
    {
        Add(new HelloDisk1());
        Add(new HelloDisk2());
    }

    private class HelloDisk1 : C64AppDisk
    {
        protected override void Initialize(C64AppInitializeContext context)
        {
            Add(new HelloBasic("10 PRINT \"HELLO WORLD 1") { Name = "HelloWorld1" });
            Add(new HelloBasic("10 PRINT \"HELLO WORLD 2") { Name = "HelloWorld2" });
        }
    }

    private class HelloDisk2 : C64AppDisk
    {
        protected override void Initialize(C64AppInitializeContext context)
        {
            Add(new HelloBasic("10 PRINT \"HELLO WORLD 3") { Name = "HelloWorld3" });
            Add(new HelloBasic("10 PRINT \"HELLO WORLD 4") { Name = "HelloWorld4" });
            Add(new HelloAsm());
        }
    }

    private class HelloBasic : C64AppBasic
    {
        public HelloBasic(string text) => Text = text;
    }

    private class HelloAsm : C64AppAsmProgram
    {
        protected override Mos6502Label Build(C64AppBuildContext context, C64Assembler asm)
        {
            asm
                .Label(out var start)
                .BeginCodeSection("Main");

            asm
                .LDA_Imm(COLOR_RED)
                .STA(VIC2_BG_COLOR0)
                .LDA_Imm(COLOR_GREEN)
                .STA(VIC2_BORDER_COLOR)
                .InfiniteLoop();

            asm.EndCodeSection();
            return start;
        }
    }
}
```

Running this program with `dotnet run -- build` will generate two disk images, `HelloDisk1.d64` and `HelloDisk2.d64`, each containing the specified BASIC and assembly programs. You can then load these disks into the VICE emulator to run the programs.

![Output building the multi-disk example](/img/RetroC64-HelloMulti-output.png)

> [!NOTE]
> As RetroC64 is still missing support for a depacker and a proper multi-part loader, the usefulness of multi-disk applications is currently limited. Future updates should enhance this functionality.
