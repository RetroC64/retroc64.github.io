---
title: 🎵 Music
---

This section demonstrates how to play SID music files on the C64 using RetroC64.

## Your first C64 music program

After following the [steps to create your first RetroC64 application](/docs), replace the content of `Program.cs` with the following code:

<a href="https://github.com/RetroC64/RetroC64-Examples/tree/main/HelloMusic" class="btn btn-primary"><span class="bi bi-github"></span> Code on GitHub ></a>    

```csharp
// Copyright (c) Alexandre Mutel. All rights reserved.
// Licensed under the BSD-Clause 2 license.
// See license.txt file in the project root for full license information.
using Asm6502;
using RetroC64;
using RetroC64.App;
using RetroC64.Music;
using static RetroC64.C64Registers;

// Program entry: build and run the C64 app containing the music player.
return await C64AppBuilder.Run<HelloMusic>(args);

/// <summary>
/// Simple C64 application that loads a SID tune, installs a raster IRQ to play it,
/// and writes metadata to the screen buffer.
/// </summary>
/// <remarks>
/// Credits for the amazing music "Racing_the_Beam.sid" from Lft (Linus Åkesson)
/// https://csdb.dk/release/?id=256179
/// </remarks>
public class HelloMusic : C64AppAsmProgram
{
    protected override void Initialize(C64AppInitializeContext context)
    {
        // Set VICE emulator sound volume (optional aesthetic tweak).
        context.Settings.Vice.SoundVolume = 100;
    }

    protected override Mos6502Label Build(C64AppBuildContext context, C64Assembler asm)
    {
        // Load SID file (Racing_the_Beam.sid) from application base directory.
        var sidRawData = File.ReadAllBytes(
            Path.Combine(AppContext.BaseDirectory, "Racing_the_Beam.sid")
        );

        // Relocate the SID to target address with chosen zero page window.
        var sidFile = context.GetService<IC64SidService>()
        .LoadAndConvertSidFile(context, sidRawData, new SidRelocationConfig()
        {
            TargetAddress = 0x1000,
            ZpLow = 0xF0,
            ZpHigh = 0xFF,
        });

        // Helper to emit init/play routines into assembly.
        var sidPlayer = new SidPlayer(sidFile, asm);

        // -------------------------------------------------------------------------
        // Assembly: code section setup & label declarations
        // -------------------------------------------------------------------------
        asm
            .LabelForward(out var irqMusic)      // Label placeholder for raster IRQ routine
            .LabelForward(out var screenBuffer)  // Label placeholder for screen buffer data
            .Label(out var startOfCode)          // Entry label for the program
            .BeginCodeSection(Name);             // Begin main code section

        // Framework-provided initialization (sets up environment, disables BASIC/KERNAL)
        BeginAsmInit(asm);

        asm
            // Install raster IRQ at line 0xF8 calling irqMusic
            .SetupRasterIrq(irqMusic, 0xF8);

        // Initialize SID player (calls SID init routine)
        sidPlayer!.Initialize();

        // Clear the visible screen using a 1KB block copy (4 * 256 bytes)
        asm
            .CopyMemoryBy256BytesBlock(screenBuffer, VIC2_SCREEN_CHARACTER_ADDRESS_DEFAULT, 4);

        // Hand over to infinite main loop (idle; music driven by IRQ)
        EndAsmInitAndInfiniteLoop(asm);

        // -------------------------------------------------------------------------
        // Raster IRQ routine: acknowledge interrupt and call SID play
        // -------------------------------------------------------------------------
        asm.Label(irqMusic)
            .PushAllRegisters() // Preserve CPU state

            .LDA(VIC2_INTERRUPT)  // Read VIC-II interrupt register (ack)
            .STA(VIC2_INTERRUPT); // Write back to clear IRQ source

        // Call generated SID play routine (advances music)
        sidPlayer.PlayMusic();

        asm
            .PopAllRegisters()           // Restore CPU state
            .RTI()                       // Return from interrupt
            .EndCodeSection();

        // -------------------------------------------------------------------------
        // Prepare screen buffer content: render metadata strings (Author, Title, Released)
        // -------------------------------------------------------------------------
        var musicX = 7;
        var musicY = 10;

        var screenBufferData = new byte[1024];        // Full screen character buffer
        var screenBufferSpan = screenBufferData.AsSpan();
        screenBufferSpan.Fill((byte)' ');             // Initialize with spaces

        // Write SID metadata into buffer at chosen positions
        C64CharSet.StringToPETScreenCode($"   MUSIC: {sidFile!.Author.ToUpperInvariant()}")
            .CopyTo(screenBufferSpan.Slice(40 * (musicY + 2) + musicX));
        C64CharSet.StringToPETScreenCode($"   TITLE: {sidFile.Name.ToUpperInvariant()}")
            .CopyTo(screenBufferSpan.Slice(40 * (musicY + 3) + musicX));
        C64CharSet.StringToPETScreenCode($"RELEASED: {sidFile.Released.ToUpperInvariant()}")
            .CopyTo(screenBufferSpan.Slice(40 * (musicY + 4) + musicX));

        // -------------------------------------------------------------------------
        // Data section: SID music block + initial screen buffer
        // Only the first 256 bytes of screenBufferData are arranged here (matches copy loop).
        // -------------------------------------------------------------------------
        asm
            .BeginDataSection("DemoData")
            .ArrangeBlocks([
                // SID relocated data/code block
                sidPlayer!.GetMusicBlock(),
                // Screen buffer (first 256 bytes used for clearing)
                new(screenBuffer, screenBufferData, 256),
            ])
            .EndDataSection();

        return startOfCode; // Entry point returned to framework.
    }
}
```

It will produce the following output and you will hear the music playing:

![C64 screen showing music metadata](/img/RetroC64-HelloMusic.png)

## Loading SID files

If you only want to inspect an existing SID file without relocating it, you can use SidFile.Load method:

```csharp
var sidRawData = File.ReadAllBytes(
    Path.Combine(AppContext.BaseDirectory, "Your_Song_Name.sid")
);

// Load the SID file without relocation
var sidFile = SidFile.Load(sidRawData);
```

The [`SidFile`](https://github.com/RetroC64/RetroC64/blob/main/src/RetroC64.Core/Music/SidFile.cs) object returned contains metadata such as the song's name, author, and release information, which you can use to display on the screen.

Examples:
- `sidFile.Name`: The title of the SID tune.
- `sidFile.Author`: The author of the SID tune.
- `sidFile.Released`: The release information of the SID tune.
- `sidFile.Songs`: The number of songs/tracks in the SID file.
- `sidFile.StartSong`: The default song index.

## Relocating SID files

By using the `IC64SidService` as shown below, you can relocate a SID file to a specific target address in C64 memory, along with a specified zero-page window for the SID player routines:

```csharp
var sidRawData = File.ReadAllBytes(
    Path.Combine(AppContext.BaseDirectory, "Your_Song_Name.sid")
);

// Relocate the SID to target address with chosen zero page window.
var sidFile = context.GetService<IC64SidService>()
.LoadAndConvertSidFile(context, sidRawData, new SidRelocationConfig()
{
    TargetAddress = 0x1000,
    ZpLow = 0xF0,
    ZpHigh = 0xFF,
});
```

The [`SidRelocationConfig`](https://github.com/RetroC64/RetroC64/blob/main/src/RetroC64.Core/Music/SidRelocationConfig.cs) allows you to specify the target address in C64 memory where the SID file will be loaded, as well as the zero page range used by the SID player routines.

{{NOTE do}}
Some SID files might not be compatible with the relocation process due to their internal structure or specific requirements. In such cases, you may need to use a different SID file or adjust the relocation parameters.

The relocation process is necessary to extract also the range of zero-page addresses used by the SID player routines, which is essential to avoid conflicts with other parts of your C64 program.

Because the relocation process can take time, the `IC64SidService` caches relocated SID files in the build cache directory (`.retroC64/cache/`) to speed up subsequent builds.
{{end}}

{{TIP do}}
The relocation code is actually a C# port of the fantastic [sidreloc](https://www.linusakesson.net/software/sidreloc/index.php) made by Linus Åkesson. The C# version is using the Asm6502 library to emulate the 6502 CPU and perform the relocation.
{{end}}

In order to extract Zero-Page addresses, the `SidFile` class provides an helper method to get the list of zero-page addresses used by the SID player routines:

```csharp
if (!sidFile.TryGetZeroPageAddresses(out byte[] zpAddresses))
{
    throw new InvalidOperationException("Unable to extract Zero-Page addresses from SID file.");
}
```

The zero-page data is stored as part of the additional data block when storing a music and can be recovered even after the music is saved to disk.

The `IC64SidService` service ensures that the SID file is properly relocated and ready to be used with the `SidPlayer` helper class.

## Playing SID Music

To play the SID music, you can use the `SidPlayer` class, which provides methods to initialize and play the music. Here's how to set it up in your assembly code:

```csharp
// Helper to emit init/play routines into assembly.
var sidPlayer = new SidPlayer(sidFile, asm);

// Initialize SID player (calls SID init routine)
sidPlayer!.Initialize();

// ... rest of your assembly code ...

// In your raster IRQ routine, call the play music method
sidPlayer.PlayMusic();
```

When the player is created, it will reserve the zero-page addresses from the `ZeroPageAllocator` in `asm.Zp`. 

It will create `zpSidPlayer` labels for each zero-page address used by the SID player routines, which you can use in your assembly code if needed. (e.g. `zpSidPlayer0`, `zpSidPlayer1`, etc.)

## Saving SID file

You can save a SID file back to disk `sidFile.Save(filePath)` or `sidFile.Save(stream)` methods.

