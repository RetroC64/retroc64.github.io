---
title: 🖥️ Assembler
---

<a href="https://github.com/xoofx/Asm6502" alt="Asm6502 website"><img align="right" width="256px" height="256px" src="https://raw.githubusercontent.com/xoofx/Asm6502/main/img/Asm6502.png" alt="Asm6502 Logo"></a>

RetroC64 relies on [Asm6502](https://github.com/xoofx/Asm6502), an assembler specially designed for 6502-based systems and for RetroC64 in particular.

It allows to write 6502/6510 assembly code directly from C# by providing a fluent, strongly typed assembler/disassembler. It also provides a cycle-accurate CPU emulator (pluggable 64 KiB memory bus) that is used by the [SID file relocator](/docs/music.md) to analyze the code and relocate it properly.

> [!TIP]
> You can find more information about Asm6502 in its documentation 📚 [here](https://github.com/xoofx/Asm6502/tree/main/doc)

This page will focus on how to best use Asm6502 in RetroC64.

## C64Assembler

As we saw in the Getting Started, the `C64Assembler` class is used to build assembly code for the C64. It extends the `Mos6502Assembler` class from Asm6502 and adds C64-specific features, such as predefined labels for C64 memory-mapped registers and various ASM helpers.

```csharp
// Import the Mos6502Factory for instruction helpers (X,Y,A registers...etc.)
using static Asm6502.Mos6502Factory; 

public class HelloAsm : C64AppAsmProgram
{
    protected override Mos6502Label Build(C64AppBuildContext context, C64Assembler asm)
    {
        asm
            .Label(out var start);

        // ... assembly code here ...
 
        return start;
    }
}
```

The address of the program entry point is returned as a `Mos6502Label` from the `Build` method.

The `C64AppAsmProgram` base class takes care of generating the proper PRG file with the correct load address.

## Zero-Page Allocator

C64Assembler provides **an integrated zero-page allocator** that can be used to allocate zero-page variables easily.

```csharp
asm
    .ZpAlloc(out var tempVar1);    
```

This will allocate a single byte variable in the zero-page and assign its address to the `tempVar1` label. This label will be visible from the debugger as well.

## Code and Data Sections

You can define code and data sections in your assembly code using the `BeginCodeSection`, `EndCodeSection`, `BeginDataSection`, and `EndDataSection` methods.

```csharp
asm
    .BeginCodeSection("Main")
        // ... code here ...
    .EndCodeSection()
    .BeginDataSection("MyData")
        // ... data here ...
    .EndDataSection();
```

Sections help organize your code and data, and they can be useful for debugging and analysis.

In particular, the code section is used for identifying regions of executable code that can be disassembled and debugged.

## Append Buffers

C64Assembler provides methods to append raw data buffers to the assembly code, which is useful for embedding binary data such as graphics or sound.

```csharp
asm
    .Append([0x01, 0x02, 0x03, 0x04]) // Append 4 raw bytes
    .AppendBytes(256, 0xFF)           // Append 256 bytes of 0xFF
    .Append((ushort)0x1234)           // Append a 16-bit word
    .Append("Hello, C64!"u8);         // Append a UTF-8 string
``` 

It is sometimes required to append multiple buffers with some buffers that might require specific alignment or fixed addresses. In that case you can use `asm.ArrangeBlocks(...)` to arrange multiple blocks with specific constraints.

```csharp
asm
    .LabelForward(out var screenBuffer)
    .LabelForward(out var spriteSinXTable);

// ... later in the code ...

asm.ArrangeBlocks(
    [
        // Allocate a screen buffer at the label with 256-byte alignment
        new(screenBuffer, screenBufferArray, 256), // aligned to 256 bytes
        // Generate a sine table for sprite X positions
        new(spriteSinXTable, Enumerable.Range(0, 256).Select(
            x => (byte)Math.Round(radius * 
            Math.Sin(Math.PI * 2 * x / 256) + centerX)).ToArray(), 256), // aligned to 256 bytes
    ]
);

static readonly byte[] screenBufferArray = new byte[1024] {
    // ... screen data here ...
}
```

If the label provided is bound to a specific address (like it is used for a SID file), it will be placed at that address. Otherwise, it will be placed in the next available space respecting the alignment constraints.

## Initialization Phase

To take control of the C64 system initialization (VIC-II, SID, CIA, etc.), you can use the `BeginAsmInit` helper method that will generate the proper initialization code for you.

```csharp
// Initialize full control of the C64 system (VIC-II, SID, CIA, etc.)
BeginAsmInit(asm, CPUPortFlags.FullRamWithIO);

// Behind the scenes, this will generate code similar to:
asm
    .SEI()
    .SetupTimeOfDayAndGetVerticalFrequency()
    .STA(0x02A6) // Store back NTSC(0)/PAL(1) flag to 0x02A6
    .DisableAllIrq()
    .SetupStack()
    .SetupRamAccess(CPUPortFlags.FullRamWithIO)
    .DisableNmi();
```

Notice that the initialization method blocks the interrupts with `SEI` and disables NMI. If your program requires interrupts or NMI, you will need to enable them again in your code.

Once you have initialized the system, you can proceed to your main program code or use the `EndAsmInitAndInfiniteLoop` helper to finalize the initialization and enter an infinite loop:

```csharp
// Your init program code here...

// Finalize initialization and enter an infinite loop
EndAsmInitAndInfiniteLoop(asm);
```

This will generate code similar to:

```csharp
asm
    .CLI()
    .InfiniteLoop();
```

> [!IMPORTANT]
> Make sure to emit a `CLI` instruction after the initialization if you want to enable interrupts, as the `SEI` instruction in the initialization will block them.
> 
> Usually during the initialization phase, you setup your IRQs, so enabling them right after the initialization is a common practice.

> [!NOTE]
> In order to access registers, RetroC64 provides a static class [`C64Registers`](https://github.com/RetroC64/RetroC64/blob/main/src/RetroC64.Core/C64Registers.cs) with most of the VIC-II, SID, and CIA registers defined as constants, with enums for bit flags where applicable.
> 
> ```csharp
> // Example of using C64Registers
> using static RetroC64.C64Registers;
> // Also import the Mos6502Factory for instruction helpers (X,Y,A registers...etc.)
> using static Asm6502.Mos6502Factory; 
> ```
