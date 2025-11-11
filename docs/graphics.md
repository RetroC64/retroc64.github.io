---
title: 🎨 Graphics
---

Currently, RetroC64 supports a minimal helper for creating sprites with SkiaSharp.

## Sprites

You can create a monochrome sprite using the `C64SpriteMono` class. This class provides a canvas to draw on using SkiaSharp methods. Once you have drawn your sprite, you can convert it to the C64 sprite bit format using the `ToBits` method.

For example, the following code creates a simple oval sprite with a stroke:

```csharp
using var sprite = new C64SpriteMono();
sprite.UseStroke(2.0f).Canvas.DrawOval(new SKRect(1, 1, 23, 20), sprite.Brush);
sprite.Canvas.Flush();
var spriteData = sprite.ToBits();
```

Then we can setup the sprites in the memory and display them:

If you create the following program, it will display 8 sprites on the screen, each with a different color and position:

<a href="https://github.com/RetroC64/RetroC64-Examples/tree/main/HelloSprites" class="btn btn-primary"><span class="bi bi-github"></span> Code on GitHub ></a>    

```csharp
// Copyright (c) Alexandre Mutel. All rights reserved.
// Licensed under the BSD-Clause 2 license.
// See license.txt file in the project root for full license information.
using Asm6502;
using RetroC64;
using RetroC64.App;
using RetroC64.Graphics;
using SkiaSharp;
using static RetroC64.C64Registers;

return await C64AppBuilder.Run<HelloSprites>(args);

/// <summary>
/// Minimal demo: draws one mono sprite shape, duplicates its pointer for 8 sprites,
/// assigns distinct colors (skipping blue to keep background clear),
/// positions them in a simple grid, and enables all.
/// </summary>
public class HelloSprites : C64AppAsmProgram
{
    protected override Mos6502Label Build(C64AppBuildContext context, C64Assembler asm)
    {
        // Reserve (forward declare) a 64-byte aligned buffer for sprite data 
        // (resolved later in data section).
        asm.LabelForward(out var spriteBuffer);

        // Build a mono sprite bitmap (an oval) via high-level drawing API 
        // then extract 1-bit planar data.
        using var sprite = new C64SpriteMono();
        sprite
            .UseStroke(2.0f)
            .Canvas.DrawOval(new SKRect(1, 1, 23, 20), sprite.Brush);
        sprite.Canvas.Flush();
        var spriteData = sprite.ToBits(); // 63 bytes sprite + 1 byte padding (64 total expected)

        // Mark program entry and start main code section.
        asm
            .Label(out var start)
            .BeginCodeSection("Main");

        // Generic C64 + assembler initialization (clears/sets up runtime, interrupts, etc.).
        BeginAsmInit(asm);
        
        // Compute sprite pointer value (each pointer = (address / 64) & 0xFF).
        // All 8 sprites will reuse the same shape data.
        var spriteAddr64 = (spriteBuffer / 64).LowByte();
        asm.LDA_Imm(spriteAddr64);
        for (int i = 0; i < 8; i++)
        {
            // Store identical pointer for sprites 0..7.
            asm.STA((ushort)(VIC2_SPRITE0_ADDRESS_DEFAULT + i));
        }

        // Assign colors: start at black, increment; skip blue to avoid blending with background.
        for (int i = 0; i < 8; i++)
        {
            var color = (byte)(COLOR_BLACK + i);
            if (color >= COLOR_BLUE)
            {
                color++; // Skip blue.
            }

            asm
                .LDA_Imm(color)
                .STA((ushort)(VIC2_SPRITE0_COLOR + i));
        }

        // Position sprites with simple horizontal & vertical offset based on sprite dimensions.
        // Uses base origin (80,65) then offsets by width/height per index (no multiplexing).
        for (int i = 0; i < 8; i++)
        {
            asm
                .LDA_Imm((byte)(80 + i * sprite.Width))
                .STA((ushort)(VIC2_SPRITE0_X + i * 2));
            asm
                .LDA_Imm((byte)(65 + i * sprite.Height))
                .STA((ushort)(VIC2_SPRITE0_Y + i * 2));
        }

        // Enable all 8 hardware sprites (bitmask 0xFF).
        asm
            .LDA_Imm(0xFF)
            .STA(VIC2_SPRITE_ENABLE);

        // Finish init and stay in an infinite idle loop (program end).
        EndAsmInitAndInfiniteLoop(asm);
        
        asm.EndCodeSection();

        // Data section: ensure sprite bitmap is placed at the predeclared label, aligned to 64 bytes.
        asm
            .BeginDataSection("Sprites")
            .ArrangeBlocks([new(spriteBuffer, spriteData.ToArray(), 64)]) // Alignment guaranteed.
            .EndDataSection();
        
        return start;
    }
}
```

It will produce the following output:

![Colored sprites displayed on a C64 screen](/img/RetroC64-HelloSprites.png)

## Bitmaps

> [!NOTE]
> More to come in the future! 
> 
> Notably the support for converting bitmaps to C64 multicolor bitmap format with proper color handling and dithering modes.
